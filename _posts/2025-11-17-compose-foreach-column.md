---
title: "How Compose Renders Repeated Composables (with Lazy keys & compiler metrics)"
excerpt: "forEach와 Lazy*에서 반복되는 컴포저블이 실제로 어떻게 붙고(emit) 재사용되는지, Slot Table/Applier/Diff, 고급 key 전략, 그리고 Compose Compiler Metrics 정리."
categories: ["Android", "Jetpack Compose", "Kotlin"]
tags: ["Compose", "Recomposition", "Slot Table", "Composer", "Applier", "key", "LazyColumn", "Tracing", "Performance", "Compiler Metrics"]
last_modified_at: 2025-11-17T18:40:00+09:00
---

# Column + forEach의 내부 동작 요약  
How `Column + forEach` behaves under the hood

Compose는 **컴포저블 호출 = UI 노드 발행(emit)** 모델을 사용합니다.  
In Compose, **calling a Composable = emitting a UI node**.

`forEach`는 `Unit`을 반환하지만, **반복마다 컴포저블을 호출**하고 부모 레이아웃(Column 등)에 **자식 노드가 추가**됩니다.  
Although `forEach` returns `Unit`, **each iteration calls a Composable** which **adds a child node** to the parent.

---

## 빠른 리마인드: Composer / Slot Table / Applier  
Quick reminder: Composer / Slot Table / Applier

- **Composer**: 컴포지션을 지휘하고 그룹/키/위치/remember 메타데이터를 관리합니다.  
  Orchestrates composition, manages groups/keys/positions/remember metadata.
- **Slot Table**: “**무엇이 어디 있었는지**”의 청사진. 재구성 때 **정확히 매칭/재사용**을 가능하게 합니다.  
  Blueprint of **what was where**; enables **matching/reuse** on recomposition.
- **Applier**: 변경 목록을 실제 UI 트리(레이아웃 노드)에 **삽입/이동/삭제**로 적용합니다.  
  Applies changes to the real UI tree via **insert/move/remove**.

---

## 의사 Slot Table 스냅샷  
Pseudo Slot Table snapshot

> 실제 내부 포맷은 다르므로, 이해를 위한 **개념적 스냅샷**입니다.  
> Conceptual snapshot for understanding; actual internal format differs.

### Pass #1 — data: `[A, B, C]`

```
Group 100: Column(key=Column)
  Group 110: content-lambda
    Group 201: SelectQuantityButton(key=A)  [remember: state@0xA]
    Group 202: SelectQuantityButton(key=B)  [remember: state@0xB]
    Group 203: SelectQuantityButton(key=C)  [remember: state@0xC]
```

### Pass #2 — data: `[B, A, C]` (순서 변경 / reordered)

```
Group 100: Column(key=Column)
  Group 110: content-lambda
    Group 202: SelectQuantityButton(key=B)  // REUSE + MOVE
    Group 201: SelectQuantityButton(key=A)  // REUSE + MOVE
    Group 203: SelectQuantityButton(key=C)  // REUSE
```

키 기반 매칭으로 노드를 재생성하지 않고 **이동/재사용**합니다.  
With key-based matching, nodes are **moved/reused** instead of recreated.

---

## Diff 로그(의사 예시)  
Pseudo diff log

```
Recomposition Start
  Match 202 (key=B): REUSE, MOVE 1 -> 0
  Match 201 (key=A): REUSE, MOVE 0 -> 1
  Match 203 (key=C): REUSE, STAY 2
Apply Changes
  Applier.move(B, 1 -> 0)
  Applier.move(A, 0 -> 1)
Recomposition End
```

---

## 성능 트레이싱 팁  
Performance tracing tips

### Layout Inspector — Composition 탭
- **재구성 횟수**와 **트리 구조** 확인. 상위에서 상태를 수집하면 하위 전체가 빈번히 갱신될 수 있습니다.  
  Inspect **recomposition counts** & structure; collecting state high up can refresh a large subtree.

### System Trace — 프레임 타임라인
- `Choreographer#doFrame`, `Recomposer`, `measure/layout/draw`를 타임라인으로 파악.  
  Visualize frame stages and where time is spent.
- 코드에 **trace 마커**를 추가해 병목 구간을 구분.  
  Add code **trace markers** to pinpoint hotspots.

```kotlin
import androidx.tracing.trace

fun expensiveWork() = trace("expensiveWork") {
    // heavy code here
}
```

---

# 🔸 Lazy 계열에서 **key 전략 심화**  
Advanced key strategies for Lazy containers

