---
title: "Compose Memory, Stability, and GC Considerations"
excerpt: "SlotTable GC, stability inference, remember lifecycle, and memory best practices"
categories: ["Jetpack Compose", "Performance", "Memory"]
tags: ["Memory", "Stability", "GC", "remember"]
last_modified_at: "2025-11-19T15:05:00+09:00"
---

# Compose Memory Management, Stability & GC in Compose

## 1. remember Lifecycle & Memory

`remember`는 SlotTable에 값을 저장하고, 해당 그룹이 유효한 동안 메모리를 유지한다.  
`remember` stores values in the SlotTable and keeps them alive while the group remains valid.

그룹이 제거되면:
- remember 값도 GC 대상이 된다  
- BackStackEntry 단위로 scope가 유지되면 state도 함께 유지

---

## 2. Stability Inference

Compose Compiler는 object의 stability를 추론해 재구성 최적화를 한다.

안정한 타입의 특징:
- data class with stable fields
- immutable collections
- primitive types

안정하지 않은 타입은 parameter 변화 시 더 자주 recomposition을 유발할 수 있다.

---

## 3. SlotTable GC 패턴

SlotTable은 자체적으로 GC를 수행하지는 않지만:

- 그룹이 제거되면 해당 group의 anchor/slot이 재사용 또는 폐기  
- Kotlin/JVM GC가 unused 객체를 수집  
- recomposition이 반복되면서 old 그룹이 교체됨

---

## 4. 메모리 사용 모범 사례

- 큰 객체를 rememberSaveable에 직접 넣지 않기  
- ViewModel에서 heavy state 관리  
- Composable은 가벼운 뷰 상태만 유지  
- Long-lived coroutine + Context leak 주의

---

## 5. CompositionLocal과 메모리

CompositionLocal에 큰 객체를 넣으면:
- 모든 자식 컴포지션이 참조 가능  
- 예상보다 더 오래 메모리에 남을 수 있음

따라서:
- 환경값(설정, config, style) 수준으로만 사용하는 것이 바람직  
- business data나 대형 캐시는 ViewModel/Repository 레벨로 분리
