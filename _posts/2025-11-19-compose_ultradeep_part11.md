---
title: "Compose Coroutines, Flow, and State in Compose"
excerpt: "LaunchedEffect, snapshotFlow, Flow collection patterns, and concurrency considerations in Compose"
categories: ["Jetpack Compose", "Coroutines", "State"]
tags: ["Flow", "LaunchedEffect", "snapshotFlow", "Concurrency"]
last_modified_at: "2025-11-19T15:01:00+09:00"
---

# Compose + Flow/Coroutines Deep Dive

## 1. LaunchedEffect와 Coroutine Scope

`LaunchedEffect`는 Composable scope에 묶인 coroutine을 실행한다.  
`LaunchedEffect` launches a coroutine tied to the composable’s lifecycle.

```kotlin
LaunchedEffect(key1 = userId) {
    viewModel.loadUser(userId)
}
```

- key가 바뀌면 기존 coroutine cancel + 새 coroutine 시작  
- key가 `Unit`이면 최초 한 번만 실행

---

## 2. rememberCoroutineScope

```kotlin
val scope = rememberCoroutineScope()

Button(onClick = {
    scope.launch {
        viewModel.doAction()
    }
}) { Text("Action") }
```

- Composition lifecycle에 묶인 scope
- UI 이벤트 + suspend 함수 연결에 필수

---

## 3. snapshotFlow

Compose state를 Flow로 변환하는 API.  
Transforms Compose state reads into a Flow.

```kotlin
val query by remember { mutableStateOf("") }

LaunchedEffect(Unit) {
    snapshotFlow { query }
        .collect { q -> println("Query: $q") }
}
```

- `snapshotFlow { }` block 내에서 읽은 상태가 변경될 때마다 emit  
- state → Flow 브리지 역할

---

## 4. Flow 수집 패턴 (collectAsState, collectAsStateWithLifecycle)

```kotlin
val uiState by viewModel.uiState.collectAsState()
```

- StateFlow/Flow를 Compose state로 노출  
- recomposition-friendly

Lifecycle-aware 버전은 `collectAsStateWithLifecycle()` 사용.

---

## 5. 동시성 고려사항 (Concurrency Considerations)

- 같은 state를 여러 coroutine이 갱신할 때 race condition 주의  
- Snapshot system은 thread-safe지만 “로직 수준” 경쟁은 직접 처리해야 함  
- viewModelScope + immutable UI state 패턴 권장

---

## 6. Flow + CompositionLocal 조합

```kotlin
val themeConfig = LocalThemeConfig.current

LaunchedEffect(themeConfig) {
    themeConfig.events.collect { event ->
        // theme-related side effect
    }
}
```

CompositionLocal 값이 변하면 새로운 Flow 수집이 시작될 수 있음.
