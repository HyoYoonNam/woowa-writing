
## Jetpack Compose

![img.png](img.png)

Jetpack Compose는 Android UI 개발을 더 쉽고 빠르게 만들어주는 구조다.
기존의 XML 기반 UI 개발 방식과 달리, 선언형 UI 방식을 사용해 상태에 따라 UI를 동적으로 갱신하도록 한다.
전통적인 View 시스템인 xml 방식과 비교했을 때 훨씬 간결하고 직관적인 코드를 작성할 수 있다.
어떤 점들이 매력적으로 다가올 수 있었을까?

### 매력 포인트
Jetpack Compose의 매력 포인트를 이야기할 때, 선언형 프로그래밍 패러다임에 대하여 설명하곤한다.
단순히 무엇을 그릴지에 집중하는 점이 매력적이다.
직접 View를 관리하고 업데이트하는 작업보다는, 단순히 상태에 맞는 UI를 선언해서 적용한다.
Jetpack Compose의 가장 매력적인 점은 코드가 **간결하고 직관적**이다.
이를 활용하면 버튼 클릭 시 텍스트가 바뀌는 화면을 만드는 것을 몇 줄의 코드로 가능해진다.

```
@Composable
fun SimpleScreen() {
    var text by remember { mutableStateOf("Hello!") }
    
    Button(onClick = { text = "Text" }) {
        Text(text)
    }
}
```

만약 동일한 기능을 XML 기반으로 구현하려면 아래와 같다.
```
// in XML
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:padding="16dp">

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello!"
        android:textSize="24sp"
        android:layout_marginBottom="16dp" />

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Click Me" />
</LinearLayout>

// in Kotlin
import android.os.Bundle
import android.widget.Button
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val textView: TextView = findViewById(R.id.textView)
        val button: Button = findViewById(R.id.button)

        button.setOnClickListener {
            textView.text = "Text"
        }
    }
}
```

물론 데이터바인딩이나 뷰 바인딩과 같은 방식을 활용하면 코드가 훨씬 간결해질 수 있지만, 이를 위한 학습 곡선도 어느정도 존재한다.
또한 개발자는 모든 바인딩에 중복 코드를 추가해야해서 실수하기 쉽다. 
Compose는 모든 UI와 로직이 하나의 함수 안에 통합되어 있어 더 간결하고 유지보수가 용이하다.

### 재사용성

Jetpack Compose의 컴포저블 함수는 레고 블록과 같다. 
한 번 정의된 UI 요소를 다양한 화면에서 재사용이 가능하다.
코드의 중복을 줄이고 컴포넌트 기반 설계가 가능하며, 레고처럼 조립하여 사용할 수 있다.

### 기존 View와의 호환성

XML 코드를 Compose로 UI를 완전히 새롭게 작성할 필요없다.
기존의 View 시스템과 Compose는 서로 호환되며, 이를 활용하면 점진적으로 적용이 가능하다.
기존 프로젝트에서 XML 기반의 UI와 Compose를 함께 사용하는 것이 가능하다.
Compose 학습을 진행하면서 기존의 프로젝트를 모듈별로 Compose로 전환하는 작업을 진행해 보았는데,
점진적 적용을 통해 한 번에 많은 파일을 바꿀 필요가 없었다.
XML 방식과 Compose 방식을 결합한 코드가 되었으며, 쉽고 빠르게 변환이 가능했다.

### 빠른 미리보기

Android Studio에서는 Compose Preview 기능을 제공한다.
이를 활용하면 실시간으로 UI를 확인할 수 있는데, 앱을 빌드하고 디바이스에 설치하는 시간을 줄여주기 때문에 개발 생산성이 크게 증가했다.
빠른 미리보기 및 실시간 업데이트를 제공하며 단순하고 유연하게 개발이 가능해졌다.

### 커스텀 뷰

