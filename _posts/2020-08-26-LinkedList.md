---
title: "[자료구조] LinkedList 구현하기 (Kotlin)"
excerpt: 
categories:
  - Computer Science
tags:
  - Kotlin
last_modified_at: 2020-08-26T16:00:00+0900
---
### 링크드 리스트(Linked List)

**[소스 참고](https://nanamare.tistory.com/117)

```kotlin
class LinkedList {
    var size: Int = 0
    private var head: Node? = null
    private var tail: Node? = null

    fun addFirst(item: Int) {
        val newNode = Node().apply {
            this.item = item
            this.next = head
        }
        head = newNode
        if (head?.next == null) {
            tail = head
        }
        size++
    }

    fun addLast(item: Int) {
        if (size == 0) {
            addFirst(item)
        } else {
            val newNode = Node().apply {
                this.item = item
            }
            tail?.next = newNode
            tail = newNode
            size++
        }
    }

    fun addAt(index: Int, item: Int) {
        if (index > size - 1) {
            error("invalid index")
        }

        if (index == 0) {
            addFirst(item)
        } else {
            val newNode = Node().apply {
                this.item = item
            }
            val node = find(index)!!
            val temp = node.next
            newNode.next = temp
            node.next = newNode

            if (newNode.next == null) {
                tail = newNode
            }
            size++
        }
    }

    fun removeFirst(): Int {
        val node = head
        head = node?.next
        size--
        return node?.item ?: error("invalid item")
    }

    fun removeAt(index: Int): Int {
        if (index > size - 1) {
            error("invalid position")
        }

        return if (index == 0) {
            removeFirst()
        } else {
            val node = find(index - 1)
            val deletedNode = node?.next
            node?.next = node?.next?.next
            size--
            deletedNode?.item ?: error("invalid item")
        }
    }

    fun find(index: Int): Node? {
        var node = head
        for (i in 0 until index) {
            node = node?.next ?: error("invalid node")
        }
        return node
    }

    fun printAll() {
        var node = head
        while (node != null) {
            println(node.item)
            node = node.next
        }
    }

    inner class Node {
        var next: Node? = null
        var item: Int? = null
    }
}
```

