# 운동도, 객체지향도 몸풀기가 기본! 객체지향 생활 체조에 대해 알아보자.

운동을 잘하고 싶다면 어떻게 해야 할까요? 꾸준한 연습이 필요합니다. 그리고 그것만큼 중요한 건 단연 몸을 풀기 위한 `체조`입니다.
객체지향도 마찬가지입니다. 꾸준한 연습은 물론 객체지향적 사고를 위한 몸풀기를 통해 웜웍을 해야 합니다. 객체지향 몸풀기, `객체지향 생활 체조`를 소개해 보겠습니다.

## 객체지향 생활 체조란?
**객체지향 생활 체조**는 ‘소트웍스 앤솔러지'라는 책에서 처음 제시한, 객체지향 프로그래밍을 잘하기 위한 9가지 원칙입니다. 객체지향 프로그래밍의 본질을 더 잘 따라갈 수 있도록 유도하고, 가독성이 높으면서 유연하고 유지보수가 쉬운 코드를 작성하기 위한 일종의 가이드라인이라고 볼 수 있습니다.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeb83FBy2qPwy1Un5o5GBHZCAXR9xK8T0uyi2eEkoCIBZz9mJe6nzOqEdTflMLZ255aHW4Dk9ntcZ8fPC0WfHYCdyfDtE2BOtPqpa_QsFk7L7nttlODJuaiGTSarYjDTQMk_bQwXwaFg8-PwBve4JgwUzD-?key=O_FxyUbZ_0aKjkAwvR6zMQ)
(출처 : 소트웍스 앤솔러지 : 소프트웨어 기술과 혁신에 관한 에세이, 마틴 파울러 지음, 정지웅 역)

## 객체지향 생활 체조 원칙이 주는 이점
앞서 소개한것처럼 객체지향 생활 체조 원칙을 잘 준수한다면 여러 이점을 누릴 수 있습니다.

### 1. 코드 가독성 향상
객체지향 생활 체조의 여러 원칙은 가독성 좋은 코드를 작성하도록 유도합니다. 
예를 들어, `한 메서드에서 들여쓰기를 한 단계만 사용하고, else를 피하는 등`의 원칙은 논리 흐름이 명확한 코드를 작성하도록 유도합니다.

### 2. 객체 응집도 증가
객체지향 생활 체조는 작성되는 클래스와 메서드가 각각 한 가지의 책임만 가지도록 유도합니다. 이를 통해 클래스나 메서드가 특정 한 가지 기능만을 담당하도록 하여 객체 간 응집도가 높은 코드를 작성하도록 합니다. 
예를 들어, `3개 이상의 인스턴스 변수를 가지지 않는다`라는 원칙은 개발자가 객체에 너무 많은 책임을 넣지 않도록 유도합니다.

### 3. 테스트 용이성 증가
객체지향 생활 체초는 각 메서드가 단일 책임을 가지도록 유도합니다. 이렇게 메서드의 책임이 작아질수록 테스트 용이성이 높아집니다.
예를 들어, `한 메서드에 들여쓰기를 한 단계만 사용하기`라는 원칙은 복잡한 로직을 여러 작은 메서드로 분리하여 작성하도록 유도합니다.
이렇게 분리된 메서드들은 각각에 대한 단위 테스트(Unit) 작성이 매우 쉬워집니다.

### 4. 변경 용이성
객체지향 생활 체조는 객체를 최대한 작게 유지하고, 각 메서드가 `단일 책임 원칙(SRP)`을 따르는 방향으로 개발자를 유도합니다.
이렇게 작성된 코드는 주변 객체에 영향을 덜 주기 때문에 변경이 매우 쉬워집니다.

### 5. 확장성 증가
객체지향 생활 체조는 확장성을 고려하여 코드를 설계하는 데 도움을 줍니다. 
예를 들어, `한 줄에 점을 하나만 찍는다`라는 원칙은 다른 객체에 대한 의존성이 적은 코드를 작성하도록 유도합니다.
이렇게 작성된 코드는 변경이 쉽고, 기능 확장에도 매우 유리한 구조를 가지게 됩니다.

---
# 객체지향 생활 체조 원칙 살펴보기
그럼 이제 9가지의 원착들을 하나씩 살펴보겠습니다. 각 원칙들에 대해 다음과 같은 내용들을 살펴보겠습니다.
1. 원칙의 목적
2. 원칙이 적용되지 않았을 때 문제점
3. 원칙을 적용하였을 때 개선되는점
4. 주의점

## 1. 한 메서드에는 오직 한 단계의 들여쓰기만 한다.
### 목적
`if`, `for`, `while`, `try-catch`문을 사용하다 보면 블록으로 인해 코드의 들여쓰기 단계가 증가합니다. 이 문법들을 남용해 들여쓰기의 깊이가 늘어날수록 코드는 읽기 힘들어집니다.
그리고 무엇보다 들여쓰기가 깊다는 건 해당 메서드가 여러 역할을 수행할 확률이 높다는 걸 의미하기도 합니다.
원칙의 내용대로 한 단계의 들여쓰기를 유지하려 한다면 복잡한 메서드를 여러 작은 메서드로 쪼개야 합니다. 그리고 이는 자연스럽게 각각의 메서드가 작은 책임만을 가지도록 유도합니다.

