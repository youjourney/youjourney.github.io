---
title: "Compose Rendering Pipeline & Skia Interaction"
excerpt: "Skia rendering pipeline, LayoutNodes, Recomposer interaction, and how CompositionLocal affects rendering"
categories: ["Jetpack Compose", "Rendering", "Skia"]
tags: ["Skia", "Rendering Pipeline", "LayoutNode", "CompositionLocal"]
last_modified_at: "2025-11-19T14:57:00+09:00"
---

# Compose Rendering Pipeline + Skia Interaction

## 1. Compose Rendering Pipeline Overview
Compose의 렌더링 파이프라인은 다음 단계로 구성된다.  
The Compose rendering pipeline consists of the following stages:

```
Composition → Layout → Drawing → Skia GPU Pipeline
```

## 2. CompositionLocal의 영향 지점  
CompositionLocal affects rendering **during composition stage**, deciding:

- 어떤 색상/타이포그래피를 사용할지  
- 레이아웃 방향(LocalLayoutDirection)  
- 간격/패딩(LocalSpacing)  
- 로케일(LocalLocaleConfig)

## 3. LayoutNode 구조
Compose UI는 LayoutNode 트리 구조로 표현된다.

```
LayoutNode
 ├─ modifiers
 ├─ measure policy
 ├─ draw block
 └─ children
```

CompositionLocal 값은 LayoutNode의 measure/draw 정책에 영향을 줄 수 있다.

## 4. Skia Rendering Flow
```
CanvasLayer → Skia Canvas → GPU Texture → Screen
```

CompositionLocal이 draw 단계에 영향을 주는 대표 사례:
- LocalContentColor
- LocalTextStyle
- LocalDensity (dp → px 변환)

## 5. Rendering Pipeline Diagram
```
CompositionLocalProvider
    ↓
Composition runs
    ↓
SlotTable stores environment
    ↓
LayoutNodes built with environment
    ↓
Skia draws using LayoutNode-rendered ops
```
