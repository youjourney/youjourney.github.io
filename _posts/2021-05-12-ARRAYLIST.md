---
title: "코틀린(Kotlin)에서 배열(Array)과 리스트(List)의 비교"
excerpt: "코틀린에서 배열과 리스트의 장단점과 성능을 확인"
categories:
- CS
tags:
- datastructure
- arrays
- list
- arraylist
- mutablelist
last_modified_at: 2021-05-12T22:59:00+0900
---

# 배열(Arrays)와 리스트(List)의 차이

## 배열(Arrays)

### 배열 특징

- 같은 자료형을 가진 값(value: 자료형)들을 하나로 나타낸 것
- 초기화와 동시에 크기(size)가 정해짐
- `메모리 공간에 연속적`으로 저장됨
- 인덱스(index)를 통하여 값(value: 자료형)에 접근
- `정적` 타입



### 배열 장점

* 인덱스(index)를 통한 검색이 용이함
* 연속된 메모리 공간을 사용하여 메모리 관리가 편리함



### 배열 단점

* 배열에서 값을 삭제하더라도 배열의 크기가 줄어들지 않아 메모리가 낭비됨
* 배열은 정적이므로 배열의 크기를 컴파일 이전에 지정해야함
* 배열은 정적이므로 배열의 크기가 정해진 이후 배열의 크기를 재조정 할수 없음





## 리스트(List)

### 리스트 특징

* `순서`가 있는 엘리먼트(element: 자료형)들의 집합, **`시퀀스(Sequence)`** 라고도 부름.
* `불연속적인 메모리 공간`을 점유하여 메모리 관리가 용이
* `동적` 타입
* 포인터를 통해 값에 접근
* 인덱스(index)는 몇 번째 데이터인가 정도의 의미를 갖음



### 리스트의 장점

* 포인터를 사용하여 다음 값의 위치를 가르키고 있으므로 삽입/삭제가 용이
* 동적타입으로 크기가 정해져있지 않음
* 빈틈 없는 엘리먼트(element: 자료형) 적재라는 장점을 취한 자료구조
* 메모리의 재사용이 용이
* 불연속적 메모리공간을 점유하여 메모리 관리가 편리



### 리스트의 단점

* 불연속적인 메모리공간을 사용하여 검색 성능이 좋지 않음
* 포인터를 통하여 다음 값을 가르키므로 배열 대비 추가적인 메모리 공간이 필요

* 빈 요소(element: null)를 허용하지 않음

  



# 코틀린(Kotlin)에서 배열(Arrays)와 리스트(ArrayList)

## 코틀린(Kotlin) 배열(Arrays)

### 코틀린 배열 특징