이 원칙은 개발자가 메서드를 잘개 나누도록 유도함으로써 **단일 책임 원칙(SRP)**을 준수하는 코드를 작성하도록 하는 것이 목적입니다.

### 원칙이 적용되지 않았을 때 문제점
```java
public int calculate(final int[] numbers) {
    int sum = 0;

    for (int number : numbers) {    // 첫 번째 들여쓰기
        if (number > 0) {    // 두 번째 들여쓰기  
            sum += number;
        }
    }

    return sum;
}
```
위 코드는 하나의 메서드에 중첩된 여러 제어문이 존재합니다. 때문에 코드를 읽고 이해하는데 어느 정도 시간이 소요됩니다. 
또 한 `숫자를 순회하며 합을 관리하는 책임`과 `숫자가 양수인지 판단하고 더하는 책임` 등 여러 역할과 책임을 수행하여 테스트 코드 작성에도 두 책임을 모두 고려해 작성해야 합니다.

### 원칙을 적용하여 개선
```java
public int calculateSum(final int[] numbers) {
    return sumPositiveNumbers(numbers);
}

private int sumPositiveNumbers(final int[] numbers) {
    int sum = 0;

    for (int number : numbers) {
        sum = addIfPositive(sum, number);  // 들여쓰기 한 단계만 적용
    }

    return sum;
}

private int addIfPositive(final int sum, final int number) {
    // 별도의 메서드로 분리하여 조건 로직 단순화
    if (number <= 0) {
        return sum;
    }
    
    return sum + number;
}
```
원칙대로 메서드가 하나의 들여쓰기를 가지도록 개선하였습니다.

덕분에 이전보다 코드를 읽는 게 더 쉬워졌습니다. 그리고 `특정 정수가 양수인지 판단하고 합산한다`라는 책임이 `순회한다`라는 책임과 별개의 메서드 역할로 분리되어 테스트를 작성하기에도 매우 편해졌습니다.
그리고 이 역할은 다른 메서드에서도 쉽게 재활용할 수 있게 되었습니다.

참고로 위 코드는 다음과 같이 `람다 & 스트림`을 사용해 개선할 수도 있습니다.
```java
public int calculateSum(final int[] numbers) {
    return sumPositiveNumbers(numbers);
}

private int sumPositiveNumbers(final int[] numbers) {
    return Arrays.stream(numbers)
                 .filter(number -> number > 0)  // 양수인 경우만 필터링
                 .sum();  // 양수들의 합 계산
}
```

### 주의할 점
메서드가 너무 간단하여 분리할 필요가 없는 경우. 예를 들어, `한두 줄로 끝나는 간단한 조건문`을 모두 분리하면 코드가 오히려 더 복잡해질 수 있습니다. 
메서드를 분리할 때는 이로인해 코드의 의도가 명확해지는지가를 판단 기준으로 삼아야합니다. **단순히 들여쓰기 줄이는 것에만 집착**해서는 안 됩니다.

## 2. else 예약어를 쓰지 않는다.
### 목적
`else`문은 대부분의 상황에서 복잡하고 의도를 명확하게 파악하기 어려운 코드 흐름을 만듭니다. 
이 원칙은 `else`문을 지양 함으로써 논리 흐름이 쉽게 이해되는 코드를 작성하도록 유도합니다.

### 원칙이 적용되지 않았을 때 문제점
```java
public String getGrade(final int score) {
    if (score >= 90) {
        return "A";
    } else if (score >= 80) {
        return "B";
    } else if (score >= 70) {
        return "C";
    } else {
        return "D";
    }
}
```
위 코드는 `else-if`문과 `else`문이 반복적으로 사용되어 코드의 논리 흐름을 쉽게 파악하기 어렵습니다.

### 원칙을 적용하여 개선
```java
public String getGrade(final int score) {
    if (score >= 90) {
        return "A";
    }

    if (score >= 80) {
        return "B";
    }

    if (score >= 70) {
        return "C";
    }

    return "D";
}
```

원칙에 따라 `else`문을 모두 제거하여 각 조건을 명확하게 표현하도록 하였습니다. 덕분에 코드의 논리 흐름이 단순해졌습니다.
코드를 읽는 사람은 개선 전 코드보다 더 수월하게 조건문을 따라갈 수 있으며, 특정 조건에 대한 변경이 필요할 때 코드를 수정하기 용이해졌습니다.

### 주의할 점
복잡한 조건 분기가 많은 상황이라면 오히려 `else`문을 제거한 코드가 더 복잡할 수 있습니다.
상황에 따라 `else`문을 적절히 활용하거나, **전략 패턴**과 같은 디자인 패턴의 사용을 고려하는 것이 더 좋은 선택지가 될 수 있습니다.

