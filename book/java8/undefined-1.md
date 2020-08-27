---
title: 디폴트 메서드
tags:
  - java
categories: JAVA
layout: post
---

# 디폴트 메서드

## 호환성 문제

* `바이너리 호환성` : 코드 수정 후 에러 없이 기존 바이너리가 실행될 수 있는 상황  _\(ex. interface에 메서드를 추가했을 때 추가된 메서드를 호출하지 않는 한 문제가 발생하지 않는것\)_
* `소스 호환성` : 코드를 고쳐도 기존 프로그램을 성공적으로 재컴파일할 수 있는 것 _\(ex. interface에 메서드를 추가하면 소스 호환성이 아님. 추가 메서드를 구현해야 하기 때문\)_
* `동작 호환성` : 코드를 바꾼 다음에도 같은 입력값이 주어지면 프로그램이 같은 동작을 실행하는 것 _\(ex. interface에 메서드를 추가하더라도 추가된 메서드를 호출할 일은 없으므로 동작 호환성은 유지\)_ 

## 디폴트 메서드

* 인터페이스는 자신을 구현하는 클래스에서 메서드를 구현하지 않을 수 있는 새로운 메서드 시그니처
* 인터페이스를 구현하는 클래스에서 구현하지 않은 메서드는 인터페이스 자체에서 기본으로 제공
* `default`라는 키워드로 시작하여 메서드 바디를 포함
* 인터페이스에 디폴트 메서드를 추가하면 소스 호환성이 유지됨

  ```java
  // Sized를 구현하는 클래스에서 isEmpty를 구현하지 않아도 됨
  public interface Sized {
    int size();
    default boolean isEmpty() {
        return size() == 0;
    }
  }
  ```

## 추상 클래스와 자바8의 인터페이스

* 둘 다 추상 메서드와 바디를 포함하는 메서드를 정의할 수 있음
* 클래스는 하나의 추상 클래스만 상속 가능. 인터페이스는 여러 개 구현 가능
* 추상 클래스는 인스턴스 변수 선언 가능. 인터페이스는 불가능.

## 디폴트 메서드 활용 패턴

1. 선택형 메서드
2. 잘 사용하지 않는 메서드에 기본 구현을 제공하여 빈 구현을 제공하지 않아도 됨

   ```java
   interface Iterator<T> {
         boolean hasNext();
         T next();
         default void remove() {
             throw new UnsupportedOperationException();
         }
   }
   ```

3. 동작 다중 상속
4. 동작을 쉽게 재사용하고 조합 가능

   ```java
   public class ArrayList<E> extends AbstractList<E> 
         implements List<E>, RandowmAccess, Cloneable, 
                       Serializable, Iterable<E>, Collection<E> {
   }
   ```

## 디폴트 메서드를 상속하면서 생기는 충돌 문제를 해결하는 규칙

1. 클래스나 슈퍼클래스에 정의된 메서드가 다른 디폴트 메서드 정의보다 우선함. 이 외의 상황에서는 서브 인터페이스에서 제공하는 디폴트 메서드가 선택됨
2. 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해서 결정해야 함

