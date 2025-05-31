---
title: "나만의 Vulkan 노트 2주차"
layout: post
toc: true
toc_sticky: true
toc_label: 목차
categories: [Software Development]
tags : [Vulkan, C++]
---


# Vulkan 키보드 입력으로 화면 조정
## 개요
키보드 A,S,D,W를 이용해서 Cube 이동 구현

### 수정 필요 부분
1. 키보드 Input 제어 (glfw 이용)
2. DeltaTime을 통한 Frame 제어
3. PushConstantData 수정 (Shader 관련)


### 수정한 부분
1. 변수 정의
- 실시간 cube 위치 정의
- 카메라 이동 속도 정의
- deltaTime 정의하여 모든 PC Frame 제어

```c++
glm::vec3 cubePosition = glm::vec3(0.0f);  // Starting at center
float moveSpeed = 2.0f;    
glm::vec3 cameraPos = glm::vec3(0.0f, 0.0f, 3.0f);
float currentTime = 0.0f; 
float deltaTime = 0.0f;
float lastFrame = 0.0f;
```


2. 키보드 입력을 듣고 처리하는 코드

```c++
if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
{
    glfwSetWindowShouldClose(window, true);
}

if (glfwGetKey(window, GLFW_KEY_A) == GLFW_PRESS)
{
    cubePosition.x -= moveSpeed * deltaTime;
}
if (glfwGetKey(window, GLFW_KEY_D) == GLFW_PRESS)
{
    cubePosition.x += moveSpeed * deltaTime;
}
if (glfwGetKey(window, GLFW_KEY_W) == GLFW_PRESS)
{
    cubePosition.z -= moveSpeed * deltaTime;
}
if (glfwGetKey(window, GLFW_KEY_S) == GLFW_PRESS)
{
    cubePosition.z += moveSpeed * deltaTime;
}
```

3. PushConstantData 변경
Shader에 전달할 데이터를 변경해줘야합니다.  
glm::translate 함수를 통해 cubePosition 만큼 이동시켜줍니다.
```c++
PushConstantData pcData{};
// 모델 매트릭스: 단위 행렬 또는 원하는 트랜스폼
// 원점에서부터 CubePosition만큼 이동
pcData.model = glm::translate(glm::mat4(1.0f), cubePosition);

```



# Vulkan 마우스 우클릭으로 화면 제어
우리가 큐브를 보고 있는 카메라를 하나 생성한다고 생각하면 됩니다.

1. 마우스 사용을 위한 glfw 설정
```c++
// Set up mouse callback
glfwSetWindowUserPointer(window, this);
glfwSetCursorPosCallback(window, mouseCallback);
glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_DISABLED);  // Hide and capture cursor
```

2. yaw, pitch 등 변수 생성
```c++
// Add these variables to the private section
float yaw = -90.0f;    // Horizontal rotation
float pitch = 0.0f;     // Vertical rotation
float lastX = 400.0f;   // Last mouse X position
float lastY = 300.0f;   // Last mouse Y position
bool firstMouse = true; // First mouse input flag
float mouseSensitivity = 0.05f;
glm::vec3 cameraFront = glm::vec3(0.0f, 0.0f, -1.0f);
glm::vec3 cameraUp = glm::vec3(0.0f, 1.0f, 0.0f);
```

3. 키보드 입력 A, S, D, W 입력값에 따른 제어 변경
기본에는 키보드 입력으로 단순히 큐브를 조종했다면  
바뀐 코드는 카메라를 조종하는 역할을 합니다.

```c++
 // Forward/Backward
if (glfwGetKey(window, GLFW_KEY_W) == GLFW_PRESS)
    cameraPos += cameraSpeed * cameraFront;
if (glfwGetKey(window, GLFW_KEY_S) == GLFW_PRESS)
    cameraPos -= cameraSpeed * cameraFront;
    
// need cross product
if (glfwGetKey(window, GLFW_KEY_A) == GLFW_PRESS)
    cameraPos -= glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
if (glfwGetKey(window, GLFW_KEY_D) == GLFW_PRESS)
    cameraPos += glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;

```
4. 3차원 카메라 이동에 따른 각도 계산
마우스 위치 변화를 계산하여 각도와 함께 카메라 위치를 조정합니다.

![3D 마우스 프로세스 설명](/assets/images/Vulkan3DProcessMouse.png)

지금 카메라 바라보는 방향을 1이라고 가정하고 고개를 든 각도와 돌린 각도를 각각 두고 계산을 해보았습니다.  
확실히 그림을 그리면서 하는게 저한텐 이해가 잘되었습니다.   

```c++
void processMouseMovement(double xpos, double ypos) {
        if (firstMouse) {
            lastX = xpos;
            lastY = ypos;
            firstMouse = false;
        }

        float xoffset = xpos - lastX;
        float yoffset = ypos - lastY; // Reversed since y-coordinates range from bottom to top
        lastX = xpos;
        lastY = ypos;

        xoffset *= mouseSensitivity;
        yoffset *= mouseSensitivity;

        yaw += xoffset;
        pitch += yoffset;

        // Constrain pitch
        if (pitch > 89.0f)
            pitch = 89.0f;
        if (pitch < -89.0f)
            pitch = -89.0f;

        // Update camera front vector
        glm::vec3 direction;
        direction.x = cos(glm::radians(yaw)) * cos(glm::radians(pitch));
        direction.y = sin(glm::radians(pitch));
        direction.z = sin(glm::radians(yaw)) * cos(glm::radians(pitch));
        cameraFront = glm::normalize(direction);
    }

```




### Reference
- [vulkan-tutorial](https://vulkan-tutorial.com/)
- [Raw-Vulkan](https://alain.xyz/blog/raw-vulkan)

