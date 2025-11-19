---
categories:
- Android
- Compose
excerpt: "Jetpack Compose 상태관리: remember, rememberSaveable,
  derivedStateOf의 동작 비교, 키 전략, 로케일 변화 처리, 화면 구조
  흐름까지 상세 정리"
last_modified_at: 2025-11-17T23:30:00+09:00
tags:
- compose
- jetpack
- state
- remember
- rememberSaveable
- derivedStateOf
title: Compose remember vs rememberSaveable vs derivedStateOf
---

# Jetpack Compose 상태 관리

remember는 재구성 사이에서 값을 기억해 UI가 다시 그려져도 메모리에
보관된 상태를 계속 사용한다.\
remember keeps values across recompositions, reusing state stored in
memory during UI redraws.

이 상태는 화면 회전이나 프로세스 종료에서는 복원되지 않는다.\
This state does not survive configuration changes or process death.

``` kotlin
val count = remember { mutableStateOf(0) }
```

키를 제공하면 해당 키가 변할 때 새로운 상태가 생성된다.\
Providing a key causes the state to be recreated when the key changes.

``` kotlin
val title = remember(locale) { getLocalizedTitle(locale) }
```

------------------------------------------------------------------------

rememberSaveable은 구성 변경이나 프로세스 종료 후에도 값을 복원하기 위해
Bundle 또는 Saver를 사용한다.\
rememberSaveable restores values after configuration changes or process
recreation using Bundles or custom Savers.

``` kotlin
val name = rememberSaveable { mutableStateOf("") }
```

로케일이 바뀌면 기존 데이터와 충돌하지 않도록 키로 locale을 포함할 수
있다.\
Including locale as a key ensures the state resets when the locale
changes, avoiding mismatches.

``` kotlin
val name = rememberSaveable(locale) { mutableStateOf("") }
```

------------------------------------------------------------------------

derivedStateOf는 입력 상태가 변할 때만 계산되는 파생 값을 만들며
불필요한 연산을 피한다.\
derivedStateOf creates derived values recalculated only when
dependencies change, avoiding unnecessary computation.

``` kotlin
val query = remember { mutableStateOf("") }
val filtered = derivedStateOf { items.filter { it.contains(query.value) } }
```

------------------------------------------------------------------------

Compose는 상태를 Slot Table에 위치 기반으로 저장하며 키 변경은 해당
슬롯을 폐기하고 새 상태를 만든다.\
Compose stores state in the Slot Table by position, and when keys
change, the slot is discarded and rebuilt.

derivedStateOf는 snapshot 관찰을 통해 의존 값이 변할 때만 업데이트한다.\
derivedStateOf listens through snapshots, updating only when
dependencies change.

------------------------------------------------------------------------

Locale 변경 감지를 위해 locale 객체 자체를 키로 사용하거나
toLanguageTag를 키로 활용할 수 있다.\
To detect locale changes, use the locale object or its language tag as
the key.

``` kotlin
remember(locale) { ... }
remember(locale.toLanguageTag()) { ... }
```

RTL/LTR 방향만 감지하고 싶다면 LayoutDirection을 키로 사용할 수 있다.\
If only text direction changes matter, LayoutDirection can be used as a
key.

``` kotlin
val dir = LocalLayoutDirection.current
remember(dir) { ... }
```

------------------------------------------------------------------------

아래는 화면 구조 전체와 상태 흐름이 결합된 실제 예시다.\
Below is a combined example showing overall screen structure and state
flow.

    ProfileScreen
     ├─ Title (locale 기반)
     ├─ NameInput (rememberSaveable)
     ├─ ValidationMessage (derivedStateOf)
     └─ SaveButton (derivedStateOf 기반)

``` kotlin
@Composable
fun ProfileScreen(locale: Locale) {

    val title = remember(locale) {
        if (locale.language == "ko") "프로필 설정" else "Profile Settings"
    }

    val name = rememberSaveable(locale) { mutableStateOf("") }

    val isValid = derivedStateOf { name.value.length >= 2 }

    Column {

        Text(title)

        TextField(
            value = name.value,
            onValueChange = { name.value = it }
        )

        if (!isValid.value) {
            Text(
                if (locale.language == "ko") "이름은 2글자 이상 필요합니다"
                else "Name must be at least 2 characters"
            )
        }

        Button(
            enabled = isValid.value,
            onClick = {}
        ) {
            Text(if (locale.language == "ko") "저장" else "Save")
        }
    }
}
```

이 예시에서 title과 name은 locale이 변경될 때 재생성되고 isValid는
name이 바뀔 때만 계산된다.\
In this example, title and name are recreated when the locale changes,
while isValid recalculates only when the name changes.

------------------------------------------------------------------------

remember는 재구성만 고려할 때, rememberSaveable은 회전·종료 후 복원까지
필요할 때,\
remember is ideal for recomposition-only persistence, rememberSaveable
for restoration through rotation or process death,

derivedStateOf는 고비용 파생 계산을 최적화할 때 사용한다.\
and derivedStateOf excels when optimizing expensive derived
computations.

키 전략은 로케일 변화와 같은 외부적 변화에 반응하는 상태 관리에서
핵심이다.\
Key strategy is essential when handling external changes such as locale
updates.
