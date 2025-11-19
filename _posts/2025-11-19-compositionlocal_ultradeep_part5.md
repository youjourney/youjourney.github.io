---
title: "Compose Compiler IR Dump + Plugin Internals"
excerpt: "How to generate IR dumps, understand Compose Compiler plugin internals, and track code lowering"
categories: ["Jetpack Compose", "Compiler"]
tags: ["IR Dump", "Compose Compiler", "Lowering Pass"]
last_modified_at: "2025-11-19T14:55:00+09:00"
---

# Compose IR Dump Generation + Compose Compiler Plugin Internals

## 1. IR Dump를 실제로 생성하는 방법

### Gradle 설정
```
tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile>().configureEach {
    kotlinOptions.freeCompilerArgs += listOf(
        "-Xir-dump",
        "-Xir-dump-directory=build/ir"
    )
}
```

### 실행 결과
- `build/ir/` 내부에 *.ir, *.txt 파일 생성  
- CompositionLocalProvider는 startGroup/endGroup로 lowering됨  
- remember는 RememberObserver 노드와 함께 저장됨

---

## 2. Compose Compiler Plugin Internals Overview

### Compose Compiler 역할
- Composable 분석
- SlotTable 삽입 코드 생성
- Group boundary 생성
- remember / CompositionLocal lowering
- Recomposer가 소비할 메타데이터 생성

### 주요 lowering phase

```
ComposableDeclarationTransformer
ComposableJvmLowering
ComposerParamRemover
LateInitRemover
SlotTableBuilderLowering
RememberLowering
CompositionLocalLowering
```

각 lowering은 다음 형태의 IR을 생성한다:

```
composer.startReplaceableGroup(key)
composer.updateProvider(LocalA, value)
composer.endReplaceableGroup()
```

---

## 3. IR Dump 분석 예시

```
%tmp0 = call composer.consume(LocalSpacing)
store %tmp0 into local variable

%composer.startReplaceableGroup(187723)
%composer.updateProvider(LocalLocale, value)
invoke children
%composer.endReplaceableGroup()
```

IR dump는 성능 최적화 및 동작 이해에 매우 유용하다.
