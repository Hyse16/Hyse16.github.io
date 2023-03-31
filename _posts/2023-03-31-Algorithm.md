---
layout: single
title: "백준 알고리즘 10816번"
categories: Algorithm
toc: true
author_profile: true
sidebar:
    nav: "docs"
---

[백준링크](https://www.acmicpc.net/problem/10816)

dict() 자료형을 사용


# 파이썬
```python
n = int(input())
card = list(map(int,input().split()))
m = int(input())
test = list(map(int,input().split()))

card.sort()

dic = dict()

for i in card:
    if i in dic:
        dic[i] += 1
    else :
        dic[i] = 1
        
for i in range(m):
    if test[i] in dic:
        print(dic[test[i]], end =' ')
    else:
        print(0, end=' ')
```