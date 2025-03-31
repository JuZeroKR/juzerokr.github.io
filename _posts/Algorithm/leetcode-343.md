---
title: "343. Integer Break"
layout: single
toc: true
toc_sticky: true
toc_label: 목차
categories:     
    - algorithm
tag:
    - DP
---

## 문제 개요

[343. Integer Break](https://leetcode.com/problems/integer-break/)


## 문제 접근 방법
DP 풀이 중 재귀 함수와 Memo 기법을 활용하면 쉽게 풀수 있다.

## 실제 코드

```c++
class Solution {
public:
    int dp[60][60];
    int recurIntegerBreak(int n, int dividedCount) {
        int maxValue = n;

        if(dividedCount == 0)
            maxValue = 1;

        if (dp[n][dividedCount] != 0)
            return dp[n][dividedCount];

        for (int i = 1; i < n; i++)
        {
            maxValue = max(maxValue, i * integerBreak(n - i, dividedCount));
        }

        dp[n][dividedCount] = maxValue;

        return maxValue;
    }

    int integerBreak(int n) {
        dp[1] = 1;
        dp[2] = 1;
        return recurIntegerBreak(n, 0);
    }
};s
```