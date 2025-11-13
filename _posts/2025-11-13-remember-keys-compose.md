---
title: "remember(keys) in Jetpack Compose: Basics & Patterns"
excerpt: "Control recomposition and caching with remember, keys, LocalConfiguration, and LocalContext."
categories: ["Android", "Jetpack Compose", "Kotlin"]
tags: ["remember", "state", "keys", "LocalContext", "LocalConfiguration", "rememberSaveable", "derivedStateOf"]
last_modified_at: 2025-11-13T19:30:00+0900
---

# `remember(keys)` in Jetpack Compose — Basics & Patterns  
**`remember(keys)`로 재구성과 캐싱을 세밀하게 제어하기** (Use `remember(keys)` to precisely control recomposition and caching.)

> 요약: `remember { … }`는 처음 한 번 계산한 값을 컴포지션에 캐시하고, `remember(key1, key2) { … }`는 **키가 바뀔 때만** 캐시를 무효화해 재계산합니다. (Summary: `remember { … }` caches once per composition slot; `remember(key1, key2) { … }` invalidates and recomputes **only when any key changes**.)

---

## 1) API & Mental Model  
**시그니처와 개념 모델** (Signature and mental model)

```kotlin
@Composable
inline fun <T> remember(vararg keys: Any?, crossinline calculation: () -> T): T
```
- `calculation`은 **처음 한 번만** 실행되어 값이 캐시됩니다. (The `calculation` runs **once** initially and is cached.)
- 이후 재구성에서 **키가 모두 동일하면** 캐시된 값을 재사용합니다. (On subsequent recompositions, if all keys are equal, the cached value is reused.)
- **한 개라도 키가 달라지면** 캐시 무효화 후 `calculation`을 다시 실행합니다. (If **any key differs**, the cache is invalidated and `calculation` is recomputed.)

### 작은 다이어그램 (Tiny diagram)
```
First composition:
  compute -> [cache value]

Recomposition w/ same keys:
  reuse   -> [cache value]

Recomposition w/ changed key:
  invalidate -> compute -> [cache new value]
```
(동일 키면 재사용, 키 변경 시 무효화/재계산. — Reuse for same keys; invalidate/recompute when keys change.)

---

## 2) 왜 `LocalConfiguration`/`LocalContext`를 키로?  
**구성/컨텍스트 의존 계산의 정확한 갱신** (Accurate refresh of configuration/context-dependent computations)

- `LocalConfiguration.current`: 로캘, 다크모드, 화면 크기, 폰트 스케일 등 **시스템 구성**이 바뀌면 값이 바뀝니다. (Changes when system configuration like locale, dark mode, screen size, or font scale changes.)
- `LocalContext.current`: 보통 `Activity` 컨텍스트이며, 구성 변경으로 액티비티가 재생성되면 **새 컨텍스트**가 됩니다. (Usually the `Activity` context; after configuration changes the activity is recreated, yielding a **new context**.)

> 문자열/치수 등 **환경 의존적 계산**을 캐시할 땐 `remember(configuration, context)`로 **필요한 때만** 재계산하세요. (For environment-dependent computations like strings/dimensions, use `remember(configuration, context)` so you recompute **only when needed**.)

```kotlin
@Composable
fun FlavorStep(optionIds: List<Int>) {
    val configuration = LocalConfiguration.current
    val context = LocalContext.current

    val options = remember(configuration, context) {
        optionIds.map { id -> context.getString(id) }
    }
    // 구성/컨텍스트가 동일한 한 options는 재사용됩니다.
    // (As long as configuration/context are equal, options is reused.)
}
```

> 참고: 로캘만 의존한다면 `configuration`만으로 충분할 때도 있지만, 컨텍스트 교체를 안전하게 반영하려면 둘 다 키로 주는 것이 일반적으로 안전합니다. (If only locale matters, `configuration` may suffice; including both keys is a safe default to reflect context swaps.)

---

## 3) 더 “추천”되는 구조 (아이템에서 변환)  
**부모는 리소스 ID만 전달, 자식에서 `stringResource()` 호출** (Pass only IDs from parent; call `stringResource()` in the child)

```kotlin
@Composable
fun SelectOptionScreen(optionIds: List<Int>) {
    LazyColumn {
        items(optionIds, key = { it }) { id ->
            // 이 아이템만 재구성될 때 문자열을 변환 (부분 재구성)
            // (Only this item recomposes when needed → finer-grained updates)
            Text(text = stringResource(id))
        }
    }
}
```
- 리스트 전체를 한 번에 문자열로 변환하지 않아 **불필요한 재계산/할당**을 줄입니다. (Avoid bulk string conversion; fewer allocations and recalculations.)

