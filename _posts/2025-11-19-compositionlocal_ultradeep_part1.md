---
title: "Compose CompositionLocal, SlotTable, Snapshot System"
excerpt: "SlotTable internals, mutation lifecycle, Snapshot system, and CompositionLocal interaction"
categories: ["Jetpack Compose", "Android Internals"]
tags: ["CompositionLocal", "SlotTable", "Snapshot", "Recomposer"]
last_modified_at: "2025-11-19T14:50:00+09:00"
---

# Compose SlotTable, Snapshot System, and CompositionLocal Internals

## 1. SlotTable 개념
Compose SlotTable은 모든 구성 가능한 UI 구조와 상태를 저장하는 내부 데이터 구조이다.  
The SlotTable stores compositional structure and state in a compact tree-like structure.

```
SlotTable
 ├─ Groups
 │    ├─ Key
 │    ├─ Providers (CompositionLocal values)
 │    ├─ Remembered values
 │    └─ Children groups
```

## 2. SlotTable Mutation Lifecycle
```
State mutation →
  Snapshot records dirty → 
    Recomposer observes → 
      Determine affected groups →
        Apply group changes → 
          Commit to SlotTable
```

CompositionLocal 변경은 **해당 Local을 읽는 그룹만** dirty로 표시한다.  
CompositionLocal changes only mark groups that *read* that Local.

## 3. Snapshot System Interaction
Snapshot은 Compose의 트랜잭션 기반 상태 관리 시스템이다.  
Snapshot is the transactional state manager for Compose.

핵심 원리:
- 상태 변경은 Snapshot에 기록됨
- Recomposer는 Snapshot을 구독하고 변경을 감지함
- Compose는 필요한 그룹만 재구성함

## 4. CompositionLocal과 SlotTable 연결
CompositionLocalProvider는 SlotTable 그룹에 “provider map”을 추가한다.  
```
CompositionLocalProvider(LocalA provides A1)
↓
Slot: {LocalA → A1}
```

자식 컴포저블이 `LocalA.current`를 호출하면 SlotTable에서 가장 가까운 provider를 탐색한다.  
When children read `LocalA.current`, the SlotTable resolves the nearest provider.
