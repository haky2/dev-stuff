# Stream

* 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소
* 스트림을 이용하면 선언형으로 컬렉션 데이터를 처리 가능 \(데이터를 처리하는 임시 구현 코드 대신 질의로 표현 &gt; 루프와 if 조건문 등의 제어 블록 없이 동작 구현\)
* 멀티 스레드 코드 없이 병렬 처리 가능
* 스트림의 주제는 계산\(필터,소트,맵\), 컬렉션의 주제는 데이터\(자료구조\)
  * fileter : 람다를 인수로 받아 스트림에서 특정 요소를 제외시킴 \(predicate를 인수로 받음\)
  * distinct : 중복을 필터링
  * map : 람다를 이용해서 한 요소를 다른 요소로 변환하거나 정보를 추출
  * flatmap : 하나의 평면화된 스트림을 반환. 스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행
  * limit : 스트림 크기를 축소한다
  * skip : n개 요소를 제외한 스트림을 반환
  * collect : 스트림을 다른 형식으로 변환

```java
import static java.util.stream.Collectors.toList;
List<String> threeHighCaloricDishNames = menu.stream()
                                             .filter(d -> d.getCalories() > 300)
                                             .distinct()
                                             .map(Dish::getName)
                                             .limit(3)
                                             .skip(2)
                                             .collect(toList());
```

* 컬렉션 : 현재 자료구조가 포함하는 모든 값을 메모리에 저장. 모든 요소는 컬렉션에 추가하기 전에 계산되어야 함 \(dvd\)
* 스트림 : 요청할 때만 요소를 계산하는 고정된 자료구조. \(streaming\)
* 스트림은 한번만 소비할 수 있음. 다시 탐색하려면 새로운 스트림을 만들어야 함
* 스트림은 내부 반복을 이용하므로 반복 과정을 신경쓰지 않아도 됨
* 데이터를 언제 계산하느냐가 컬렉션과 스트림의 가장 큰 차이점 산
  * 컬렉션: 모든 요소는 컬렉션에 추가하기 전에 계산
  * 스트림: 요청할 때만 요소를 계산 \(사용자가 요청한 값만 계산하여 추\)
* 중간연산 : filter, map 처럼 스트림을 반환하면서 다른 연산과 연결될 수 있는 연산. 파이프라인 구성 가능. 결과값 없음
* 최종연산 : count, collect 처럼 스트림 파이프라인을 처리해서 스트림이 아닌 결과를 반환하는 연산
* 중간 연산은 최종 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않음\(lazy\)
* 필터링은 프레디케이트와 고유 요소 필터링 지원
  * 프레디케이트 : 불린을 반환하는 함수
  * 고유 요소 : 객체의 hashCode, equals로 결정 \(distinct\)
* 검색과 매칭 메소드 제공
  * anyMatch : 프레디케이트가 주어진 스트림에서 적어도 한 요소와 일치하는지 확인. 불린 반환
  * allMatch : 스트림의 모든 요소가 주어진 프레디케이트와 일치하는지 검사
  * noneMatch : allMAtch와 반대연산 \(일치하는 요소가 없는지 확인\)
  * findAny : 현재 스트림에서 임의의 요소를 반환 \(Optional 클래스로 반환. 값이 없을때 어떻게 처리할 것인지 강제하는 기능 제공. null 익셉션 피하기 위해\)
  * findFirst : 첫 번째 요소 반환 \(병렬 실행에서는 첫 번째 요소를 찾기 어려움\)
* 쇼트서킷 : 표현식에서 하나라도 거짓이라는 결과가 나오면 나머지 표현식의 결과와 상관없이 전체 결과도 거짓이 됨.
* 리듀싱 : 모든 스트림 요소를 처리해서 값으로 도출하는 과정
* 두개의 인수 \(초기값, 오퍼레이터 표현식\)

```java
int sum = numbers.stream().reduce(0, (a,b) -> a+b);
int sum = numbers.stream().reduce(0, Integer::sum);
Optional<Integer> max = numbers.stream().reduce(0, Integer::max);
```

* 기본형 특화 스트림
* 숫자 스트림 : mapToInt, mapToDouble, mapToLong
  * 객체 스트림으로 복원 : boxed
  * 값으로 스트림 만들기 : Stream.of
  * 배열로 스트림 만들기 : Arrays.stream
  * 파일로 스트림 만들기 : Files.lines
  * 함수로 스트림 만들기 : Stream.iterate, Stream.generate

## 참고 <a id="&#xCC38;&#xACE0;"></a>

* [http://soomong.net/blog/2017/07/30/book-java8-in-action/](http://soomong.net/blog/2017/07/30/book-java8-in-action/)
* [https://www.sangkon.com/java8\_study\_part1/](https://www.sangkon.com/java8_study_part1/)

