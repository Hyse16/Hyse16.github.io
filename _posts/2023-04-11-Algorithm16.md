---
layout: single
title: "백준 알고리즘 2775번"
categories: Algorithm
toc: true
author_profile: true
sidebar:
    nav: "docs"
---

[백준링크](https://www.acmicpc.net/problem/2775)


# 파이썬
```python
case = int(input())
for i in range(case):
    k = int(input())
    n = int(input())
    people=[i for i in range(1,n+1)]
    
    for _ in range(k):
        for j in range(1, n):
            people[j]+= people[j-1]
    print(people[n-1])
```