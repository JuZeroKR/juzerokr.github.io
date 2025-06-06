---
title: "나만의 Vulkan 노트 3주차"
layout: post
toc: true
toc_sticky: true
toc_label: 목차
categories: [Software Development]
tags : [Vulkan, C++]
---


# Vulkan을 이용한 창 분리 (작성중)
## 개요
내가 만들 게임은 여러 창을 사용할 예정이기 때문에, 화면을 분리하는 방법에 대해 공부할 예정이다. 여기서 Scissor와 Viewport를 이이용할 것이다. 

## Viewport와 Scissor
Viewport는 화면에 보여질 내용을 Rendering할 스크린의 영역을 의미한다. 그리고 Scissor는 Rendering 동안 보여질 픽셀 단위의 정해진 지역을 이야기합니다.  

즉, Viewport는 전체 영역을 정의하고 Scissor로 영역을 필터링해준다고 이해했습니다.  

## CommandBuffer와 데이터 저장 
들어가기전 commandBuffer와 데이터 저장에 대해 공부해보고자 합니다.  

CommandBuffer란 GPU에 보낼 명령을 CPU에서 저장시키는 용도로 사용됩니다.  

순서는 다음과 같습니다.  
1. `vkBeginCommandBuffer` (기록 시작)
2. `vkCmdBeginRenderPass` (GPU 명령 시작)
   1. Frame Buffer에 그릴지 지정
   2. 영역 지정
3. `vkCmdBindPipeline` (그래픽 파이프라인 바인딩)
   1. 쉐이더, 상태 지정
4. 그래픽 명령
5. `vkCmdEndRenderPass` (GPU 명령 시작)
6. `vkEndCommandBuffer` (기록 마침)
7. `vkQueueSubmit` (GPU에 제출)



## 추가 및 수정 코드
우선 사용방법을 익히기 위해 화면로 나누는 코드부터 구성했습니다.  

Viewport와 Scissor 설정을 Dynamic State로 지정하여 유동성있게 구성해주었고, `VK_STRUCTURE_TYPE_PIPELINE_DYNAMIC_STATE_CREATE_INFO`을 통해 설정을 하였다.  

화면에 Draw하기전 CommandBuffer를 통해 값을 넘겨주면 됩니다.  

이제 CommandBuffer에 들어갈 데이터를 알아보려고 합니다.  

- Viewport과 Scissor의 경우 위에서 설명했으니 스킵
- Push Constant : 작은 크기의 데이터를 전달할때 사용 (Uniform Buffer보다 빠르고, 드로우마다 값이 자주 바뀌는 데이터에 적합)
- Index : Index Buffer를 사용해서 도형을 그리는 명령



### Reference
- [vulkan-tutorial](https://vulkan-tutorial.com/)
- [Raw-Vulkan](https://alain.xyz/blog/raw-vulkan)