가장 매력을 느꼈던 부분은 커스텀 뷰를 자유롭게 구성이 가능한 점이었다.
Composable 함수를 활용하여 UI를 구현하였는데, 특정 스타일이나 반복되는 레이아웃을 재사용할 수 있었다.
하나의 CustomButton을 만들어서 재사용한다고 생각했을 때 XML 방식과 차이를 비교해봤다.
Composable 함수에서는 커스텀 뷰를 아래처럼 구상할 수 있다.
```
@Composable
fun CustomButton(text: String, onClick: () -> Unit) {
    Button(
        onClick = onClick,
        colors = ButtonDefaults.buttonColors(backgroundColor = Color.Blue)
    ) {
        Text(
            text = text,
            color = Color.White,
            fontSize = 18.sp
        )
    }
}

@Composable
fun SimpleScreen() {
    var buttonText by remember { mutableStateOf("Click") }
    
    CustomButton(text = buttonText) {
        buttonText = "Button Clicked!"
    }
}
```
text나 클릭 이벤트를 인자로 전달받아 커스텀이 가능하다.
color와 fontSize에 대하여도 파라미터로 추가하여 커스텀이 가능하다.
SimpleScreen이라는 하나의 뷰를 구성하는 함수에서 커스텀 뷰를 사용하였다.

XML 코드로 동작 방식은 아래와 같다.
```
// in style
<resources>
    <style name="CustomButtonStyle">
        <item name="android:background">#0000FF</item>
        <item name="android:textColor">#FFFFFF</item> 
        <item name="android:textSize">18sp</item> 
        <item name="android:padding">16dp</item> 
    </style>
</resources>

// in xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical">
    <Button
        android:id="@+id/customButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Click"
        style="@style/CustomButtonStyle" />
</LinearLayout>

// in Activity
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import android.widget.Button

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val customButton: Button = findViewById(R.id.customButton)
        customButton.setOnClickListener {
            customButton.text = "Button Clicked"
        }
    }
}

```
커스텀 버튼의 스타일을 생성하고, 스타일을 적용한 버튼을 활용하여 액티비티나 프래그먼트에서 활용한다.
MainActivity에서 버튼을 참조한 후 클릭 이벤트를 설정하여 커스텀 버튼을 사용할 수 있다.

Compose에서 커스텀 뷰를 정의하는 것이 직관적이고 간단하며, Kotlin 파일로 UI와 로직을 동시에 처리하였다.
여기서 말하는 로직은 비즈니스 로직이 아닌 UI 동작에 대한 로직이다.
XML 코드에서 만약 background, fontSize 등을 넘겨주려면 추가로 바인딩어댑터를 활용하여 동적으로 제어해야 한다.
Compose를 활용하여 커스텀 뷰를 구현한다면 코드 간결성과 직관성이 뛰어나다는 것을 확인할 수 있다.

## 선언형 패러다임

기존의 안드로이드 UI 시스템은 XML 레이아웃 파일을 사용하여 위젯 트리를 인스턴스화 하고,
이를 통해 사용자 인터페이스를 초기화하였다.
UI 위젯이 독립적으로 내부 상태를 유지하며 getter 및 setter 메서드를 통해 UI 상태를 변경할 수 있었다.
이러한 패러다임을 명령형 패러다임이라 부르며, UI를 직접 조작하는 특징을 가지고 있다.
이는 명령형 접근 방석이며, 객체지향 프로그래밍과 절차지향 프로그래밍 방식이 이를 따르고 있다.
명령형 패러다임에서는 UI 요소가 상태를 자체적으로 관리하기 때문에, 복잡한 UI 구조에서는 상태의 일관성을 유지하는 것이 어렵다.
Jetpack Compose는 선언형 패러다임을 채택하여 무엇을 보여줄 것인가에 집중하였다.

### Stateless

