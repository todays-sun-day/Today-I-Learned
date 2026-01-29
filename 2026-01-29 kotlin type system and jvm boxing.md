# Kotlin 타입 시스템과 JVM 동작 원리 (Boxing & Memory)
## 🎯 학습 목표
- Kotlin의 자료형이 JVM에서 어떻게 최적화되는지 이해한다.
- `val`, `var`의 차이와 Kotlin의 안전성을 파악한다.
- Bytecode 분석을 통해 '객체 생성 비용'에 대해 알아본다.

## 🔍 고찰: Kotlin의 타입 시스템
### 1. Primitive vs Wrapper (Int vs Int?)
Kotlin은 코드 작성 시에는 모두 객체처럼 보이지만, 컴파일 시점에 최적화를 진행한다.
- 분석 코드
```kotlin
val age: Int = 25            // Non-nullable
val nullableAge: Int? = null // Nullable
```
- Bytecode 분석 (Java Decompiled)
  - `Int`: 자바의 원시 타입인 `int`로 변환됨.
  - `Int?`: 자바의 래퍼 클래스인 `Integer`로 변환됨. (Heap 메모리에 객체 생성, Boxing 발생)
  
  **Why?**: 자바의 원시 타입(`int`)은 `null`을 가질 수 없기 때문에, Kotlin에서 Nullable(`?`)을 허용하려면 반드시 객체 타입인 `Integer`로 변환해야 한다.

### 2. Generic에서의 박싱
- `List<Int>`와 같이 제네릭을 사용하는 경우, 내부의 숫자들은 무조건 `Integer` 객체로 변환된다. 자바의 제네릭은 타입 인자로 원시 타입을 허용하지 않기 때문이다.
- Insight: 안드로이드처럼 리소스가 제한된 환경에서 대량의 숫자 데이터를 다룰 때는 `List<Int>`보다 `IntArray`를 사용하는 것이 메모리 효율적일 수 있다.

### Before & After
- Before
<img width="607" height="169" alt="image" src="https://github.com/user-attachments/assets/25c55468-932b-4841-a640-7e962cc62856" />

- After
<img width="546" height="235" alt="image" src="https://github.com/user-attachments/assets/df21a0c8-8c80-418d-89e6-9ec04c81f71a" />

## 🧩 Algorithm & Bytecode Analysis
### 백준 2908번 - 상수
- 핵심로직: `splitToSequence(" ").map { it.trim().reversed().toIntOrNull() ?: 0 }.maxOrNull()`
- Bytecode 분석:
  - `splitToSequence`: `split`과 달리 `ArrayList`를 즉시 생성하지 않는다.
    대신 `Iterator`를 통해 데이터를 하나씩 꺼내 쓰는 지연 계산 방식을 취하므로 힙 메모리 점유율이 낮다.
  - `maxOrNull`: 전체 리스트를 메모리에 적재하지 않고, 시퀀스를 순회하며 현재까지의 최대값만 변수에 저장한다.
    이 과정에서 중간 컬렉션 생성이 생략되어 GC의 부담을 획기적으로 줄인다.
  - 객체 생성 최소화: `map` 연산이 수행될 때마다 새로운 리스트를 만드는 대신, 요소가 필요할 때만 변환 로직을 실행하므로 잦은 힙 할당을 방지한다.
