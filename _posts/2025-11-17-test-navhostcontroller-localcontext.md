---
title: "Why pass LocalContext.current to TestNavHostController"
excerpt: "Compose 테스트에서 TestNavHostController에 LocalContext.current(컴포지션에 연결된 액티비티 컨텍스트)를 전달해야 하는 이유를, 실제 동작 원리와 대안/주의사항까지 자세히 정리."
categories: ["Android", "Jetpack Compose", "Testing", "Navigation"]
tags: ["LocalContext", "TestNavHostController", "Compose", "Navigation", "AndroidX Test"]
last_modified_at: 2025-11-17T14:30:00+09:00
---

# TestNavHostController에 `LocalContext.current`를 넘기는 이유 — Why we pass `LocalContext.current` to `TestNavHostController`

Compose 테스트에서 `TestNavHostController(LocalContext.current)`를 쓰는 핵심 이유는 **네비게이션 컨트롤러가 액티비티 레벨 환경(뒤로가기 디스패처, 생명주기, SavedState, 테마/리소스 등)** 에 의존하기 때문이며, `LocalContext.current`가 바로 **현재 컴포지션이 붙어 있는 테스트 액티비티의 컨텍스트**를 보장하기 때문이다 — The key reason is that `NavController` depends on **Activity-level facilities (OnBackPressedDispatcher, lifecycle, saved state, theme/resources)**, and `LocalContext.current` guarantees the **context of the test Activity that hosts the current composition**.

---

## 한 줄 요약 — TL;DR
- 테스트에서 `LocalContext.current`는 **테스트 규칙이 띄운 `ComponentActivity`** 의 컨텍스트다 — In tests it equals the **`ComponentActivity` launched by the compose rule**.  
- 이 컨텍스트는 `NavController`가 필요로 하는 **OnBackPressedDispatcherOwner / LifecycleOwner / SavedStateRegistryOwner** 연결을 자연스럽게 제공한다 — It naturally provides `OnBackPressedDispatcherOwner / LifecycleOwner / SavedStateRegistryOwner`.  
- `Application` 컨텍스트를 쓰면 뒤로가기나 상태 복원이 깨질 수 있다 — Using an application context can break back navigation and saved-state behavior.

---

## 무엇을 요구하나 — What does `TestNavHostController` need?
`TestNavHostController`(및 내부 `NavController`)는 다음과 같은 **액티비티 기반** 기능과 결합되어 동작한다 — It relies on these **Activity-based** features:

1. **뒤로가기 디스패처 연동 — OnBackPressedDispatcher integration**  
   실제 뒤로가기는 액티비티의 `OnBackPressedDispatcher`를 통해 동작한다 — Back navigation is wired through the Activity's dispatcher.

2. **생명주기 & SavedState — Lifecycle & SavedState**  
   목적지 전환에서 SavedState 복원/저장을 하며, 이는 `SavedStateRegistryOwner`/`LifecycleOwner`가 있는 컨텍스트에서 자연스럽게 연결된다 — Destination changes use saved state which is provided via Activity owners.

3. **리소스/테마/밀도 — Resources/Theme/Density**  
   네비게이션 그래프 인플레이트, 문자열/테마/밀도는 현재 화면 컨텍스트를 전제로 한다 — Graph inflate and theme/density assume the current screen context.

따라서 **액티비티 컨텍스트**가 필요하고, 테스트에서는 `LocalContext.current`가 그 요구를 정확히 충족한다 — Therefore we need an Activity context; in tests `LocalContext.current` satisfies this precisely.

---

## 왜 하필 `LocalContext.current`인가 — Why specifically `LocalContext.current`?
- `LocalContext`는 **CompositionLocal**로, **지금 이 컴포지션을 호스팅하는 액티비티**의 컨텍스트를 반환한다 — It's the CompositionLocal that returns the **Activity hosting the current composition**.  
- `createAndroidComposeRule<ComponentActivity>()`는 테스트용 액티비티를 띄우고, `setContent{}` 안에서 `LocalContext.current`를 읽으면 **그 액티비티 컨텍스트**가 온다 — The rule launches a test Activity and inside `setContent{}` you read that exact Activity context.

