---
title: "나만의 Vulkan 노트"
layout: post
toc: true
toc_sticky: true
toc_label: 목차
categories: [Software Development]
tags : [Vulkan, C++]
---


# Vulkan Api를 이용한 Game 개발

Vulkan Api 이해하기 위해 Game을 개발할 예정이다.  
단순한 3D 총알 피하기 게임이지만 시점의 변화를 다양하게 줄 것이다.  
플레이어는 캐릭터를 몇몇 제한적인 카메라 시점 통해 조종할 수 있다.
처음에는 쉬운 시점 2개로 시작하지만, 시간이 지날수록 4개..8개로 늘어난다.  
당신은 몇초동안 살아남을 수 있는지 측정하는 게임이다.

## 주 개발 목표 (25-05-11)
- Vulkan Api와 Rendering 과정 이해
- Gpu 동작 과정이 Software적으로 어떻게 제어하는지 정리
- Rendering 과정 최적화 경험

## 이번주 진행과정
- Vulkan Tutorial을 통해 2D Triangle 출력
- 해당 코드를 3D Cube 출력되게 수정

## Vulkan 초기 설정

1. VulkanInstance (Vulkan 사용을 위한 애플리케이션)
2. Physical Device (시스템에 있는 GPU 목록에서 원하는 기능을 지원하는거 선택)
3. Logical Device (Physical Device의 안전한 접근을 위한 가상 인터페이스. 커맨드 큐, 기능 확장 등을 설정)
4. Swapchain 설정 (GPU가 렌더링한 이미지를 화면에 출력하기 위한 버퍼 큐 및 설정)
5. ImageViews 설정 (Swapchain 이미지에 접근하기 위한 뷰(view))
6. SwapChain에 Rendering 정보 추가 (렌더링할 때 어떤 형식(format)의 이미지에 어떤 순서로 처리할지 정의)
7. Grapic Pipeline 생성 (IA, Viewport, Rasterization 등등)
8. FrameBuffer 생성 (Swapchain 이미지에 실제로 렌더링할 수 있게 연결된 Render Binding)
9. CommandPool 생성 (GPU 작업 명령을 저장하는 버퍼를 관리하기 위한 객체)
10. VertexBuffer (정점(Vertex) 데이터를 GPU에 넘기기 위한 버퍼)
11. CommandBuffer (GPU 작업 명령을 저장하는 버퍼)
12. Semaphore 및 ,Fence 생성 (GPU 작업의 동기화를 위한 도구.)



### Reference
- [vulkan-tutorial](https://vulkan-tutorial.com/)
- [Raw-Vulkan](https://alain.xyz/blog/raw-vulkan)