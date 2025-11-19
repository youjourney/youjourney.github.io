---
title: "Jetpack Compose CompositionLocal + Diagram + State 비교 예시"
excerpt: "CompositionLocal의 구조·동작·코드 예시·상태 모델 비교·프로젝트 구조화"
categories: ["Jetpack Compose", "Android", "UI Architecture"]
tags: ["CompositionLocal", "Compose", "State", "Design System", "Android"]
last_modified_at: "2025-11-19T14:45:00+09:00"
---

# Jetpack Compose CompositionLocal

Jetpack Compose의 `CompositionLocal`은 UI 트리 아래로 환경값을 전달하는 강력한 메커니즘이다.  
`CompositionLocal` in Jetpack Compose is a powerful mechanism for providing environment values down the UI tree.

---

# 📌 0. CompositionLocal 개념 요약 (Concept Summary)

CompositionLocal은 **상위 → 하위**로 값을 전달하는 Ambient Context 시스템이다.  
CompositionLocal is an ambient context system for delivering values from parent → child.

주로 다음 용도로 사용한다:  
Used commonly to deliver:

- Context / View 같은 안드로이드 환경 요소  
- Theme / Typography / ContentColor 같은 디자인 시스템 값  
- 포커스 / 키보드 / Lifecycle 같은 UI 시스템 값  
- App configuration, Feature Flags 같은 환경 정보  

---

# 📌 1. CompositionLocal 구조 다이어그램 (Diagram)

아래는 CompositionLocal의 값 전달 구조를 단순화한 한 컷 모델이다.  
Below is a simplified one-cut mental model of how CompositionLocal works.

```
Root
 └─ CompositionLocalProvider(LocalA = A1, LocalB = B1)
       ├─ ScreenA
       │     └─ ComponentA1 (LocalA = A1)
       └─ ScreenB
             └─ CompositionLocalProvider(LocalA = A2)
                   └─ ComponentB1 (LocalA = A2, LocalB = B1)
```

핵심 개념:  
Key ideas:

- CompositionLocal은 값이 “가장 가까운 provider”에서 결정된다.  
  Value resolves from the *nearest provider* in the tree.
- 파라미터 전파가 불필요하다.  
  No need to pass through parameters.
- provider를 중첩하면 override된다.  
  Nested providers override parent providers.

---

# 📌 2. 기본 사용 패턴 (Basic Usage Pattern)

```kotlin
val LocalGreeting = compositionLocalOf { "Hello" }

@Composable
fun GreetingHost() {
    CompositionLocalProvider(LocalGreeting provides "안녕") {
        Child()
    }
}

@Composable
fun Child() {
    Text(LocalGreeting.current)
}
```

---

# 📌 3. 주요 CompositionLocal 정리 (Major CompositionLocals)

## ✔ LocalContext
Android Context 제공  
Provides Android Context

사용 예시(Toast, Intent, resources):  
```kotlin
val context = LocalContext.current
Toast.makeText(context, "Hello", Toast.LENGTH_SHORT).show()
```

## ✔ LocalView
Compose가 그려지는 실제 Android View  
The actual Android View hosting Compose

사용 예시(WindowInsetsController 등):  
```kotlin
val view = LocalView.current
if (!view.isInEditMode) {
    SideEffect { setupEdgeToEdge(view) }
}
```

## ✔ LocalDensity
dp/sp ↔ px 변환을 위한 Density 제공  
Provides Density for converting dp/sp ↔ px

```kotlin
val px = with(LocalDensity.current) { 16.dp.toPx() }
```

## ✔ LocalConfiguration
화면 사이즈, 로케일, 회전 정보 제공  
Provides configuration like screen size, locale, orientation

```kotlin
if (LocalConfiguration.current.orientation == ORIENTATION_LANDSCAPE) { ... }
```

## ✔ LocalLifecycleOwner
LifecycleOwner 접근 제공  
Access to LifecycleOwner

## ✔ LocalFocusManager / LocalSoftwareKeyboardController
포커스 이동과 키보드 제어 제공  
Focus movement and keyboard control

---

# 📌 4. CompositionLocal vs remember / rememberSaveable / derivedStateOf 비교

Compose 상태 모델과 CompositionLocal의 역할을 명확히 구분할 필요가 있다.  
It’s important to clearly differentiate CompositionLocal from state models.

