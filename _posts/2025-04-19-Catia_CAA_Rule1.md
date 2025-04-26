---
title: "CATIA Rule 개발 시 Tip 정리"
layout: post
toc: true
toc_sticky: true
toc_label: 목차
categories: [CATIA-CAA]
tags : [CATIA, CAA, Rule]
---


# CATIA Rule 이란
CATIA Rule은 특정 파라미터에 대한 규칙 또는 제약 조건을 설정하는 기능입니다.  
파라미터 변경되면 그에 따라 설정된 값이 자동으로 변경됩니다.


---


# CATIA CAA로 Rule을 개발할 때 방식
CAA로 CATIA Rule을 개발하는 방법은 대표적으로 2가지가 있습니다.


---


## 1. CATI API 이용
`CATICkeParmFactory` 인터페이스의 `CreateProgram` 메소드를 통해 Rule 생성이 가능합니다.  
이때 Reture값으로 `CATICkeRelation_var`이 나오게 됩니다.  

주의할 점은 4번째 인자에 `CATICkeListOf` 리스트를 넣어줘야합니다.  
- 리스트의 경우 Rule에 작성된 파라미터를 의미하고 생성전에 미리 담아줘야 합니다.
- 만약 파라미터가 맞지 않는 경우 `S_FAIL`이 반환됩니다.  

Rule과 파라미터간의 구문 내 명명 규칙은 다음과 같습니다.  
- 첫번째 파라미터 -> a1  
- 두번째 파라미터 -> a2  
- N번째 파라미터 -> aN


---


## 2. CATIA API 이용
`CATIARelations`의 `CreateProgram` 메소드를 통해 Rule 생성이 가능합니다.  
이때 `CATIAPart`의 `get_Relation` 메소드를 통해 `CATIARelations`을 가져올 수 있습니다. (이는 최상위 트리의 Relations입니다.)  
-  `CreateProgram`의 3번째 인자는 `CATBSTR`형태로 Rule 전체 구문을 넣어주면됩니다.


---


## 선택한 방식 및 이유
저 같은 경우에는 두번째 방식을 사용했습니다.  
Geometric Set과 일반 트리에 있는 Geometric Feature들을 Rule 구문에 작성하려했지만, 이들을 파라미터로 정의하는 방법을 찾지 못했습니다.  


따라서 두번째 API를 통해서 인자들을 직접 문자열 형태(``을 둘러싼 경로)로 만들어 구문을 직접 작성하였습니다. 


---


# 개발 팁

## Part 최상위 노드 가져오기
Part 파일의 최상위 노드라 하면 일반적으로 CATPart(Part파일 이름 형태)입니다.  
직접 파라미터로 이를 지정하면 Update 문제가 발생할 수 있습니다.  
이런 경우 구문 내 `Relations.Owner` 문법을 통해 CATPart을 가져올 수 있었습니다.  

```
Let root(Feature)
root Relations.Owner
```
이때 Relations는 CATIA Rule을 담고 있는 Relations을 의미합니다.

## 모든 Feature들을 가져와서 Show 속성만 있는 Feature만 Show 하는 방법
트리 내 모든 Feature에 Show속성이 있는게 아니었습니다.
그래서 Feature 중에서 속성이 Show있는지 검사하고, 있다면 이 속성을 이용해서 Show을 호출했습니다.  
```
Let featureList(List)
featureList = root.Children
Let feat(Feature)
Let hasShowAttr(Boolean)
for feat inside featureList
{
    hasShowAttr = feat->HasAttribute("Show")
    if(hasShowAttr)
    {
        feat->SetAttributeBoolean("Show", false)
    } 
}
```

## Query 통해서 트리 내 모든 GeoSet 가져오기.
Rule 구문 내에서 모든 GeometricSet을 가져오고 싶었습니다.  
자료를 찾아보니 Rule 내에서는 재귀 탐색이 불가능하였습니다.  
그렇기 때문에 `Query`을 이용해서 모든 GeometricSet을 가져올 수 있었습니다.  

```
Let geoSets(List)
geoSets = `Feature이름`->Query("OpenBodyFeature", "")
```

## Feature List 내 특정 Feature만 가져오기
`IsSupporting` 함수를 통해 특정 Feature인지 아닌지 검사가 가능합니다.
```
`Feature이름`->IsSupporting("Solid")
```

Solid 타입인지 검사하는 코드입니다.

## CATIARelation의 get_Value시 주의할점 

CATIARelation의 `get_Value` 함수를 통해 Rule 구문을 가져왔을경우 모든 파라미터 앞에 Part 파일의 주소가 붙습니다.  
당연한 이야기이지만.. 이것을 그대로 Rule 구문에 넣어서 Modify을 하면 리턴은 성공이 나오지만 구문 자체는 바뀌지 않습니다.  (문법 오류로 바뀌지 않는듯합니다.)


## 기존 생성되어 있던 CATIA Rule가져오기 (25-04-26 추가)

CATIAPart 인터페이스에서 CATIARelations를 가져옵니다.  
CATIARelations method인 get_Count와 Item을 사용하여 반복문을 통해 순차적으로 가져오면됩니다.  
이때 주의해야할 점은 Rule를 제외하고 다른 포맷의 Relation까지 얻게됩니다.  
이를 CATISpecObject_var로 형변환 시킨 후 GetType을 통해 `RelationExpPg`인 Realtion만 선택하면됩니다.


## Rule 내에 파라미터로 작성되어있던 Object가 Delete되었을 때 (25-04-26 추가)

이 경우 우리는 구문 내에 Relations\\(Parameter이름)\\(Object이름) 형태와 함께 deleted라는 문구를 볼 수 있습니다.  
하지만, 생성된 Rule을 한번 클릭해야 위 문구로 변환이되며, 만약 클릭하지 않는다면.. 원래 구문 형태의 포맷이 나오게됩니다.

