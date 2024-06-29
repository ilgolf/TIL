# Coroutine Scope

## 코루틴이란?

Coroutine Scope는 Coroutine의 Life Cycle과 Context를 정의하는 역할을 한다. 이를 통해 coroutine structure concurrency를 달성할 수 있다.

### 주요 개념

1. Life Cycle 관리: 
    - coroutine Scope 내에서 시작된 Coroutine은 scope가 활성화된 동안에만 실행한다.
    - scope가 취소되면 해당 scope내의 모든 coroutine도 자동으로 취소된다.

2. Context 전파: 
    - coroutine scope는 coroutine context를 포함하며, 이 context는 scope 내의 모든 coroutine에 상속된다.
    - 이 context에는 Job, Dispatchers 등의 요소가 포함된다.

3. Coroutine Builder
    - `launch`, `async` 등의 함수는 코루틴 스코프 내에서 코루틴을 시작하는데 사용
    - 이러한 빌더는 coroutine이 scope의 수명 주기에 따라 관리되도록 한다.

## Scope 종류

1. Global Scope 

    - 특정 수명 주기에 종속되지 않는 글로벌 coroutine scope
    - 이 scope 내에서 시작된 coroutine 애플리케이션의 수명 주기 동안 지속 됨

```kotlin
GlobalScope.launch {
    // 장시간 실행되는 코루틴
}
```

2. Coroutine Scope

    - 일반적인 coroutine scope로, custom scope를 만들 때 사용
    - 이 scope는 context를 필요로 하며, 여기에는 Job 또는 Dispatcher 등이 포함될 수 있다.

```kotlin
customScope.launch {
    // custom scope에 속한 coroutine
}
```

3. Main Scope
    - 주로 UI 애플리케이션에서 사용
    - Main Thread에 바인딩 되어 UI 수명 주기에 맞춰 동작

```kotlin
val mainScope = MainScope()

mainScope.launch {
    // main thread에 속한 코루틴
}
```

## Coroutine scope 사용 예제

보통 Coroutine은 적은 메모리로 높은 동시성을 취하기 위해 또는 Structured Cocurrency를 달성하기 위해 사용됩니다. 

```kotlin
import kotlinx.coroutines.*

suspend fun fetchData(): List<String> = coroutineScope {
    val data1 = async { fetchData1() }
    val data2 = async { fetchData2() }
    listOf(data1.await(), data2.await())
}

suspend fun fetchData1(): String {
    delay(1000) // I/O 작업 시뮬레이션
    return "Data from source 1"
}

suspend fun fetchData2(): String {
    delay(2000) // I/O 작업 시뮬레이션
    return "Data from source 2"
}
```

또는 `runblocking`을 사용하여 동기식 코드를 작성할 수 있다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val data = fetchData()
    println(data)
}

suspend fun fetchData(): List<String> = coroutineScope {
    val data1 = async { fetchData1() }
    val data2 = async { fetchData2() }
    listOf(data1.await(), data2.await())
}

suspend fun fetchData1(): String {
    delay(1000) // I/O 작업 시뮬레이션
    return "Data from source 1"
}

suspend fun fetchData2(): String {
    delay(2000) // I/O 작업 시뮬레이션
    return "Data from source 2"
}
```
