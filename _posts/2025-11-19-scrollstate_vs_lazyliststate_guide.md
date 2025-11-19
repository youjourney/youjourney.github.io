---
title: "ScrollState vs LazyListState 비교"
excerpt: "ScrollState와 LazyListState의 동작 방식, 성능·가상화 차이, suspend API와 CoroutineScope의 사용"
categories: ["Jetpack Compose", "Android", "UI"]
tags: ["ScrollState", "LazyListState", "CoroutineScope", "Animation", "Performance"]
last_modified_at: "2025-11-19T16:20:00+09:00"
---

# ScrollState vs LazyListState 비교

`ScrollState`와 `LazyListState`는 둘 다 Jetpack Compose에서 스크롤 상태를 다루는 도구지만,  
**어떤 컴포넌트에 쓰느냐, 어떻게 스크롤을 표현하느냐, 성능이 어디까지 나가느냐**에서 완전히 다른 특성을 가진다.  
`ScrollState` and `LazyListState` are both used to handle scroll state in Jetpack Compose, but they differ in **which components they work with, how they represent scroll, and how they scale performance-wise**.

---

## 1. ScrollState란 무엇인가? (What is ScrollState?)

`ScrollState`는 **단일 축(가로 또는 세로)에 대한 스크롤 오프셋(px)** 을 표현하는 상태 객체다.  
`ScrollState` represents the **scroll offset (in pixels) along a single axis (horizontal or vertical)**.

```kotlin
val scrollState = rememberScrollState()

Column(
    modifier = Modifier.verticalScroll(scrollState)
) {
    // 모든 자식을 한 번에 쭉 렌더링
    // Renders all children at once
}
```

- `scrollState.value` → 현재 스크롤 위치(px)  
  `scrollState.value` → current scroll offset in pixels  
- 0이면 맨 위/맨 왼쪽, 값이 커질수록 아래/오른쪽으로 스크롤된 상태  
  0 means top/left; larger values mean more scrolled down/right.  
- 리스트가 **작거나 단순한 UI**일 때 사용하기 좋다.  
  It's suitable for **small or simple scrollable content**.

---

## 2. LazyListState란 무엇인가? (What is LazyListState?)

`LazyListState`는 `LazyColumn`/`LazyRow`에서 쓰이는 스크롤 상태로,  
**어떤 아이템 인덱스가 화면에 보이는지 + 그 인덱스에서의 오프셋**을 표현한다.  
`LazyListState` is used with `LazyColumn`/`LazyRow` and represents **which item index is visible and the offset within that item**.

```kotlin
val listState = rememberLazyListState()

LazyColumn(state = listState) {
    items(1000) { index ->
        Text("Item $index")
    }
}
```

대표적인 프로퍼티:  
Key properties:

- `listState.firstVisibleItemIndex`  
  → 첫 번째로 보이는 아이템의 인덱스  
  → index of the first visible item  
- `listState.firstVisibleItemScrollOffset`  
  → 그 인덱스에서 얼마나 스크롤됐는지(px)  
  → pixel offset for that first visible item  

이 구조 덕분에 `LazyColumn`/`LazyRow`는 **보이는 영역 근처의 아이템만 compose & measure & draw** 하고,  
나머지는 필요할 때 다시 그린다 (Recycler-like 가상화).  
Thanks to this structure, `LazyColumn`/`LazyRow` only compose/measure/draw items near the viewport and re-create items on demand (Recycler-like virtualization).

---

## 3. ScrollState vs LazyListState 한눈에 비교 (Side-by-side Comparison)