```kotlin
@get:Rule
val composeRule = createAndroidComposeRule<ComponentActivity>()

private lateinit var navController: TestNavHostController

@Before
fun setUp() {
    composeRule.setContent {
        val ctx = LocalContext.current // 테스트 액티비티 컨텍스트 — test Activity context
        navController = TestNavHostController(ctx).apply {
            navigatorProvider.addNavigator(ComposeNavigator())
        }
        CupcakeApp(navController = navController)
    }
}
```

---

## 동등한 안전 대안 — Safe alternatives
다음도 동일하게 안전하다 — These are equally safe:

```kotlin
// 1) 액티비티 직접 참조 — Direct activity reference
composeRule.setContent {
    val activity = composeRule.activity
    navController = TestNavHostController(activity).apply {
        navigatorProvider.addNavigator(ComposeNavigator())
    }
    CupcakeApp(navController)
}
```

피해야 할 예 — Avoid:

```kotlin
// Application 컨텍스트 — breaks back press / lifecycle wiring
val appCtx = ApplicationProvider.getApplicationContext<Context>()
navController = TestNavHostController(appCtx) // ❌ 권장하지 않음 — Not recommended
```

---

## Compose 룰과의 정합성 — Threading & rule consistency
`setContent {}` 블록은 **메인 스레드**에서 실행되며, 그 안에서 `LocalContext.current`로 얻은 컨텍스트는 **현재 컴포지션과 동일한 생명주기/테마 구성**을 공유한다 — The block runs on the main thread; the context shares the composition’s lifecycle/theme configuration.

---

## 자주 겪는 문제 — Common pitfalls
- **뒤로가기가 먹지 않는다**: `Application` 컨텍스트로 생성했을 가능성 — Back press not working usually means you used an application context.  
- **SavedState가 복원되지 않는다**: 액티비티 오너가 아닌 컨텍스트 사용 — Saved state broken due to non-Activity context.  
- **Preview와 섞어 사용**: 테스트가 아닌 프리뷰에서 `LocalContext`는 에디터 컨텍스트일 수 있다 — In previews the context is editor-provided; don’t conflate with test runtime.

---

## 미니 체크리스트 — Mini checklist
- [x] `setContent { ... }` 내부에서 컨트롤러 초기화 — Initialize inside `setContent {}`.  
- [x] `LocalContext.current` 또는 `composeRule.activity` 사용 — Use `LocalContext.current` or `composeRule.activity`.  
- [x] `ComposeNavigator()`를 `navigatorProvider`에 추가 — Add `ComposeNavigator()` to the provider.  
- [x] `NavHost`와 같은 트리 구성 이후 테스트 수행 — Run assertions after composing the tree.

---

## 작은 Q&A — Small Q&A

**Q. 그냥 전역에서 한 번 만들어 쓰면 안 되나요? — Can I create it globally?**  
A. 액티비티가 준비되기 전일 수 있고 잘못된 컨텍스트를 잡는다 — The Activity may not be ready and you risk binding the wrong context.

**Q. 왜 굳이 컨텍스트가 필요하죠? — Why does it need a context at all?**  
A. 리소스/테마/그래프 인플레이트와 플랫폼 오너들(뒤로가기/생명주기/저장상태)에 접근해야 한다 — For resources/theme/graph inflate and to access platform owners.

---

## 전체 예시 — Full example

```kotlin
@get:Rule
val composeRule = createAndroidComposeRule<ComponentActivity>()

lateinit var navController: TestNavHostController

@Before
fun setup() = composeRule.setContent {
    val context = LocalContext.current
    navController = TestNavHostController(context).apply {
        navigatorProvider.addNavigator(ComposeNavigator())
    }
    CupcakeApp(navController)
}

@Test
fun startDestination_is_Start() {
    assertThat(navController.currentDestination?.route)
        .isEqualTo(CupcakeScreen.Start.name)
}
```

---

## 핵심 정리 — Key takeaways
- `LocalContext.current`는 **테스트 컴포지션을 호스팅하는 액티비티 컨텍스트** — It’s the Activity context hosting the composition.  
- `NavController`는 **액티비티 기반 오브젝트**들과 강하게 결합 — Strongly tied to Activity-owned objects.  
- 올바른 컨텍스트 선택이 **뒤로가기, 생명주기, SavedState, 리소스 해석**을 좌우 — The correct context ensures proper back press, lifecycle, saved state, and resources.