## 3. 모든 원시값과 문자열을 포장한다.
### 목적
원시값 타입의 변수에는 특별한 의미(행위)를 담지 못하며, 여러 로직에 공유되는 과정에서 의도지 않은 값 변경 등의 실수가 발생할 수 있습니다.
이 원칙은 원시값을 도메인 개념에 맞는 객체로 포장하여 의미를 명확히 하고, 개발자의 실수를 방지하는 것을 목적으로 합니다.

또 한, 값에 대한 검증을 포장된 객체에 위임할 수 있어 응집도와 테스트 용이성을 높이는 효과도 얻을 수 있습니다.

### 원칙이 적용되지 않았을 때 문제점
```java
public class User {

    private final String name;
    private final int age;

    public User(final String name, final int age) {
        this.name = name;
        this.age = age;
    }
}
```

위 코드는 변수로 원시값 객체를 사용하고 있습니다. 때문에 이름과 나이에 대한 별도의 검증 로직을 `User`객체 내부에 작성해 줘야 합니다.
이로인해 `User` 객체의 코드양과 함께 테스트 범위가 커졌습니다. 

또한 `age`와 `name`이라는 변수 이름을 제외하고는 의미를 명확하게 전달할 방법이 없어 메서드 외부에서 값을 사용하는 개발자가 원시값 타입만 보고 실수할 가능성이 있습니다.

### 원칙을 적용하여 개선
```java
public class User {

    private final UserName name;
    private final Age age;

    public User(final UserName name, final Age age) {
        this.name = name;
        this.age = age;
    }
}

public class UserName {

    private final String value;

    public UserName(final String value) {
        if (value == null || value.isBlank()) {
            throw new IllegalArgumentException("사용자 이름은 null 혹은 공백일 수 없습니다.");
        }

        this.value = value;
    }
}

public class Age {

    private final int value;

    public Age(final int value) {
        if (value <= 0) {
            throw new IllegalArgumentException("사용자 나이는 음수일 수 없습니다.");
        }

        this.value = value;
    }
}
```

원칙에 따라 이름과 나이를 별도의 값 객체로 포장하였습니다. 
이제 포장된 각 객체가 자신의 검증을 수행합니다. 덕분에 테스트 범위를 각 객체로 쪼갤 수 있어 테스트 작성이 편해졌습니다.

그리고 더 이상 변수 이름만으로 해당 값의 의미를 판단하지 않기 때문에 컴파일 혹은 IDE의 도움을 받아 잘못된 값이 입력되는 걸 방지할 수 있습니다.

### 주의할 점
하나의 값에 하나의 객체가 생기기에 관리할 클래스가 늘어나고 그만큼 관리 복잡도가 증가한다는 단점이 있습니다.

중요한 **도메인 개념**(ex: 사용자 이름, 이메일 주소 등)에 관련된 값은 적극적으로 포장하여 응집도와 테스트 용이성을 높이는 게 좋습니다.
반대로 특정한 책임이 딱히 없이 단순 수치 계산에 사용되는 값(ex: PI)들은 오히려 관리할 클래스의 개수만 늘어나니 굳이 포장하기보다는 계산 책임을 가진 객체에 상수로 관리하는 게 좋습니다.

무조건 원칙을 따르기 위해 값을 포장하는 게 아닌, 별도의 책임을 가진 존재로 재창조하는 것에 대해 늘 먼저 고민하는 것이 가장 중요합니다.

## 4. 한 줄에 하나의 점만 찍는다.
### 목적
이 원칙은 **디미터 법칙**을 준수하도록 유도하는 것이 목적입니다.
디미터 법칙은 다른 객체가 어떠한 데이터를 가졌는지 속사정을 몰라야 하는 것을 지향합니다.
이 원칙을 `Tell, Don't Ask, 묻지 말고 시켜라!`라고도 부릅니다. 요지는 서버 객체가 행위를 하는 구체적 과정을 클라이언트 객체가 어떻게 하는지 관심 가질 필요 없이 결과만 가져오면 된다는 것을 의미합니다.

이 원칙은 **객체 간의 결합도를 낮추고, 캡슐화를 강화하는 코드**를 작성하도록 유도합니다. 만약 메서드 내부에 `.`이 많이 발생하는 로직이 존재한다면 해당 객체가 여러 객체의 세부 사항을 너무 많이 알고 있다는 신호일 수 있습니다.
이는 객체 간 응집도가 떨어지고 결합도가 높을 가능성이 높을 수 있다는 걸 암사합니다. 즉 캡슐화가 깨진 코드일 수 있습니다.

한 줄에 하나의 `.`만 찍게 함으로써 객체의 내부 사정을 과하게 간섭하지 않도록 유도하는 것이 이 원칙의 목적이라고 할 수 있습니다.

