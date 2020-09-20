---
title: 새로운 날짜와 시간 API
tags:
  - LocalDate
  - java
categories: JAVA
layout: post
---

# 새로운 날짜와 시간 API

| Class | Description |
| :---: | :--- |
| LocalDate | - 시간을 제외한 날짜를 표현하는 불변 객체 - 어떤 시간대 정보도 포함하지 않음 - 년, 달, 요일 등을 반환하는 메서드 제공 |
| LocalTime | - 시간을 표현하는 객체 |
| LocalDataTime | - LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스 - 날짜와 시간을 모두 표현할 수 있음 |
| Instant | - 기계의 관점에서는 연속된 시간에서 특정 지점을 하나의 큰 수로 표현하는 것이 가장 자연스러운 시간 표현 방법 |
| Duration | - 두 시간 객체 사이의 지속시간을 생성 |
| Period | - 두 날짜 객체 사이의 간격을 생성 |
| TemporalAdjusters | 다양한 날짜와 시간 상황에서 사용할 수 있도록 제공되는 인터페이스 |

## DateTimeFormatter

* 날짜와 시간 관련 포매팅/파싱 클래스
* Thread safe
* DateTImeFormatterBuilder : 복합적인 포매터를 저의해서 좀 더 세부적으로 포매터를 제어

## ZoneId, ZonedDateTime

* 시간대를 제공하는 불변 클래스
* 표준 시간이 같은 지역을 묶어서 시간대를 규정

## 요약

* 자바 8 이전 버전에서 제공하는 기존 Date 및 관련 클래스에는 결함이 존재
* 새로운 날짜와 시간 API에서 날짜와 시간 객체는 모두 불변
* 새로운 API는 사람과 기계가 편리하게 관리 할 수 있도록 두 가지 표현 방식 제공
* 날짜와 시간 객체를 절대적인 방법과 상대적인 방법으로 처리할 수 있음
* TemporalAdjuster를 이용하면 단순히 값을 바꾸는 것 이상의 복잡한 동작을 수행
* 날짜와 시간 객체를 특정 포맷으로 출력하고 파싱하는 포매터를 정의할 수 있음 

## 참고

* [ZonedDateTime](https://www.daleseo.com/java8-zoned-date-time/)

