---
title: "Loongarch64折腾日记"
categories:
  - Blog
tags:
  - 杂项
---

欣闻龙芯3A6000处理器正式上市，性能达到了较好水平（已经远高于我爹2015年买的mac，我目前正在密集使用它，以期尽快报废），而且价格合理（16g内存+1tb固态硬盘才不到4000，甚至带光驱）。
国产芯片如此争气，这使得立志少买不买电子产品的我打算在回国之后买一台，用实际行动支持国产。
但我也注意到，尽管龙芯做了很多适配工作，*uix对loongarch64架构的支持仍然有点小问题。
<!-- 一方面是龙芯比较新，另一方面是国外开源工作者很傲慢，给他们提交几段代码添加个新架构的支持就好像杀了他们一样。 -->
<!-- 给人的感觉是，长了张白皮，再加上他们洋人的处理器，写出的代码在磁盘里就能少占一半字节。 -->
所以我在QEMU虚拟机里面安装了loongarchlinux，做一些编译和安装工作。
以下是我的踩到的坑，未来会不断更新，希望能有帮助。

## Neovim和LuaJIT

目前（2023年12月10日）LoongArchLinux软件仓库没提供neovim。
这是因为neovim的正常编译需要LuaJIT，而LuaJIT拒绝了龙芯方面提交的Loongarch64支持补丁。
<!-- 其实这段补丁就是跟主体代码独立的一段汇编语言，如果是Loongarch64架构就按这个规则去编译，也不碍着别的架构的事儿。 -->
LuaJIT的作者Mike Pall说：你这个代码质量不高（他也没证据），而且龙芯不见得能活过五年。
这就太不像话了：龙芯能不能活过五年，取决于软件生态好不好，即取决于编译器、运行时等底层基础设施是否支持它。
现在你不支持他，我连neovim都没法用，“活不过五年”不就成了一个自我实现的预言吗？
<!-- 何况你luajit只支持到lua5.1，现在基本是一个停摆的阶段，谁先凉还不一定呢。 -->

Neovim的编译需要带luajit和luarocks，而且是编译程序下载源代码后，把luajit和几个lua包编译进去。
很自然，在编译到luajit时，系统就会提示不支持loongarch64架构。
网上有人教过一个办法，把luajit和luarock在编译时都给去掉。
我自己实验的结果是结果是lazyvim全家桶用不了。
好在龙芯方面给Mike Pall提供的代码是公开的，[可以下载](https://github.com/loongson/LuaJIT/tree/v2.1-loongarch64)。
我考虑的是直接在编译过程中，把这个luajit给换掉，看看能否编译通过。

如果带luarock的话，在用neovim编译程序编译出的luajit编译luarocks时（我没写错，就是这个流程），会报segematation fault。
我还没弄清楚这是龙芯给的luajit补丁的问题，还是我有什么地方没调对。
此外我还试了不用luajit编译luarocks，而是用自己的lua5.1编译luarocks，


## 需要更新包依赖才能正常编译的软件

下面这些软件调用了跟系统底层相关的包，老版本的包不支持Loongarch，因此需要手动调整包依赖。
暂时没有发现调整后的程序出现bug。

### Joshuto

tui文件管理器中，nnn最快，但丑陋而且难配置——全部的配置都是在环境变量里面实现的。
ranger等用户友好，但速度慢。
joshuto是用rust写的，和ranger操作方式类似，但速度快。
且和kitty搭配工作良好，可以发挥kitty显示图片的优势。
需要调整的包依赖是：在Cargo.toml中，rustyline从"^12"改为"^13"，同时删去Cargo.lock，让系统重新生成。
这是因为12版的rustyline依赖于0.26版的nix包（顾名思义这是关于*nux系统底层信息的包），13版的rustyline依赖于0.27版的nix包，后者才支持loongarch。

附PKGBUILD:

```
# Maintainer: Taiming Wang <taimingwang[at]ruc[dot]edu[dot]cn>

pkgname=joshuto-loong64
pkgver=0.9.6
pkgrel=1
pkgdesc="ranger-like terminal file manager written in Rust"
arch=("loong64")
url="https://github.com/kamiyaa/joshuto"
license=("LGPL3")
depends=("gcc-libs")
optdepends=(
    'xclip: X11 clipboard support'
    'wl-clipboard: Wayland clipboard support'
    'fzf: fuzzy finder support'
    'zoxide: zoxide support'
)
makedepends=("cargo")
conflicts=("joshuto" "joshuto-git" "joshuto-bin")
source=("joshuto-${pkgver}.tar.gz::https://github.com/kamiyaa/joshuto/archive/refs/tags/v${pkgver}.tar.gz")
sha256sums=('78d03e0c7971fe715da7e89c6e848112eedb993f04625e720997156c74578d42')

prepare() {
  cd "${srcdir}/joshuto-${pkgver}"
  rm Cargo.lock #不删除lock，就还是按照原来的依赖安装
  sed -i 's|rustyline = "^12"|rustyline = "^13"|g' Cargo.toml #用sed改了一行。未来的版本可能会有变化，请注意。
}

build() {

  cd "${srcdir}/joshuto-${pkgver}"
    cargo build --release
}

package() {
  cd "${srcdir}/joshuto-${pkgver}"
  install -Dm755 "target/release/joshuto" "${pkgdir}/usr/bin/joshuto"
}
```

我本来想把这个PKGBUILD发到LAUR上面的，但看武老师太累了，不忍心为了一个小众软件给他添加工作量，因此只放在这里供大家参考。

### Lazygit

Lazygit是tui下的git管理器，用go编写。
其实也有gitui等竞品，但我现在用的lazyvim依赖于它。
之前由于loongarch下golang toolchain版本低，无法编译lazygit。
现在只需要把pty包调整为最新版即可。

## 结语

跟国外开源社区打交道，才意识到了为什么胡老坚持认为“独立自主就是以我为主”。
因为即便号称是开源，但人家是否支持你，把你的补丁加进来，完全取决于人家的想法，这就不是真正的自主可控。
哪怕就算补丁跟其他架构下的业务逻辑完全无关，根本不碍别人的事情，现在人家就是出于某个原因，就是要卡死你，那能怎么办呢？
只能再搞个fork，结果还是要回到以我为主。
说穿了，你不去抓权，别人就要抓权，然后抓你。
指望一个我不争你也不抢的乌托邦社区，是极其幼稚的。