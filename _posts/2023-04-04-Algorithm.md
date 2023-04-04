---
layout: single
title: "백준 알고리즘 2217번"
categories: Algorithm
toc: true
author_profile: true
sidebar:
    nav: "docs"
---

[백준링크](https://www.acmicpc.net/problem/2217)


그리디 알고리즘

# 파이썬
```python
n = int(input())
s = []
for i in range(n):
    s.append(int(input()))
    
s.sort(reverse=True)
res = []
for j in range(n):
    res.append(s[j]*(j+1))
    
    
print(max(res))


```