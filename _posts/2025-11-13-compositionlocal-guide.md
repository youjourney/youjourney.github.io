---
title: "CompositionLocal in Jetpack Compose"
excerpt: "CompositionLocal이 무엇인지, 언제/어떻게 써야 하는지, compositionLocalOf vs staticCompositionLocalOf 차이와 성능·패턴 정리."
categories: ["Android", "Jetpack Compose", "Kotlin"]
tags: ["CompositionLocal", "Compose", "Kotlin", "Android", "State", "Theming"]
last_modified_at: 2025-11-13T20:36:00+09:00
---

# CompositionLocal 가이드 (Jetpack Compose)

**요약 — Summary**  
CompositionLocal은 컴포저블 트리 아래로 값을 “암묵적으로” 전파하는 메커니즘이다 
— CompositionLocal is a mechanism to implicitly propagate values down a composable tree so descendants can read them without explicit parameters. 주로 테마/레이아웃/컨텍스트 같은 **환경 값(ambient)** 을 전달할 때 사용한다 
— It’s ideal for environment-like values such as theme, layout metrics, or Android `Context`.

---

## 1) 핵심 개념 — Core Idea

- **정의**: 부모가 `CompositionLocalProvider(LocalX provides value)`로 값을 제공하면, 하위 어디서든 `LocalX.current`로 읽을 수 있다 — Parent provides a value via `CompositionLocalProvider(LocalX provides value)`, and any descendant reads it with `LocalX.current`.  
- **효과**: 파라미터 드릴링 없이 공통 “환경”을 공유할 수 있다 — Eliminates parameter drilling for shared “environment” data.  
- **용도**: 테마, 색상, 간격(Spacing), 로케일, 밀도, 레이아웃 방향, `Context` 등 — Themes, colors, spacing tokens, locale, density, layout direction, `Context`, etc.

---

## 2) 기본 사용 — Basic Usage

### 읽기 (Consume)
```kotlin
@Composable
fun NeedsContext() {
    val context = LocalContext.current   // Read from CompositionLocal
    // use context...
}
```

### 제공 (Provide)
```kotlin
val LocalSpacing = compositionLocalOf<Spacing> {
    error("No Spacing provided") // Fallback if not provided
}

@Composable
fun App() {
    CompositionLocalProvider(
        LocalSpacing provides Spacing(s = 8.dp, m = 16.dp, l = 24.dp)
    ) {
        Content() // descendants can read LocalSpacing.current
    }
}
```

### 하위 사용 (Use Downstream)
```kotlin
@Composable
fun Content() {
    val spacing = LocalSpacing.current
    Box(Modifier.padding(spacing.m)) { /* ... */ }
}
```

**포인트 — Key Point**: `LocalX.current`를 **실제로 읽는** 컴포저블만 값 변경 시 리컴포지션된다(동적 로컬의 경우) — Only composables that *read* `LocalX.current` will recompose when the value changes (for dynamic locals).

---

## 3) 언제 쓰나 — When to Use

- **환경 값**을 전역/섹션 전역으로 공유할 때 — For environment-like values shared across sections.  
- **디자인 토큰**(색/타이포/간격/라운드) — Design tokens (color/typography/spacing/corner radii).  
- **레이아웃 맥락**(밀도/방향/로케일) — Layout context (density/direction/locale).  
- **얕은 의존성**(logger/tracker) — Shallow dependencies like a logger.

> 비즈니스 상태나 무거운 의존성 주입은 ViewModel/DI가 적합 — Use ViewModel/DI for business state or heavy dependencies.

---

## 4) `compositionLocalOf` vs `staticCompositionLocalOf`

두 팩토리는 **리컴포지션 추적 범위**가 다르다 — They differ in recomposition tracking.

| 항목 — Aspect | `compositionLocalOf` (Dynamic) | `staticCompositionLocalOf` (Static) |
|---|---|---|
| 읽기 추적 — Read tracking | O (읽은 곳만 재구성) — Yes (only readers recompose) | X (하위 전체 재구성) — No (whole subtree recomposes) |
| 변경 빈도 — Change frequency | 자주 바뀌는 값 — Often-changing values | 거의 안 바뀌는 값 — Rarely-changing values |
| 권장 사례 — Good fit | 런타임 토글/플래그 — runtime toggles/flags | 고정 디자인 토큰 — fixed design tokens |

**요령 — Tip**: 값이 **자주 바뀌면** `compositionLocalOf`, **거의 고정이면** `staticCompositionLocalOf`.

---

## 5) 실전 예제 — Practical Examples

### (A) 스페이싱 토큰 — Spacing Tokens (거의 고정 → static)
```kotlin
@Stable
data class Spacing(val s: Dp, val m: Dp, val l: Dp)

val LocalSpacing = staticCompositionLocalOf<Spacing> {
    error("No Spacing provided")
}

@Composable
fun AppTheme(content: @Composable () -> Unit) {
    val spacing = remember { Spacing(8.dp, 16.dp, 24.dp) } // memoize
    CompositionLocalProvider(LocalSpacing provides spacing) {
        content()
    }
}

@Composable
fun CardWithSpacing() {
    val spacing = LocalSpacing.current
    Card(Modifier.padding(spacing.m)) { /* … */ }
}
```

