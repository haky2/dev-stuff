# 람다 표현식

* 람다표현식 : 메서드로 전달할 수 있는 익명 함수를 단순화한 것
* 파라미터 리스트, 화살표, 람다의 바디로 구성
* \(parameters\) -&gt; expression 표현식
* \(parameters\) -&gt; { statements; } 구문
* 함수형 인터페이스라는 문맥에서 람다 표현식을 사용
* 함수형 인터페이스 : 정확히 하나의 추상 메서드를 지정하는 인터페이스 \(Comparator, Runnable …\)
* 람다표현식 &gt; 전체 표현식을 함수형 인터페이스의 인스턴스로 취급
* @FunctionalInterface
* Predicate, Consumer, Function …
* 람다 캡처링 : 자유변수를 활용 가능 \(final로 선언 됐거나 이와 같은 내용으로 사용 가능한 변수만\)
* 람다가 정의된 메서드의 지역 변수의 값은 바꿀 수 없다 \(클로저와의 차이\)
* 메서드 레퍼런스 : 특정 메서드만을 호출하는 람다의 축약형
* 메서드명 앞에 구분자\(::\)를 붙이는 방식으로 활용
* ClassName::Method
* 생성자 레퍼런스 : ClassName::new
* 자바 컴파일러는 람다 표현식이 사용된 콘텍스트를 활용해서 람다의 파라미터 형식을 추로한다 \(형식추론 » 파라미터 타입 제거 가능\)
* 실행 어라운드 패턴 : 코드 중간에 실행해야 하는 메서드에 꼭 필요한 코드\)

## 참고 <a id="&#xCC38;&#xACE0;"></a>

* [http://soomong.net/blog/2017/07/30/book-java8-in-action/](http://soomong.net/blog/2017/07/30/book-java8-in-action/)
* [https://www.sangkon.com/java8\_study\_part1/](https://www.sangkon.com/java8_study_part1/)
* [Functional Interface](https://cornswrold.tistory.com/306?category=755361)