| 항목 / Aspect | ScrollState | LazyListState |
|--------------|-------------|---------------|
| 사용 컴포넌트 / Used with | `Column`/`Row` + `verticalScroll`/`horizontalScroll` | `LazyColumn` / `LazyRow` |
| 스크롤 표현 / Scroll representation | 단일 px 오프셋 (single pixel offset) | (firstVisibleIndex + offset) 조합 |
| 가상화 / Virtualization | 없음 (all items composed) | 있음 (only visible+buffer composed) |
| 성능 / Performance | 아이템 수 많으면 비용 커짐 | 많은 아이템에도 적합 |
| 상태 제어 API / Control APIs | `scrollTo`, `scrollBy`, `animateScrollTo`, `animateScrollBy` | `scrollToItem`, `animateScrollToItem` 등 |
| 스크롤 단위 / Scroll unit | 픽셀 기준 (pixel-based) | 아이템 기준 (item-based) |

정리하면:  
In summary:

- 아이템 수가 **적고 단순한 UI** → ScrollState  
  For **small, simple content** → use `ScrollState`.  
- 아이템이 **많고 리스트형 구조** → LazyListState + LazyColumn/LazyRow  
  For **large lists** → use `LazyListState` with Lazy components.

---

## 4. suspend API와 CoroutineScope: 왜 필요한가? (Why do we need suspend APIs and CoroutineScope?)

### 4.1 suspend 함수란? (What is a suspend function?)

`suspend` 함수는 **중간에 잠시 멈췄다가 나중에 다시 이어서 실행될 수 있는 함수**다.  
A `suspend` function is one that **can pause and resume later without blocking a thread**.

예:  

```kotlin
suspend fun doSomething() {
    delay(1000)
    println("Done")
}
```

- 내부에서 `delay`, 네트워크, 애니메이션 등 비동기/시간이 걸리는 작업을 하되  
- 호출한 스레드를 막지 않고 동작한다.  
It can perform asynchronous work such as delays, networking, or animations without blocking the caller thread.

---

### 4.2 suspend 함수는 왜 그냥 호출할 수 없을까? (Why can’t we just call suspend functions?)

`suspend` 함수는 반드시 **CoroutineScope 내에서** 호출돼야 한다.  
A `suspend` function must be called from within a `CoroutineScope`.

```kotlin
val scope = CoroutineScope(Dispatchers.Main)

scope.launch {
    doSomething() // OK
}

// doSomething() // ❌ scope 없이 직접 호출 불가
```

Compose에서 스크롤 API들 중 많은 것들이 `suspend`로 정의되어 있다:  
Many scroll APIs in Compose are defined as `suspend`:

- `scrollState.scrollTo(...)`  
- `scrollState.animateScrollTo(...)`  
- `lazyListState.animateScrollToItem(...)`  
등등.

이 함수들은 내부에서 애니메이션 프레임 처리, 시간 지연 등을 하기 때문에 **코루틴**이라는 실행 컨텍스트가 필요하다.  
These functions run animations and respond to frame updates, so they must run inside a coroutine context.

---

### 4.3 rememberCoroutineScope()의 역할 (Role of rememberCoroutineScope)

`rememberCoroutineScope()`는 **해당 Composable의 수명에 묶인 CoroutineScope**를 만들어준다.  
`rememberCoroutineScope()` creates a `CoroutineScope` that is tied to the lifecycle of the composable.

```kotlin
@Composable
fun ScrollButtons(scrollState: ScrollState) {
    val scope = rememberCoroutineScope()

    Button(onClick = {
        scope.launch {
            scrollState.animateScrollTo(0) // suspend 함수
        }
    }) {
        Text("Scroll to Top")
    }
}
```

- `onClick`은 `suspend`가 아니기 때문에  
  그 안에서 직접 `scrollState.animateScrollTo()`를 호출할 수 없다.  
  Since `onClick` is not a `suspend` function, you cannot directly call `scrollState.animateScrollTo()` inside it.  
- 그래서 `scope.launch { ... }` 블록 안에서 `suspend` 함수를 호출하는 것.  
  Hence we wrap it in `scope.launch { ... }` to call the suspend function.

LaunchedEffect와 비교하면:  
Compared to `LaunchedEffect`:

- `LaunchedEffect(key)` → Composable이 등장할 때 자동으로 코루틴 실행  
  `LaunchedEffect(key)` automatically launches a coroutine when the composable enters or the key changes.  
- `rememberCoroutineScope()` → 클릭/제스처 등 이벤트 발생 시 개발자가 직접 launch  
  `rememberCoroutineScope()` lets you manually launch coroutines in response to events like clicks or gestures.

---

## 5. ScrollState를 사용한 스크롤 제어 예시 (ScrollState with manual control)

```kotlin
@Composable
fun ScrollStateDemo() {
    val scrollState = rememberScrollState()
    val scope = rememberCoroutineScope()

    Column {
        Row(Modifier.horizontalScroll(scrollState)) {
            repeat(1000) { index ->
                Text("[$index] ", Modifier.padding(8.dp))
            }
        }

        Row(verticalAlignment = Alignment.CenterVertically) {
            Text("Scroll: ")
            Button(onClick = {
                scope.launch {
                    scrollState.scrollBy(-200f) // 즉시 왼쪽으로 200px
                }
            }) { Text("<-") }

            Button(onClick = {
                scope.launch {
                    scrollState.animateScrollBy(200f) // 부드럽게 오른쪽으로 200px
                }
            }) { Text("->") }
        }
    }
}
```

- `scrollBy` → **즉시 점프**(no animation)  
  `scrollBy` → jumps instantly (no animation).  
- `animateScrollBy` → **부드러운 스크롤**(애니메이션)  
  `animateScrollBy` → smooth scrolling with animation.

---

## 6. LazyListState를 사용한 아이템 기반 스크롤 제어 (Item-based scrolling with LazyListState)

```kotlin
@Composable
fun LazyListStateDemo() {
    val listState = rememberLazyListState()
    val scope = rememberCoroutineScope()

    Column {
        LazyRow(state = listState) {
            items(1000) { index ->
                Text("[$index] ", Modifier.padding(8.dp))
            }
        }

        Row {
            Button(onClick = {
                scope.launch {
                    listState.animateScrollToItem(0) // 0번 아이템으로 이동
                }
            }) { Text("First") }

            Button(onClick = {
                scope.launch {
                    listState.animateScrollToItem(999) // 마지막 아이템으로 이동
                }
            }) { Text("Last") }
        }
    }
}
```

- `animateScrollToItem(index)` → 특정 위치의 **아이템 기준**으로 스크롤  
  Scrolls to a specific item index rather than pixel offset.  
- 리스트가 길어도 실제로는 **보이는 아이템 몇 개만 그리므로** 메모리 사용이 상대적으로 적다.  
  Even with a long list, only visible items are composed, keeping memory usage lower.

---

## 7. 가상화(virtualization)와 스크롤 전략 비교 (Virtualization and scroll strategy)

`Row/Column + ScrollState`는 **가상화 없음**:  
`Row/Column + ScrollState` **does not virtualize** items:

- 1000개 아이템 → 1000개 모두 compose & layout & draw 한다.  
  1000 items → all 1000 are composed and laid out.  

`LazyColumn/LazyRow + LazyListState`는 **Recycler-like 가상화**:  
`LazyColumn/LazyRow + LazyListState` **virtualizes** items similar to RecyclerView:

- 화면 + 버퍼에 필요한 소량의 아이템만 활성화  
  Only items near the viewport are active.  
- 스크롤하면서 보이지 않는 아이템은 제거하고, 새로 필요한 아이템만 compose  
  Invisible items are disposed, and newly visible items are composed on demand.  

결론:  
Conclusion:

- 정적/짧은 목록: `ScrollState` 기반 UI로 충분  
  Short/static content: `ScrollState` is fine.  
- 동적/긴 리스트: `LazyListState` + Lazy 컴포넌트가 정석  
  Long, dynamic lists: `LazyListState` with Lazy components is the recommended strategy.

