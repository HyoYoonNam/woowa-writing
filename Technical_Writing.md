# Flow 는 무엇일까 👀

안드로이드를 공부하다 보면 'Flow'라는 단어를 한 번쯤은 만나게 된다.
구글에 검색해보니 특히 LiveData를 대체할 새로운 대안으로 Flow가 주목받고 있었다. 
그렇다면 Flow란 무엇일까?



코틀린 `Flow` 에 대한 정의를 찾아보니 다음과 같이 적혀있었다.

> An asynchronous data stream that sequentially emits values and completes normally or with an exception.
>

글을 읽어보니 **비동기(asynchronous)** 와  **데이터 스트림(data stream)** 이라는 키워드가 핵심인 것 같다. 
'비동기'라는 단어는 많이 들어봤어도, '데이터 스트림'은 너무 생소했다. 일단 데이터 스트림이 무엇인지부터 알아보자.

## 데이터 스트림 (Data Stream)

데이터 스트림은 **데이터가 순차적으로 흐르는 흐름**을 의미한다. 일반적으로 데이터가 시간에 따라 순차적으로 생산되고 소비된다는 특징을 가지고 있다.


### Producer 와 Consumer

데이터 스트림에서 중요한 역할을 하는 두 가지 구성 요소가 있다. 바로 생산자 (Producer)와 소비자 (Consumer) 이다.

- **생산자**는 데이터를 생성하는 주체로, API 서버, 센서, 데이터베이스 등을 예시로 들 수 있다. 데이터 스트림에 새로운 데이터를 추가하거나 이벤트를 발생시킨다.
- **소비자**는 생산자가 생성한 데이터를 소비하는 주체를 의미한다. 데이터 처리를 수행하는 클라이언트, UI 등이 포함된다.

![image2.png](img%2Fimage2.png)

Flow는 생산자, 중간 연산자, 최종 연산자로 구성된다. 
- **생산자** (Producer)는 Flow를 생성하고 데이터를 방출하는 역할을 한다. 일반적으로 `flowOf`, `flow`, `asFlow`와 같은 함수를 사용한다.


- **중간 연산자** (Intermediary)는 Flow에 대해 변환하거나 필터링을 수행한다. 이 때, 원래의 `Flow`를 변경하지 않고 새로운 `Flow`를 반환한다. 중간 연산자는 `map`, `filter`, `flatMap`, `take`와 같은 함수를 포함한다.


- **최종 연산자** (Consumer)는 Flow의 실행을 완료하고 실제로 데이터를 수집하는 역할을 한다. `collect`, `single`, `toList`와 같은 함수를 사용할 수 있다.

