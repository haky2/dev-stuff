# 스트림으로 데이터 수집

* 스트림에서 Collector 인터페이스 구현한다는 것은 스트림의 요소를 어떤 식으로 도출할지 지정하는 것
* 함수형 API의 장점은 가독성, 유지보수성, 조합성, 재사용성 등
* Collectors 제공하는 메소드의 기능
  * 스트림 요소를 하나의 값으로 리듀싱하고 요약
  * 요소 그룹화
  * 요소 분할

## 리듀싱과 요약 <a id="&#xB9AC;&#xB4C0;&#xC2F1;&#xACFC;-&#xC694;&#xC57D;"></a>

* 컬렉터로 스트림의 모든 항목을 하나의 결과로 합칠 수 있음
* 요약 연산 : 스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 리듀싱 기능 \(summingInt, averagingLong, summarizingDouble…\)

## Collectors에서 제공하는 메서드 <a id="collectors&#xC5D0;&#xC11C;-&#xC81C;&#xACF5;&#xD558;&#xB294;-&#xBA54;&#xC11C;&#xB4DC;"></a>

| 메서드 | 인수 | 설명 |
| :--- | :--- | :--- |
| `counting` | - | 요소 갯수 카운팅 |
| `maxBy` | Comparator | 요소 최대값 반환 |
| `minBy` | Comparator | 요소 최솟값 반환 |
| `summingInt` `summingLong` `summingDouble` | ToIntFunction ToLongFunction ToDoubleFunction | 각 형식으로 데이터 합계 계산 |
| `averagingInt` `averagingLong` `averagingDouble` | ToIntFunction ToLongFunction ToDoubleFunction | 각 형식으로 데이터 평균 계산 |
| `summarizingInt` `summarizingLong` `summarizingDouble` | ToIntFunction ToLongFunction ToDoubleFunction | 각 형식으로 데이터 카운팅, 합계, 평균 등 계산 |
| `joining` | - | 각 객체의 toString을 호출해서 하나의 문자열로 연결하여 반환 |
| `reducing` | 초기값, 변환함수, BinaryOperator |  |

#### \* BinaryOperator : 같은 종류의 두 항목을 하나의 값으로 반환 <a id="-binaryoperator--&#xAC19;&#xC740;-&#xC885;&#xB958;&#xC758;-&#xB450;-&#xD56D;&#xBAA9;&#xC744;-&#xD558;&#xB098;&#xC758;-&#xAC12;&#xC73C;&#xB85C;-&#xBC18;&#xD658;"></a>

### reducing <a id="reducing"></a>

```java

int totalCalories = menu.stream()
                        .collect(reducing(0, Dish::getCalories, (i, j) -> i + j));

// 빈 스트림이 넘어왔을 경우 시작값이 없기 때문에 Optional로 받음
Optional<Dish> mostCalorieDish = menu.stream()
                                     .collect(reducing((d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));

```

* `collect` vs `reduce`
  * collect : 도출하려는 결과를 누적하는 컨테이너로 바꾸는 연산
  * reduce : 두 값을 하나로 도출하는 불변형 연산
  * 여러 스레드가 동시에 같은 데이터 구조체를 고치면 리스트 자체가 망가지므로, 리듀싱 연산을 병렬로 수행할 수 없다. 가변 컨테이너 관련 작업이면서 병렬성을 확보하려면 collect 메서드로 리듀싱을 구현하는 것이 바람직.

## 그룹화 <a id="&#xADF8;&#xB8F9;&#xD654;"></a>

* 분류 함수 : 스트림을 그룹화 하는 함수

## Collectors에서 제공하는 메서드 <a id="collectors&#xC5D0;&#xC11C;-&#xC81C;&#xACF5;&#xD558;&#xB294;-&#xBA54;&#xC11C;&#xB4DC;-1"></a>

| 메서드 | 인수 | 설명 |
| :--- | :--- | :--- |
| `groupingBy` | 분류함수 | 분류함수를 기준으로 그룹화 |
| `collectingAndThen` | 컬렉터, 분류함수 | 적용할 컬렉터와 변환 함수를 인수로 받아 다른 컬렉터를 반환 |
| `mapping` | 변환함수, 컬렉터 | 변환 함수를 이용하여 결과 객체를 컬렉터에 누적하여 반환 |

```java

Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(
    groupingBy(dish -> {
        if (dish.getCalories() <= 400) {
            return CaloricLevel.DIET;
        } else if (dish.getCalories() <= 700) {
            return CaloricLevel.NORMAL;
        } else {
            return CaloricLevel.FAT;
        }
    }));

```

#### \* 다수준 그룹화를 하려면 첫번째 인수로 분류함수, 두번째 인수로 원하는 컬렉터를 넘기면 됨 <a id="-&#xB2E4;&#xC218;&#xC900;-&#xADF8;&#xB8F9;&#xD654;&#xB97C;-&#xD558;&#xB824;&#xBA74;-&#xCCAB;&#xBC88;&#xC9F8;-&#xC778;&#xC218;&#xB85C;-&#xBD84;&#xB958;&#xD568;&#xC218;-&#xB450;&#xBC88;&#xC9F8;-&#xC778;&#xC218;&#xB85C;-&#xC6D0;&#xD558;&#xB294;-&#xCEEC;&#xB809;&#xD130;&#xB97C;-&#xB118;&#xAE30;&#xBA74;-&#xB428;"></a>

## 분할 <a id="&#xBD84;&#xD560;"></a>

* 분할 함수라 불리는 predicate를 분류 함수로 사용하는 그룹화 기능
* boolean을 반환하므로 최대 두 개의 그룹으로 분류됨. true / false

```java

Map<Boolean, List<Dish>> partitionedMenu = menu.stream().collect(partitioningBy(Dish::isVegetarian));

Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType = menu.stream().collect(
    partitioningBy(Dish::isVegetarian, groupingBy(Dish::getType)));

```

