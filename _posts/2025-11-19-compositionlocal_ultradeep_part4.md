---
title: "Compose SlotTable Mutation & FrameClock"
excerpt: "Step-by-step SlotTable mutation diagram + Recomposer FrameClock/MonotonicFrameClock internals"
categories: ["Jetpack Compose", "Runtime Internals"]
tags: ["SlotTable", "FrameClock", "Recomposer"]
last_modified_at: "2025-11-19T14:54:00+09:00"
---

# Compose SlotTable Mutation Step-by-step + FrameClock Internals

## 1. SlotTable Mutation Step-by-step Diagram (애니메이션 개념 다이어그램)

아래는 SlotTable이 recomposition 동안 실제로 어떻게 변경되는지를 단계별 애니메이션처럼 구성한 다이어그램이다.  
This diagram visualizes how SlotTable mutates step-by-step during recomposition.

```
[1] read CompositionLocal
     ↓
SlotTable lookup nearest provider

[2] State mutation happens
     ↓
Snapshot marks changed state

[3] Recomposer observes snapshot
     ↓
Marks dependent groups as `dirty`

[4] Recompose only dirty groups
     ↓
Group re-entered, new providers applied

[5] SlotTable writes updated nodes
     ↓
Commit phase

[6] UI refreshed
```

각 단계는 transactional하며, Snapshot → Recomposer → SlotTable → UI 순환을 가진다.

---

## 2. SlotTable 더 깊은 내부 구조 (Deep Internal Structure)

SlotTable의 핵심 구성요소:

```
SlotTable
 ├─ anchors[]        // Group 위치 고정
 ├─ slots[]          // 실제 값 저장
 ├─ groups[]         // composable boundaries
 │     ├─ parent pointer
 │     ├─ key index
 │     ├─ providerMask
 │     └─ slotIndex range
 └─ gapBuffer        // 삽입/삭제 최적화 구조
```

특징:
- GapBuffer 기반으로 삽입/이동 비용이 낮다
- Group/Slot이 연속 메모리 배열로 저장되어 매우 빠르다
- CompositionLocal provider mask는 bit flag 기반 최적화

---

## 3. Recomposer FrameClock / MonotonicFrameClock

Recomposer는 `FrameClock`과 결합하여 **UI 프레임 단위**로 recomposition을 실행한다.  
Recomposer coordinates with FrameClock to align recompositions with UI frames.

### FrameClock 개념

```
requestFrame()
 ↓
FrameClock posts frame
 ↓
Recomposer runs applyChanges()
 ↓
UI redraw
```

### MonotonicFrameClock

Compose가 제공하는 frame clock이며, Android choreographer와 연결된다.

Compose는 다음 루프를 기반으로 실행됨:

```
while (true) {
    await frame from MonotonicFrameClock
    apply pending changes
}
```

---

## 4. Recomposer Scheduler Internals

Recomposer는 세 가지 작업 큐를 가진다.

```
pendingRecompositions
applyChangesQueue
sideEffectsQueue
```

재구성은 다음 조건에서 실행됨:
- Snapshot 변경 발생
- FrameClock frame 도착
- UI thread idle 시 yield

Compose의 scheduler는 coroutine 기반 cooperative scheduling을 사용한다.
