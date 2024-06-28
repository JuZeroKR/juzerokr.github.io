---
title: "1603. Design Parking System"
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

[1603. Design Parking System](https://leetcode.com/problems/design-parking-system/description/)


## 문제 접근 방법
- 단순 구현 문제라 어렵지 않았다.


## 실제 코드

```c++
class ParkingSystem {
public:
    int curSmallParkingLot;
    int curMediumParkingLot;
    int curBigParkingLot;

    int numSmallParkingLot;
    int numMediumParkingLot;
    int numBigParkingLot;

    ParkingSystem(int big, int medium, int small) {
        numSmallParkingLot = small;
        numMediumParkingLot= medium;
        numBigParkingLot = big;

        curSmallParkingLot = 0;
        curMediumParkingLot = 0;
        curBigParkingLot = 0;
    }
    
    bool addCar(int carType) {
        if(carType == 3 && numSmallParkingLot == curSmallParkingLot)
            return false;

        if(carType == 2 && numMediumParkingLot == curMediumParkingLot)
            return false;

        if(carType == 1 && numBigParkingLot == curBigParkingLot)
            return false;

        if(carType == 3)
            curSmallParkingLot++;
        else if(carType == 2)
            curMediumParkingLot++;
        else if(carType == 1)
            curBigParkingLot++;

        return true;
    }
};

/**
 * Your ParkingSystem object will be instantiated and called as such:
 * ParkingSystem* obj = new ParkingSystem(big, medium, small);
 * bool param_1 = obj->addCar(carType);
 */
```