### 원칙이 적용되지 않았을 때 문제점
```java
public class Order {

    private final Customer customer;

    public Order(final Customer customer) {
        this.customer = customer;
    }

    public String getCustomerStreetName() {
        return customer.getAddress().getStreet().getName();  // 여러 단계의 객체 접근
    }
}

class Customer {

    private final Address address;

    public Address getAddress() {
        return address;
    }
}

class Address {

    private final Street street;

    public Street getStreet() {
        return street;
    }
}

class Street {

    private final String name;

    public String getName() {
        return name;
    }
}
```

위 코드는 `.`이 과하게 사용되는 로직이 존재합니다. `Order` 클래스가 `Customer`, `Address`, `Street` 클래스의 세부 구조에 대한 지식을 모두 가지고 있는 거 같습니다.

지금과 같은 구조에서 `Address` 클래스의 구조가 바뀐다면, `Order` 클래스도 수정해야 합니다.
예를 들어, `Address` 클래스가 더 이상 `Street`클래스를 포함하지 않게 되면 `Order` 클래스가 깨지게 됩니다.

한 코드가 여러 객체의 내부를 탐색하므로 객체 간 결합도와 의존성이 커지고 있는 상황입니다.

### 원칙을 적용하여 개선
```java
public class Order {

    private final Customer customer;

    public Order(final Customer customer) {
        this.customer = customer;
    }

    public String getCustomerStreetName() {
        return customer.getStreetName();  // Customer에게 메시지를 보내서 정보 요청
    }
}

class Customer {

    private final Address address;

    public Customer(final Address address) {
        this.address = address;
    }

    public String getStreetName() {
        return address.getStreetName();  // 내부 구조를 노출하지 않고 필요한 정보만 제공
    }
}

class Address {

    private final Street street;

    public Address(final Street street) {
        this.street = street;
    }

    public String getStreetName() {
        return street.getName();  // Address가 내부적으로 처리
    }
}

class Street {

    private final String name;

    public Street(final String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

`목적` 단락에서 소개드린 **디미터 법칙**을 코드에 적용하였습니다. 이제 세부 객체에 직접 접근하지 않고 서버 객체에만 요청을 보내고 있습니다.

`Order` 클래스는 이제 `Customer` 클래스의 구조에 대해서만 알고 있으며, `Address`나 `Street`의 내부 구조를 알 필요가 없어졌습니다.
덕분에 객체 간의 결합도가 낮아졌습니다. `Address`나 `Street` 클래스의 구조가 아무리 변경돼도 `Customer` 클래스만 수정하면 그만입니다. 유지보수성이 높아졌기에 가능한 현상입니다.

각 클래스에 내부 상황이 외부에 노출되지 않고 있기 때문에 **코드의 캡슐화가 강화되었다**고 표현할 수 있겠습니다.

### 주의할 점
너무 단순한 데이터 클래스(DTO, VO) 간 상호작요에서 무조건적인 원칙 준수를 강행하면 오히려 코드의 복잡도가 높아질 수 있습니다.
캡슐화를 준수하는 코드는 그만큼 여러 고민을 만들기에 빠르게 개발해야 하는 작은 프로젝트에서는 진행 속도를 더디게 만들 수 있습니다.

디미터 법칙을 적용할 때는 **어떤 객체가 핵심적인 역할을 수행하는지**를 파악하고, 해당 객체의 책임부터 명확히 위임하도록 시작해 보는 것이 좋습니다.

## 5. 줄여 쓰지 않는다. (축약 금지)
### 목적
변수명, 메서드명, 클래스명 등을 과도하게 줄여 작성하면 의미가 불분명해지고, 코드를 읽는 사람이 그 의도를 쉽게 이해할 수 없게 돼버립니다.
이 원칙은 축약된 이름 대신, 명확하고 읽기 쉬운 이름을 사용하여 코드의 가독성을 높이는 것이 목적입니다.

줄임말이나 약어를 피하고, 구체적이고 직관적인 이름을 사용해야 합니다.

### 원칙이 적용되지 않았을 때 문제점
```java
public class Emp {

    private final String nm;  // 직원의 이름
    private final int sal;    // 직원의 급여

    public Emp(final String nm, final int sal) {
        this.nm = nm;
        this.sal = sal;
    }

    public String getNm() {
        return nm;
    }

    public int getSal() {
        return sal;
    }
}
```
위 코드에 사용된 `Emp`, `nm`, `sal` 변수명은 너무 과하게 축약되어 있습니다. 때문에 변수명만으로 의미를 파악하기 어렵습니다.
만약 다른 개발자가 이 코드를 유지보수 해야 한다면 코드의 의미를 빨리 해석하지 못해 시간이 오래 걸리거나, 이전에 코드를 작성한 사람에게 추가 설명을 계속 요구하는 등 개발 생산성이 크게 떨어질 수 있습니다.

### 원칙을 적용하여 개선
```java
public class Employee {

