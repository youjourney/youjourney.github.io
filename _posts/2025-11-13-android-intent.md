---
title: "Android Intent Guide"
excerpt: "Android Intent의 개념, 구조, 유형, 예외 처리, 플래그"
categories: ["Android", "Kotlin", "Jetpack"]
tags: ["Intent", "Activity", "ACTION_SEND", "Implicit", "Explicit", "Chooser"]
last_modified_at: 2025-11-13T17:57:00+0900
---


## 🧩 Intent란? · What is an Intent?
- **Intent는 Android에서 컴포넌트 간 통신(IPC)을 위한 메시지 객체입니다. –** An Intent is a message object for inter-component communication (IPC) in Android.
- **Activity/Service/BroadcastReceiver에 작업을 요청하고 데이터를 전달합니다. –** It requests actions and passes data to Activities, Services, and BroadcastReceivers.

---

## 🧭 주요 목적 · Main Purposes
- **다른 화면(액티비티) 실행. –** Start another Activity.  
- **백그라운드 작업(서비스) 요청. –** Request background work via a Service.  
- **브로드캐스트 전송으로 이벤트 알림. –** Notify events by sending a broadcast.  
- **Extras로 데이터 전달. –** Pass data using extras.

---

## 🗂 인텐트 유형 · Types of Intents
**1) 명시적(Explicit)**: **실행할 컴포넌트를 정확히 지정합니다. –** You explicitly specify the target component to launch.
```kotlin
val intent = Intent(this, DetailActivity::class.java)
startActivity(intent)
```
**2) 암시적(Implicit)**: **수행할 작업만 정의하고, 대상 앱은 시스템이 결정합니다. –** You define the action; the system resolves a compatible app.
```kotlin
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://developer.android.com"))
startActivity(intent)
```

---

## 🧱 Intent의 구조 · Structure of Intent
```text
Intent {{ 
  action: String?      // 수행할 작업 (예: ACTION_SEND) – The action to perform (e.g., ACTION_SEND)
  data: Uri?           // 처리할 데이터 – The data to operate on
  type: String?        // MIME 타입 (예: text/plain) – MIME type (e.g., text/plain)
  extras: Bundle?      // 추가 데이터 – Additional key-value data
  flags: Int?          // 실행 플래그 – Launch flags
  component: ComponentName? // 명시 대상 – Explicit target component
}}
```

---

## ⚙️ Intent Filter와 매칭 · Filters & Matching
```xml
<intent-filter>
    <action android:name="android.intent.action.SEND" />
    <category android:name="android.intent.category.DEFAULT" />
    <data android:mimeType="text/plain" />
</intent-filter>
```
- **시스템은 action/category/data(MIME) 3요소 매칭으로 처리 앱을 결정합니다. –** The system chooses a handler app by matching action/category/data (MIME).

---

## 📤 공유 예제 (`ACTION_SEND`) · Sharing Example
```kotlin
private fun shareSoldDessertsInformation(intentContext: Context, dessertsSold: Int, revenue: Int) {
    val sendIntent = Intent().apply {
        action = Intent.ACTION_SEND
        putExtra(Intent.EXTRA_TEXT,
            intentContext.getString(R.string.share_text, dessertsSold, revenue))
        type = "text/plain"
    }
    val shareIntent = Intent.createChooser(sendIntent, null)
    try {
        ContextCompat.startActivity(intentContext, shareIntent, null)
    } catch (e: ActivityNotFoundException) {
        Toast.makeText(
            intentContext,
            intentContext.getString(R.string.sharing_not_available),
            Toast.LENGTH_LONG
        ).show()
    }
}
```
- **`createChooser()`는 항상 안전한 공유 선택 UI를 띄웁니다. –** `createChooser()` always presents a safe share chooser UI.
- **처리 앱이 없을 수 있으므로 try/catch로 예외를 처리합니다. –** Wrap in try/catch because no app may handle the intent.

---

## 🧠 내부 동작 · Internal Flow
```text
앱이 Intent 생성 → 시스템에 전달 → PackageManager가 필터 매칭
→ 후보 앱 목록 산출 →(필요시) Chooser 표시 → 사용자 선택 → 대상 앱 실행
App creates Intent → Pass to system → PackageManager matches filters
→ Build candidate list → (If needed) show chooser → User selects → Target launches
```

---

## 🖼 UI 플로우 다이어그램 · UI Flow Diagram
```text
사용자 탭 → 공유 함수 호출 → ACTION_SEND 인텐트 구성
→ createChooser 호출 → 앱 리스트 표시 → 사용자 선택 → 대상 앱 실행 및 데이터 전달
User taps → call share() → build ACTION_SEND
→ call createChooser → show app list → user picks → target app receives data
```

---

## 🧩 예외 처리 · Exception Handling
```kotlin
try {{
    startActivity(intent)
}} catch (e: ActivityNotFoundException) {{
    // 처리 가능한 앱이 없을 때 사용자에게 안내 – Notify user when no app can handle
}}
```
- **암시적 인텐트는 항상 예외 가능성을 고려해야 합니다. –** Always consider exceptions for implicit intents.

---

## 🧪 Intent vs Bundle vs NavArgs
| 항목 · Item | Intent | Bundle | NavArgs |
|---|---|---|---|
| 대상 · Target | Activity / Service / Receiver | Fragment / Activity | Compose Nav Graph |
| 타입 안정성 · Type Safety | 낮음/Low | 중간/Medium | 높음/High (Safe Args) |
| 용도 · Usage | 컴포넌트 실행 & 앱 간 통신 – Start components & cross-app | 화면 간 데이터 – Screen-to-screen data | Compose 내 네비게이션 – Navigation in Compose |

---

## 🧭 Flags & Best Practices
- **`FLAG_ACTIVITY_NEW_TASK`: 새 태스크에서 시작. –** Start in a new task.  
- **`FLAG_ACTIVITY_CLEAR_TOP`: 타겟 위 스택 제거. –** Clear activities above target.  
- **`FLAG_ACTIVITY_SINGLE_TOP`: 상단 동일 인스턴스 재사용. –** Reuse top-most instance.

**모범 사례 · Best Practices**
- **암시적 인텐트는 try/catch로 안전 처리. –** Wrap implicit Intents in try/catch.  
- **내부 화면 전환은 명시적 인텐트 사용. –** Use explicit intents for in-app navigation.  
- **`createChooser()`로 사용자 선택 보장. –** Use `createChooser()` to ensure safe user choice.  
- **URI & MIME 타입을 일관되게 관리. –** Keep URI & MIME types consistent.

---

## 📊 전체 시스템 흐름 · Full System Flow
```text
 ┌────────────────────────────┐
 │        Your App            │
 └────────────┬───────────────┘
              │ 인텐트 생성 – Create Intent
              ▼
       Chooser UI (선택창) – Chooser UI
              ▼
     PackageManager 매칭 – PackageManager match
              ▼
     후보 액티비티 목록 – Candidate activities
              ▼
     사용자 선택 – User selects
              ▼
     대상 액티비티 실행 – Target Activity launches
              ▼
     extras 수신 & 처리 – Receives extras & handles
```

---

## ✅ 요약 · Summary
- **Intent는 Android 통신의 핵심이며, 명시적/암시적을 상황에 맞게 선택해야 합니다. –** Intent is core to Android; choose explicit/implicit appropriately.
- **예외 처리와 플래그, 필터 매칭을 이해하면 견고한 UX를 구현할 수 있습니다. –** With exceptions, flags, and filter matching in mind, you can build robust UX.