Compose의 위젯은 기본적으로 Stateless한 특성을 가지고 있다.
UI 위젯이 상태를 유지하지 않으며, getter나 setter 메서드를 통해 외부와 상호작용하지 않는다.
UI를 변경하려면 동일한 Composable 함수를 새로운 인수로 호출하여 UI를 갱신한다.
이러한 방식으로 UI 상태와 비즈니스 로직을 분리할 수 있다.

### 큰 차이는 무엇일까

명령형과 선언형 패러다임의 차이에서 상태 관리 방식이 꼭 언급된다.
명령형 패러다임은 각 위젯이 자체 상태를 관리하여 UI를 조작하며, 
선언형 패러다임에서는 UI 상태가 외부로부터 주어져 외부 상태에 따라 갱신된다.
UI 외부에서 상태를 가질 수 있는 ViewModel에서 컴포저블 함수에 필요한 상태를 제공하고, 
상태가 변경되면 Compose에서 이를 감지하여 UI를 업데이트한다.

## Side effects

컴포즈의 편리성에 대해서만 이야기하였는데, 이번에는 부수효과에 대해서 이야기해보려 한다.
부수효과는 선언적 UI 시스템에서 외부 시스템과 상호작용 시 발생할 수 있다.
함수형 프로그래밍에서 말하는 순수 함수의 개념과 대비되는 개념이라고 볼 수 있다.
순수 함수는 같은 입력에 대해 항상 같은 출력을 반환하며 외부에 영향을 주지 않는다.
반면에 부수효과는 함수가 외부 상태에 영향을 미치거나, 외부 상태에 의존할 때 발생한다.
Compose는 기본적으로 선언형 패러다임에 기반을 두고 있지만, 때로는 UI를 구축하는 과정에서 부수효과가 필요할 수도 있다.
올바른 부수효과 관리를 지원한다면, 컴포즈의 장점을 올바르게 활용할 수 있다. 

### 부수효과의 필요성

Side effects는 약물에서 부작용이란 뜻이다.
프로그래밍 관점에서 비슷한 의미이며, 어떤 행위나 현상에 관한 주요한 작용이 아닌 그에 따르는 부차적인 작용을 가리키는 어휘이다.
부수효과는 UI에서 비동기 작업, 외부 시스템과의 상호작용, 상태 변경과 같은 컴포넌트 외부의 시스템과의 상호작용이 필요할 때 유용하다.
예를 들면 네트워크 요청, 파일 저장, 애니메이션, 외부 이벤트 처리 등이 있다.

Jetpack Compose에서는 이러한 부수효과를 관리하기 위한 여러 도구를 제공한다.
부수효과를 안전하고 예층 가능하게 처리하는데 사용되는 Composable 함수들에 대하여 소개하려 한다.

### LaunchedEffect

LaunchedEffect는 Compose의 코루틴을 활용하여, 컴포저블이 처음 실행되거나 재구성될 때 실행할 작업을 정의할 수 있다.
이 함수는 주로 비동기 작업이나 외부에서 제공된 데이터를 처리하는 데 사용된다.

```
LaunchedEffect(key1) {
    // 비동기 작업 실행
}
```

화면이 처음 로드될 때의 네트워크 요청을 보내 데이터를 불러오거나,
특정 키 갑싱 변경될 때만 작업을 재실행하는 등의 작업을 처리할 수 있다.

### rememberUpdatedState

rememberUpdatedState는 특정 값이 변경되더라도 이전의 값에 안전하게 접근할 수 있도록 도와준다.
주로 콜백을 안정적으로 유지하고 싶을 때 유용하게 사용할 수 있다.

```
val updatedValue = rememberUpdatedState(newValue)
LaunchedEffect(key1) {
    // updatedValue는 항상 최신 상태로 유지
}
```

이는 타이머나 비동기 작업 중 최신 상태에 기반한 작업을 해야 할 때, 이전 상태로 인해 발생할 수 있는 버그를 방지한다.

### DisposableEffect

