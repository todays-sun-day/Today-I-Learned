# 📅 Kotlin 기초: 식(Expression)과 데이터 클래스

## 🎯 학습 목표
- 코틀린의 '식(Expression)' 중심 설계가 자바와 어떻게 다른지 이해한다.
- `data class`의 `copy()` 메서드를 통해 불변성을 유지하는 방법을 익힌다.

---

## 🔍 고찰: 문(Statement)이 아닌 식(Expression)
- 발견: 코틀린에서 `if`는 값을 반환하는 '식'이다.
- 코드 비교:
  ```kotlin
  // Java 스타일 (문)
  var result = ""
  if (a > b) result = "A" else result = "B"

  // Kotlin 스타일 (식)
  val result = if (a > b) "A" else "B"
  ```
- 이해: `if`가 값을 반환하므로 결과를 바로 `val`에 대입할 수 있다. 덕분에 불변성을 유지하기가 훨씬 쉬워진다.
  
---

## 🛠 바이트코드 분석: data class
- 실험: `data class` 선언 시 생성되는 메서드 확인
- 분석 결과:
  - `copy()`: 내부적으로 `new User(...)`를 호출하여 새로운 객체를 반환함을 확인. 원본 데이터의 오염을 방지한다.
  - `toString()`, `equals()`: 수동으로 작성할 필요 없이 컴파일러가 바이트코드 레벨에서 자동으로 생성해 준다.
- 결론: 데이터 전달 객체(DTO)를 만들 때 `data class`를 쓰면 가독성과 안전성을 동시에 챙길 수 있다.

- 코틀린 코드
  <img width="649" height="146" alt="image" src="https://github.com/user-attachments/assets/33b6d8a8-c20e-404e-8b73-452d43759a87" />

- 바이트 코드
  - `copy()`
    
    <img width="527" height="158" alt="image" src="https://github.com/user-attachments/assets/a2b6a138-27a2-4355-af45-95ebd9004df6" />
  - `toString()`
 
    <img width="830" height="75" alt="image" src="https://github.com/user-attachments/assets/bd1efd4c-3c5d-44e2-9918-af68fd3863dd" />
  - `equals()`
 
    <img width="519" height="400" alt="image" src="https://github.com/user-attachments/assets/85a3234b-0285-47fb-ad0e-55dc417dec2e" />

---

## 🧩 Algorithm: 최소, 최대 - 백준 10818
- 구현 코드:
  ```kotlin
  fun main() {
    val size = readln().toInt()
    val input = readln().split(" ").map { it.toInt() }
    val intArray = IntArray(size)

    for (i in 0..<size) {
        intArray[i] = input[i]
    }

    print("${intArray.min()} ${intArray.max()}")
  }
  ```
- 엔지니어링 이해
  - 박싱(Boxing) 방지: `List<Int>` 대신 `IntArray`를 사용하여 JVM 레벨의 원시 타입 배열(`int[]`)로 컴파일되게 함으로써 메모리 효율을 극대화했다.
  - 최신 문법 활용: `0..<size` 범위를 사용하여 가독성을 높였다.
  - 비용 인지: `split(" ")`이 매번 새로운 `List<String>`을 생성한다는 점을 인지하고, 성능과 가독성 사이의 트레이드오프를 고려했다.
