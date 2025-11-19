---
title: "Compose Recomposer Profiling & Performance Analysis"
excerpt: "Recomposer event profiler, frame analysis, and methods for observing CompositionLocal effects"
categories: ["Jetpack Compose", "Performance"]
tags: ["Recomposer", "Profiler", "Performance", "Frame Analysis"]
last_modified_at: "2025-11-19T14:58:00+09:00"
---

# Compose Recomposer Profiler & Performance Guide

## 1. Recomposer Event Profiler  
Recomposer는 다음 이벤트를 기록한다:

- Snapshot state reads/writes  
- Group dirty marking  
- applyChanges duration  
- Skipped recompositions  
- Frame scheduling delays  

You can monitor them using:

```
adb shell setprop debug.compose.log.recomposer true
```

## 2. UI Thread / Recomposer Thread 분석  
Recomposer는 coroutine 기반이므로, scheduler profiling도 중요하다.

관찰 포인트:
- frame boundary timing
- choreographer-driven delays
- skipped compositions
- Long applyChanges blocks

## 3. CompositionLocal이 성능에 미치는 영향  
CompositionLocal is fast **unless**:

- Local value changes frequently
- Many composables read the same Local
- Provider deeply nested

## 4. Frame Analysis Diagram
```
FrameStart →
    RunPendingWork →
        Snapshot apply →
            Recompose →
                Layout →
                    Draw →
FrameEnd
```

CompositionLocal 변경은 “Recompose” 단계만 영향을 준다.

## 5. Tools for Profiling
- Android Studio Layout Inspector (Live Compose Tree)
- Systrace / Perfetto
- compose-runtime tracing flags
- Compiler IR dump + tracing integration