---

## 8. ScrollState + CompositionLocal 조합 예시  
(Using CompositionLocal with ScrollState)

전역(혹은 특정 화면 전체)에서 **같은 ScrollState를 공유하고 싶을 때** CompositionLocal이 유용하다.  
CompositionLocal is useful when you want to **share the same ScrollState across multiple composables**.

```kotlin
val LocalMainScrollState = staticCompositionLocalOf<ScrollState> {
    error("ScrollState not provided")
}

@Composable
fun ScrollHost(content: @Composable () -> Unit) {
    val scrollState = rememberScrollState()
    CompositionLocalProvider(LocalMainScrollState provides scrollState) {
        content()
    }
}
```

어디서든:  
Anywhere inside:

```kotlin
@Composable
fun ContentList() {
    val scrollState = LocalMainScrollState.current
    Column(Modifier.verticalScroll(scrollState)) {
        // content...
    }
}

@Composable
fun ScrollToTopButton() {
    val scrollState = LocalMainScrollState.current
    val scope = rememberCoroutineScope()

    Button(onClick = {
        scope.launch { scrollState.animateScrollTo(0) }
    }) {
        Text("맨 위로 / Scroll to top")
    }
}
```

이렇게 하면 화면 여러 곳에서 하나의 스크롤 상태를 공유하며 제어할 수 있다.  
This lets multiple composables share and control the same scroll state.

---

## 9. SlotTable과 ScrollState (How ScrollState lives in the SlotTable)

`rememberScrollState()`는 내부적으로 거의 다음과 같은 패턴으로 동작한다고 보면 된다:  
Internally, `rememberScrollState()` behaves roughly like:

```kotlin
@Composable
fun rememberScrollState(initial: Int = 0): ScrollState {
    return remember { ScrollState(initial) }
}
```

SlotTable 관점에서:

- `remember { ... }`에 의해 **ScrollState 인스턴스가 해당 Composable 위치에 저장**된다.  
  The `ScrollState` instance is stored in the SlotTable at the composable’s position.  
- recomposition 시에는 새로 만들지 않고 기존 인스턴스를 재사용한다.  
  On recomposition, the existing instance is reused instead of being recreated.  
- Composable가 트리에서 제거되면, 그 그룹과 함께 해당 ScrollState도 GC 대상이 된다.  
  When the composable leaves the tree, the group (and its ScrollState) becomes eligible for GC.

즉, ScrollState는 **“SlotTable에 remember된 상태 중 하나”**라고 이해하면 된다.  
In other words, `ScrollState` is simply one of many “remembered states” stored in the SlotTable.

---

## 10. 마무리 (Wrapping up)

- **ScrollState**  
  - 픽셀 기반 스크롤 오프셋  
  - 모든 아이템을 한 번에 렌더링  
  - 간단한 스크롤 UI에 적합  

- **LazyListState**  
  - 아이템 인덱스 기반 스크롤  
  - Recycler-like 가상화 지원  
  - 긴/동적 리스트에 최적  

- **suspend API + rememberCoroutineScope**  
  - `scrollTo`, `animateScrollTo`, `animateScrollToItem` 등은 모두 `suspend`  
  - `rememberCoroutineScope()`로 UI 이벤트에서 안전하게 호출  

- **CompositionLocal + ScrollState**  
  - 스크롤 상태를 화면 단위 전역 환경 값처럼 공유 가능  
  - 다만 비즈니스 데이터보다는 “환경 값” 수준에 사용하는 걸 권장  

이 내용을 바탕으로, 프로젝트에서 **어디에 ScrollState를 쓰고, 언제 LazyListState로 넘어가야 할지** 기준을 세워두면  
성능과 구조 양쪽에서 훨씬 안정적인 Compose 코드를 만들 수 있다.  
With this understanding, you can choose when to use `ScrollState` vs `LazyListState` and design more robust, performant Compose UIs.