Lazy 계열(`LazyColumn`, `LazyRow`, `LazyVerticalGrid`)은 **지연 컴포지션 + 가상화**가 핵심입니다. **키 전략**은 노드 재사용과 스크롤 포지션, 애니메이션 안정성에 직접적 영향을 줍니다.  
Lazy containers virtualize items; **key strategy** directly affects node reuse, scroll position, and animations stability.

## 1) 올바른 key 지정의 기본  
Basics

```kotlin
LazyColumn {
    items(
        items = users,
        key = { user -> user.id } // ✅ stable, unique
    ) { user ->
        UserRow(user)
    }
}
```

- **고유하고 변하지 않는 식별자**(DB id, UUID 등)를 키로 사용.  
  Use a **unique, stable identifier** (DB id, UUID).
- 인덱스를 키로 쓰면 **중간 삽입/삭제**에서 모든 이후 아이템이 **다른 항목으로 매칭**되어 스크롤 위치, 애니메이션, remember 상태가 흔들립니다.  
  Using index causes mis-match on mid insert/remove, breaking scroll/animations/remember.

## 2) items vs itemsIndexed — 언제 어떤 것을?  
When to use `items` vs `itemsIndexed`

- `items(list, key=...)`는 **키만** 필요할 때 간단합니다.  
  Use `items` when you only need keys.
- 인덱스가 필요하면 `itemsIndexed(list, key=...)`를 사용하되 **반드시 key를 제공**하세요.  
  If you need index, use `itemsIndexed` but **still provide a key**.

```kotlin
itemsIndexed(
    items = messages,
    key = { _, msg -> msg.id } // ✅ provide stable key
) { index, msg ->
    MessageRow(index, msg)
}
```

## 3) 복합 키 전략 (fallback)  
Composite keys (fallback)

데이터에 고유 id가 없다면, **안정 속성들의 조합**으로 키를 구성하세요.  
If no single id, compose a key from stable fields.

```kotlin
key = { item -> "${item.type}:${item.code}:${item.version}" }
```

> 주의: **자주 변하는 필드**를 포함하면 키가 바뀌어 재생성이 발생합니다.  
> Beware: Including **frequently changing fields** makes keys unstable.

## 4) remember / rememberSaveable와의 상호작용  
Interaction with remember / rememberSaveable

- `remember`는 **슬롯 위치+키**에 저장됩니다. 키가 흔들리면 상태가 초기화됩니다.  
  `remember` is stored at slot position + key; unstable keys reset state.
- 스크롤 위치/입력값 보존이 필요하면 `rememberSaveable`을 쓰고, **안정 키**를 유지하세요.  
  For preserving scroll/input, use `rememberSaveable` with **stable keys**.

## 5) 애니메이션과 key — `animateItemPlacement()`  
Animations and keys

```kotlin
items(items = todos, key = { it.id }) {
    TodoRow(
        todo = it,
        modifier = Modifier.animateItemPlacement()
    )
}
```

키가 바뀌면 애니메이션 연속성이 깨집니다. **항상 동일 item은 동일 key**가 되도록 보장하세요.  
Changing keys breaks animation continuity; **same item must keep the same key**.

## 6) Paging/Streaming 시나리오  
Paging/Streaming scenarios

- **페이지 경계**가 바뀌어도 **항목의 id는 유지**되어야 합니다.  
  Keep item `id` stable across page boundaries.
- 서버가 아이템을 **치환**하는 API라면, 클라이언트에서 **불변 id 생성**(예: hash of immutable fields)을 고려하세요.  
  If server replaces entries, consider client-side **immutable ids** (e.g., hash of stable fields).

## 7) 성능 팁  
Performance tips

- `key`를 제공하면 Compose가 **MOVE**로 처리할 수 있어 **재조립/재측정 감소**에 유리합니다.  
  Keys enable **MOVE** instead of recreate; fewer remeasures/rebuilds.
- 다만 **항상 키를 계산하는 비용**도 있으므로, 데이터가 *진짜로* 안정 id를 가지고 있을 때만 지정하세요.  
  There’s a small cost; only specify keys when data truly has stable ids.

---

# 🔸 Compose Compiler Metrics 설정  
Compose Compiler Metrics configuration

> 버전에 따라 설정 문법이 조금씩 다릅니다. 아래는 Kotlin DSL 예시를 **두 가지 버전**(Kotlin 1.9.x / Kotlin 2.x)로 제공합니다.  
> Syntax varies by versions; below are Kotlin DSL examples for **Kotlin 1.9.x** and **Kotlin 2.x**.

