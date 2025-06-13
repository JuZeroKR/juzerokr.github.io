---
title: "나만의 Vulkan 노트 4주차"
layout: post
toc: true
toc_sticky: true
toc_label: 목차
categories: [Software Development]
tags : [Vulkan, C++]
---


# Vulkan을 이용한 Back View 구현
## 개요
오늘은 흔한 3D게임의 BackView 시점을 구현해볼 예정입니다.  
그리고 키보드와 마우스로 큐브와 시점을 조정하여 3D 게임에 기본이 되는 움직임을 만들어볼 예정입니다.  

WASD키를 이용해서 큐브를 움직이고, 카메라는 큐브 뒤에 일정 거리 이상 거리를 유지하도록 구현하겠습니다.  

저번주에 했던 화면 나누기를 잠시 off 시켜놓고 진행하겠습니다.  

## 코드 수정 부분  

### 큐브 이동 여부 확인을 위한 격자 바닥 생성  
현재 화면에는 큐브만 보이는 상태입니다. 큐브를 이동시켜봤자 아무런 변화를 못느끼겠죠.  
그렇기에 저는 임시 바닥을 생성하여 큐브 이동을 확인할 예정입니다.  

먼저 격자의 vertex 위치를 생성하여 저장해둡니다.

```c++
void VulkanCore::generateGridData() {
    gridVertices.clear();
    gridIndices.clear();
    
    const int gridSize = 30;  // 30x30 격자 (더 넓은 바닥)
    const float gridSpacing = 0.5f;  // 더 촘촘한 격자 간격
    const float gridOffset = (gridSize - 1) * gridSpacing * 0.5f;  // 중앙 정렬
    
    // 격자 정점 생성
    for (int x = 0; x < gridSize; x++) {
        for (int z = 0; z < gridSize; z++) {
            float posX = x * gridSpacing - gridOffset;
            float posZ = z * gridSpacing - gridOffset;
            
            // 검은 배경에 어울리는 격자 패턴
            glm::vec3 color;
            if ((x + z) % 2 == 0) {
                color = glm::vec3(0.3f, 0.3f, 0.3f);  // 밝은 회색
            } else {
                color = glm::vec3(0.1f, 0.1f, 0.1f);  // 어두운 회색
            }
            
            gridVertices.push_back({{ posX, -1.0f, posZ }, color});
        }
    }
    
    // 격자 인덱스 생성 (삼각형 쌍)
    for (int x = 0; x < gridSize - 1; x++) {
        for (int z = 0; z < gridSize - 1; z++) {
            int i = x * gridSize + z;
            int iNext = (x + 1) * gridSize + z;
            
            // 첫 번째 삼각형
            gridIndices.push_back(i);
            gridIndices.push_back(iNext);
            gridIndices.push_back(i + 1);
            
            // 두 번째 삼각형
            gridIndices.push_back(iNext);
            gridIndices.push_back(iNext + 1);
            gridIndices.push_back(i + 1);
        }
    }
}
```

이후에 격자 정보를 넘기기 위한 Buffer를 생성해줍니다. (Index, Vertex)  