## ✔ 비교 테이블 (Comparison Table)

| 기능 | CompositionLocal | remember | rememberSaveable | derivedStateOf |
|------|------------------|----------|------------------|----------------|
| 목적 | 환경값 전달 | 단일 구성에서 값 기억 | 프로세스 복원 가능한 기억 | 기존 상태에서 파생된 메모이즈 상태 |
| 데이터 성격 | Context-like / Theme-like | UI State | UI State + SavedState | Derived UI value |
| 트리 전파 | Yes | No | No | No |
| recomposition | current 읽을 때 영향 | 값 변경 시 UI 갱신 | 저장/복원 목적 | dependency 값 변경 시 갱신 |
| 예 | LocalView, LocalContext | TextField state | Form inputs | filtered list |

## ✔ 정적 예시 (Static Example)

```kotlin
val LocalUser = compositionLocalOf<User?> { null }

@Composable
fun Screen() {
    CompositionLocalProvider(LocalUser provides User("Admin")) {
        UserPanel()
    }
}

@Composable
fun UserPanel() {
    val user = LocalUser.current      // global-like context
    var count by remember { mutableStateOf(0) }     // per-composable state
}
```

---

# 📌 5. 실전 프로젝트 구조 내 CompositionLocal 설계 예시  
(Real Project Architecture Examples)

---

## ✔ 예시 1 — AppConfig (Feature Flags + App-level Settings)

### 파일 구조  
```
ui/
  theme/
    Color.kt
    Type.kt
    Shape.kt
    Theme.kt
    AppConfig.kt   ← 추가
```

### AppConfig.kt

```kotlin
data class AppConfig(
    val useNewChatUI: Boolean,
    val apiEndpoint: String,
)

val LocalAppConfig = staticCompositionLocalOf<AppConfig> {
    error("AppConfig not provided")
}
```

### 적용

```kotlin
@Composable
fun MyApp(appConfig: AppConfig) {
    CompositionLocalProvider(LocalAppConfig provides appConfig) {
        AppRoot()
    }
}

@Composable
fun SomeScreen() {
    val config = LocalAppConfig.current
    if (config.useNewChatUI) NewChat() else OldChat()
}
```

---

## ✔ 예시 2 — Design System Tokens

Typography, Color, Shape 외에도  
`Spacing`, `Elevation`, `MotionSpec` 같은 항목도 CompositionLocal 로 제공할 수 있다.

### Spacing.kt

```kotlin
data class Spacing(
    val small: Dp = 4.dp,
    val medium: Dp = 12.dp,
    val large: Dp = 24.dp
)

val LocalSpacing = staticCompositionLocalOf { Spacing() }
```

### 사용

```kotlin
Column(Modifier.padding(LocalSpacing.current.medium)) { ... }
```

---

## ✔ 예시 3 — Locale-aware CompositionLocal (로케일 반영 전략)

Locale, LayoutDirection, Formatting Rules 등을 일관되게 전달할 수 있다.

```kotlin
data class LocaleConfig(
    val locale: Locale,
    val dateFormatter: DateFormat,
)

val LocalLocaleConfig = compositionLocalOf<LocaleConfig> {
    error("LocaleConfig not provided")
}
```

---

# 📌 6. CompositionLocal 남용에 대한 주의 사항  
(Cautions about Overuse)

- 전역 상태처럼 변하면 유지보수가 어려워짐  
  Overusing turns it into global state
- 비즈니스 로직 전달에 사용하면 안 됨  
  Should not carry business logic
- “환경 값 전달용”에만 사용해야 안전  
  Safe only for environment-like values

---

# 📌 7. 최종 요약 (Final Summary)

- CompositionLocal은 **환경값 전달 전용** 메커니즘이다.  
  It is a mechanism specifically for delivering environment values.
- 상태 보관은 remember / rememberSaveable 를 사용해야 한다.  
  Use remember / rememberSaveable for state.
- derivedStateOf는 상태에서 파생 값 계산에 사용한다.  
  derivedStateOf is for derived values.
- 실전 프로젝트에서는 **Theme 시스템 + Custom Local** 조합으로  
  강력한 UI/UX 디자인 시스템을 구축할 수 있다.  
  Real projects combine custom locals with the theme system to build strong design systems.

---