## (A) Kotlin 1.9.x / Compose 1.5~1.6 계열 예시  
Example for Kotlin 1.9.x

**module-level** `build.gradle.kts`

```kotlin
android {
    // ...
}

kotlin {
    sourceSets.all {
        languageSettings.optIn("androidx.compose.compiler.plugins.kotlin.ComposeCompilerApi")
    }
}

tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile>().configureEach {
    kotlinOptions {
        freeCompilerArgs += listOf(
            "-P", "plugin:androidx.compose.compiler.plugins.kotlin:reportsDestination=${"$"}{project.buildDir}/compose-reports",
            "-P", "plugin:androidx.compose.compiler.plugins.kotlin:metricsDestination=${"$"}{project.buildDir}/compose-metrics",
            "-P", "plugin:androidx.compose.compiler.plugins.kotlin:strongSkipping=true"
        )
    }
}
```

- **reportsDestination**: 재구성 가능성/건너뛰기(skippability) 보고서.  
  Where textual **reports** (restartable/skippable, etc.) are written.
- **metricsDestination**: 함수/클래스 단위 **메트릭**(안정성/재구성 특성 등).  
  Destination for **metrics** (stability, recomposition traits).
- **strongSkipping**: 더 공격적인 스킵 최적화 시도. (프로젝트에 따라 영향 다름)  
  Enables stronger skipping optimizations (project-dependent effect).

## (B) Kotlin 2.x / Compose 1.7+ 계열 예시  
Example for Kotlin 2.x

```kotlin
import org.jetbrains.kotlin.gradle.dsl.JvmTarget

android {
    // ...
}

kotlin {
    compilerOptions {
        freeCompilerArgs.addAll(
            "-P", "plugin:androidx.compose.compiler.plugins.kotlin:reportsDestination=${"$"}{project.buildDir}/compose-reports",
            "-P", "plugin:androidx.compose.compiler.plugins.kotlin:metricsDestination=${"$"}{project.buildDir}/compose-metrics",
            "-P", "plugin:androidx.compose.compiler.plugins.kotlin:strongSkipping=true"
        )
        jvmTarget.set(JvmTarget.JVM_17)
    }
}
```

### 결과 파일 읽는 법  
How to read outputs

- `build/compose-reports`  
  - `composables.txt`: 각 컴포저블의 **restartable**, **skippable**, **readonly**, **isolated** 등의 플래그와 호출 그래프.  
    Flags per composable and call graphs.
- `build/compose-metrics`  
  - 안정성(inferred stability) 분석, 그룹/슬롯 관련 수치.  
    Inferred stability, groups/slots related metrics.

> 리포트에서 **skippable**이 많을수록, 동일 입력에 대해 **재구성 스킵** 가능성이 큽니다.  
> More **skippable** generally means more recompositions can be skipped for equal inputs.

### 흔한 해석 팁  
Practical reading tips

- 특정 컴포저블이 **restartable만 있고 skippable이 없다면**, **불안정 파라미터**나 **캡처된 람다**를 점검.  
  If a composable is restartable but not skippable, check **unstable params** or **captured lambdas**.
- `@Stable` 주석은 **재구성 스킵을 보장**하지 않습니다. 멤버 안정성/변경 추적이 실제로 성립해야 합니다.  
  `@Stable` does not guarantee skipping; true stability semantics must hold.

---

## Lazy vs Column — 언제 어떤 것을?  
When to use Lazy vs Column

- **Column + forEach**: 항목 수가 적고 스크롤이 없다면 간단하고 비용이 적음.  
  For small, non-scrolling content, simple and cheap.
- **LazyColumn**: 항목이 많거나 스크롤이 필요하면 지연 컴포지션/측정으로 효율적. **반드시 key 전략**을 고려.  
  For large/scrolling lists, efficient via virtualization; **consider key strategy**.

---

## 최종 체크리스트  
Final checklist

- 동적 목록엔 **안정 키** 제공 (`items(list, key={{ it.id }})`).  
  Provide **stable keys**.
- `remember/rememberSaveable` 상태는 **키+위치**에 종속. 키가 흔들리지 않게 설계.  
  Remembered state depends on **key+position**; keep keys steady.
- 과도한 재구성은 **상태 수집 위치**를 낮추고, **metrics/reports**로 확인.  
  Reduce recompositions by collecting state lower; verify with **metrics/reports**.
- 성능 분석은 **Layout Inspector + System Trace + trace 마커**로 삼각 측량.  
  Triangulate performance with **Layout Inspector + System Trace + trace markers**.