```c++
void VulkanCore::createGridBuffers() {
    // Grid Vertex Buffer
    VkBufferCreateInfo vertexBufferInfo{};
    vertexBufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
    vertexBufferInfo.size = sizeof(gridVertices[0]) * gridVertices.size();
    vertexBufferInfo.usage = VK_BUFFER_USAGE_VERTEX_BUFFER_BIT;
    // Only one queue family uses the resource
    vertexBufferInfo.sharingMode = VK_SHARING_MODE_EXCLUSIVE;

    if (vkCreateBuffer(device, &vertexBufferInfo, nullptr, &gridVertexBuffer) != VK_SUCCESS) {
        throw std::runtime_error("failed to create grid vertex buffer!");
    }

    // the requirements for allocating memory for a resource
    VkMemoryRequirements memRequirements;
    vkGetBufferMemoryRequirements(device, gridVertexBuffer, &memRequirements);

    // VkMemoryAllocateInfo : Info in memory to Vulkan
    VkMemoryAllocateInfo allocInfo{};
    allocInfo.sType = VK_STRUCTURE_TYPE_MEMORY_ALLOCATE_INFO;
    allocInfo.allocationSize = memRequirements.size;
    allocInfo.memoryTypeIndex = findMemoryType(memRequirements.memoryTypeBits, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT);

    if (vkAllocateMemory(device, &allocInfo, nullptr, &gridVertexBufferMemory) != VK_SUCCESS) {
        throw std::runtime_error("failed to allocate grid vertex buffer memory!");
    }

    // Binding
    vkBindBufferMemory(device, gridVertexBuffer, gridVertexBufferMemory, 0);

    // create connection between cpu and gpu
    void* data;
    vkMapMemory(device, gridVertexBufferMemory, 0, vertexBufferInfo.size, 0, &data);
    memcpy(data, gridVertices.data(), (size_t)vertexBufferInfo.size);
    vkUnmapMemory(device, gridVertexBufferMemory);

    // Grid Index Buffer
    VkBufferCreateInfo indexBufferInfo{};
    indexBufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
    indexBufferInfo.size = sizeof(gridIndices[0]) * gridIndices.size();
    indexBufferInfo.usage = VK_BUFFER_USAGE_INDEX_BUFFER_BIT;
    indexBufferInfo.sharingMode = VK_SHARING_MODE_EXCLUSIVE;

    if (vkCreateBuffer(device, &indexBufferInfo, nullptr, &gridIndexBuffer) != VK_SUCCESS) {
        throw std::runtime_error("failed to create grid index buffer!");
    }

    vkGetBufferMemoryRequirements(device, gridIndexBuffer, &memRequirements);

    allocInfo.allocationSize = memRequirements.size;
    allocInfo.memoryTypeIndex = findMemoryType(memRequirements.memoryTypeBits, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT);

    if (vkAllocateMemory(device, &allocInfo, nullptr, &gridIndexBufferMemory) != VK_SUCCESS) {
        throw std::runtime_error("failed to allocate grid index buffer memory!");
    }

    vkBindBufferMemory(device, gridIndexBuffer, gridIndexBufferMemory, 0);

    vkMapMemory(device, gridIndexBufferMemory, 0, indexBufferInfo.size, 0, &data);
    memcpy(data, gridIndices.data(), (size_t)indexBufferInfo.size);
    vkUnmapMemory(device, gridIndexBufferMemory);
}

```

recordCommandBuffer에 격자 화면을 그리기 위한 코드를 추가해줍니다.

```c++
// Draw grid floor
VkDeviceSize gridOffsets[] = {0};
vkCmdBindVertexBuffers(commandBuffer, 0, 1, &gridVertexBuffer, gridOffsets);
vkCmdBindIndexBuffer(commandBuffer, gridIndexBuffer, 0, VK_INDEX_TYPE_UINT16);

// Grid transformation matrix (keeping it at y = -1)
struct {
    glm::mat4 model;
    glm::mat4 view;
    glm::mat4 proj;
} gridMatrices;

gridMatrices.model = glm::mat4(1.0f); // Grid는 이미 y=-1에 위치
gridMatrices.view = matrices.view;
gridMatrices.proj = matrices.proj;

vkCmdPushConstants(commandBuffer, pipelineLayout,
    VK_SHADER_STAGE_VERTEX_BIT, 0,
    sizeof(gridMatrices), &gridMatrices);

vkCmdDrawIndexed(commandBuffer,
    static_cast<uint32_t>(gridIndices.size()),
    1, 0, 0, 0);

vkCmdEndRenderPass(commandBuffer);

if (vkEndCommandBuffer(commandBuffer) != VK_SUCCESS) {
    throw std::runtime_error("failed to record command buffer!");
}

```




### Reference
- [vulkan-tutorial](https://vulkan-tutorial.com/)
- [Raw-Vulkan](https://alain.xyz/blog/raw-vulkan)

