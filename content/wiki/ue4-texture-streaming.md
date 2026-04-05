---
title: "UE4 Texture 流式加载原理详解"
date: 2026-04-05
draft: false
tags: ["UE4", "Texture", "流式加载", "图形渲染", "内存优化"]
categories: ["技术笔记"]
description: "深入解析 UE4 Texture 流式加载的核心原理，包括 Mipmap 分级加载、流式管理器架构、完整加载流程及配置参数。"
showToc: true
TocOpen: true
---

## 一、流式加载的概念

**流式加载**是一种按需加载资源的技术，核心思想是：
- **不一次性加载所有资源**，而是根据实际需要动态加载
- **根据距离、可见性等因素**，动态调整资源的质量
- **有效控制内存占用**，避免内存溢出

## 二、Texture 流式加载的实现原理

### 1. 基于 Mipmap 的分级加载

Texture 流式加载的核心是 **Mipmap（多级渐远纹理）** 技术：

```
原始纹理 (2048x2048) 
    ↓
Mip 0: 2048x2048 (高分辨率，近距离使用)
Mip 1: 1024x1024
Mip 2: 512x512
Mip 3: 256x256
...
Mip N: 1x1 (最低分辨率，远距离使用)
```

### 2. 关键数据结构

从 `Texture2D.h` 中可以看到：

```cpp
// Texture的平台数据，包含所有Mipmap层级
FTexturePlatformData *PlatformData;

// Mipmap数据存储在PlatformData->Mips中
FORCEINLINE int32 GetNumMips() const
{
    if (PlatformData)
    {
        return PlatformData->Mips.Num();
    }
    return 0;
}
```

### 3. 流式加载管理器

UE4 使用专门的流式加载管理器来处理 Texture：

#### FStreamableManager（通用异步加载）

从 `StreamableManager.h` 可以看到：

```cpp
// 异步加载资源，返回一个Handle用于跟踪加载状态
TSharedPtr<FStreamableHandle> RequestAsyncLoad(
    TArray<FSoftObjectPath> TargetsToStream, 
    FStreamableDelegate DelegateToCall,
    TAsyncLoadPriority Priority
);

// 同步加载资源
UObject* LoadSynchronous(const FSoftObjectPath& Target);
```

#### FRenderAssetStreamingManager（Texture 专用）

从 `StreamingManagerTexture.cpp` 可以看到：

```cpp
// 纹理流式加载管理器，负责：
// 1. 根据距离计算需要的Mipmap层级
// 2. 异步加载/卸载Mipmap数据
// 3. 管理纹理内存池
class FRenderAssetStreamingManager
{
    // 内存池大小
    int64 EffectiveStreamingPoolSize;
    
    // 内存余量
    int64 MemoryMargin;
    
    // 是否使用动态流式加载
    bool bUseDynamicStreaming;
};
```

## 三、Texture 流式加载的完整流程

### 举例说明：一个角色贴图的流式加载

假设有一个 2048x2048 的角色贴图：

<pre style="font-family: monospace; overflow-x: auto; line-height: 1.6;">
┌─────────────────────────────────────────────────────────┐
│  1. 初始化阶段                                            │
│  ┌──────────────────────────────────────────────────┐  │
│  │ 只加载最低分辨率的Mipmap（如64x64）作为基础        │  │
│  │ 内存占用：约 8KB                                   │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  2. 玩家接近角色（距离 &lt; 5米）                            │
│  ┌──────────────────────────────────────────────────┐  │
│  │ FRenderAssetStreamingManager检测到距离变化        │  │
│  │ 计算需要加载的Mipmap层级：Mip 0-3                 │  │
│  │ 异步加载高分辨率Mipmap数据                        │  │
│  │ 内存占用：约 5MB                                  │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│  3. 玩家远离角色（距离 &gt; 50米）                           │
│  ┌──────────────────────────────────────────────────┐  │
│  │ 检测到距离增加，不再需要高分辨率                   │  │
│  │ 卸载高分辨率Mipmap，只保留低分辨率                │  │
│  │ 内存占用：约 64KB                                 │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
</pre>

### 关键代码流程

从 `Texture2D.h` 中：

```cpp
// 流入高分辨率Mipmap
bool StreamIn(int32 NewMipCount, bool bHighPrio) override;

// 流出高分辨率Mipmap
bool StreamOut(int32 NewMipCount) override;

// 获取当前驻留在内存中的Mipmap数量
int32 GetNumResidentMips() const final override;
```

## 四、流式加载的优势

1. **内存优化**：只加载当前需要的分辨率，大幅减少内存占用
2. **性能优化**：远距离物体使用低分辨率纹理，减少 GPU 负担
3. **加载速度**：初始只加载低分辨率数据，快速进入游戏
4. **动态适应**：根据硬件性能自动调整纹理质量

## 五、配置参数

从 `BaseDeviceProfiles.ini` 中可以看到纹理 LOD 组配置：

```ini
; 纹理LOD组配置，控制不同类型纹理的流式加载行为
TextureLODGroups=(Group=TEXTUREGROUP_World,
    MinLODSize=1,
    MaxLODSize=16384,
    LODBias=0,
    MinMagFilter=linear,
    MipFilter=point)

; Shadowmap的流式Mipmap数量
TextureLODGroups=(Group=TEXTUREGROUP_Shadowmap,
    NumStreamedMips=3)
```

## 总结

UE4 的 Texture 流式加载通过 **Mipmap 分级** 和 **距离感知** 实现了高效的内存管理。核心机制是：

- 根据物体与相机的距离动态调整纹理分辨率
- 使用异步加载避免卡顿
- 通过内存池管理纹理内存
- 预计算流式加载数据提升性能

这种机制使得大型开放世界游戏能够在有限的内存中运行，同时保持良好的视觉质量。
