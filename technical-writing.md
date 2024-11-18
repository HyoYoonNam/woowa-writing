

![image](https://github.com/user-attachments/assets/c7b9ff9b-cfd2-4132-b9f6-14edc38085e7)





# UI 이벤트 처리
[공식 문서](https://developer.android.com/topic/architecture/ui-layer/events?hl=ko)에서 UI 이벤트는 다음과 같이 설명합니다.

> UI 이벤트는 UI 레이어에서 처리해야 하는 동작으로, UI 또는 ViewModel에 의해 처리됩니다. 가장 일반적인 유형의 이벤트는 사용자 이벤트입니다. 사용자는 화면을 탭하거나 제스처를 생성하는 등의 방법으로 앱과 상호작용하여 사용자 이벤트를 발생시킵니다. 그런 다음 UI는 onClick() 리스너와 같은 콜백을 사용하여 이러한 이벤트를 소비합니다.
>

![image (5)](https://github.com/user-attachments/assets/d4f88982-d6b2-418c-ae5d-34c06564da70)



그렇다면 `ViewModel`에서 `UI 이벤트`를 처리하는 다양한 방법에 대해 알아보겠습니다.



# 1. LiveData

![image (6)](https://github.com/user-attachments/assets/55c2a1fd-b38e-45ea-afd8-0602b68e0c13)


일반적으로 LiveData는 데이터가 변경될 때 활성 옵저버에게만 업데이트를 전달합니다. 다만, 옵저버가 비활성 상태에서 활성 상태로 전환될 경우, 마지막으로 활성 상태였던 시점의 값으로 업데이트를 받습니다.
 
 
 
> 💡 즉, `LiveData`는 옵저버가 비활성에서 활성으로 전환될 때 마지막 값을 전달하여 UI를 `최신 상태`로 유지합니다.





그러나 LiveData를 사용하는 것은 문제가 있습니다.

### 문제 발생 시나리오

1. MainActivity에서 Toast를 띄우라는 UI 이벤트가 발생합니다.
2. 이후, DetailActivity로 이동한 후 다시 MainActivity로 돌아옵니다.
3. LiveData를 Observe하고 있던 옵저버는 비활성 상태에서 활성 상태로 전환되며 다시 관찰을 시작합니다.
4. 이때, 1번에서 발생한 Toast 띄우기 이벤트가 다시 관찰되면서 의도하지 않게 Toast가 발생하는 문제가 발생합니다.



# 2. SingleLiveEvent


![image (7)](https://github.com/user-attachments/assets/7576ef09-240b-4135-a005-b3545eed5851)

> 💡 `SingleLiveEvent`는 단발성 이벤트를 처리하기 위해 고안된 개념으로, 이벤트를 한 번만 전파하고 소모할 수 있도록 LiveData와 결합된 이벤트 래퍼입니다.




```kotlin
open class Event<out T>(private val content: T) {

    var hasBeenHandled = false
        private set

    fun getContentIfNotHandled(): T? {
        return if (hasBeenHandled) {
            null
        } else {
            hasBeenHandled = true
            content
        }
    }

    fun peekContent(): T = content
}
```

이 Event Wrapper 개념은 LiveData 공식 문서의 [추천 자료](https://medium.com/androiddevelopers/livedata-with-snackbar-navigation-and-other-events-the-singleliveevent-case-ac2622673150)에 있습니다.



# ViewModels and LiveData: Patterns + AntiPatterns

![image \(8\).png](https://techcourse-storage.s3.ap-northeast-2.amazonaws.com/cc9871e901314a68ad3a36a278c19a60)

LiveData 공식 문서의 [추천 자료](https://medium.com/androiddevelopers/viewmodels-and-livedata-patterns-antipatterns-21efaef74a54)에서 ViewModel과 LiveData를 함께 사용하는 것은 다음과 같다고 설명합니다.

> 이상적으로, ViewModel은 Android에 대한 참조를 가지지 않아야 합니다. 참조를 가지지 않으면 테스트 용이성, 메모리 누수 방지, 모듈화가 향상됩니다. 일반적인 규칙은 ViewModel에 `android.*` 임포트가 없도록 하는 것입니다.(단, `android.arch.*`와 같은 예외는 허용됩니다)
> 



> 💡 즉, `ViewModels`과 `LiveData`를 사용하는 패턴은 안티 패턴이라고 설명합니다.




# 3. Channel

### ViewModel에서 안드로이드 프레임워크의 종속성을 벗어나기 위해 Channel을 도입하여 문제를 개선할 수 있습니다.

- Channel은 Android 프레임워크와 독립적으로 동작하며, **코루틴 기반의 비동기 데이터 전송**을 지원하여 테스트 시 Android 의존성을 줄이고, 모듈화 및 유지보수성을 향상시킵니다.
- Channel은 **라이프사이클에 구애받지 않고** 데이터를 **단일 소비자**에게 전달할 수 있어 이벤트 처리에 더욱 유연하게 대응할 수 있습니다.



> 💡 즉,` Channel`을 도입함으로써 ViewModel에서 `안드로이드 의존성`을 줄이고, `단일 소비자 기반`의 `효율적인 데이터 처리`를 할 수 있습니다.




### 이전 코드

```kotlin
// sealed interface
sealed interface Toast {
    data object ShowToast
    data object ShowXXX
    data object ShowYYY
}

// ViewModel
private val _showToastEvent: MutableLiveData<Event<Toast>> = MutableLiveData(null)
val showToastEvent: LiveData<Event<Toast>> get() = _showToastEvent

// UI
viewModel.showToastEvent.observeEvent(this) { toastEvent ->
    when (toastEvent) {
        is Event.ShowToast -> // TODO
        is Event.ShowXXX -> // TODO
        is Event.ShowYYY -> // TODO
    }
}
```



### 이후 코드

```kotlin
// ViewModel
private val _showToastEvent = Channel<Toast>()
val showToastEvent = _showToastEvent.receiveAsFlow() 

// UI
lifecycleScope.launch {
    viewModel.showToastEvent.collect { toastEvent ->
        when (toastEvent) {
            is Event.ShowToast -> // TODO
            is Event.ShowXXX -> // TODO
            is Event.ShowYYY -> // TODO
        }
    }
}
```

기존의 SingleLiveEvent를 Channel로 변경하고, observe 대신 collect하는 방식으로 수정하면 됩니다. 이제 UI에서는 하나의 showToastEvent를 collect하여 Toast 유형에 맞게 처리하는 방식으로 간단히 구현할 수 있습니다.



하지만 Channel만을 사용하는 것도 문제가 있습니다.

### 문제 발생 시나리오

1. ViewModel에서 서버와 통신하면서 위치 데이터를 주기적으로 `emit`합니다.
2. UI에서는 위치 데이터가 변경되는것을 감지하고 있다가 변경될 때마다 화면에 새로 그리게 됩니다.
3. 이때, 홈 버튼을 눌러 앱을 백그라운드로 내린다면, 위치 데이터를 계속해서 감지하며 화면을 새로 그리게 되는 문제가 발생합니다.

> 💡즉, 사용자가 UI를 보고 있지 않을 때도 데이터를 `observe`하고 있어 `메모리 누수`가 발생합니다.



### 해결 방안

Lifecycle에서 `repeatOnLifecycle()` 이라는 함수를 사용하면 됩니다.



### repeatOnLifecycle()

`repeatOnLifecycle()`은 `Lifecycle` 상태에 맞춰 코루틴을 자동으로 관리하는 기능을 제공합니다. 이 함수는 지정된 `Lifecycle.State`(보통 `STARTED`나 `RESUMED`)에 도달하면 코루틴을 실행하고, 해당 상태에서 벗어나면 자동으로 중단합니다. 이러한 자동 관리를 통해 개발자는 코루틴의 시작과 중지를 일일이 처리할 필요가 없게 됩니다.



### **이후 코드**

```kotlin
// UI
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.showToastEvent.collect { toastEvent ->
            when (toastEvent) {
                is Event.ShowToast -> // TODO
                is Event.ShowXXX -> // TODO
                 is Event.ShowYYY -> // TODO
            }
        }
    }
}
```



그러나, Channel은 여러 개의 구독자에게 동일한 이벤트를 수신하지 못한다는 단점이 있습니다.



# 4. SharedFlow

- SharedFlow는 **코루틴 기반의 Flow**를 사용하여 **여러 구독자에게 데이터를 동시에** 전달할 수 있습니다.
- SharedFlow는 **브로드캐스트 방식**으로 여러 구독자가 동일한 데이터를 받아 처리할 수 있습니다.




> 💡 즉, SharedFlow는 `복수의 구독자`에게 데이터를 `브로드캐스트 방식`으로 전달하며, 라이프사이클에 의존하지 않는 이벤트 처리가 가능합니다.




### **이후 코드**

```kotlin

// ViewModel
private val _showToastEvent = MutableSharedFlow<Toast>()
val showToastEvent = _showToastEvent.asSharedFlow() 

// UI
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.showToastEvent.collect { toastEvent ->
            when (toastEvent) {
                is Event.ShowToast -> // TODO
                is Event.ShowXXX -> // TODO
                is Event.ShowYYY -> // TODO
            }
        }
    }
}

```

기존의 Channel에서 SharedFlow로만 수정하면 됩니다.



그러나 SharedFlow만 사용하는 것도 문제가 있습니다.

### 문제 발생 시나리오

목록에서 item을 선택하고 서버의 응답에 따라 상세 화면으로 이동하는 로직이라고 가정해봅시다.

1. 목록에서 특정 item을 선택합니다.
2. 서버에서 Item에 대한 상태 체크가 끝나기전에 홈 버튼을 눌러 앱을 백그라운드로 내립니다.
3. 이때, 상세 화면으로 이동하라는 이벤트를 `emit`해도 `onStop()` 상태이기 때문에 이벤트를 받지 못하는 문제가 발생합니다.






> 💡 즉, event를 observe하고 있는 곳이 아무데도 없다면, 해당 event는 `유실`된다는 것입니다.



# 5. EventFlow

EventFlow는 이벤트가 발생했을 때 이를 캐시한 후, 해당 이벤트가 소비(consume)되었는지 여부에 따라 새로운 옵저버가 구독할 때 이벤트를 전달할지를 결정하는 구조입니다.


> 💡 즉, 소비되지 않은 이벤트를 `캐시`하고 있다가 `소비`하는 형태입니다.


```kotlin
interface EventFlow<out T> : Flow<T> {

    companion object {

        const val DEFAULT_REPLAY: Int = 3
    }
}

interface MutableEventFlow<T> : EventFlow<T>, FlowCollector<T>

@Suppress("FunctionName")
fun <T> MutableEventFlow(
    replay: Int = EventFlow.DEFAULT_REPLAY
): MutableEventFlow<T> = EventFlowImpl(replay)

fun <T> MutableEventFlow<T>.asEventFlow(): EventFlow<T> = ReadOnlyEventFlow(this)

private class ReadOnlyEventFlow<T>(flow: EventFlow<T>) : EventFlow<T> by flow

private class EventFlowImpl<T>(
    replay: Int
) : MutableEventFlow<T> {

    private val flow: MutableSharedFlow<EventFlowSlot<T>> = MutableSharedFlow(replay = replay)

    @InternalCoroutinesApi
    override suspend fun collect(collector: FlowCollector<T>) = flow
        .collect { slot ->
            if (!slot.markConsumed()) {
                collector.emit(slot.value)
            }
        }

    override suspend fun emit(value: T) {
        flow.emit(EventFlowSlot(value))
    }
}

private class EventFlowSlot<T>(val value: T) {

    private val consumed: AtomicBoolean = AtomicBoolean(false)

    fun markConsumed(): Boolean = consumed.getAndSet(true)
}

```

그러나 EventFlow만 사용하는 것도 문제가 있습니다.

### 문제 발생 시나리오

이벤트 객체가 있고 이를 AFragment, BFragment에서 collect하고 있다고 가정해봅시다.

1. 이벤트가 `emit`되면 AFragment, BFragment에서 collect 됩니다. 
(근소한 차이로 AFragment에서 먼저 collect 되었다고 가정)
2. 이때 AFragment에서 이벤트의 comsumed는 true가 됩니다.
3. 그 이후 BFragment에서 이벤트가 collect되어야하지만, 이벤트는 이미 comsumed 되었기 때문에 collect되지 않는 문제가 발생합니다.



> 💡 즉, 여러 구독자에게 데이터를 동시에 전달하는 SharedFlow의 장점이 사라집니다.



# 6. EventFlow + HashMap

EventFlow + HashMap은 이벤트가 발생했을 때 이를 캐시하고, 이벤트의 소비 여부에 따라 새로 구독하는 옵저버에게 이벤트를 전파할지 여부를 결정하는 구조입니다.

> 💡 즉, 소비되지 않은 `이벤트`를 `캐시`하고 있다가, `새로운 옵저버`가 `구독`할 때 해당 이벤트를 `전달`하는 형태입니다.


```kotlin
private class EventFlowImpl<T>(
    replay: Int
) : MutableEventFlow<T> {

    private val flow: MutableSharedFlow<EventFlowSlot<T>> = MutableSharedFlow(replay = replay)

    private val slotStore: ArrayDeque<Slot<EventFlowSlot<T>>> = ArrayDeque()

    @InternalCoroutinesApi
    override suspend fun collect(collector: FlowCollector<T>) = flow
        .collect { slot ->

            val slotKey = collector.javaClass.name + slot

            if(isContainKey(slotKey)) {
                if(slotStore.size > MAX_CACHE_EVENT_SIZE) slotStore.removeFirst()
                slotStore.addLast(Slot(slotKey, EventFlowSlot(slot.value)))
            }

            val slotValue = slotStore.find { it.key == slotKey }?.value ?: slot

            if (slotValue.markConsumed().not()) {
                collector.emit(slotValue.value)
            }
        }

    override suspend fun emit(value: T) {
        flow.emit(EventFlowSlot(value))
    }

    fun isContainKey(findKey: String): Boolean {
        return slotStore.find { it.key == findKey } == null
    }
}

private data class Slot<T>(
    val key: String,
    val value: T
)

```


### HashMap의 역할

이 구조에서 `HashMap`은 각 이벤트와 해당 이벤트를 소비하는 옵저버의 상태를 관리하는 데 사용됩니다.

- `HashMap`의 키는 현재 `collect`하고 있는 옵저버의 이름과 해당 슬롯의 `toString()` 값을 결합하여 생성됩니다. 이를 통해 각 옵저버를 고유하게 식별할 수 있으며, 어떤 옵저버가 어떤 이벤트를 수신할 자격이 있는지를 명확히 알 수 있습니다.
- `HashMap`의 값은 이벤트와 동일한 값을 가지는 새로운 이벤트가 저장됩니다. 이 구조 덕분에 새로운 옵저버가 구독할 때, 이전에 발생한 이벤트를 적절히 전달받을 수 있습니다.



# 정리하면..
- **LiveData**는 이벤트 발생 후 구독자가 활성화되면 가장 최근 값을 재전달하기 때문에 `단발성 이벤트 처리`에는 `적합`하지 `않`습니다.
- **SingleLiveEvent**는 `한 번만 이벤트`를 전송할 수 있지만, `안티 패턴`입니다.
- **Channel**은 `단일 소비자`에게 이벤트를 효율적으로 `전달`할 수 있지만, `여러 소비자`에게 데이터를 `전달`할 수 `없`습니다.
- **SharedFlow**는 `여러 소비자`에게 브로드캐스트 방식으로 데이터를 `전달`할 수 있지만, 데이터 `유실`이 발생할 수 있습니다.
- **EventFlow**는 소비되지 않은 이벤트를 `캐시`하여 `새로운 옵저버`에게 `전달`할 수 있지만, `복수 소비자 환경`에서는 한계가 있습니다.
- **EventFlow + HashMap**은 캐시된 이벤트를 새로운 옵저버에게 전달하면서, `복수 소비자 환경`에서도 이벤트 관리가 가능하지만, 코드가 `복잡`할 수 있습니다.



# 결론적으로..
> 💡 이벤트 처리는 보통 한 곳에서 이루어지므로, 코드가 간결하고 이해하기 쉬운 `Channel`을 사용하는 것이 적합 할 것 같다고 생각합니다. 그 외에 특별한 요구 사항이 있을 때는 `EventFlow + HashMap`을 사용하는 것이 좋을 것 같습니다.




# 참고문헌

https://developer.android.com/topic/architecture/ui-layer/events

https://developer.android.com/topic/libraries/architecture/livedata

https://medium.com/androiddevelopers/livedata-with-snackbar-navigation-and-other-events-the-singleliveevent-case-ac2622673150

https://medium.com/androiddevelopers/viewmodels-and-livedata-patterns-antipatterns-21efaef74a54

https://medium.com/prnd/mvvm의-viewmodel에서-이벤트를-처리하는-방법-6가지-31bb183a88ce

https://jinukeu.hashnode.dev/android-channel-vs-sharedflow
