---
title: "Understanding Lambdas with Receiver in Kotlin"
excerpt: "A clear explanation of Kotlin's 'lambda with receiver' syntax, how it differs from normal lambdas, and why it's powerful in DSLs and Compose."
categories:
  - Development
tags:
  - Android
  - Compose
  - DSL
  - Jetpack Compose
  - Kotlin
  - Lambda
  - Receiver
last_modified_at: 2025-11-10T00:00:00+0900
---
## 📘 Introduction

Kotlin’s **lambda with receiver** is one of its most elegant and powerful language features. It allows a lambda function to behave as if it were an extension function — meaning you can access the receiver object’s members directly, without referring to it explicitly.

This concept powers many of Kotlin’s expressive APIs, like ``, ``, and even **Jetpack Compose**’s layout scopes.

---

## 💡 Basic Concept

A **lambda with receiver** looks like this:

```kotlin
val builder: StringBuilder.() -> Unit = {
    append("Hello, ")
    append("World!")
}
```

Here:

- `StringBuilder.() -> Unit` means this lambda can be called **on** a `StringBuilder`.
- Inside the lambda, you can call `append()` directly — as if you were inside the `StringBuilder` class.

You can invoke it like this:

```kotlin
val result = StringBuilder().builder()
println(result.toString()) // Hello, World!
```

---

## 🧩 Normal Lambda vs Lambda with Receiver

| Type                     | Declaration                | Access inside Lambda         | Example                        |
| ------------------------ | -------------------------- | ---------------------------- | ------------------------------ |
| **Normal Lambda**        | `(StringBuilder) -> Unit`  | Must use parameter reference | `{ sb -> sb.append("Hello") }` |
| **Lambda with Receiver** | `StringBuilder.() -> Unit` | Access members directly      | `{ append("Hello") }`          |

In other words:

- **Normal lambda** → external control (`it` or parameter name).
- **Lambda with receiver** → internal control (implicit `this`).

---

## 🏗️ How It Works Internally

When you write:

```kotlin
val block: T.() -> R
```

It’s essentially syntactic sugar for:

```kotlin
val block: (T) -> R  // A regular lambda taking T as a parameter
```

But with the ``** bound to **`` inside the lambda body.

---

## 🧠 Real-World Examples

### 1. Kotlin Standard Library

```kotlin
val list = buildList {
    add("Apple")
    add("Banana")
    add("Cherry")
}
```

- `buildList {}` takes a `MutableList<T>.() -> Unit` lambda.
- You can call `add()` directly because the receiver is the list itself.

### 2. Jetpack Compose Example

```kotlin
@Composable
fun Row(content: @Composable RowScope.() -> Unit)
```

Here, `RowScope.() -> Unit` is a **lambda with receiver**, meaning everything inside the `Row` block (like `Text()` or `Button()`) can directly access the `RowScope`’s properties and modifiers.

```kotlin
Row {
    Text("Hello")
    Button(onClick = { }) {
        Text("Click me")
    }
}
```

---

## 🧱 Why Use Lambdas with Receiver?

- ✅ **Cleaner syntax** – no need to reference the object repeatedly.
- ✅ **Improved readability** – perfect for DSL (Domain Specific Language) design.
- ✅ **Safer APIs** – restricts what’s accessible inside the block.

This is why libraries like Compose, Anko, and Ktor use it to create **declarative DSLs** that read like natural language.

---

## 🧭 Summary

| Concept        | Description                                              |
| -------------- | -------------------------------------------------------- |
| **Definition** | A lambda that operates with an implicit `this` receiver. |
| **Syntax**     | `ReceiverType.() -> ReturnType`                          |
| **Benefit**    | Cleaner, DSL-friendly syntax.                            |
| **Usage**      | Kotlin builders, Jetpack Compose, custom DSLs.           |

