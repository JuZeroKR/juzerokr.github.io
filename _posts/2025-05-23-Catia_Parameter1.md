---
title: "CATIA에서 Parameter Value 삽입"
layout: post
toc: true
toc_sticky: true
toc_label: 목차
categories: [Software Development]
tags : [CATIA, CAA, Parameter]
---

# 개요
CATIA에서 Parameter는 모델의 형상이나 특성을 정의하는 값입니다.  
이 Parameter의 Value 변경하는 코드를 공유할 것입니다.  
Parameter는 single Value와 Multi Value가 있는데, 이 코드의 경우 두 Value에서 동작합니다.  

오늘은 String Type Parameter를 예시로 들겠습니다.


## 코드 해석
바로 코드 해석으로 들어가겠습니다.  
CATIA Automation을 이용해서 Value 접근하였습니다.  

```
CATIkeUnit_var spCurrentUnit = NULL_var;
CATIkeMagnitude_var spMagnitude = iospParam->Type();
CATListOfCATUnicodeString valueList;
CATUnicodeString usUnit;

// 단위 문자열 추출
if (!!spMagnitude)
{
    CATIkeUnit_var spCurrentUnit = spMagnitude->CurrentUnit();
    if (!!spCurrentUnit)
        usUnit = spCurrentUnit->Symbol();
}

// Parameter 인터페이스를 통해 Value 가져오기
CATIParameter* pCATIParameter = NULL;
if (SUCCEEDED(iospParam->QueryInterface(IID_CATIParameter, (void**)&pCATIParameter)))
{
    // String Type
    CATIAStrParam* pCATIAStrParam = NULL;
    if (SUCCEEDED(pCATIParameter->QueryInterface(IID_CATIAStrParam, (void**)&pCATIAStrParam)))
    {
        // 먼저 Value 개수 얻고
        CATLONG valueSize = 0;
        pCATIAStrParam->GetEnumerateValuesSize(valueSize);
        
        // 싱글 혹은 멀티에 따라 다르게 처리 
        // 기존 Value를 가져온 다음 신규 Value 값 추가
        if (valueSize <= 0)
        {
            // Single Type
            BSTR bsValue;
            pCATIAStrParam->get_Value(bsValue);
            CATUnicodeString iousStr;
            iousStr.BuildFromBSTR(bsValue);
            valueList.Append(iousStr + usUnit);
        }
        else
        {
            // Multi Type
            CATUnicodeString* ivalueList = new CATUnicodeString[valueSize];
            CATSafeArrayVariant* pArrayVariant;
            pArrayVariant = BuildSafeArrayVariant(ivalueList, valueSize);
            HRESULT rc = pCATIAStrParam->GetEnumerateValues(*pArrayVariant);
            if (SUCCEEDED(rc))
            {
                unsigned long size = valueSize;
                CATUnicodeString* iovalueList = new CATUnicodeString[valueSize];
                ConvertSafeArrayVariant(*pArrayVariant, iovalueList, ivalueList, valueSize);
                for (int v = 0; v < valueSize; v++)
                {
                    CATUnicodeString iousStr = iovalueList[v];
                    valueList.Append(iousStr + usUnit);
                }
                delete[] iovalueList;
            }
            FreeVariantSafeArray(*pArrayVariant);
            delete[] ivalueList;
        }

        // Value를 SetEnumerateValues를 통해 저장
        valueList.Append(iusValueName);
        CATSafeArrayVariant* pArrayVariant;
        CATUnicodeString* ivalueList = new CATUnicodeString[valueList.Size()];
        for (int i = 0; i < valueList.Size(); i++)
            ivalueList[i] = valueList[i + 1];

        pArrayVariant = BuildSafeArrayVariant(ivalueList, valueList.Size());
        pCATIAStrParam->SetEnumerateValues(*pArrayVariant);
    }
}
```

## 마치며
코드 예제에는 삽입만 존재하지만 만약 삭제를 구현하고 싶으시다면 똑같이 GetEnumerateValuesSize를 통해 Value 리스트가져오고 리스트 값을 변경한다음 SetEnumerateValues을 이용하여 저장하면 됩니다.