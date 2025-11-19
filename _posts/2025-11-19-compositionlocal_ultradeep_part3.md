---
title: "Compose CompositionLocal Navigation + ThemeBuilder + Compiler Logs"
excerpt: "Navigation integration, App-wide ThemeBuilder system, and IR dump analysis"
categories: ["Jetpack Compose", "Architecture", "Navigation"]
tags: ["Navigation", "ThemeBuilder", "IR Dump"]
last_modified_at: "2025-11-19T14:53:00+09:00"
---

# Compose Navigation, ThemeBuilder Architecture, IR Logs

## 1. Navigation과 CompositionLocal

### Graph-level Provider
```
CompositionLocalProvider(LocalSession provides session) {
    NavHost(...)
}
```

### Screen-level Override
```
composable("settings") {
    CompositionLocalProvider(LocalLocale provides localeKR) {
        SettingsScreen()
    }
}
```

BackStackEntry는 해당 CompositionLocal 스코프까지 보존함.  
Back navigation restores previous CompositionLocal environments.

---

## 2. App-wide ThemeBuilder Pattern

### 구조
```
theme/
  ThemeBuilder.kt
  Color.kt
  Typography.kt
  Spacing.kt
  Elevation.kt
  LocaleConfig.kt
```

### ThemeBuilder 구현 예시
```kotlin
@Composable
fun AppTheme(
    darkTheme: Boolean,
    locale: LocaleConfig,
    content: @Composable () -> Unit
) {
    CompositionLocalProvider(
        LocalLocaleConfig provides locale,
        LocalSpacing provides Spacing(),
        LocalElevation provides Elevation()
    ) {
        MaterialTheme(
            colorScheme = if (darkTheme) DarkColors else LightColors,
            typography = AppTypography,
            content = content
        )
    }
}
```

---

## 3. Compiler IR Dump 예시

로컬 값을 읽는 부분의 IR dump 형태 예시:  
Example IR dump:

```
%tmp0 = call %composer.consume(LocalSpacing)
store %tmp0 → local variable
```

CompositionLocalProvider의 IR:
```
%composer.startReplaceableGroup(key)
%composer.updateProvider(LocalLocaleConfig, value)
invoke children
%composer.endReplaceableGroup()
```

IR dump는 내부 동작 추적과 성능 최적화 분석에 유용하다.