DisposableEffect는 Compose에서 Compose의 생명주기와 관련된 부수효과를 관리하기 위한 함수이다.
특정 컴포저블이 UI에서 제거될 때, 이를 처리하는 정리 작업을 함께 정의할 수 있다.

```
DisposableEffect(Unit) {
    // 생명주기 시작 시 실행할 작업
    onDispose {
        // 컴포저블이 소멸될 때 정리할 작업
    }
}
```

이는 리소스 해제, 리스너 등록 및 해제, 애니메이션 취소 등의 동작에 활용할 수 있다.

### SideEffect

SideEffect는 컴포지션이 완료된 후 호출되는 부수효과를 처리할 때 사용된다.
Compose 트리가 변경된 후 외부 시스템과 상호작용해야 할 경우 유용하다.

```
SideEffect {
    // 컴포지션 완료 후 실행할 코드
}
```

이는 로그를 남기거나, 외부 상태를 갱신하는 작업 등 컴포지션 이후 실행되어야 할 작업에 활용된다.

### productState

productState는 상태를 생성하기 위해 비동기 작업을 수행할 때 사용된다.
이 함수는 Compose에서 상태 기반의 비동기 작업을 처리할 때 유용하며, 특정 값이 비동기로 로드되어야 할 때 사용된다.

```
val result by produceState(initialValue = null) {
    val data = fetchDataFromNetwork()
    value = data
}
```

이는 네트워크 호출이나 데이터베이스와 같은 외부 리소스를 비동기로 호출해 UI 상태로 반영할 때 사용된다.

### rememberCoroutineScope

rememberCoroutineScope는 컴포저블 내에서 rememberCoroutineScope를 기억하고, 이를 통해 코루틴을 실행할 수 있다.
사용자가 특정 이벤트를 발생시켰을 때 코루틴을 통해 작업을 비동기로 처리하고 싶을 때 사용한다.

```
val coroutineScope = rememberCoroutineScope()
Button(onClick = {
    coroutineScope.launch {
        // 비동기 작업
    }
}) {
    Text("Run Async Task")
}
```

이는 버튼 클릭 시 네트워크 요청이나 I/O 작업을 실행하는 경우 사용한다.

### Compose와 부수효과의 차별성

Compose에서 부수효과는 반드시 UI의 선언형 특성을 해치지 않도록 안전하게 처리되어야 한다.
Compose는 상태 관리와 재구성을 기반으로 하기 때문에, 부수효과가 잘못 관리되면 재구성 과정에서 의도치 않은 동작이나 버그가 발생할 수 있다.
이러한 부수효과 관리 도구들은 이런 문제를 방지하고 예측 가능한 방식으로 외부 상태와의 상호작용을 처리할 수 있도록 돕는다.

부수효과를 안전하게 사용하기 위해서는 필요한 경우에만 사용하며 로직을 분리하는 것이 좋다.
불필요한 부수효과는 복잡성을 증가시킬 수 있으며, 예상치 못한 버그를 발생시키는 코드를 작성하기 충분하다.
Compose의 상태 관리와 밀접하게 연결되어 있으므로, 상태가 변경될 때만 부수효과가 실행되도록 설계하는 것이 좋다.
또한 DisposableEffect나 LaunchedEffect와 같은 도구들을 활용하면 생명주기에 고려하여 리소스를 적절히 해제할 수 있다.

## Compose의 계층 구조와 Recomposition

Jetpack Compose는 트리 계층 구조를 따른다.
UI를 선언적으로 구성하는 방식에서 Composable 함수 호출을 통해 UI를 구성하고, 각 Composable 함수는 또 다른 Composable 함수를 호출한다.
이는 트리 구조를 형성하며 안드로이드 XML 레이아웃에서 볼 수 있는 위젯 트리와 유사하지만 몇 가지 차별화된 특성을 가진다.

### 트리 계층 구조 

![img_1.png](img_1.png)

