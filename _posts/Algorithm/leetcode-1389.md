---
title: "1389. Create Target Array in the Given Order"
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

[1389. Create Target Array in the Given Order](https://leetcode.com/problems/create-target-array-in-the-given-order/description/)


## 문제 접근 방법
- 단순 구현 문제라 어렵지 않았다.


## 실제 코드

```c++
class Solution {
public:
    vector<int> createTargetArray(vector<int>& nums, vector<int>& index) {
        vector<int> res;
        for(int i = 0; i < nums.size(); i++)
        {
            int num = nums[i];
            int targetIndex = index[i];
            if(res.size() < targetIndex)
                res.push_back(num);
            else
                res.insert(res.begin() + targetIndex, nums[i]);
        }

        return res;
    }
};
```