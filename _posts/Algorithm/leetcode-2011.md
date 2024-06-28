---
title: "2011. Final Value of Variable After Performing Operations"
layout: single
toc: true
toc_sticky: true
toc_label: 목차
categories:     
    - Algorithm
tag:
    - 
---

## 문제 개요

[2011. Final Value of Variable After Performing Operations](https://leetcode.com/problems/final-value-of-variable-after-performing-operations/)


## 문제 접근 방법
- ++, -- 구분해주면 되는데, x++ or ++x 와 같이 두번째 인자만 확인하면 +인지 -인지 알수 있다.


## 실제 코드

```c++
class Solution {
public:
    int finalValueAfterOperations(vector<string>& operations) {
        int x = 0;
        for(int i = 0; i < operations.size(); i++)
        {
            if(operations[i][1] == '+')
            {
                x++;
            }else
                x--;
        }
        return x;
    }
};
```