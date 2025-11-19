---
title: "Compose CompositionLocal, Compiler IR & Effect APIs"
excerpt: "IR lowering of CompositionLocal and comparisons with Effect APIs"
categories: ["Jetpack Compose", "Compiler", "Effects"]
tags: ["IR", "rememberUpdatedState", "DisposableEffect", "LaunchedEffect"]
last_modified_at: ""2025-11-19T14:51:00+09:00""
---

# Compose Compiler IR + Effect API Comparative Guide

## 1. Compiler IR: CompositionLocal Lowering
### LocalA.current
```
LocalA.current
↓ IR lowering
composer.consume(LocalA)
```

### Provider lowering
```
CompositionLocalProvider(LocalA provides A1) {
    ...
}
↓
composer.startReplaceableGroup(key)
composer.updateProvider(LocalA, A1)
emit children
composer.endReplaceableGroup()
```

## 2. Effect APIs 심층 비교

### rememberUpdatedState
ongoing coroutine 또는 callback이 최신 값을 참조하도록 유지  
Keeps latest value available to ongoing effects.

### DisposableEffect
CompositionLocal이 바뀌거나 제거될 때 unregister 필요할 때  
Used for lifecycle-bound registrations.

### LaunchedEffect
CompositionLocal 변경 시 coroutine을 재실행해야 할 때  
Runs coroutine when key (often Local value) changes.

## 3. 타임라인 비교
```
remember:
  persists across recomposition

rememberUpdatedState:
  updates reference only, keeps coroutine alive

DisposableEffect:
  cleanup on key change

LaunchedEffect:
  restart effect on key change
```