> 주의: `remember { … }` 블록 안에서는 `@Composable`(예: `stringResource`)를 호출할 수 없습니다. **필요하면 `context.getString(id)`**를 사용하세요. (You cannot call `@Composable` functions like `stringResource` inside `remember { … }`. Use `context.getString(id)` instead when needed.)

---

## 4) `rememberSaveable` / `derivedStateOf`와의 비교  
**상태 보존/파생 상태 최적화** (State persistence / derived state optimization)

- `remember`  
  : 컴포지션 생명주기 동안만 유지. 구성 변경 시 **키가 없으면** 그대로, **키가 있으면** 재계산. (Lives for the composition; recomputes on key change.)
- `rememberSaveable`  
  : `Bundle`에 직렬화해 **프로세스 종료/회전 후에도 보존**. (Serializes to a `Bundle` to **survive process death/rotation**.)
- `derivedStateOf { … }`  
  : 다른 `State`에서 **파생 값**을 메모이즈; 의존 `State`가 바뀔 때만 재계산. (Memoizes a value **derived from other `State`s**; recomputes only when dependencies change.)

```kotlin
val query by rememberSaveable { mutableStateOf("") } // 회전 후에도 유지 (survives rotation)
val all by remember { mutableStateOf(fullList) }

// 키 기반 캐시 (recompute only when query/all change)
val filtered1 = remember(query, all) { all.filter { it.matches(query) } }

// 스냅샷 의존 기반 파생 (recompute when either state changes)
val filtered2 by derivedStateOf { all.filter { it.matches(query) } }
```

---

## 5) 성능 팁 & 안티패턴  
**현장에서 자주 겪는 실수들** (Common pitfalls in the field)

1. 무거운 작업을 `remember`에서 동기 실행 ✗  
   (Don’t run heavy blocking work inside `remember`.)  
   → 파일 I/O·디코딩·네트워크는 `LaunchedEffect`/`produceState`로 백그라운드 처리.  
   (Use `LaunchedEffect`/`produceState` with coroutines for heavy work.)

2. `Context`를 전역 싱글톤에 보관 ✗  
   (Don’t keep an `Activity` context in long-lived singletons.)  
   → `applicationContext` 사용 또는 필요한 데이터만 캐시.  
   (Use `applicationContext` or cache only the extracted data.)

3. 키를 빼먹어 재계산 타이밍 놓침 ✗  
   (Forgetting keys → stale cache.)  
   → 로캘/테마/배율 의존이면 `remember(configuration, context)`를 습관화.  
   (Include `configuration`/`context` when you depend on them.)

---

## 6) FAQ 빠르게 정리 (Lightning FAQ)

- **키 비교는 어떻게 하나요?** `==`(구조적 동등)로 비교합니다. (Keys are compared using structural equality `==`.)
- **값은 언제 사라지나요?** 컴포저블이 컴포지션에서 제거되거나, 키가 바뀌거나, 같은 슬롯이 아니게 되면 사라집니다. (When the composable leaves the composition, keys change, or the slot changes.)
- **`remember`와 `rememberSaveable` 선택?** 사용자 입력처럼 복원이 필요한 상태는 `rememberSaveable`, 그 외엔 `remember`. (Use `rememberSaveable` for restorable user-facing state; otherwise `remember`.)

---

## 7) 종합 예시 (End-to-end example)

```kotlin
@Composable
fun FlavorOptions(optionIds: List<Int>) {
    val configuration = LocalConfiguration.current
    val context = LocalContext.current

    // 환경 의존 변환을 키 기반으로 캐시
    // (Cache environment-dependent transformation with keys)
    val options = remember(configuration, context) {
        optionIds.map(context::getString)
    }

    LazyColumn {
        items(options, key = { it }) { label ->
            Text(
                text = label,
                style = MaterialTheme.typography.bodyLarge,
                modifier = Modifier.padding(16.dp)
            )
        }
    }
}
```

이 예시는 구성/컨텍스트 변화에만 반응해 문자열 목록을 재계산하고, 평소 재구성에선 리스트를 재사용합니다. (Recomputes only when configuration/context change; otherwise reuses the list across recompositions.)
