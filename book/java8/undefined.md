---
title: '리팩토링, 테스팅, 디버깅'
tags:
  - java
categories: JAVA
layout: post
---

# 리팩토링, 테스팅, 디버깅

## 코드 가독성

* 우리가 구현한 코드를 다른 사람이 쉽게 이해하고 유지보수할 수 있게 만드는 것

## 익명 클래스를 람다 표현식으로 리팩토링하기

* 람다 표현식은 익명 클래스보다 코드를 좀 더 간결하게 만듦

  ```java
  // 익명 클래스를 사용한 코드
  Runnable r1 = new Runnable() {
    public void run() {
        System.out.println("Hello");
    }
  }
  // 람다 표현식을 사용한 코드
  Runnable r2 = () -> System.out.println("Hello");
  ```

* 익명 클래스를 람다 표현식으로 변환할때의 주의점 1. this, super 키워드 : 익명 클래스에서의 this는 익명 클래스 자신, 람다식에서의 this는 람다를 감싸는 클래스 \(super도 동일하게 적용\)

  > Similar rule applies for super and this keyword when using inside anonymous inner class and lambda expression. In case of anonymous inner class this keyword refers to local scope and super keyword refers to the anonymous class’s super class. While in case of lambda expression this keyword refers to the object of the enclosing type and super will refer to the enclosing class’s super class.   
  >   
  > [http://www.code2succeed.com/lambdas-vs-anonymous-inner-classes/](http://www.code2succeed.com/lambdas-vs-anonymous-inner-classes/) 2. 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있으나, 람다 표현식은 불가능\(shadow variable\).   
  > 컴파일 오류 발생 3. 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩\(형식 추론\)에 따른 모호함이 초래됨.   
  > 익명 클래스는 인스턴스화할 때 명시적으로 형식이 정해지는 반면 람다식은 콘텍스트에 따라 달라지기 때문
  >
  > ```java
  > // 모호함이 발생할 경우 형변환을 하여 해결
  > doSomething((Task)() -> System.out.println(“Hello”));
  > ```

## 람다 표현식을 메서드 레퍼런스로 리팩토링하기

* 람다 표현식을 별도의 메서드로 추출한 다음에 메서드 레퍼런스를 이용하면 가독성을 높일 수 있음
* comparing, maxBy, summingInt, maximum 같은 정적 헬퍼 메서드와 Collectors API를 활용

  ```java
  // 메서드 레퍼런스 활용
  Map<CaloricLevel, List<Dish>> dishesByCaloricLevel 
    = menu.stream().collect(groupingBy(Dish::getCaloricLevel));
  // 정적 헬퍼 메서드 활용
  inventory.sort(comparing(Apple::getWeight));
  // Collectors API 활용
  menu.stream().collect(summingInt(Dish::getCalories));
  ```

## 명령형 데이터 처리를 스트림으로 리팩토링하기

* 스트림 API는 데이터 처리 파이프라인의 의도를 명확하게 보여줌
* 쇼트서킷과 게으름이라는 최적화와 멀티코어 아키텍처\(병렬처리\)를 활용할 수 있음

## 코드 유연성 개선

* 함수형 인터페이스 적용
  * 조건부 연기 실행
  * 실행 어라운

## 람다로 객체지향 디자인 패턴 리팩토링하기

1. 전략 패턴
2. 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법
3. 알고리즘을 나타내는 `인터페이스`, 인터페이스를 `구현한 클래스`, 전략 객체를 사용하는 `클라이언트`로 구성
4. 전략을 구현하는 새로운 클래스를 구현할 필요 없이 람다 표현식을 직접 전달하면 코드가 간결해짐

   ```java
   Validator numericValidator = new Validator((String s) -> s.matches("[a-z]+"));
   boolean b1 = numericValidator.validate("aaa");
   Validator numericValidator = new Validator((String s) -> s.matches("\\d+"));
   boolean b1 = numericValidator.validate("bbb");
   ```

5. 템플릿 메서드
6. 알고리즘의 개요를 조금 고쳐 유연하게 사용해야 하는 경우 사용하는 기법

   ```java
   // 추상 클래스를 상속받지 않고 직접 람다 표현식을 전달해서 다양한 동작을 추가
   new OnlineBankingLambda().processCustomer(1337, 
             (Customer c) -> System.out.println("Hello " + c.getName()));
   ```

7. 옵저버
8. 어떤 이벤트가 발생했을 때 한 객체\(주체\)가 다른 객체 리스트\(옵저버\)에 자동으로 알림을 보내야 하는 상황에서 사용

   ```java
   // 옵저버를 명시적으로 인스턴스화하지 않고 람다 표현식을 직접 전달해서 실행할 동작을 지정
   f.registerObserver((String tweet)) -> {
         if (tweet != null && tweet.contains("mony")) {
             System.out.println("Breaking news in NY! " + tweet);
         }
   });
   ```

9. 의무 체인
10. 작업처리 객체의 체인을 만들 때 사용.
11. 한 객체가 어떤 작업을 처리한 다음에 다른 객체로 결과를 전달하고, 다른 객체도 해야할 작업을 처리한 다음에 또 다른 객체로 전달하는 식

    ```java
    // 작업 처리
    UnaryOperator<String> headerProcessing =
          (String text) -> "From Raoul, Mario and Alan: " + text;
    UnaryOperator<String> spellCheckerProcessing =
          (String text) -> text.replaceAll("labda", "lambda");
    // 동작 체인으로 두 함수를 조합
    Function<String, String> pipeline = headerProcessing.andThen(spellCheckerProcessing);
    String result2 = pipeline.apply("Aren't labdas really sexy?!!");
    ```

12. 팩토리
13. 인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 사용

    ```java
    // 생성자도 메서드 레퍼런처럼 접근 가능
    Supplier<Product> loanSupplier = Loan::new;
    Loan loan = loanSupplier.get();
    ```

## 람다 테스팅

* 람다 표현식 자체를 테스트하는 것보다는 람다 표현식이 사용되는 메서드의 동작을 테스트하는 것이 바람직

## 람다 디버깅

* 람다 표현식을 사용하면 스택 트레이스를 이해하기 어려움
* 스트림 파이프라인에서 요소를 처리할 때 peek 메서드로 중간값 확인 가능