이 외에도 다양한 연산자들은 [공식 홈페이지](https://kotlinlang.org/docs/flow.html)를 참조하여 사용법을 익히는 것을 추천한다.

<br>

### Hot Stream 과 Cold Stream

데이터 스트림은 데이터 생산 및 소비 방식에 따라 Hot 과 Cold 로 구분할 수 있다.
TV 방송과 OTT 서비스를 예시로 Hot 과 Cold의 차이점을 알아보자.

**TV 방송**은 시청자와 관계없이 계속해서 송출된다. 시청자가 있든 없든 정해진 시간표에 맞춰 방송이 진행되며, 시청자가 방송을 시청하기 시작하면 그 시점에서 최신 방송을 보게 된다. 
이처럼 TV 방송은 방송 자체가 이미 진행 중인 상태에서 시청자가 참여하는 방식으로, 시청자가 방송을 보기 시작한 시점부터 데이터가 전달되는 Hot Stream의 특성을 가진다.

반면, **OTT 서비스**(예: 넷플릭스)에서는 사용자가 영화를 선택하고 재생을 시작하는 순간, 그 콘텐츠의 데이터가 생산되기 시작한다. 
즉, 사용자가 요청하는 시점에 맞춰 콘텐츠가 스트리밍되며, 각 사용자는 자신만의 독립적인 스트림을 경험하게 된다. 
이러한 방식은 각 소비자에게 독립적인 데이터를 제공하는 Cold Stream의 특성을 가진다.

Hot과 Cold Stream에 대한 정의를 정리하자면 다음과 같다.

- **Hot Stream** : 데이터를 소비하는 것과 무관하게 원소를 생성한다. 소비자가 구독하기 전에 이미 데이터가 생산되고 있으며, 구독자가 데이터를 요청하면 최신 데이터를 수신하게 된다.
- **Cold Stream** : 데이터 요청이 있을 때만 작업을 수행한다. 소비자가 데이터 스트림에 연결할 때 데이터 생산이 시작된다. 해당 방식은 각각의 소비자에게 독립적인 데이터를 제공한다.

![Flows are Cold..](img/image3.png)

Flow는 Cold Stream 에 속하기 때문에, 소비자가 `collect` 하기 전까지는 시작되지 않는다는 특징을 가지고 있다.

<br>

코드를 보면서 마저 이해해보자.

```kotlin
fun simple(): Flow<Int> = flow { 
    println("Flow started")
    for (i in 1..5) {
        delay(100)
        emit(i)
    }
}
```

`Flow<Int>` 타입을 반환하는 `simple()` 함수는 생성자 역할을 하며, emit 을 통해 `Int` 값을 발행한다.

```kotlin
fun main() = runBlocking<Unit> {
    println("Calling simple function...")
    val flow = simple()
    println("Calling collect...")
    flow.collect { value -> println(value) } 
    println("Calling collect again...")
    flow.collect { value -> println(value) } 
}
```

`main()` 함수에서는 `collect`를 통해 생산자로부터 흐름을 수집하고 있다. 
이때 생산자가 `emit`을 통해 발행하는 각 값들을 받을 수 있다.


호출 결과는 다음과 같다.
```kotlin
Calling simple function...
Calling collect...
Flow started
1
2
3
Calling collect again...
Flow started
1
2
3
```

일반적인 예상과 다르게`simple()` 함수가 호출되는 시점에서 "Flow started" 가 출력되지 않는다.
이는 Cold Stream의 특징을 아주 잘 보여준다. 

"Flow started" 는 `collect` 메서드를 호출할 때 출력 된다.
이를 통해 소비자가 collect 로 데이터를 수집하기 전까지는 데이터를 생산하지 않는다는 것을 알 수 있다.

또한 Flow를 collect 할 때마다 "Flow started" 가 동일하게 출력되는 것을 보아, 매번 Flow가 새로 시작되었음을 의미한다는 것을 알 수 있다. 이러한 특성 덕분에 각 소비자는 Flow를 수집하는 시점에서 새로운 데이터 스트림을 받게 된다.


## 비동기

다음은 **비동기(asynchronous)** 키워드에 초점을 맞춰보려고 한다.

```kotlin
fun simple(): Flow<Int> = flow { 
    for (i in 1..5) {
        delay(100) // 무거운 작업 진행 중 ..
        emit(i) 
    }
}

fun main() = runBlocking<Unit> {
    launch {
        for (k in 1..3) {
            println("I'm not blocked $k")
            delay(100)
        }
    }
    
    val flow = simple()
    flow.collect { value -> println(value) } 
}
```

Flow 빌더 함수 코드를 보면 익숙한 `delay` 메서드가 보인다. `delay` 는 `suspend` 함수이다!

Flow 빌더 내에서 `suspend` 함수를 호출할 수 있다는 의미는 무엇일까?
이는 Flow를 정의하는 과정에서 비동기 작업을 쉽게 수행할 수 있다는 것을 나타낸다. 
실제 사용시에는 네트워크 요청, 또는 다른 장시간 수행되는 작업들을 메인 스레드를 차단하지 않고도 수행할 수 있음을 의미한다.

또한, 코틀린에서는 비동기 작업을 수행하는 함수에 일반적으로 `suspend` 수정자가 붙는다.
그러나 위의 코드에서 `simple` 함수는 `suspend`로 표시되지 않았지만, 비동기 작업을 수행하고 있는`Flow`를 반환하고 있다.
이는 `simple` 함수가 호출될 때, 호출자는 즉시 반환되며 중단되지 않음을 뜻한다. 즉, Flow 자체가 실행되기 전까지는 어떤 비동기 작업도 수행되지 않는다는 것을 의미한다!

Flow의 값은 `collect` 함수를 호출할 때 실제로 수집되며, 이 시점에서 Flow가 비로소 시작된다.
따라서, `collect` 는 반드시 `suspend` 함수로 구현되어야 하며, Flow에서 값을 수집하기 위한 비동기 작업을 수행하게 된다.

호출 결과는 다음과 같다.

```kotlin
I'm not blocked 1
1
I'm not blocked 2
2
I'm not blocked 3
3
```

`simple()`에서 생성된 Flow를 비동기적으로 수집함으로써 UI 스레드를 차단하지 않고도 네트워크 요청이나 데이터 처리와 같은 긴 작업을 효과적으로 관리할 수 있다는 것을 알 수 있다.

### 코루틴 (Coroutine)
Flow가 비동기적으로 유연한 처리가 가능한 이유는 바로 **코루틴**에 있다!
(만약 코틀린 코루틴에 대한 이해가 없다면, 코루틴과 관련된 설명을 읽고 오는 것을 추천한다.)
따라서 Flow는 코루틴의 **구조적 동시성**(structured concurrency)을 기반으로 동작한다.
코루틴 스코프는 코루틴이 실행되는 범위와 환경을 정의한다. 
이를 통해, 같은 스코프 내에서 실행되는 모든 코루틴은 동일한 실행 규칙을 따르며, 코루틴의 실행과 종료가 예측 가능해지고 자원을 효율적으로 관리할 수 있다.

이러한 구조적 동시성은 `Flow`의 비동기 작업들이 상위 스코프의 생명 주기에 맞춰 자동으로 취소되거나 재시작되도록 한다. 
예를 들어, `Flow.collect`를 특정 스코프에서 실행하면, 상위 스코프가 종료될 때 `collect` 작업도 함께 취소된다. 

간단한 코드 예시를 살펴보자.
```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..5) {
        delay(500) // 무거운 작업 중 ..
        emit(i)
    }
}

fun main() = runBlocking {
    val job = launch {
        val task1 = launch {
            simple().collect { value ->
                println("Task 1 collected: $value")
            }
        }

        val task2 = launch {
            simple().collect { value ->
                println("Task 2 collected: $value")
            }
        }

        delay(1500) 
        println("Cancelling job...")
        job.cancelAndJoin()  // ⛔️ 상위 job 취소
        println("Job cancelled")
    }
    job.join() 
}
```
`task1`과 `task2`는 각각 `simple().collect { ... }`를 실행하면서 병렬적으로 데이터를 수집한다. 
1.5초 뒤, `job.cancelAndJoin()`을 통해 상위 job이 취소하게 되면 출력 결과는 다음과 같다.
```
Task 1 collected: 1
Task 2 collected: 1
Task 1 collected: 2
Task 2 collected: 2
Task 1 collected: 3
Task 2 collected: 3
Cancelling job...
Job cancelled
```

상위 job이 취소되면, 그 안에서 실행 중인 하위 task1과 task2 코루틴도 취소되며, Flow의 실행 역시 중단된다.
이처럼 상위 코루틴이 하위 코루틴의 생명 주기를 관리하고 취소할 수 있는 구조적 동시성을 Flow에서도 확인할 수 있다. 이로써 Flow는 메모리 누수를 방지하고, 자원을 효율적으로 관리하며, 예외 처리를 용이하게 하여 안정적인 비동기 작업을 구현할 수 있게 한다.

<br>

## 그래서 Flow는 ... 🥸
지금까지 **데이터 스트림**과 **비동기**라는 키워드를 토대로 Flow에 대한 정의를 알아보았다. 
Flow의 가장 큰 장점은 Cold Stream 특성 덕분에 소비자가 collect를 호출하기 전까지 데이터 생산이 이루어지지 않는다는 점이다.
이는 메모리 사용을 최적화하고, 필요할 때만 데이터를 처리할 수 있게 해준다.
비동기적 특성으로 인해 UI 스레드를 차단하지 않으면서 네트워크 요청이나 긴 작업을 처리할 수 있어, 사용자 경험을 향상시키는 데 기여한다.
또한, 생성자, 중간 연산자, 최종 연산자를 활용하여 복잡한 비동기 데이터 흐름을 간결하고 직관적으로 다룰 수 있다. 이로써 개발자는 데이터의 변환 및 필터링을 손쉽게 수행할 수 있으며, 코루틴의 구조적 동시성을 통해 안정적인 비동기 작업을 구현할 수 있다.

이러한 장점들을 가진 Flow를 활용하여 효과적인 비동기 프로그램을 만들어보자!


### 참고 자료 📚
https://kotlinlang.org/docs/flow.html
https://developer.android.com/kotlin/flow <br>
https://www.youtube.com/watch?v=fSB6_KE95bU