Compose에서는 UI가 트리 구조로 표현되며, 각 Composeable 함수는 트리와 노드로 볼 수 있다.
이 트리는 UI가 재구성될 때 중요한 역할을 하며, 각 자식 요소는 부모에 종속되고 UI 트리의 루트는 앱의 최상위 UI 요소이다.
예를 들면, Column이나 Row(XML에서의 LinearLayout)와 같은 레이아웃 컨테이너는 여러 자식 Composable 요소들을 포함하고,
그 자식들이 다시 그들의 자식을 포함할 수 있다.

### 선언적 UI와 Recomposition

Compose의 핵심은 상태 기반 UI 업데이트이다.
상태가 변경될 때, Compose는 트리 계층 구조를 따라 필요한 부분만 다시 그린다.
이를 Recomposition이라고 부르며, 상태가 변경된 특정 노드와 그 자식들만 다시 그려지기 때문에 성능적으로 효율적이다.
일반적인 트리 계층과 마찬가지로 루트 노드, 내부 노드, 리프 노드로 구분할 수 있다.
- 루트 노드 : 앱의 촤상위에서 UI 트리를 구성하는 시작점이며, setContent 내부에서 첫 Compsoable 함수가 루트 노드가 될 수 있다.
- 내부 노드 : 각 내부 노드는 하나 이상의 자식을 포함할 수 있는 부모 노드이다. 이 노드는 레이아웃이나 컨테이너 역할을 하며, 자식들이 화면에서 어떻게 배치될지 결정한다.
- 리프 노드 : 더 이상 자식을 가지지 않는 UI 요소이다. Text, Button과 같은 단일 요소는 리프 노드로 구성되어 UI 트리 최하단에 위치하게 된다.
```
setContent {
    Column { // Root
        Text("Hello")
        Row { // Inner
            Button(onClick = {}) { // Leaf
                Text("Button 1")
            }
            Button(onClick = {}) {
                Text("Button 2")
            }
        }
    }
}
```

위 코드를 보면 Column이 루트 노드로서 Text와 Row를 자식으로 포함하고 있다.
여기서 Row는 다시 두 개의 Button을 자식으로 가지며 각 버튼은 Text를 포함하는 구조를 가진다.
이에 순서대로 루트 노드, 내부 노드, 리프 노드로 구분할 수 있다.

### Recomposition 과정

Compose의 장점 중 하나는 UI의 특정 부분만 재구성할 수 있다.
상태 변경이 발생했을 때, Compose는 트리 전체를 다시 그리지 않고 변경된 상태와 연관된 Composable 함수만 선택적으로 다시 호출하여 갱신한다.

> 재구성 트리 탐색 : 상태가 변경되면 Compose는 트리의 노드를 탐색하며, 해당 상태와 관련된 노드만 다시 그린다.
> 이를 통해 불필요한 UI 갱신을 방지하고, 성능 최적화를 도모할 수 있다.

### 기존 XML 트리와 차이는?

기존의 안드로이드 XML 기반 UI에서는 레이아웃 파일을 통해 계층 구조를 정의하지만,
이 구조는 고정적이며 런타임에 동적으로 변경하기 어렵다.
반면 Compose에서는 선언형으로 UI를 정의하기 때문에, 상태에 따라 트리 구조가 동적으로 변할 수 있다.
이는 재구성이 일어나는 부분만 다시 그려지므로 효율적이다.

## Recomposition 최적화

Recomposition은 UI가 상태 변경에 따라 다시 그려지는 과정이다.
하지만 때로는 특정 Composable이 Recomposition되는 것을 방지하거나 불필요한 재구성을 줄이고 싶을 때가 있다.
이를 위해 몇가지 기법을 사용할 수 있으며, key를 활용한 remember, rememberUpdateState와 같은 상태 유지 도구를 통해 성능 최적화를 할 수 있다.

### remember

