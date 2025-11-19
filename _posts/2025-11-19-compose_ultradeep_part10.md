---
title: "Compose Animation & ConstraintLayout Internals"
excerpt: "Jetpack Compose Animation system, Transition APIs, AnimationClock vs FrameClock, and ConstraintLayout under the hood"
categories: ["Jetpack Compose", "Animation", "Layout"]
tags: ["Animation", "Transition", "ConstraintLayout", "FrameClock"]
last_modified_at: "2025-11-19T15:00:00+09:00"
---

# Compose Animation & ConstraintLayout Internals

## 1. Compose Animation System Overview

Compose 애니메이션 시스템은 **state → animationSpec → animationClock** 기반으로 동작한다.  
The animation system in Compose is built on **state → animationSpec → animationClock**.

주요 API:

- `animate*AsState` (e.g. `animateFloatAsState`)
- `updateTransition`
- `AnimatedVisibility`
- `rememberInfiniteTransition`
- `Animatable`

### 1.1 animate*AsState

```kotlin
val alpha by animateFloatAsState(
    targetValue = if (visible) 1f else 0f,
    animationSpec = tween(durationMillis = 300)
)
```

- 단일 값이 targetValue로 “부드럽게 이동”  
- 간단한 UI 상태 전환에 적합

---

## 2. Transition API & State-driven Animation

### 2.1 updateTransition

```kotlin
enum class BoxState { Collapsed, Expanded }

val transition = updateTransition(targetState = state, label = "BoxTransition")

val height by transition.animateDp(label = "height") { target ->
    if (target == BoxState.Expanded) 200.dp else 80.dp
}
```

- 여러 애니메이션 값을 **한 상태 머신**에서 관리  
- label은 디버깅에 유용

---

## 3. AnimationClock vs FrameClock

### FrameClock
- UI 프레임 단위로 재구성을 관리
- `withFrameNanos { }` 기반

### AnimationClock
- AnimationSpec(스프링, tween 등)에 따라 값 보간
- frameClock에서 tick을 받으며 진행

Animation 흐름:

```
State change →
  Transition sets targetValue →
    AnimationClock calculates intermediate values on each frame →
      Recompose affected animating Composables
```

---

## 4. rememberInfiniteTransition

```kotlin
val infiniteTransition = rememberInfiniteTransition()
val pulse by infiniteTransition.animateFloat(
    initialValue = 0.8f,
    targetValue = 1.2f,
    animationSpec = infiniteRepeatable(
        animation = tween(500),
        repeatMode = RepeatMode.Reverse
    )
)
```

- 종료되지 않는 애니메이션(로딩, 강조 등)에 사용
- frameClock과 연결되어 프레임마다 value 갱신

---

## 5. ConstraintLayout Internals (Compose)

Compose ConstraintLayout은 **ConstraintSet + LayoutNode** 기반으로 동작한다.

```kotlin
ConstraintLayout {
    val (title, button) = createRefs()

    Text(
        "Title",
        Modifier.constrainAs(title) {
            top.linkTo(parent.top)
            start.linkTo(parent.start)
        }
    )
}
```

### 내부 개념:

- ConstraintSet은 제약(Constraints)을 정의하는 DSL
- Measure policy가 Constraint를 해석하고 각 child의 위치를 계산
- SlotTable에는 ConstraintLayout 그룹 + 자식 LayoutNode 정보가 들어간다

ConstraintLayout vs Column/Row:

- ConstraintLayout: 상대적/복잡한 위치 지정에 유리
- Column/Row: 선형, 단순 레이아웃에 유리
