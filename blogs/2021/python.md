---
title: python自学笔记
date: 2020-05-20 
tags:
 - python
---
## 自学python
大三自学过python这门语言和js一样是门弱类型的语言，也是目前非常的流行，尽管我们有这门课但是我还是自己去找视频学习了一下。但是光靠这门语言找工作也是不好找的，不要被现在python的火热蒙蔽了。我有个学长主学python，结果大四没找到工作然后自己最后还是去找培训机构花了2w学了java，找了1w的工作了。所以说不建议靠这门语言找工作，也许以后会有更好的就业前景，但是现在不建议。
## 自己当初学的时候一些有意思的知识点也都记录了下来，分享出来
```python
'''scores={'邓小荣':88,'红文葛':77,'周逛':66}
keys=scores.keys()
values=scores.values()
lst=list(keys)
lsts=list(values)
itemas=list(scores.items())
print(keys)
print(values)
print(lst)
print(lsts)
print(itemas)
for items in scores:
    #print(items,scores[items],scores.get(items))
k={'邓小荣dxr','红文葛hwg','周逛zg'} #花括号字典就乱序排列  就是集合（没有value的字典）
v={88,77,66}
lst={k.upper():v for k,v in zip(k,v)} #upper()将改成大写字母
print(lst)
s={10,20,30,40,50,60}
```
## 集合元素的判断操作
```python
print(10 in s)
print(20 in s)
print(40 not in s)
```
## 集合元素的新增操作 （无序)
```python
s.add(70)
print(s)
s.update([100,500,600])
s.update((77,88,99))
print(s)
```
## 集合元素的删除
```python
s.remove(100) #(没有的话抛异常)
print(s)
s.discard(500) #(没有的话不抛异常)
print(s)
s.pop()
print(s)
```
## 判断集合的关系
```python
s1={10,20,30,40,50}
s2={10,20,30}
s3={10,20}
print(s2.issubset(s1)) #判断s2是否为s1的子集
print(s1.issuperset(s3)) #判断s1是否为s2的超集
print(s2.isdisjoint(s3)) #判断两个集合是否有交集
print(s1.intersection(s2))  print(s1 & s2 )#给出两个集合的交集
print(s1.union(s2))  print(s1 | s2 )#给出两个集合的交集
print(s1.difference(s2))  print(s1-s2) #给出两个集合的差集
print(s1.symmetric_difference(s2)) #print(s1 ^ s2) #给出两个集合的对称差集
s1='abc%'
s2='abc%'
print(s1 is s2)
print(s1 == s2)
```
## 字符串的替换
```python
s='hello,python'
print(s.replace('python','java'))
s1='hello,python,python,python,python'
print(s.replace('python','java',4))
```
## 列表的添加
```python
lst={'hello','java','python'}
ls=('hello','java','python')
print('*'.join(lst))
print(''.join(lst))
print('*'.join(ls))
print('*'.join('python'))
print('apple'>'app')
print('apple'>'banana')
print(ord('a'),ord('b')) #ord函数可以返回字母的原始值
print(chr(97),chr(98)) #chr 反之
```
## 字符串的切片操作 产生新的对象
```python
s='hello,world'
s1=s[5]
s2=s[6]
s3='!'
news=s1+s2+s3
print(s1)
print(s2)
print(news)
```
## 函数的创建与调用
```python
def giao(a,b): #形参
    c=a+b
    return c
result=giao(20,30) #实参
res=giao(b=10,a=20)
print(result)
print(res)
```
## 可变参数* 为一个元组
```python
def fun(*arg):
    print(arg)
    print(arg[0])
fun(10)
fun(20,30)
fun(10,20,30,40)
  只能一个一个不能 def fun1(**args,*args):     def fun1(**args,**args):
在一个函数定义过程中，既有个数可变的关键字形参，要求，个数可变的位置形参，放在个数可变的关键字形参之前
比如  def fun1(*arg,**a):
          pass
```
## 可变参数 为一个字典
```python
def fun1(**args):
    print(args)
fun1(a=10)
fun1(a=10,b=20,c=30)

def fun(a,b,c,d):  #如果是 def fun(a,b,,*,c,d): 那么就是要求*号之后的参数都要求用关键字传递参数，所以只有108 109 行能输出
    print('a=',a)
    print('b=',b)
    print('c=',c)
    print('d=',d)
fun(10,20,30,40) #位置实参传递
fun(a=10,b=20,c=30,d=40) #关键字实参传递
fun(10,20,c=30,d=40) #前两个采用实参传递，后两个采用关键字传递
```
## 递归函数
```python
def fac(a):
    if a==1:
        return 1
    else:
        return a*fac(a-1)
print(fac(6))
```
## 递归斐波那契数列
```python
def fun1(n):
    if n==1:
        return 1
    elif n==2:
        return 1
    else:
        n=fun1(n-1)+fun1(n-2)
        return n
print(fun1(6))
'''
```
## python爬虫
pyhton爬虫和可视化挺好玩的，但是可视化我没深入研究,仅仅了解了一下.爬虫自己爬取到了图片,小说等,还有就是做了一个学生管理系统,有简陋的可视化界面,存入数据库,可增删改.需要源码联系我.
