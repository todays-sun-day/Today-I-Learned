# 📅 [Level0 | 1주 차] Kotlin 문자열 처리 최적화와 스코프 함수의 효율성 분석

## 🎯 학습 목표
- 문자열 연쇄 호출(`split`, `map`) 시 발생하는 JVM 내부의 컬렉션 생성 비용을 파악한다.
- 스코프 함수(`apply`, `also`)가 `inline`으로 동작하여 오버헤드를 줄이는 원리를 이해한다.
- 정규식(Regex)을 활용해 레이싱카 미션의 `InputView`를 견고하게 설계한다.

---

## 🔍 고찰: 문자열 체이닝의 이면 (The Cost of Chaining)
### 1. `split().map()` 연쇄 호출 분석
코틀린의 가독성 좋은 체이닝 뒤에는 임시 객체 생성이라는 비용이 숨어 있다.

- 실습 코드:
  ```kotlin
  val result = "pobi,woni,jun".split(",").map { it.uppercase() }
  ```

- 디컴파일 코드:
  ```java
  public static final void main() {
      CharSequence var10000 = (CharSequence)"pobi,woni,jun";
      String[] var1 = new String[]{","};
      Iterable $this$map$iv = (Iterable)StringsKt.split$default(var10000, var1, false, 0, 6, (Object)null);
      int $i$f$map = 0;
      Collection destination$iv$iv = (Collection)(new ArrayList(CollectionsKt.collectionSizeOrDefault($this$map$iv, 10)));
      int $i$f$mapTo = 0;

      for(Object item$iv$iv : $this$map$iv) {
         String it = (String)item$iv$iv;
         int var9 = 0;
         String var12 = it.toUpperCase(Locale.ROOT);
         Intrinsics.checkNotNullExpressionValue(var12, "toUpperCase(...)");
         destination$iv$iv.add(var12);
      }

      List result = (List)destination$iv$iv;
   }
  ```

- Bytecode 분석 결과:
  - `split(",")`: 내부적으로 `ArrayList<String>`을 생성하여 분리된 문자열들을 담는다.
  - `.map { ... }`: 위에서 생성된 `ArrayList`를 순회하며 새로운 `ArrayList<String>`을 또 생성하여 반환한다.
- Insight: 데이터의 양이 적은 미션 수준에서는 가독성이 우선이지만, 대량의 데이터를 다룰 때는 `asSequence()`를 사용하여 중간 컬렉션 생성을 방지하는 최적화가 필요함을 인지했다.

### 2. Scope Functions와 Inline
- 테스트 코드
```kotlin
fun main() {
    val person = Person().apply { this.name = "Sun-a"}
}
```

- 디컴파일 코드
```java
public static final void main() {
      Person $this$main_u24lambda_u240 = new Person();
      int var3 = 0;
      $this$main_u24lambda_u240.setName("Sun-a");
   }
```

1. `apply` vs `also`: 언제 무엇을 쓸까?
- `apply`: 객체 생성 직후 "이 객체에 설정을 적용한다"는 맥락. 수신 객체(`this`)를 반환한다.
- `also`: "그리고 또한 이런 작업(로그, 유효성 검사)을 한다"는 부수 효과 맥락. 수신 객체 자체를 반환한다.
2. 왜 성능 저하가 없을까? (Inline)
- 바이트코드 확인 결과, 이 함수들은 `inline` 키워드로 선언되어 있다.
- 실제 컴파일된 자바 코드에서는 별도의 함수 호출 없이 람다 내부의 로직이 호출부로 직접 치환된다.
  즉, 가독성은 챙기면서 함수 호출 오버헤드는 0에 수렴한다.

---

## 🚀 레이싱카(Racing Car) 확장 실습
### 정규식(Regex)을 활용한 `InputView` 설계  
[풀이한 레포 Pull Request 바로가기](https://github.com/woowacourse8/kotlin-racingcar-7/pull/2)
- 문자열 함수를 여러 번 중첩해서 검사하는 것보다, 정규식 하나로 패턴을 정의하는 것이 유지보수와 가독성 측면에서 유리하다.

---

## 🧩 Algorithm 
### 1157번 - 단어 공부
- 핵심 로직
```kotlin
val counts = IntArray(26)
readln().uppercase().forEach { counts[it - 'A']++ }
```
- Bytecode 및 메모리(Heap/Stack) 분석
  - 원시 타입 배열 활용: `IntArray`는 바이트코드 수준에서 자바의 원시 타입 배열인 `int[]`로 컴파일된다.
  - 박싱(Boxing) 제거: `Map<Char, Int>`를 사용할 때 발생하는 `Character` 및 `Integer` 객체 생성이 전혀 없다.
    데이터를 객체로 감싸지 않고 힙(Heap) 영역의 연속된 공간에 원시 값으로 직접 저장한다.
  - 상수 시간 접근: 해시 함수를 거치는 `Map`과 달리, 메모리 주소 계산(`it - 'A'`)을 통해 인덱스로 즉시 접근하므로 오버헤드가 거의 0에 수렴한다.
- 효율성 및 GC 영향
  - 메모리 최적화: 알파벳 개수는 26개로 고정되어 있으므로, 프로그램 실행 동안 단 하나의 배열 객체만 유지하면 된다.
  - GC(Garbage Collection) 부담 경감: 잦은 래퍼 객체 생성이 없으므로 가비지 컬렉터가 치워야 할 쓰레기가 발생하지 않아 실행 속도가 향상된다.  
    
