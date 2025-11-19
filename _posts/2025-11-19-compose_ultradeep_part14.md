---
title: "Compose Platform Views & Interop"
excerpt: "AndroidView, ComposeView, and how legacy View hierarchy interops with Compose and CompositionLocal"
categories: ["Jetpack Compose", "Interop"]
tags: ["AndroidView", "ComposeView", "Interop"]
last_modified_at: "2025-11-19T15:04:00+09:00"
---

# Compose Platform Views & Interop

## 1. AndroidView in Compose

```kotlin
AndroidView(
    factory = { context ->
        TextView(context).apply { text = "Legacy View" }
    },
    update = { view ->
        view.text = "Updated"
    }
)
```

- Compose 트리 안에 기존 View 삽입  
- LocalContext, LocalDensity를 활용해 크기/스타일 동기화 가능

---

## 2. ComposeView in View Hierarchy

XML 또는 코드에서 ComposeView 사용:

```xml
<androidx.compose.ui.platform.ComposeView
    android:id="@+id/compose_view"
    ... />
```

```kotlin
composeView.setContent {
    MyComposeContent()
}
```

ComposeView 내부에서 CompositionLocal이 설정된다.  
CompositionLocal is tied to the content inside ComposeView.

---

## 3. CompositionLocal과 View 계층 상호작용

- LocalView.current는 ComposeView 또는 host View를 반환  
- LocalContext.current는 해당 View의 Context를 반환  
- AndroidView 내부에서는 View 세계와 Compose 세계를 연결하는 bridge 역할

---

## 4. Interop 모범 사례

- ViewModel은 양쪽에서 공동 사용  
- shared data는 Flow/LiveData로 노출  
- Compose는 관찰자 역할, View는 producer or peer

CompositionLocal은 주로 Compose side의 환경전달용으로 사용.