    private final String name;  // 명확한 이름 사용
    private final int salary;

    public Employee(final String name, final int salary) {
        this.name = name;
        this.salary = salary;
    }

    public String getName() {
        return name;
    }

    public int getSalary() {
        return salary;
    }
}
```

축약된 이름 대신 `Employee`, `name`, `salary`와 같이 의미가 명확한 이름으로 변경하였습니다. 
덕분에 변수명만으로 해당 코드가 무엇을 의미하는지 명확히 이해할 수 있게 되었습니다. 불필요한 주석을 추가하거나 설명해야 하는 비용도 아낄 수 있습니다.

### 주의할 점
통용적으로 사용되는 약어는 굳이 풀어쓰지 않아도 괜찮습니다. 개발자들 사이에서 흔히 쓰이는 약어들은 이미 널리 통용되고 쉽게 이해할 수 있기 때문에, 오히려 풀어서 사용하면 코드가 불필요하게 길어지고 가독성이 떨어질 수 있습니다. (ex: DB, URL, SSL, 등)

변수나 메서드 이름은 팀 컨벤션으로 정하는 게 가장 좋습니다. 작명하기 전 팀 내 코딩 컨벤션이나 표준 명명 규칙을 확인하고, 널리 사용되는 약어는 컨벤션이 허락하는 선에서 그대로 사용하되, 모호하거나 생소한 약어는 명확한 이름으로 변경합니다.

## 6. 모든 엔티티를 작게 유지한다.
### 목적
클래스나 메서드의 코드양이 길어진다면 여러 책임을 동시에 가지고 있다는 신호일 수 있습니다.
위에서 계속 언급했듯 여러 책임을 하나의 객체가 가진다면 `단일 책임 원칙(SRP)`을 위반합니다. 객체 간 응집도가 떨어져 유지보수가 어려운 코드가 될 가능성이 높습니다.

이 원칙은 클래스와 메서드를 작게 나누어 `단일 책임 원칙(SRP)`을 준수하는 코드를 작성하도록 유도하는 게 목적입니다. `단일 책임 원칙(SRP)`을 준수하는 코드는 유지보수성이 매우 높아집니다.

### 원칙이 적용되지 않았을 때 문제점
```java
public class OrderProcessor {

    public void processOrder(final String orderDetails) {
        validateOrder(orderDetails);
        saveOrder(orderDetails);
        sendConfirmationEmail(orderDetails);
    }

    private void validateOrder(final String orderDetails) {
        // 주문 검증 로직
    }

    private void saveOrder(final String orderDetails) {
        // 주문 저장 로직
    }

    private void sendConfirmationEmail(final String orderDetails) {
        // 이메일 전송 로직
    }
}
```

`OrderProcessor` 클래스는 검증, 저장, 이메일 전송 등 많은 기능을 담당하고 있습니다. 이는 곧 여러 책임을 동시에 가지고 있다는 의미입니다.
이로 인해 클래스의 역할은 모호해지고, 하나의 메서드를 수정할 때 다른 메서드에 그 파급효과가 퍼질 가능성이 매우 높아집니다.
또 한 여러 책임에 대한 테스트를 작성해야 하므로 `OrderProcessor`에 대한 테스트 코드 작성이 매우 까다로워집니다.

### 원칙을 적용하여 개선
```java
public class OrderProcessor {

    private final OrderValidator validator;
    private final OrderRepository repository;
    private final EmailService emailService;

    public OrderProcessor(final OrderValidator validator, final OrderRepository repository, final EmailService emailService) {
        this.validator = validator;
        this.repository = repository;
        this.emailService = emailService;
    }

    public void processOrder(final String orderDetails) {
        if (validator.isValid(orderDetails)) {
            repository.save(orderDetails);
            emailService.sendConfirmationEmail(orderDetails);
        }
    }
}

class OrderValidator {
    public boolean isValid(final String orderDetails) {
        // 주문 검증 로직
        return true;
    }
}

class OrderRepository {
    public void save(final String orderDetails) {
        // 주문 저장 로직
    }
}

