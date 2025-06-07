---
title: "나만의 Vulkan 노트 3주차"
layout: post
toc: true
toc_sticky: true
toc_label: 목차
categories: [Software Development]
tags : [Vulkan, C++]
---


# Vulkan을 이용한 창 분리
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

맨 처음엔 큐브 1개만 보이고 이후에 2개 -> 3개 -> 4개.. 16분할까지 3초마다 바뀔 예정입니다.

### main 게임 로직 변경

우선 스테이지를 하드코딩해주고, Stage를 3초마다 바뀌는 로직을 추가해주었다.  
스테이지가 변경될때마다 카메라 수를 업데이트 해주고 콜백 데이터 또한 변경해주었다. 

```c++
float lastFrame = 0.0f;
int lastStage = 1;  // 마지막으로 변경된 스테이지 추적
std::vector<int> stages = {1, 2, 3, 4, 6, 8, 9, 10, 12, 16};  // 진행할 스테이지 순서
int currentStageIndex = 0;  // 현재 스테이지 인덱스

while (!glfwWindowShouldClose(window))
        {
            float currentTime = glfwGetTime();
            float deltaTime = currentTime - lastFrame;
            lastFrame = currentTime;

            // 현재 게임 시간 계산
            float gameTime = gameManager->GetCurGameTime(currentTime);
            
            // 3초마다 다음 스테이지로 진행
            if (gameTime >= 3.0f * (currentStageIndex + 1) && currentStageIndex < stages.size() - 1) {
                currentStageIndex++;
                gameManager->SetStage(stages[currentStageIndex]);
                
                // 새로운 뷰포트 수로 MultiCameraManager 업데이트
                multiCameraManager = std::make_unique<MultiCameraManager>(gameManager->GetCurrentViewportCount());
                vulkanCore->setMultiCameraManager(multiCameraManager.get());
                
                // 콜백 데이터 업데이트
                callbackData.multiCameraManager = multiCameraManager.get();
                glfwSetWindowUserPointer(window, &callbackData);
            }

            // .. 이전과 동일
        }
```

## CommandBuffer에 데이터 저장
Viewport 개수에 따라 반복문을 실행시켜준다.  
이때 각 Viewport 마다 영역을 설정해주고 카메라를 세팅해준다.  

```c++
for(int viewportIndex = 0; viewportIndex < multiCameraManager->GetActiveViewportCount(); viewportIndex++) {
   VkViewport viewport = multiCameraManager->CalculateViewport(swapChainExtent, viewportIndex);
   VkRect2D scissor = multiCameraManager->CalculateScissor(swapChainExtent, viewportIndex);

   vkCmdSetViewport(commandBuffer, 0, 1, &viewport);
   vkCmdSetScissor(commandBuffer, 0, 1, &scissor);

   // Get camera for this viewport
   Camera* currentCamera = multiCameraManager->GetCamera(viewportIndex);

   PushConstantData pcData{};
   pcData.model = glm::translate(glm::mat4(1.0f), cubePosition);
   pcData.view = currentCamera->GetViewMatrix();
   pcData.proj = glm::perspective(glm::radians(45.0f),
      viewport.width / viewport.height,
      0.1f, 10.0f);

   pcData.proj[1][1] *= -1.0f;

   vkCmdPushConstants(commandBuffer, pipelineLayout,
      VK_SHADER_STAGE_VERTEX_BIT, 0,
      sizeof(PushConstantData), &pcData);

   vkCmdDrawIndexed(commandBuffer,
      static_cast<uint32_t>(indices.size()),
      1, 0, 0, 0);
}

```




### Reference
- [vulkan-tutorial](https://vulkan-tutorial.com/)
- [Raw-Vulkan](https://alain.xyz/blog/raw-vulkan)

