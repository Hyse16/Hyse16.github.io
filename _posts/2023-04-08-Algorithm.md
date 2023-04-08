---
layout: single
title: "백준 알고리즘 1003번"
categories: Algorithm
toc: true
author_profile: true
sidebar:
    nav: "docs"
---

[백준링크](https://www.acmicpc.net/problem/1003)


피보나치 함수
<p>
다이나믹 프로그래밍

<p>



# 파이썬

```python
import sys
input = sys.stdin.readline

def res(n):
    if len(zero) <= n:
        for i in range(len(zero), n+1):
            zero.append(zero[i-1]+zero[i-2])
            one.append(one[i-1]+one[i-2])
            
    print(zero[n],one[n])


t = int(input())
zero = [1,0,1]
one = [0,1,1]
for i in range(t):
    n = int(input())
    res(n)


```