* 코틀린에서 Arrays는 **Primitive type arrays**임
* [Primitive type arrays](https://kotlinlang.org/docs/basic-types.html#primitive-type-arrays){:target="_blank"} 에는 ByteArray, ShortArray, IntArray, BooleanArray 등이 있음



### 코틀린 배열 내부 코드

* **Kotlin(1.5) Code**

```Kotlin
public class Array<T> {
  public inline constructor(size: Int, init: (Int) -> T)
  public operator fun get(index: Int): T
  public operator fun set(index: Int, value: T): Unit
  public val size: Int
  public operator fun iterator(): Iterator<T>
}
```





## 코틀린(Kotlin) 리스트(List)

### 코틀린 리스트 특징

* 코틀린의 ArrayList는 자바의 ArrayList( java.util.ArrayList\<E> )를 그대로 사용함.

* 자바의 ArrayList는 최상위 클래스인 Object의 배열(Array)로 이루어져있음.
* ArrayList의 초기 생성시, 초기 생성값이 주어지지 않으면 기본값 사이즈 10으로 설정됨.



### 코틀린 배열과 리스트 차이점

|         -         | 배열(Arrays)               | 리스트(List)                                                 |
| :---------------: | -------------------------- | :----------------------------------------------------------- |
|     Iterator      | Iterator을 내부적으로 구현 | Iterable 인터페이스를 구현하여,<br />interface 구현에 의해 만들어짐 |
|       Index       | Index를 통한 접근 가능     | Index를 통한 접근 가능                                       |
| isEmpty, contains | 불가                       | 가능                                                         |
|    add, remove    | 불가                       | 가능                                                         |

* 배열은 Collection에서 제공하는 기능을 사용할 수 없거나 직접 구현해서 사용해야 함



### 코틀린 리스트 인터페이스 내부 코드

* **Kotlin(1.5) Code**

```kotlin
// kotlin/Collections.kt

public interface List<out E> : Collection<E> {
  // Query Operations
	override val size: Int
  override fun isEmpty(): Boolean
  override fun contains(element: @UnsafeVariance E): Boolean
  
  
  // Bulk Operations
  override fun containsAll(elements: Collection<@UnsafeVariance E>): Boolean
  
  
  // Positional Access Operations
  // Returns the elements at the specified index in the list.
  public operator fun get(index: Int): E
  
  
  // Search Operations
  // 리스트 내부의 특정 인덱스 요소를 반환
  // Returns the index of the first occurrence of the specified element in the list, or -1 if the specified element is not contained in the list.
  public fun indexOf(element: @UnsafeVariance E): Int
  
  // 리스트 내부에서 특정한 마지막 요소를 반환, 마지막 요소가 없을 경우 -1를 반환
 	// Returns the index of the last occurrence of the specified element in the list, or -1 if the specified element is not contained in the list.
  public fun lastIndexOf(element: @UnsafeVariance E): Int
  
  
  // List Iterators
  // 이 리스트의 반복기를 반환
  // Returns a list iterator over the elements in this list (in proper sequence).
  public fun listIterator(): ListIterator<E>
  
  // 특정 인덱스에서 시작하는 이 리스트의 반복기를 반환
  // Returns a list iterator over the elements in this list (in proper sequence), starting at the specified index.
  public fun listIterator(index: Int): ListIterator<E>
  
  
  // View
  // fromIndex(포함)부터 toIndex(제외)까지 list를 뷰의 형태로 반환, 반환 후 기존 리스트에 구조적인 변화가 생겼다면, 반환되었던 뷰 형태의 subList는 정의되지 않은 상태로 바뀜
  // Returns a view of portion of this list between the specified fromIndex(inclusive) and toIndex(exclusive). The returned list is backed by this list, so non-structural changes in the returned list are reflected in this list, and vice-versa.
  // Structural changes in the base list make the behavior of the view undefined.
  public fun subList(fromIndex: Int, toIndex: Int): List<E>
  
}

private class ArrayAsCollection<T>(val values: Array<out T>, val isVarargs: Boolean) : Collection<T> {
  override val size: Int get() = values.size
  override fun isEmpty(): Boolean = values.isEmpty()
  override fun contains(element: T): Boolean = values.contains(element)
  override fun containsAll(elements: Collections<T>): Boolean = element.all { contains(it) }
  override fun iterator(): Iterator<T> = values.iterator()
  public fun toArray(): Array<Out Any?> = value.copyToArrayOfAny(isVarargs)
  
  // 읽기만 가능한 비어 있는 리스트 하나를 반환, 반환된 리스트는 serializable.
  // Returns an empty read-only list. The returned list is serializable.
  public fun <T> emptyList(): List<T> = EmptyList
  
  // 주어진 요소들의 리스트를 새로운 읽기만 가능한 하나의 리스트로 반환, 반환된 리스트는 serializable.
  // Returns a new read-only list of given elements. The returned list is serializable.
  public fun <T> listOf(vararg element: T): List<T> = if (element.size > 0) elements.asList() else emptyList()
  public fun <T> listOf(): List<T> = emptyList()
  
  // 하나의 새로운 빈 ArrayList를 MutableList 객체에 담아 반환.
  // Returns an empty new ArrayList.
  public inline fun <T> mutableListOf(): MutableList<T> = ArrayList()
  
  // 하나의 새로운 빈 ArrayList를 반환.
  // Returns an empty new ArrayList.
  public inline fun <T> arrayListOf(): ArrayList<T> = ArrayList()
  
  // 주어진 요소들로 이루어진 ArrayList를 MutableList 객체에 담아 반환.
  // Returns a new MutableList with the given elements.
  public fun <T> mutableListOf(vararg element: T): MutableList<T> = if (elements.size == 0) ArrayList() else ArrayList(ArrayAsCollection(elements, isVarargs = true))
  
  // 주어진 요소들로 이루어진 ArrayList를 반환.
  // Returns a new ArrayList with the given elements.
  public fun <T> arrayListOf(vararg elements: T): ArrayList<T> = if (elements.size == 0) ArrayList() else ArrayList(ArrayAsCollection(elements, isVarargs = true))
  
  // 사이즈가 정해지고 각 요소들은 특정한 init 함수의 호출에 의해 계산된, 하나의 읽기만 가능한 MutableList<T>를 List<T>에 담아 반환
  // init 함수는 첫 번째 요소부터 순차적으로 각 리스트 요소에 대해 호출된다. init 함수는 index가 주어진 list 요소의 값을 반환해야 한다.
  // Creates a new read-only list with the specified size, where each element is calculated by calling the specific init function.
  // The function init is called for each list element sequentially starting from the first one. it should return the value for a list element given its index.
  public inline fun <T> List(size: Int, init: (index: Int) -> T): List<T> = MutableList(size. init)
  
  // 사이즈가 정해지고 각 요소들은 특정한 init 함수의 호출에 의해 계산된, ArrayList<T>를 MutableList<T>에 담아 반환
  // init 함수는 첫 번째 요소부터 순차적으로 각 리스트 요소에 대해 호출된다. init 함수는 index가 주어진 list 요소의 값을 반환해야 한다.
  // The function init is called for each list element sequentially starting from the first one. it should return the value for a list element given its index.
  public inline fun <T> MutableList<size: Int, init: (index: Int) -> T): MutableList<T> {
    val list = ArrayList<T>(size)
    repeat(size) { index -> list.add(init(index)) }
    return list
  }
 
  // 이하 생략
}
```





## 코틀린 리스트(ArrayList) 내부 코드

### 코틀린 코드 부분

* Kotlin(1.5) Code

```kotlin
// TypeAliases.kt
package kotlin.collections

@SinceKotlin("1.1") public actual typealias ArrayList<E> = java.util.ArrayList<E>
```



### 자바 원본 코드

* 자바 코드의 의하면 ArrayList는 내부적으로 Object의 배열로 이루어져 있음
* 배열의 크기를 늘이거나 줄일때, 새로운 크기의 배열을 생성하여 값을 복사하는 작업이 이루어짐

* Java Code

```java
// ArrayList.java

public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
 
  @java.io.Serial
  private static final long serialVersionUID = 9999L
    
  // 초기 기본 용량.
  // Default init capacity.
  private static final int DEFAULT_CAPACITY = 10;
  private static final Object[] EMPTY_ELEMENTDATA = {};
  private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
  private Object[] elementData;
  private int size;
  
  // 하나의 특정된 초기 용량을 갖는 빈 list를 생성.
  // 파라미터는 0부터 Integer.MAX_VALUE까지 가능하며, 음의 값이 들어왔을 때 Exception 발생.
  // Constructs an empty list with the specified initial capacity.
  // Prarams: initialCapacity - the initial capacity of the list.
  // throw IllegalArgumentException - if the specified initial capacity is negative.
  public ArrayList(@Range(from = 0, to = java.lang.Integer.MAX_VALUE) int initalCapacity) {
    if (initialCapacity > 0) {
      this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
      this.elementData = EMPTY_ELEMENTDATA;
    } else {
      throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
    }
  }
  
  // 초기 용량이 기본(10)인 비어있는 list를 생성.
  // Constructs an empty list with an initial capacity of ten.
  public ArrayList() { this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA; }
  
  // 특정 collection의 요소들이 그 collection의 반복기에 의해 순서대로 반환되는 값을 포함하는 list 하나를 생성
  // Constructs a list containing the elements of the specified collection, in the order they are returned by the collection's iterator.
  // Params: c - the collection whose elements are to be placed into this list.
  // Throw: NullPointerException - if the specified collection is null
  public ArrayList(@NotNull@Flow(sourceIsContainer = true, targetIsContainer = true)Collection<? extends E> c) {
    Object[] a = c.toArray();
    if ((size = a.length) != 0) {
      if (c.getClass() == ArrayList.class) {
        elementData = a;
      } else {
        elementData = Arrays.copyOf(a, size, Object[].class);
      }
    } else {
      // replace with empty array.
      elementData = EMPTY_ELEMENTDATA;
    }
  }
  
  // 이 ArrayList 인스턴스의 사이즈를 list의 현재 사이즈로 조정한다.
  // 프로그램은 이 기능을 ArrayList의 저장공간 최소화를 하기 위해 쓸 수 있다.
  // Trims the capacity of this {@code ArrayList} instance to be the list's current size.
  // An application can use this operation to minimize the storage of an {@code ArrayList} instance.
  public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
      elementData = (size == 0) ? EMPTY_ELEMENTDATA : Arrays.copyOf(elementData, size);
    }
  }
  
  // 이 ArrayList 인스턴스의 용량을 키우기 이전에, 인스턴스가 가장 적은 요소들의 수를 특정된 최소용량 파라미터로 유지할 수 있는지 확인한다.
  // Increase the capacity of this ArrayList instance,
  // if necessary, to ensure that it can hold at least the number of elements specified by the minimum capacity argument.
  // Params: minCapacity - the desired minimum capacity.
  public void ensureCapacity(int minCapacity) {
    if (minCapacity > elementData.length && 
        !(elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA && 
          minCapacity <= DEFAULT_CAPACITY)) {
      modCount++;
      grow(minCapacity);
    }
  }
  
  // 용량을 키울때, 가장 적은 요소들의 수를 특정된 최소용량 파라미터로 유지할 수 있는지 확인한다.
  // Increase the capacity to ensure that it can hold ar least the number of elements specified by the minimum capacity argument.
  // Params: minCapacity - the desired minimum capacity.
  // Throws: OutOfMemoryError - if minCapacity is less than zero.
  private Object[] grow(int minCapacity) {
    int oldCapacity = elementDate.length;
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
      int newCapacity = ArraysSupport
        .newLength(oldCapacity, 
                   minCapacity - oldCapacity, 
                   oldCapacity >> 1);
      return elementData = Arrays.copyOf(elementData, newCapacity);
    } else {
      return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
    }
  }
  
  private Object[] grow() { return grow(size + 1); }
  
  // list안의 요소의 갯수를 반환
  // Returns the number of elements in this list.
 	public int size() { return size; }
  
  
  // 생략
  
  
  // 이 도우미 메소드는 add(E)가 35 아래의 바이트사이즈 코드를 유지할 수 있도록 분할한다.
  // 이는 add(E)가 C1-complied loop에서 호출될때 돕는다.
  // This helper method split out from add(E) to keep method bytecode size under 35(the -XX:MaxInlineSize default value),
  // which helps when add(E) is called in a C1-compiled loop.
  private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length) {
      elementData = grow()
    }
    elementData[s] = e;
    size = s + 1;
  }
  
  // 특정한 요소를 list의 마지막에 추가한다.
  // 성공적으로 추가되었을때, true(boolean)을 반환한다.
  // Appends the specified element to the end of this list.
  // Params: e - element to be appended to this list.
  // Returns: true (as specified by Collection.add)
  public boolean add(E e) {
    modCount++;
    add(e, elementData, size);
    return true;
  }
  
  // 특정 요소를 list의 특정 위치에 삽입한다.
  // 만약 특정 위치 오른쪽에 위치한 요소들이 있다면, 오른쪽으로 이동시키고 요소들의 index에 1을 더한다.
  // Inserts the specified element at the specified position in this list.
  // Shifts the element currently at that position (if any) and any subsequent elements to the right (adds one to their indices).
  // Params: index - index at which the specified element is to be inserted, element - element to be inserted
  // Throws: IndexOutOfBoundsException
  public void add(int index, E element) {
    rangeCheckForAdd(index);
    modCount++;
    final int s;
    Object[] elementData;
    if ((s = size) == (elementData = this.elementData).length) {
      elementData = grow();
    }
    System.arraycopy(elementData, 
                     index, 
                     elementData, 
                     index + 1, 
                     s - index);
    elementData[index] = element;
    size = s + 1;
  }
  
  public E remove(int index) {
    Objects.checkIndex(index, size);
    final Object[] es = elementData;

    E oldValue = (E) es[index];
    fastRemove(es, index);
    
    return oldValue;
  }
  
  
  // 생략
  
  
  // (만약 존재한다면) 특정된 요소를 (순차적으로 탐색하면서) 처음 발견한 것을 지운다.
  // 만약 특정 요소를 list에서 포함하고 있지 않으면 list는 변화되지 않는다.
  // Removes the first occurrence of the specified element from this list,
  // if it is present.
  // if the list does not contain the element, it is unchanged.
  // More formally, removes the element with the lowest index i such that Object.equals.(o, get(i))
  // Returns true if list contained the specified element (or equivalently, if this list changed as a result of the call).
  // Params: o - element to be removed from this list, if present.
  // Returns: true if this contained the specified element.
  public boolean remove(Object o) {
    final Object[] es = elementData;
    final int size = this.size;
    int i = 0;
    found: {
      if (o == null) {
        for(; i < size; i++) 
          if (es[i] == null)
            break found;
      } else {
        for(; i < size; i++)
          if (o.equals(es[i]))
            break found;
      }
      return false;
    }
    fastRemove(es, i);
    return true;
  }
  
  // private 제거 메소드로 bounds 체크하는 것을 건너뛰고,
  // 제거된 값은 반환되지 않는다.
  // Private revome method that skips bounds checking and does not return the value removed
  private void fastRemove(Object[] es, int i) {
    modCount++;
    final int newSize;
    if ((newSize = size - 1) > i)
      System.arraycopy(es, i + 1, es, i, newSize - 1);
    es[size = newSize] = null;
  }
  
  
  // 생략
}
```



# 배열과 리스트의 성능 비교

![배열과 리스트 성능 비교]( {{site.baseurl}}/2021-05-12-ARRAYLIST/image1.png )

|    Array    |    List    | ArrayList  | MutableList |
| :---------: | :--------: | :--------: | :---------: |
| 11.380855ms | 3547.518ms | 0.392117ms | 3902.1213ms |

* (환경에 따라 다름) 인덱스로 요소에 접근시 Array보다 List가 약 300배 느림

* Array가 ArrayList보다 약 40배 느린데, 이 것은 아직 이유를 찾지 못함
  ArrayList가 java에서 내부적으로 Object[]으로 구현되어 있어서 그럴 것이라고 추측됨