class EmailService {
    public void sendConfirmationEmail(final String orderDetails) {
        // 이메일 전송 로직
    }
}
```

기존 `OrderProcessor`의 책임을 각각 `OrderValidator`, `OrderRepository`, `EmailService`겍체들에 위임하였습니다.
`OrderProcessor` 클래스는 이제 각 클래스를 조합하여 전체 주문 프로세스를 관리하는 역할만 수행합니다. 

개별 로직을 나눈 클래스들을 활용해서 프로세스를 관리하기에 객체 간 응집도가 높아졌습니다.
또한 디미터 법칙에 따라 세부 작업을 담당 객체에 요청하기에 작업의 세부 방식이 변경되어도 다른 클라이언트 객체들은 변경의 파급 효과를 신경 쓸 필요가 없어졌습니다. 객체 간 결합도가 낮아진 것입니다.

개선된 코드는 기존 코드보다 새로운 기능을 추가하기 쉬우며 각 기능에 대한 테스트를 독립적으로 작성해 수행할 수 있게 되었습니다.

### 주의할 점
너무 과도하게 클래스나 메서드를 분리하면 그만큼 관리할 엔티티가 늘어나고, 코드를 읽는 사람의 입장에서 탐색해야 할 엔티티 개수가 굉장히 많아지므로 오히려 코드를 읽기 어렵게 만들 수 있습니다.

만약 특정 객체에 대한 테스트를 작성하려 하는데, 가지고 있는 기능이 많아 테스트 작성이 까다롭다면 엔티티를 분리하라는 신호일 수 있습니다.

## 7. 3개 이상의 인스턴스 변수를 가진 클래스를 사용하지 않는다.
### 목적
하나의 클래스에 너무 많은 인스턴스 변수가 존재한다면, 그 클래스는 여러 책임을 동시에 수행하고 있을 가능성이 높습니다.
이는 **단일 책임 원칙(SRP)**을 위반하고 있을 가능성이 높습니다. 또한 클래스가 너무 많은 데이터를 다루기 때문에 코드가 복잡해질 수 있습니다.

이 원칙은 인스턴스 변수의 개수를 제한하여 각 클래스가 **명확한 책임**을 가져 응집도를 높이고 결합도를 낮추도록 유도하는 것이 목적입니다.

### 원칙이 적용되지 않았을 때 문제점
```java
public class Order {

    private final String productName;
    private final int productPrice;
    private final String customerName;
    private final String customerEmail;

    public Order(final String productName, final int productPrice, final String customerName, final String customerEmail) {
        this.productName = productName;
        this.productPrice = productPrice;
        this.customerName = customerName;
        this.customerEmail = customerEmail;
    }

    public String getProductName() {
        return productName;
    }

    public int getProductPrice() {
        return productPrice;
    }

    public String getCustomerName() {
        return customerName;
    }

    public String getCustomerEmail() {
        return customerEmail;
    }
}
```

`Order` 4개의 인스턴스 변수를 가지고 있습니다. 여러 종류의 데이터를 동시에 다루고 있는 것입니다. 
이로 인해 클래스는 여러 책임을 수행하게 되어 어떤 목적을 수행하는 건지 명확하지 않습니다. 이는 코드의 유지보수성을 떨어뜨립니다.

만약 고객 정보나 제품 정보가 변경된다면 `Order` 클래스 역시 변경의 파급 효과를 받게 됩니다.

### 원칙을 적용하여 개선
```java
public class Order {

    private final Product product;         // 제품 정보 객체로 분리
    private final Customer customer;       // 고객 정보 객체로 분리

    public Order(final Product product, final Customer customer) {
        this.product = product;
        this.customer = customer;
    }

    public Product getProduct() {
        return product;
    }

    public Customer getCustomer() {
        return customer;
    }
}

class Product {

    private final String name;
    private final int price;

    public Product(final String name, final int price) {
        this.name = name;
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public int getPrice() {
        return price;
    }
}

class Customer {

    private final String name;
    private final String email;

    public Customer(final String name, final String email) {
        this.name = name;
        this.email = email;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }
}
```

`Product`와 `Customer` 클래스를 분리하여 `Order` 클래스의 인스턴스 변수를 2개로 줄였습니다. 이제 각 클래스가 자신의 역할에 집중할 수 있도록 책임이 분리되어 코드의 응집도가 높아졌습니다.

만약 제품 정보나 고객 정보가 변경되더라도 다른 클래스에 아무런 영향이 가지 않습니다. 또한 각 책임을 담당하는 객체의 테스트를 독립적으로 작성하기도 편해졌습니다.

### 주의할 점
모든 클래스의 인스턴스 변수를 강제적으로 줄이려고 하면, 오히려 클래스의 수가 과도하게 늘어 코드의 복잡도가 증가할 수 있습니다.
특히 **DTO(Data Transfer Object)**와 같이 단순히 데이터를 전달하는 용도의 객체에서는 이 규칙을 너무 엄격하게 적용하는 게 오히려 비효율적일 수 있습니다.

이 원칙의 목적은 클래스가 여러 종류의 데이터를 처리하지 못하게 제안함으로써 `단일 책임 원칙(SRP)`을 준수하게 하는 것입니다.
객체 간 응집도가 낮고, 결합도가 높다고 판단되면 객체들의 책임을 분리하기 위해 인스턴스 변수를 줄여봅시다. 이때 무조건 변수명을 2개 이하로 줄이는 것에 집착하기보다 적절한 단일 책임을 가지도록 개선하는 것에 중점을 두는 것이 매우 중요합니다.

## 8. 일급 컬렉션을 사용한다.
### 목적
`List`, `Set`, `Map` 등 자바 컬렉션을 사용할 때 이 컬렉션이 객체 외부로 직접 노출되면 여러 문제가 발생할 수 있습니다.
컬렉션을 사용하는 곳마다 관련된 로직이 중복되거나, 컬렉션의 상태가 외부에서 의도치 않게 변경될 수 있습니다.

**일급 컬렉션**은 컬렉션을 포장한 객체를 의미합니다. 컬렉션과 관련된 모든 로직을 한곳에 모아 응집도를 높이고 캡슐화를 강화하기 위해 제안되었습니다.
이 원칙은 컬렉션을 일급 컬렉션으로 포장하여 컬렉션 내부의 상태를 안전하게 보호하고, 컬렉션과 관련된 로직을 한곳에서 관리하여 코드의 **응집도**와 **재사용성**을 높이도록 유도하는 게 목적입니다.

### 원칙이 적용되지 않았을 때 문제점
```java
public class Team {

