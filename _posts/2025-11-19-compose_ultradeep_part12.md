---
title: "Compose Layout System Internals"
excerpt: "Measure, layout, placement, and how CompositionLocal feeds into layout behavior"
categories: ["Jetpack Compose", "Layout"]
tags: ["Layout", "MeasurePolicy", "Density", "Intrinsic"]
last_modified_at: "2025-11-19T15:02:00+09:00"
---

# Compose Layout System Internals

## 1. Layout Pipeline

Compose Layout은 세 단계로 구성된다.  
Compose layout has three major stages.

```
Measure → Layout (position) → Draw
```

---

## 2. MeasurePolicy

커스텀 레이아웃은 `measurePolicy`로 동작한다.

```kotlin
@Composable
fun MyLayout(content: @Composable () -> Unit) {
    Layout(
        content = content,
        measurePolicy = { measurables, constraints ->
            val placeables = measurables.map { it.measure(constraints) }
            val width = placeables.maxOf { it.width }
            val height = placeables.sumOf { it.height }

            layout(width, height) {
                var y = 0
                placeables.forEach { p ->
                    p.place(0, y)
                    y += p.height
                }
            }
        }
    )
}
```

---

## 3. Constraints & Density

Constraints:

- minWidth, maxWidth
- minHeight, maxHeight

Density:

- dp/sp를 px로 변환  
- `LocalDensity`로 제공

CompositionLocal은 Layout behavior를 간접적으로 제어할 수 있다.  
예: LocalSpacing, LocalLayoutDirection, LocalDensity.

---

## 4. Intrinsic Measurements

`intrinsicSize`는 컨텐츠 기반 크기 계산에 사용된다.  
Intrinsic sizes allow measuring based on content.

`Modifier.width(IntrinsicSize.Min)` 등.

---

## 5. LayoutNode와 SlotTable

SlotTable이 UI 구조를 표현하면, LayoutNode는 실제 레이아웃 트리를 표현한다.

```
SlotTable (logical)
LayoutNode tree (physical)
```

CompositionLocal 값은 SlotTable → LayoutNode로 전달되어 measure/draw에 영향을 준다.