### (B) 기능 플래그 — Feature Flags (자주 변경 → dynamic)
```kotlin
data class FeatureFlags(val newDesign: Boolean)

val LocalFlags = compositionLocalOf { FeatureFlags(newDesign = false) }

@Composable
fun FeatureFlagProvider(newDesign: Boolean, content: @Composable () -> Unit) {
    CompositionLocalProvider(LocalFlags provides FeatureFlags(newDesign)) {
        content()
    }
}

@Composable
fun Home() {
    val flags = LocalFlags.current
    if (flags.newDesign) NewHome() else OldHome()
}
```
플래그가 바뀌면 **읽은 위치만** 리컴포즈 — Only readers recompose when the flag changes.

---

## 6) LocalContext는 왜 “Local”인가 — Why `LocalContext` is Local

- `LocalContext`는 Android `Context`를 컴포즈 트리로 전달하는 CompositionLocal이다 — `LocalContext` is a CompositionLocal for Android `Context`.  
- **컴포저블 본문 안에서만 읽기 가능** — Can only be read inside composable bodies.  
- ViewModel/Repository에서 `Context`가 필요하면 **DI/파라미터 주입**을 고려 — Inject or pass it via DI/parameters in non-composable layers.

---

## 7) 리컴포지션 & 성능 — Recomposition & Performance

- **읽기 추적**(dynamic local): `LocalX.current`를 **읽은** 컴포저블만 값 변경 시 다시 그린다 — Only readers recompose when the value changes.  
- **Static local**은 읽기 추적이 없어 **Provider 하위 전체**를 다시 그린다 — Static locals recompose the entire provided subtree.  
- **Stable/불변 데이터**를 제공하라: `@Stable` data class, 불변 필드 — Prefer `@Stable`/immutable payloads.  
- **스코프 최소화**: 필요한 섹션에만 `CompositionLocalProvider` — Keep provider scopes tight.  
- **콜백은 최신 참조로**: `rememberUpdatedState(callback)`로 대규모 리컴포즈를 방지 — Use `rememberUpdatedState` for callbacks to reduce recomposition churn.  
- **생성 비용 큰 객체는 memoize**: `remember { … }`로 캐싱 후 제공 — Memoize heavy objects before providing.

---

## 8) ViewModel / DI / remember 와의 차이 — How It Differs

- **ViewModel**: 화면/프로세스 생명주기 기반의 상태·로직 보관 — Holds UI state/logic across lifecycle.  
- **DI(Hilt/Koin)**: 객체 그래프 구성·주입 — Builds/injects object graphs.  
- **remember**: **현재 컴포지션 위치**에 메모이즈 — Memoizes at the current composition node.  
- **CompositionLocal**: **트리 하위로 값 전달 경로** — Creates an environment channel down the tree.

> 결론: CompositionLocal은 **환경 값** 전달에, ViewModel/DI는 **상태·의존성 관리**에, `remember`는 **메모이즈**에 각각 특화 — Use the right tool for the right concern.

---

## 9) 미니 다이어그램 — Mini Diagram

```
CompositionLocalProvider(LocalX provides value)
│
├── A()              // doesn't read LocalX → unaffected by updates
└── B() ── reads LocalX.current → recomposes when value changes
``

- 동적 로컬: `B()`만 재구성 — Dynamic: only `B()`  
- 정적 로컬: 하위 전체 재구성 — Static: whole subtree

---

## 10) 트러블슈팅 — Troubleshooting

- **"No X provided" 에러**: 해당 Local을 제공하지 않았다 → 부모에서 `CompositionLocalProvider`로 제공 — Provide it at an ancestor.  
- **리컴포즈 폭발**: 자주 바뀌는 값을 `staticCompositionLocalOf`로 제공 — Using static local for frequently-changing values causes subtree-wide recomposition.  
- **미리보기(Preview) NPE**: Preview에서 Provider가 빠진 경우 — Wrap preview content with a provider or give a safe default.  
- **ViewModel에서 Local 사용 시도**: 불가 — CompositionLocal은 컴포저블 컨텍스트 전용, ViewModel에는 DI/파라미터 사용.

---

## 11) 치트시트 — Cheat Sheet

```kotlin
// Define (dynamic vs static)
val LocalX = compositionLocalOf<MyType> { error("No value") }
// val LocalX = staticCompositionLocalOf<MyType> { error("No value") }

// Provide
CompositionLocalProvider(LocalX provides myValue) {
    Child()
}

// Consume
val x = LocalX.current

// Stable payload example
@Stable
data class MyTokens(val radius: Dp, val padding: Dp)
```

---

## 12) 결론 — Takeaways

- CompositionLocal은 **환경 값 전파**에 최적 — Best for environment propagation.  
- 값의 **변경 빈도**에 따라 `compositionLocalOf`/`staticCompositionLocalOf`를 선택 — Choose dynamic vs static based on change frequency.  
- **Stable/불변** 데이터, **스코프 최소화**, **콜백 최신화**가 성능 핵심 — Favor stable/immutable payloads, tight scopes, and updated callbacks.  
- 비즈니스 상태/의존성은 **ViewModel/DI**로 — Use ViewModel/DI for business state and dependencies.
