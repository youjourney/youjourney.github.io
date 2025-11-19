---
title: "Sealed Class vs Data Class(Inner) vs Enum Class 비교 "
excerpt: "Kotlin sealed class, 내부 data class, enum class의 차이와 목적"
categories: ["Kotlin", "Android"]
tags: ["sealed class", "enum", "data class", "kotlin"]
last_modified_at: "2025-11-19T14:00:00+09:00"
---

# Sealed Class vs 내부 Data Class vs Enum Class 비교

sealed class, sealed class 내부 data class, enum class는 모두 '정해진 집합'을 표현하지만 서로 다른 의도와 구조를 가진다.  
Sealed classes, sealed class inner data classes, and enum classes all represent finite sets but serve different structural and semantic purposes.

이 문서는 세 구조를 목적·특징·사용 예시 중심으로 비교한다.  
This document compares them based on their purpose, characteristics, and use cases.

---

# 1. Enum Class

## 개념  
enum class는 고정된 상수 값의 집합을 나타낸다.  
Enum classes represent a fixed set of constant values.

각 항목은 동일한 구조를 가지며 단일 인스턴스로 존재한다.  
Each entry shares the same structure and exists as a single instance.

```kotlin
enum class MenuType {
    ENTREE,
    SIDE,
    ACCOMPANIMENT
}
```

## 특징  
- 모든 항목이 동일한 생성자 구조를 가진다.  
  All entries share the same constructor schema.  
- 단순한 값 그룹을 표현하기에 적합하다.  
  Best suited for simple value groups.  
- ordinal/name 등 기본 메타데이터를 제공한다.  
  Provides built-in metadata such as ordinal and name.  
- 상속 불가.  
  Cannot be subclassed.  
- when 구문에서 exhaustive 체크 가능.  
  Allows exhaustive checking in `when` statements.

## 사용 예  
- 버튼 상태 (Enabled / Disabled / Loading)  
  Button states (Enabled / Disabled / Loading)  
- 고정된 규칙적인 값 집합  
  Fixed, predictable value groups

---

# 2. Sealed Class

## 개념  
sealed class는 **타입의 닫힌 집합(closed set of types)**을 정의한다.  
A sealed class defines a **closed set of types**.

하위 타입은 같은 파일 안에서만 선언할 수 있다.  
Subclasses can only be declared in the same file.

```kotlin
sealed class MenuItem {
    data class EntreeItem(val name: String, val price: Double) : MenuItem()
    data class SideDishItem(val name: String, val price: Double) : MenuItem()
    data class AccompanimentItem(val name: String, val price: Double) : MenuItem()
}
```

## 특징  
- 하위 타입이 파일 내부에 고정된다.  
  Subclass types are fixed within the file.  
- 각 하위 타입에 서로 다른 속성과 로직 부여 가능.  
  Each subtype may have different properties and behaviors.  
- 클래스 계층 표현 가능.  
  Enables modeling complex type hierarchies.  
- when 구문에서 else 없이 exhaustive 매칭 가능.  
  Enables exhaustive `when` expressions without requiring an `else`.

## 사용 예  
- 네트워크 상태 모델링  
  Network state modeling  
- UI State 표현  
  UI state representation  
- 구조가 복잡하며 상태별로 속성이 다를 때  
  When states differ in structure or complexity

---

# 3. 왜 sealed class 내부에 data class를 선언하는가?

sealed class와 내부 data class 조합은 **ADT(Algebraic Data Type)** 모델링을 구현하는 가장 강력한 방식이다.  
Combining sealed classes with inner data classes is a powerful pattern for representing **Algebraic Data Types (ADTs)**.

## 이유 1 — 명확한 타입 분류  
Reason 1 — Clear type classification  
각 종류를 명확히 다른 타입으로 분리할 수 있다.  
Each variant can be represented as a distinctly typed model.

## 이유 2 — data class의 강점 활용  
Reason 2 — Leverage the strengths of data classes  
- equals/hashCode 자동 생성  
  Auto-generated equals/hashCode  
- copy() 제공  
  Provides copy()  
- toString() 자동 생성  
  Auto toString()  
→ 상태 모델에 최적  
→ Ideal for representing stateful data models

## 이유 3 — 타입 + 데이터 구조를 모두 표현  
Reason 3 — Express both type and structured data  
enum은 “값”만 표현하지만 sealed class + data class는 타입과 값 구조를 모두 표현한다.  
Enums represent only values, while sealed class + data class pairs represent both types and structured data.

---

# 4. Sealed Class vs Enum Class 비교 표

| 구분 | Enum Class (KR/EN) | Sealed Class (KR/EN) |
|------|---------------------|------------------------|
| 표현 대상 / Representation | 값(Value) | 타입(Type) |
| 구조 / Structure | 모든 항목 동일 / Uniform | 타입마다 상이 / Varies per subtype |
| 상속 / Inheritance | 불가 / Not allowed | 같은 파일 내에서만 가능 / Allowed within same file |
| 데이터 보유 / Data | 제한적 / Limited | 풍부함 / Rich & flexible |
| when exhaustive | 지원 / Supported | 지원 / Supported |
| 적합한 용도 / Best for | 단순 상수집합 / Simple constants | 복잡한 타입 계층 / Complex type hierarchies |

---

# 5. Sealed Class vs Sealed Interface

Kotlin 1.5 이후 sealed interface도 사용 가능하다.  
Since Kotlin 1.5, sealed interfaces are available.

| 항목 | Sealed Class | Sealed Interface |
|------|--------------|------------------|
| 다중 상속 | 불가 / No | 가능 / Yes |
| 필드 보유 | 가능 / Yes | 제한적 / Limited |
| 행동 모델링 | 보통 / Moderate | 매우 강함 / Very strong |
| 데이터 모델링 | 우수 / Strong | 약함 / Weak |
| 계층 유연성 | 중간 / Medium | 매우 높음 / High |

---

# 6. 어떤 걸 선택해야 할까?

| 상황 (KR/EN) | 선택 (Choice) |
|--------------|---------------|
| 단순 상수 집합 / Simple constant set | enum class |
| 상태 + 데이터가 복잡 / Complex state + data | sealed class + data class |
| 다중 상속 필요 / Need multiple inheritance | sealed interface |
| 안전한 exhaustive 브랜칭 / Safe exhaustive branching | sealed class or enum |
| 항목마다 다른 속성 필요 / Per-item unique attributes | sealed class |

---

# 결론 (Conclusion)

- enum은 ‘값 집합(value set)’을 표현한다.  
  Enums represent sets of values.  
- sealed class는 ‘타입 집합(type set)’을 표현한다.  
  Sealed classes represent sets of types.  
- sealed class + 내부 data class 조합은 ADT 모델링에 이상적이다.  
  Sealed classes with inner data classes form an ideal ADT modeling structure.  
- 상황에 따라 올바른 구조를 선택하면 유지보수성과 안정성이 높아진다.  
  Choosing the right structure improves maintainability and type safety.

