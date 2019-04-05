# 时间序列分析

* 授课教师：范红岗
* 首选参考书：James D. Hamilton： Time Series Analysis，Princeton University Press，1994
* 成绩：平时作业和考勤占40%，期末60%。



关于lambda函数

如果函数有多个参数, 但每个参数的序列元素数量不一样, 会根据最少元素的序列进行：
```python
listx = [1,2,3,4,5,6,7]       # 7 个元素
listy = [2,3,4,5,6,7]         # 6 个元素 
listz = [100,100,100,100]     # 4 个元素
list_result = map(lambda x,y,z : x**2 + y + z,listx, listy, listz)
print(list(list_result))
[103, 107, 113, 121]
```
明显可以看出是由于 lambda 中的 z 参数,实际是使用了 listz, 而 listz 里面只有 4 个元素, 所以即使 listx 有 7 个元素, listy 有 6 个元素,也不会继续执行了，只执行了 4 个元素的的计算。

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
# 字典用法扩展: 用字典映射简单的switch case 语句
# 其他语言中有switch case用法,但是python中没有,不过可以用字典映射来代替
day = 1

likeswitch = {
    0 : 'Monday',
    1 : 'Tuesday',
    2 : 'Wednesday'
    3 : 'Thursday'
    4 : 'Friday'
    5 : 'Saturday'
    6 : 'Sunday'
}

# 这里是用get()方法,是为了在字典中如果没有找到key时,就返回'Unknow',
# 相当于switch case 中的default
day_name = likeswitch.get(day,'Unknow')  #这里用到了字典中的get()方法.

print(day_name)
# 运行结果:Tuesday
# -----简单的字典映射switch case-----


# -----字典映射复杂的switch case----
# switch中,case下面可以写代码块,其实Python的字典也可以.

print('请输入您想翻译星期几:')
userinput = input("输入星期几(如星期一)：")     #输入 '星期一'
#userinput = raw_input("输入星期几(如星期一)：")    # Python2.x

def translateMonday():
    return 'Monday'

def translateTuesday():
    return 'Tuesday'

def translateWednesday():
    return 'Wednesday'

def translateThursday():
    return 'Thursday'

def translateFriday():
    return 'Friday'

def translateSaturday():
    return 'Saturday'

def translateSunday():
    return 'Sunday'

def unknow():
    return '请输入正确的信息'

# 从这里可以得出,字典的value也可以是函数.
# value的值,直接写函数名
# 这里的key是字符串,value是函数
chineseName = {
    '星期一': translateMonday,
    '星期二': translateTuesday,
    '星期三': translateWednesday,
    '星期四': translateThursday,
    '星期五': translateFriday,
    '星期六': translateSaturday,
    '星期日': translateSunday
}

# 注意这里的第二个参数也必须是一个函数,否则会报错的.
# get()方法后面还有一个()
englishName = chineseName.get(userinput,unknow)()

print(englishName)

# 输入信息: 星期一
# 输出结果: Monday
# 扩展了字典中 key和value的知识点,value可以是函数,那么就可以用这样的方法来实现比较复杂的业务逻辑了.
# 完美代替了switch case
```