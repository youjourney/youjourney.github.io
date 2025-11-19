---
title: "Compose & Skia GPU Rendering"
excerpt: "RenderNode, DisplayList, GPU pipeline, and how Compose translates UI to Skia GPU calls"
categories: ["Jetpack Compose", "Rendering", "GPU"]
tags: ["Skia", "GPU", "RenderNode", "DisplayList"]
last_modified_at: "2025-11-19T15:03:00+09:00"
---

# Compose & Skia GPU Rendering

## 1. Skia 기반 렌더링 개요

Android는 Skia를 통해 Canvas API를 구현하고, GPU 가속을 통해 그린다.  
Android uses Skia as its 2D rendering engine, GPU accelerated.

Compose는 LayoutNode의 draw 명령을 Canvas(=Skia) 호출로 변환한다.  
Compose translates LayoutNode draw ops into Skia canvas calls.

---

## 2. RenderNode & DisplayList

- RenderNode: draw 명령과 properties를 저장  
- DisplayList: batched draw commands for GPU

Compose는 각 LayoutNode에 대해 draw block을 가지고 있고, 이것이 RenderNode / DisplayList로 연결된다.

---

## 3. CompositionLocal이 GPU 단계에 미치는 영향

직접적으로는 GPU 파이프라인을 건드리지 않지만:

- LocalContentColor → paint color에 반영  
- LocalDensity → stroke width, radius 등에 반영  
- LocalLayoutDirection → clip/translate 연산에 반영  

즉, CompositionLocal은 “Rendering parameters”를 결정한다.

---

## 4. Rendering Pipeline Flow

```
SlotTable + CompositionLocal
    ↓
LayoutNode build
    ↓
Measure & Layout
    ↓
Draw scope → Canvas commands
    ↓
Skia → GPU
    ↓
Screen
```

---

## 5. 성능 관련 포인트

- overdraw 최소화  
- unnecessary recomposition 피하기  
- heavy draw 작업은 Canvas clip/transform 최적화  
- large lists에서는 LazyX 컴포넌트 사용
