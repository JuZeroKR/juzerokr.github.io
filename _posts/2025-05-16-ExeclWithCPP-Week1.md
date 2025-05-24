---
title: "C++에서 Excel(COM Automation)을 이용해서 이미지 삽입"
layout: post
toc: true
toc_sticky: true
toc_label: 목차
categories: [Software Development]
tags : [Excel, C++]
---

# 개요
C++을 이용해 Excel 파일을 생성하고 조작할 일이 있었습니다.  
Microsoft에서 제공하는 `COleDispatchDriver` 클래스를 사용하여 Excel과 연동하였고,  
다행히 기능별로 잘 래핑된 Wrapper 클래스들이 존재해 비교적 쉽게 사용할 수 있었습니다 [Reference-1].

텍스트나 셀 설정과 같은 기본 기능은 무리 없이 동작했습니다.  
하지만 이미지를 삽입한 후 크기를 줄이면 해상도가 현저히 저하되는 문제가 발생했습니다.  
이 글에서는 해상도 저하 문제를 해결하기 위한 시도와 접근 방법을 공유하고자 합니다.

---

## 이미지 삽입 방식 비교

이미지를 Excel에 삽입하는 방법은 크게 두 가지가 있습니다.

1. `Shape.AddPicture` 사용 (msoPicture = 13)
2. `Shape.Fill.UserPicture` 사용 (msoAutoShape = 1)

두 방법 모두 이미지 삽입은 가능하지만, 해상도와 추가 효과 적용 면에서 차이가 있습니다.

- **1번 방식**은 이미지 삽입 후 `ScaleHeight` 또는 `put_Height()` 등을 통해 크기를 조절하면 해상도가 떨어지는 문제가 발생합니다.
- **2번 방식**은 크기를 조절해도 해상도가 잘 유지되며 깨끗합니다.

하지만 문제는 **배경 제거 효과(투명도 설정)**입니다.  
`PictureFormat`을 이용해 흰색 배경을 제거하고 싶었는데, **2번 방식에서는 `PictureFormat.TransparencyColor` 설정 시 오류가 발생했습니다**.

즉,
- **고해상도를 원한다면 2번 방식이 적합하고**
- **배경 제거 효과를 원한다면 1번 방식이 필요**합니다.

---

## 버튼과 VBA 삽입을 이용하여 이미지 해상도 유지  
제가 여러 방법으로 해본 결과 CPP로 삽입된 이미지를 축소시키면 해상도 저하를 피할 수 없다고 느꼈습니다.  
즉, VBA를 제공하여 사용자가 버튼 클릭으로 인해 자동으로 이미지를 축소시켜주는 방향으로 개발을 하였습니다.

### VBA 코드 삽입
hex의 경우 각 함수별 dispid입니다.

```c++
// Button 삽입
OLECHAR* szName = L"Buttons";

// m_Sheet -> WorkSheet 객체
Buttons buttons(m_Sheet.Buttons());
LPDISPATCH pBtnDisp = buttons.Add(left, top, width, height); // 0xb5
Button btn(pBtnDisp);

// 이벤트 설정
btn.SetCaption((LPCTSTR)strButtonText);   // 0x8b
btn.SetOnAction(_T("Module1.ResizeAllPictures"));  // 0x254

// 동적 삽입
VBProject vbProject = m_Book.GetVBProject(); // 0x5bd
VBComponents vbComps = vbProject.GetVBComponents(); // 0x87

LPDISPATCH modDisp = vbComps.AddModule(1); // 0xc
VBComponent vbComp(modDisp);
CodeModule codeMod(vbComp.GetCodeModule()); // GetCodeModule 0x32

CString resizeMacro =
    _T("Public Sub ResizeAllPictures()\r\n")
    _T("    Dim shp As Shape\r\n")
    _T("    Dim scaleFactor As Double\r\n")
    _T("    scaleFactor = 0.3\r\n")
    _T("    For Each shp In ActiveSheet.Shapes\r\n")
    _T("        If shp.Type = msoPicture Then\r\n")
    _T("            shp.LockAspectRatio = msoTrue\r\n")
    _T("            shp.ScaleHeight scaleFactor, msoTrue\r\n")
    _T("            shp.ScaleWidth scaleFactor, msoTrue\r\n")
    _T("        End If\r\n")
    _T("    Next shp\r\n")
    _T("End Sub\r\n");

codeMod.AddFromString(resizeMacro); // 0x60020004

```


---


### Reference
1. [Reference-Sourcode](https://github.com/dwatow/xls2oul)