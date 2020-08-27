---
title: Optional
tags:
  - java
categories: JAVA
layout: post
---

# Optional

## null로 인해 발생하는 문제

* NullPointerException
* 코드 가독성 저하
* 아무런 의미가 없음
* 자바 철학에 위배 \(Java는 모든 포인터를 숨겼으나 null은 예외\)
* 형식 시스템에 구멍을 만듦 \(null이 퍼졌을 때 애초에 null이 어떤 의미로 사용되었는지 알 수 없음\)

## Optional

* 선택형값을 캡슐화하는 클래스
* 값이 없는 상황을 표현
* 모든 null을 Optional로 대치하는 것은 바람직하지 않음
* Optional의 역할은 더 이해하기 쉬운 API를 설계하도록 돕는 것
* 메서드의 시그니처만 보고 선택형값인지 여부를 구별 가능
* 직렬화를 구현하지 않음

## Optional 적용 패턴

| 패턴 | 예시 |
| :--- | :--- |
| 빈 Optional | Optional optCar = Optional.empty\(\); |
| null이 아닌 값으로 Optional | //car가 null이라면 NPE 발생 Optional optCar = Optional.of\(car\); |
| null값으로 Optional | //car가 null이라면 빈 Optional 반환 Optional optCar = Optional.ofNullable\(car\); |

## map, flatmap을 지원하는 Optional

```java
public String getCarInsuranceName(Optional<Person> person) {
    return person.flatMap(Person::getCar)
                 .flatMap(Car::getInsurance)
                 .map(Insurance::getName)
                 .orElse("Unknown");
}
```

## 두 optional 합치기

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(
                            Optional<Person> person, Optional<Car> car) {
    return person.flatMAp(p -> car.map(c -> findCheapestInsurance(p, c)));
}
```

## Optional 클래스의 메서드

| 메서드 | 설명 |
| :--- | :--- |
| empty | 빈 Optional 인스턴스 반환 |
| filter | 값이 존재하며 프레디케이트와 일치하면 값을 포함하는 Optional을 반환하고, 값이 없거나 프레디케이트와 일치하지 않으면 빈 Optional 반환 |
| flatMap | 값이 존재하면 인수로 제공된 함수를 적용한 결과 Optional을 반환하고, 값이 없으면 빈 Optional을 반환 |
| get | 값이 존재하면 Optional이 감싸고 있는 값을 반환하고, 값이 없으면 NoSuchElementException이 발생 |
| ifPresent | 값이 존재하면 지정된 Consumer를 실행하고, 값이 없으면 아무 일도 일어나지 않음 |
| isPresent | 값이 존재하면 true를 반환하고, 값이 없으면 false를 반환 |
| map | 값이 존재하면 제공된 매핑 함수를 적용 |
| of | 값이 존재하면 값을 감싸는 Optional을 반환하고, 값이 null이면 NullPointException을 발생 |
| ofNullable | 값이 존재하면 값을 감싸는 Optional을 반환하고, 값이 null이면 빈 Optional을 반환 |
| ofElse | 값이 존재하면 값을 반환하고, 값이 없으면 디폴트값을 반환 |
| ofElseGet | 값이 존재하면 값을 반환하고, 값이 없으면 Supplier에서 제공하는 값을 반환 |
| ofElseThrow | 값이 존재하면 값을 반환하고, 값이 없으면 Supplier에서 생성한 예외를 발생 |

## 기본형 Optional과 이를 사용하지 말아야 하는 이유

* Optional의 최대 요소 수는 한 개이므로 성능을 개선할 수 없음
* 기본형 특화 Optional은 map, flatMap, filter 등을 지원하지 않음
* 기본형 특화 Optional로 생성한 결과는 다른 일반 Optional과 혼용할 수 없음

## 참고

* [Java Optional 바르게 쓰기](http://homoefficio.github.io/2019/10/03/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0/)

