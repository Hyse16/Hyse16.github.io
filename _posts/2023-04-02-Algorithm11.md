---
layout: single
title: "백준 알고리즘 1463번"
categories: Algorithm
toc: true
author_profile: true
sidebar:
    nav: "docs"
---

[백준링크](https://www.acmicpc.net/problem/1463)


BottomUp방법 사용

# 파이썬
```python
x=int(input()) 
d=[0]*(x+1) 

for i in range(2,x+1):
    d[i]=d[i-1]+1 
    if i%2==0: 
        d[i]=min(d[i], d[i//2]+1)
    if i%3==0:
        d[i]=min(d[i], d[i//3]+1)
        
print(d[x])

```