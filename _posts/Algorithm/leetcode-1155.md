---
title: "1155. Number of Dice Rolls With Target Sum"
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

[1155. Number of Dice Rolls With Target Sum](https://leetcode.com/problems/number-of-dice-rolls-with-target-sum/description/)


## 문제 접근 방법
- 처음에 점화식을 세웠는데 수가 너무 컸기 때문에 오류가 발생 (모듈러 연산으로 인해 음수가 나올때가 있음)
- 그래서 -1 한 개수의 주사위에서 모든 경우의 수를 더하는 방식으로 변경
- 즉 target 7이고 주사위가 3개일때, 2개의 주사위로 1부터 6까지 등등 7을 만들수 있는 모든 경우의 수를 더한다.


## 실제 코드

```c++
class Solution {
public:
    int numRollsToTarget(int n, int k, int target) {
        vector<vector<long long>> dp(n, vector<long long>(max(target, k), 0));
        int mod = pow(10, 9) + 7;
        for(int i = 0 ; i < k ; i++)
        {
            dp[0][i] = 1;
        }

        for(int i = 1 ; i < n ; i++)
        {
            for(int j = 0; j < target; j++)
            {
                for(int dice = 1 ; dice <= k ; dice++)
                {
                    int sum = 0;
                    if(j-dice >= 0)
                    {
                        sum = dp[i-1][j-dice];
                    }
                    dp[i][j] = (dp[i][j] + sum) % mod;
                }
            }
        }
        return dp[n-1][target-1];
    }
};
```