    private final List<Member> members;  // 컬렉션이 직접 노출됨

    public Team(final List<Member> members) {
        this.members = members;
    }

    public List<Member> getMembers() {
        return members;  // 컬렉션을 직접 반환하여 외부에서 수정 가능
    }

    public void addMember(Member member) {
        members.add(member);
    }
}

class Member {

    private final String name;

    public Member(final String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

`Team` 클래스가 내부 컬렉션(`members`)을 외부에 직접 노출하고 있습니다. 
만약 `getMembers()`를 호출한 후, `members` 리스트에 직접 접근하면 요소를 제거하거나 추가할 수 있어 객체의 상태가 예기치 않게 변경될 수 있습니다.

또한, 컬렉션과 관련된 로직(`add`, `remove` 등)이 여러 곳에 분산되면 코드의 응집도가 낮아지고, 컬렉션 상태를 일관되게 유지하기가 어렵습니다.

### 원칙을 적용하여 개선
```java
public class Team {

    private final Members members;  // 일급 컬렉션으로 포장

    public Team(final List<Member> members) {
        this.members = new Members(members);
    }

    public Members getMembers() {
        return members;
    }
}

class Members {  // 일급 컬렉션

    private final List<Member> members;

    public Members(final List<Member> members) {
        this.members = new ArrayList<>(members);  // 외부 리스트를 내부적으로 복사하여 안전하게 관리
    }

    public void addMember(final Member member) {
        members.add(member);
    }

    public List<Member> getMembers() {
        return Collections.unmodifiableList(members);  // 외부에 불변 리스트 제공
    }
}
```

컬렉션을 `Members` 일급 컬렉션으로 포장하여 컬렉션과 관련된 로직을 `Members` 클래스에 위임하였습니다.

이제 컬렉션의 상태 변경과 외부에 어떤 형태로 컬렉션을 내보낼지에 대한 결정을 일급 컬렌션인 `Members` 내부에서 정의할 수 있습니다.
덕분에 관련된 로직에 응집도가 높아지고, 외부에서 맘대로 컬렉션의 상태를 변경할 여지가 사라졌습니다. 컬렉션 상태의 일관성이 보장되는 것입니다.

그리고 컬렉션에 관련된 로직의 테스트를 독립적으로 작성하기도 매우 쉬워졌습니다.

### 주의할 점
단순히 데이터 전달 목적만 가진 DTO, 컬렉션의 상태를 일시적으로만 사용하는 객체에 경우 오히려 일급 컬렉션을 활용하는 게 코드 복잡도를 높일 수 있습니다.

컬렉션의 상태를 관리하거나 변경하는 로직이 많을 경우에만 일급 컬렉션을 활용합니다. 만약 단순히 데이터를 담기 위한 용도라면 일급 컬렉션을 사용하는 게 코드 복잡도만 높일 수 있습니다.

## 9. Getter/Setter/Property를 무분별하게 사용하지 않는다.
### 목적
`Getter`, `Setter`를 무분별하게 남용하면 객체의 상태가 외부에서 쉽게 변경될 수 있습니다. 즉, 객체의 캡슐화가 깨질 수 있습니다.
또한 객체가 무언가 책임을 가지고 행동하는 자율적인 존재가 아닌, 단순히 데이터를 저장하고 전달하는 DTO처럼 동작하게 되어 객체지향의 의미가 퇴색될 수 있습니다.

이 원칙은 **객체가 자신의 상태를 스스로 관리**하고, 외부에서 객체의 상태를 직접 조작하지 않도록 유도하는 것이 목적입니다.

### 원칙이 적용되지 않았을 때 문제점
```java
public class User {

    private final String name;
    private final int age;

    // Getter
    public String getName() {
        return name;
    }

    // Setter
    public void setName(final String name) {
        this.name = name;
    }

    // Getter
    public int getAge() {
        return age;
    }

    // Setter
    public void setAge(int age) {
        this.age = age;
    }
}
```

위 코드의 `User` 객체는 `setName` & `setAge` 메서드를 통해 외부에서 객체의 상태를 마음대로 조작할 수 있습니다. 객체의 내부 상태를 안전하게 보호할 수 없는 상태입니다. 이는 **캡슐화 위반** 입니다.

객체가 자신이 관리해야 할 데이터를 외부에 쉽게 노출하기 때문에, 객체가 **자율적으로 상태를 관리하는 것**이 아닌, 외부에서 제어하는 **수동적인 객체**가 돼버립니다. 즉, 객체의 책임이 모호해집니다.
객체의 데이터가 외부에서 변경될 수 있기 때문에, 객체가 원래의 목적에 맞는 책임을 수행하지 못하고, 객체의 데이터와 행동이 분리되어 **응집도가 떨어 집**니다.

### 원칙을 적용하여 개선
```java
public class User {

    private final String name;
    private final int age;

    public User(final String name, final int age) {
        this.name = name;
        this.age = age;
    }

    // 이름을 변경하는 메서드
    public void changeName(final String newName) {
        if (newName == null || newName.isBlank()) {
            throw new IllegalArgumentException("Name cannot be null or empty");
        }
        this.name = newName;
    }

    // 나이를 변경하는 메서드
    public void haveBirthday() {
        this.age += 1;
    }

    // 나이에 대한 정보 제공 (단순히 상태를 전달하는 게터가 아닌 의미 있는 동작)
    public boolean isAdult() {
        return age >= 18;
    }
}
```
`setXXX` 메서드 대신 명확한 역할이 드러나는 메서드로 대체했습니다.

이제 `User` 객체는 스스로 자신의 상태를 관리하고, 본인만 알고 있는 상태 변경에 대한 규칙을 적용하여 행위를 수행합니다.
`changeName` 메서드는 단순히 `setName`처럼 상태를 외부값 그대로 변경하는 것이 아니라, **이름이 올바른지 유효성을 검증**하는 로직을 포함하는 등 능동적인 역할을 가지고 있습니다.
`haveBirthday`와 같은 **의미 있는 행동**을 통해 객체의 상태가 변화하도록 설계하여 **객체지향의 설계 원칙**을 준수하였습니다.
`isAdult` 메서드는 단순히 나이를 반환하는 `Getter`가 아니라, 객체의 상태에 따라 능동적으로 특정한 상태를 반환하는 행위를 수행합니다.

즉, 객체의 캡슐화가 강화되었다고 할 수 있습니다.

### 주의할 점
단순한 데이터 전달 객체(DTO)인 경우 `Getter`와 `Setter`를 사용하는 것이 더 효율적일 수 있습니다. 
**DTO**는 주로 데이터를 전달하거나, 계층 간의 데이터 교환을 위해 사용되므로, 이때는 `Getter`와 `Setter`를 통해 필드에 접근해도 무방합니다.

또한 원시값과 문자열을 래핑한 객체 역시 `Getter`를 통해 데이터를 반환하도록 로직을 작성해야 하는 상황이 올 수 있습니다.

**비즈니스 로직이 포함된 도메인 객체**에서는 `Getter`와 `Setter`를 줄이고, 해당 필드와 관련된 의미 있는 메서드를 만들어 **객체가 자신의 상태를 스스로 관리**하도록 설계합니다. 
반대로 **단순 데이터 전송 혹은 관리**가 목적인 DTO와 Wrapping 객체는 예외적으로 `Getter`와 `Setter`를 허용하여, 필요한 데이터를 효율적으로 전달하도록 합니다.

이 원칙은 무조건 모든 `Getter`와 `Setter`를 금지하라는 것이 아닙니다. 
중요한 것은 **객체가 자신의 상태를 안전하게 관리하도록 설계**하고, 외부에서 객체의 상태를 직접 조작하지 못하게 하는 것이 본질이라는 걸 기억해야 합니다.

---
# 마무리
지금까지 생활 체조 원칙들에 대해 알아보았습니다. 빠르게 적용할 수 있는 원칙도 있겠지만 아직 이해하기 힘든 원칙들도 많을 것입니다. 개발자는 손을 더럽힐 때 가장 많은 걸 얻어가는 직업입니다.
단순히 원칙들을 암기하기보다 직접 코드에 적용하면서 전/후 코드를 비교해 보며 학습해 봅시다.

주의점에서 언급 드렸듯, 개발 세계에는 **은탄환**(Silver bullet)이 존재하지 않습니다. 그 어떤 원칙도 모든 상황에서 정답이 되어주지 않습니다. 단순히 원칙을 지키는 것에 집착하는 게 아니라, 객체지향 그 자체에 목적을 두어야 합니다.
필요하다면 지금 코드에 객체지향이 정말 적절한지도 늘 고민해야 합니다. 개발자는 현재의 문제를 명확히 인지하고, 적절한 도구를 찾아 해결하는 것이 궁극적 목표입니다.

지금 본인이 마주한 문제에 객체지향이 해답일 거라는 확신이 들었다면, 생활 체조와 함께 문제를 해결해 봅시다.
