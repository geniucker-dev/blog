---
title: Python 八皇后问题
categories:
  - [笔记]
  - [算法]
tags:
  - Python
  - 算法
  - 八皇后问题
date: 2020-03-26 14:25:59
updated: 2020-03-26 14:25:59
---

---
> &emsp;&emsp;八皇后问题，是一个古老而著名的问题，是回溯算法的典型案例。该问题是国际西洋棋棋手马克斯·贝瑟尔于1848年提出：在8×8格的国际象棋上摆放八个皇后，使其不能互相攻击，即任意两个皇后都不能处于同一行、同一列或同一斜线上，问有多少种摆法。 高斯认为有76种方案。1854年在柏林的象棋杂志上不同的作者发表了40种不同的解，后来有人用图论的方法解出92种结果。计算机发明后，有多种计算机语言可以解决此问题。<p align="right">——百度百科</p>


这是上学期学Python的时候老师留给我们的一个思考题，一开始我和我一个同学是这么写的：
```python
k=0
for a in range(1,9):
    a==a
    for b in range(1,9):
        if b==a or (abs(b-a)==1):continue
        else:
            b==b
            for c in range(1,9):
                if c==b or c==a or (abs(c-b)==1) or (abs(c-a)==2):continue
                else:
                    c==c
                    for d in range(1,9):
                        if d==c or d==b or d==a or (abs(d-c)==1) or (abs(d-b)==2) or (abs(d-a)==3):continue
                        else:
                            d==d
                            for e in range(1,9):
                                if e==d or e==c or e==b or e==a or (abs(e-d)==1) or (abs(e-c)==2) or (abs(e-b)==3) or (abs(e-a)==4):continue
                                else:
                                    e==e
                                    for f in range(1,9):
                                        if f==e or f==d or f==c or f==b or f==a or (abs(f-e)==1) or (abs(f-d)==2) or (abs(f-c)==3) or (abs(f-b)==4) or (abs(f-a)==5):continue
                                        else:
                                            f==f
                                            for g in range(1,9):
                                                if g==f or g==e or g==d or g==c or g==b or g==a or (abs(g-f)==1) or (abs(g-e)==2) or (abs( g-d)==3) or (abs(g-c)==4) or (abs(g-b)==5) or (abs(g-a)==6):continue
                                                else:
                                                    g==g
                                                    for h in range(1,9):
                                                        if h==g or h==f or h==e or h==d or h==c or h==b or h==a or (abs(h-g)==1) or (abs(h-f)==2)  or (abs(h-e)==3) or (abs(h-d)==4) or (abs(h-c)==5) or (abs(h-b)== 6) or (abs(h-a)==7):continue
                                                        else:
                                                            h==h
                                                            k+=1
                                                            print('result',k,':','(1,',a,'),(2,',b,'),(3,',c,'),(4,',d,'),(5,',e,'),(6,',f,'),(7,',g,'),(8,',h,')')

```
这显然是我们初学者最容易想到的一种办法。由于“学业比较繁忙”，当时一直没有去思考更好的实现，直到现在才重新开始思考这个问题。从原程序看，循环嵌套了8次，由此我想到（网上看到的）让函数自己调用自己——递归。

整体思路：N皇后问题每行有且仅有一个皇后，用数组（Python只能列表）表示皇后的坐标（下标为行，至为列，为了方便第一行下标仍为0），在每一行都要用循环分别判断是否与前面任意一行在同一列或同一斜排，都满足条件进入下一行，直到行数=N进行输出。下面给出代码（为了方便注释，有些可以合并的地方我分行写了）：
```python
num=0 #用来计数
def huangHou(A,hang=0):#A是一个列表，hang表示第n行
    global num
    if len(A)==hang:#递归终止条件
        num += 1
        print(num,end='：')
        for i in range(len(A)):#坐标形式输出
            print('('+str(i+1)+','+str(A[i])+')',end='')
        print('')
        return
    else:
        for i in range(len(A)):#遍历第hang行的每一个位置
            tem = True#用于标记是否满足条件
            A[hang] = i#填入列表
            for j in range(hang):#判断两个条件
                if (A[j]==A[hang])or(abs(A[j]-A[hang])==hang-j):
                    tem = False
                    break
            if tem:huangHou(A,hang+1)#若满足条件，进入下一行
huangHou([None]*8)#只要填入有N个元素的列表即可解决N皇后问题

```
这样不但结构清晰还简短了许多。
本人小白一只，若有错误欢迎在评论区留言指正（最好留下邮箱方便联系）
