---
title: "Compose Multi-Module ThemeBuilder + Architecture"
excerpt: "Multi-module design system strategy with ThemeBuilder and large-scale architecture"
categories: ["Jetpack Compose", "Architecture", "Design System"]
tags: ["ThemeBuilder", "Multi-module", "Architecture"]
last_modified_at: "2025-11-19T14:56:00+09:00"
---

# Compose ThemeBuilder + Multi-module Architecture

## 1. ThemeBuilder의 Multi-Module 확장 전략

대규모 앱에서는 ThemeBuilder를 계층적으로 설계해야 한다.

### 구조 예시

```
core/
  theme/
    Color.kt
    Typography.kt
    Spacing.kt
    Elevation.kt
    ThemeBuilder.kt

feature-home/
  ui/
  HomeThemeOverlay.kt

feature-settings/
  ui/
  SettingsThemeOverlay.kt
```

### ThemeBuilder 기본 계층

1) core-theme  
2) feature-overlay themes  
3) runtime override via CompositionLocalProvider  

---

## 2. Feature Module Overlay Example

```kotlin
@Composable
fun HomeTheme(content: @Composable () -> Unit) {
    CompositionLocalProvider(
        LocalSpacing provides Spacing(medium = 20.dp)
    ) {
        content()
    }
}
```

Home 화면 전용 spacing을 적용.

---

## 3. Large-scale Compose Architecture Guide

### 3-1. Recommended Layers

```
app/
  ui/
    navigation/
    theme/
core/
  domain/
  data/
  design-system/
features/
  featureA/
  featureB/
```

### 3-2. Compose 상태 관리 원칙

- UI state는 ViewModel에서 Flow/StateFlow로 관리  
- CompositionLocal은 환경 값 전달 전용  
- Navigation과 CompositionLocal은 BackStackEntry 스코프 기반으로 관리  
- ThemeBuilder는 최상단(AppTheme)에서 통합 제공  

---

## 4. Snapshot System Locking & Observer Dispatch

### Snapshot Locking 원리

```
snapshot.enterMutableState()
  assign write observers
  modify state
snapshot.record
snapshot.leaveMutableState()
```

### Observer Dispatch 흐름

```
state write →
  snapshot notifies observer →
    Recomposer schedules frame →
      recomposition → applyChanges
```

Snapshot은 thread-safe하며 lock-free 구현인 경우도 있음 (atomic operations 기반).
