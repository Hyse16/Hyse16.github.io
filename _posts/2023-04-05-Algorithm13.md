---
layout: single
title: "백준 알고리즘 13305번"
categories: Algorithm
toc: true
author_profile: true
sidebar:
    nav: "docs"
---

[백준링크](https://www.acmicpc.net/problem/13305)


그리디 알고리즘

# 파이썬
```python
n = int(input())
road = list(map(int,input().split()))
oil = list(map(int,input().split()))

min_price = road[0]*oil[0]
min_oil = oil[0]

for i in range(1,n-1):
    if min_oil > oil[i]:
        min_oil = oil[i]
    min_price += min_oil*road[i]
print(min_price)


```