remember는 컴포저블 재구성이 발생하더라도 해당 컴포저블에서 특정 값을 유지할 수 있도록 한다.
이는 무분별한 재구성을 막고 재구성 시 값이 초기화되지 않도록 도와준다.

```
@Composable
fun MyComponent() {
    val value = remember { calculateExpensiveValue() }
    Text("Value: $value")
}
```

재구성이 발생해도 remember에 저장된 value는 유지되며, 값이 다시 계산되지 않는다.

### rememberUpdatedState

rememberUpdatedState는 remember와 비슷하지만, 상태가 변경될 때마다 최신 값을 유지한다.
콜백 함수가 변경되거나 오래 걸리는 작업에서 최신 상태를 참조하는 경우에 사용된다.

```
@Composable
fun MyComponent(onClick: () -> Unit) {
    val updatedOnClick = rememberUpdatedState(onClick)

    Button(onClick = { updatedOnClick.value() }) {
        Text("Click me")
    }
}
```

onClick 값이 업데이트되더라도 전체 컴포저블을 다시 그리지 않고 콜백 함수만 최신 상태를 반영한다.

### key 활용

key는 컴포저블 트리에서 특정 컴포저블을 고유하게 식별하기 위해 사용된다.
재구성 시 컴포저블이 변하지 않도록 하거나, 특정 조건에 따라 새로운 컴포저블을 생성하도록 할 수 있다.

```
@Composable
fun MyList(items: List<String>) {
    Column {
        items.forEach { item ->
            key(item) {
                Text("Item: $item")
            }
        }
    }
}
```

key를 사용하면, 아이템이 변경될 때 고유한 키 값이 변경되는 경우에만 해당 컴포저블을 다시 생성하거나 재구성한다.

### derivedStateOf

derivedStateOf는 복잡한 계산이 자주 일어나지 않도록 상태를 저장하는데 사용된다.
상태가 변경될 때만 다시 계산되므로 불필요한 재구성을 줄일 수 있다.

```
@Composable
fun MyComponent(list: List<Int>) {
    val sortedList by remember {
        derivedStateOf { list.sorted() }
    }
    Text("Sorted List: $sortedList")
}
```

list가 변경되지 않으면, sortedList는 다시 계산되지 않고 재구성 없이 값을 유지한다.

### Immutable 어노테이션

Immutable 어노테이션을 활용하면 불필요한 재구성을 방지할 수 있다.
이는 상태를 불변하게 만들며, 데이터 구조를 Immutable하게 유지하여 상태 여부를 빠르게 파악하고 불필요한 재구성을 피하도록 한다.

```
@Immutable
data class Person(val name: String, val age: Int)
```

### Stable 어노테이션

Stable 어노테이션은 데이터 클래스 또는 객체가 재구성을 일으키지 않도록 힌트를 제공한다.
이 어노테이션을 통해 상태가 안정적임을 표시하고, 불필요한 재구성을 방지할 수 있다.

```
@Stable
class MyClass {
    var value by mutableStateOf(0)
}
```

@Stable로 마킹된 클래스는 그 자체로 안정적인 상태를 유지하며, 내부 상태가 변하지 않으면 재구성이 발생하지 않는다.

## 마무리

Jetpack Compose는 선언형 패러다임을 기반으로 한 UI 프레임워크로, 직관적이고 간결한 UI 구축 도구를 제공한다.
UI와 비즈니스 로직의 분리, 상태 관리와 재구성을 효과적으로 제어한다면 성능을 극대화 할 수 있다.
Compose를 처음 접한 개발자라면 새로운 선언형 패러다임이 낯설게 느껴질 수 있지만, 시간이 지날수록 유연성과 효율성을 체감할 수 있을 것이다.
꾸준히 발전하는 개발 생태계에서 효율적이고 직관적인 UI 개발 경험을 원한다면 Compose 학습에 대하여 강력하게 추천한다.
