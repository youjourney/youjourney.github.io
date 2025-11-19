---
title: "Compose Historical Ambients, Navigation Interaction & Advanced Architecture"
excerpt: "History of Ambient → CompositionLocal, Navigation BackStackEntry & saveable state interaction, and high-end architecture"
categories: ["Jetpack Compose", "Architecture", "History"]
tags: ["Ambient", "CompositionLocal", "Navigation", "Architecture"]
last_modified_at: "2025-11-19T14:59:00+09:00"
---

# Compose Ambient History, Navigation Saveable Interaction & Enterprise Architecture

## 1. Ambient → CompositionLocal 역사적 전환

### Ambients 등장
초기 Compose에는 `Ambient`라는 API가 있었다.

문제점:
- 이름이 ambiguous  
- Lifecycle ambiguous  
- Provider override 구조가 복잡함  

### CompositionLocal로 진화
이후 API가 명확한 CompositionLocal로 대체됨.

새 API의 개선점:
- 더 직관적인 이름 (`LocalContext`, `LocalView`, etc.)
- provider system 단순화
- SlotTable integration 강화

---

## 2. Navigation BackStackEntry + rememberSaveable Interaction

Navigation BackStackEntry는 화면당 saveable state scope를 제공한다.

### Key principle
```
rememberSaveable is scoped to NavBackStackEntry
```

즉:
- 동일 화면으로 돌아오면 이전 rememberSaveable 값 복원  
- CompositionLocal override는 BackStackEntry 단위로 유지됨  
- Local configuration + saveable UI state 조합이 가능

예시:

```kotlin
composable("edit") { entry ->
    val state = rememberSaveable(entry) { mutableStateOf("") }
    val locale = LocalLocale.current
}
```

---

## 3. High-level Enterprise Compose Architecture

### 권장 계층 구조
```
app/
  theme/                  ← global theme builder
  navigation/
core/
  design-system/          ← CompositionLocal tokens
  data/
  domain/
features/
  featureA/
  featureB/
```

### Core principles
- CompositionLocal은 “환경 값” 전달용  
- 비즈니스 로직 전달 금지  
- rememberSaveable은 화면 단위 state 저장  
- ViewModel은 UI state 생산자  
- Navigation BackStack 단위로 CompositionLocal 복원

---

## 4. Architecture Diagram
```
AppTheme
 ├─ CompositionLocalProviders
 │      ├─ Locale
 │      ├─ Spacing
 │      ├─ Elevation
 │      └─ Density override
 │
 └─ NavHost
        ├─ ScreenA (BackStackEntry A)
        │        ├─ rememberSaveable scope A
        │        └─ Locale overridden? yes/no
        └─ ScreenB (BackStackEntry B)
```

CompositionLocal + Navigation + SaveableStateHolder는 Compose 아키텍처의 핵심 축이다.
