# @Transactional 알아보기

### Transaction이란?

트랜잭션은 데이터베이스에서 상태를 변화시키기 위해 수행하는 하나의 논리적인 작업 단위를 의미합니다. </br>
이는 여러 개의 데이터베이스 연산을 포함할 수 있으며, 이러한 연산은 모두 성공적으로 완료되거나, 하나라도 실패하면 전체 작업이 취소되어야 합니다. </br> 
트랜잭션은 데이터의 무결성과 일관성을 보장하기 위해 매우 중요합니다. </br>

트랜잭션이 안전하게 수행되기 위해서는 4가지 필수 특성인 ACID 조건을 충족해야 합니다.

#### Transaction 특징

1. 원자성(Atomicity)
: 트랜잭션은 모든 작업이 하나의 단위로 실행되도록 보장합니다. </br>
즉, 트랜잭션 내에서 수행되는 모든 작업은 모두 성공하거나, 하나라도 실패하면 전체가 롤백됩니다.

2. 일관성(Consistency)
: 트랜잭션이 완료된 후 데이터베이스는 항상 일관된 상태를 유지해야 합니다. </br>
트랜잭션 내의 작업이 성공적으로 완료되면 시스템의 상태는 정합성을 유지합니다.

3. 격리성(Isolation)
: 트랜잭션은 동시에 실행되는 다른 트랜잭션과 격리된 상태에서 동작해야 합니다. </br>
이는 동시성 문제를 방지하기 위한 것으로, 여러 트랜잭션이 동시에 같은 데이터를 수정하려 할 때 발생할 수 있는 충돌을 막기 위해 격리 수준을 설정하여 제어합니다.

4. 내구성(Durability)
: 트랜잭션이 성공적으로 완료되면 그 결과는 영구적으로 저장됩니다. </br>
즉, 트랜잭션이 커밋되면 시스템에 장애가 발생하더라도 그 결과는 손실되지 않고 데이터베이스에 변경 사항이 확실히 반영되도록 보장됩니다.

이러한 ACID 조건을 충족하기 위해 트랜잭션에서는 commit과 rollback을 사용합니다. </br>
데이터베이스 작업의 성공 여부에 따라 트랜잭션을 확정하거나 취소할 수 있습니다.

- **Commit**
: 트랜잭션 내에서 실행된 모든 작업이 성공적으로 완료되었음을 확인하고, 데이터베이스에 그 변경 사항을 반영하는 과정입니다. </br>
Commit이 완료되면 해당 트랜잭션에서 실행된 모든 변경 사항이 영구적으로 저장됩니다.

- **Rollback**
: 트랜잭션 내에서 오류가 발생했거나 작업이 실패한 경우, 해당 트랜잭션 내에서 수행된 모든 변경 사항을 취소하는 과정입니다. </br> 
Rollback이 발생하면 트랜잭션이 시작된 이후의 모든 데이터베이스 변경 사항은 무효가 되고, 데이터베이스는 트랜잭션 이전 상태로 되돌아갑니다.

### 프로그램에 의한 트랜잭션 적용
@Transactional 어노테이션이 도입되기 전에는 트랜잭션 관리를 위해 프로그래밍 방식으로, 수동으로 트랜잭션을 시작하고 종료하는 코드를 작성해야 했습니다. </br>
이 방식은 주로 데이터베이스 관련 API나 트랜잭션 매니저를 활용하여 트랜잭션을 명시적으로 관리하는 방법입니다.

예를 들어, 은행에서 A 계좌에서 B 계좌로 송금하는 작업을 생각해 볼 수 있습니다. </br>
이 작업은 두 개의 계좌에서 돈을 이동시키며, 성공적으로 완료되거나 실패할 경우 모두 롤백되어야 합니다. 

이를 코드로 나타내면 다음과 같습니다.

```java
public class BankService {

    private final PlatformTransactionManager transactionManager;

    public BankService(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }

    public void transferFunds(Account fromAccount, Account toAccount, double amount) {
        // 트랜잭션 상태를 가져옵니다.
        TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());

        try {
            // 송금 비즈니스 로직 수행
            withdraw(fromAccount, amount); // 출금
            deposit(toAccount, amount); // 입금

            // 모든 작업이 성공적으로 완료되면 커밋
            transactionManager.commit(status);
        } catch (Exception e) {
            // 오류가 발생하면 롤백
            transactionManager.rollback(status);
            throw new RuntimeException("송금 실패: " + e.getMessage(), e);
        }
    }

    private void withdraw(Account account, double amount) {
        // 계좌에서 금액을 출금하는 로직
        account.setBalance(account.getBalance() - amount);
    }

    private void deposit(Account account, double amount) {
        // 계좌에 금액을 입금하는 로직
        account.setBalance(account.getBalance() + amount);
    }
}
```

이 코드에서 A 계좌에서 금액을 출금하고 B 계좌에 금액을 입금하는 작업이 하나의 트랜잭션 내에서 실행됩니다. </br>
두 작업이 모두 성공하면 커밋이 호출되어 A 계좌의 출금과 B 계좌의 입금이 확정됩니다.

#### 프로그램에 의한 트랜잭션 관리의 문제점

위의 코드에서 볼 수 있듯이, 수동으로 트랜잭션을 관리하는 방식은 다음과 같은 문제점이 있습니다.

- 중복 코드: 트랜잭션을 시작하고 커밋하거나 롤백하는 코드가 반복적으로 작성되어야 합니다. 이는 코드의 유지보수를 어렵게 합니다.

- 에러 가능성: 트랜잭션 경계를 직접 관리하다 보면 예외 상황에서 롤백을 잊거나 잘못된 시점에 트랜잭션을 커밋하는 등의 실수를 범할 수 있습니다. 
이는 데이터의 일관성을 해칠 수 있습니다.

- 가독성 저하: 비즈니스 로직과 트랜잭션 관리 코드가 섞이면서 코드가 복잡해지고, 전체적인 가독성이 떨어집니다. 
이는 나중에 코드를 유지 보수하거나 수정할 때 어려움을 초래합니다.

이렇게 수동으로 트랜잭션을 관리하는 방식은 여러 단점이 있어, @Transactional 어노테이션과 같은 자동화된 트랜잭션 관리 방법이 필요하게 되었습니다.

### 선언적 트랜잭션 적용

선언적 트랜잭션은 @Transactional 어노테이션을 사용하여 트랜잭션을 관리하는 방법입니다. </br>
이 방식은 비즈니스 로직과 트랜잭션 관리 코드를 분리하여 코드의 가독성과 유지보수성을 높여줍니다.

```java
public class BankService {

    // PlatformTransactionManager는 더 이상 필요하지 않습니다.

    @Transactional
    public void transferFunds(Account fromAccount, Account toAccount, double amount) {
        withdraw(fromAccount, amount);
        deposit(toAccount, amount);
    }

    private void withdraw(Account account, double amount) {
        account.setBalance(account.getBalance() - amount);
    }

    private void deposit(Account account, double amount) {
        account.setBalance(account.getBalance() + amount);
    }
}
```

위 코드로부터 알 수 있듯이, 수동으로 트랜잭션을 관리했을 때보다 코드가 굉장히 깔끔해진 것을 알 수 있습니다.

먼저 트랜잭션 관리 코드가 비즈니스 로직과 분리되어, 불필요한 중복 코드가 줄었습니다. </br>
이는 코드의 가독성을 높이고 유지보수를 쉽게 만들고, 비즈니스 로직이 명확하게 드러나게 됩니다. </br>
또한 트랜잭션 경계를 자동으로 관리하므로, 예외 상황에서 롤백을 잊거나 잘못된 시점에 커밋하는 등의 실수를 줄일 수 있습니다.

### @Transactional 동작 원리

그럼 스프링에서 기존의 복잡했던 코드들을 어떻게 @Transactional 어노테이션을 사용함으로써 바꿀 수 있었는지 동작 원리에 관해 설명하겠습니다.

1. 프록시 객체의 생성:
- @Transactional 어노테이션이 붙은 클래스나 메서드에 대해 스프링은 자동으로 트랜잭션 프록시를 생성합니다.
- 이 프록시는 원래의 클래스나 메서드 호출을 가로채어, 트랜잭션 관리 로직(시작, 커밋, 롤백 등)을 추가합니다.
- 예시: 만약 위 코드처럼 BankService 클래스의 transferFunds 메서드에 @Transactional이 붙어 있다면, 스프링은 이 메서드를 호출할 때 프록시를 통해 트랜잭션을 관리합니다.
2. 트랜잭션의 시작:
- 메서드가 호출되면 프록시는 먼저 새로운 트랜잭션을 시작합니다.
- 예시: transferFunds 메서드가 호출되면, 스프링은 새로운 트랜잭션을 시작합니다.
3. 영속성 컨텍스트의 생성 및 연결:
- 트랜잭션이 시작되면, JPA를 사용할 경우 영속성 컨텍스트가 생성되어 데이터베이스와의 연결이 설정됩니다. 이 컨텍스트는 데이터 변경 사항을 관리합니다.
- 예시: 송금 작업 중, 출금과 입금 작업이 영속성 컨텍스트에 반영되어 변경 사항이 추적됩니다.
4. 비즈니스 로직 실행:
- 트랜잭션 내에서 비즈니스 로직이 실행됩니다. 이 단계에서 데이터베이스에 대한 변경 작업이 이루어집니다.
- 예시: withdraw 메서드에서 A 계좌에서 금액을 출금하고, deposit 메서드에서 B 계좌에 금액을 입금합니다.
5. 트랜잭션의 커밋 또는 롤백:
- 모든 작업이 성공적으로 완료되면, 프록시는 트랜잭션을 커밋하여 데이터베이스에 변경 사항을 확정합니다. 만약 예외가 발생하면, 프록시는 자동으로 롤백하여 모든 변경 사항을 취소합니다.
- 예시: A 계좌에서 출금과 B 계좌에 입금이 모두 성공하면 트랜잭션이 커밋됩니다. 만약 중간에 오류가 발생하면 모든 작업이 롤백되어 데이터가 원래 상태로 되돌아갑니다.

이처럼 @Transactional을 통해 트랜잭션 관리가 자동으로 이루어지지만, 사용 시 몇 가지 주의해야 할 점이 있습니다. </br> 
외부 메서드와 내부 메서드에 대한 트랜잭션 적용 결과가 다를 수 있기 때문입니다. </br>
예를 들어, 내부 메서드에 @Transactional을 적용한 경우, 그 메서드가 외부 메서드에서 직접 호출되면 트랜잭션 관리가 제대로 되지 않을 수 있습니다. 

따라서 이러한 상황을 잘 이해하기 위해 밑에서 자세히 설명하겠습니다.

### @Transactional 주의점

@Transactional 어노테이션을 사용할 때, 외부 메서드와 내부 메서드에 대한 적용 결과에 대해 주의해야 할 점이 있습니다. </br>
아래에서는 이러한 상황을 코드로 설명하고 있습니다.

모든 코드는 외부 클래스에서 TransactionalService의 outerMethod를 호출한다는 가정하겠습니다.

1. 외부 및 내부 메서드 모두에 @Transactional을 적용한 경우

```java
@Service
public class TransactionalService {

    @Transactional
    public void outerMethod() {
        // 외부 메서드의 비즈니스 로직
        innerMethod(); // 내부 메서드 호출
    }

    @Transactional
    public void innerMethod() {
        // 내부 메서드의 비즈니스 로직
    }
}
```

이 경우, outerMethod와 innerMethod 모두 트랜잭션이 적용됩니다. </br>
스프링의 트랜잭션 전파 기본값은 Required로 설정되어 있어, outerMethod에서 트랜잭션이 시작되고, innerMethod는 같은 트랜잭션 안에서 실행됩니다. </br>
(트랜잭션 전파 속성은 아래에서 설명하겠습니다.)

2. 외부 메서드에만 @Transactional을 적용한 경우

```java
@Service
public class TransactionalService {

    @Transactional
    public void outerMethod() {
        // 외부 메서드의 비즈니스 로직
        innerMethod(); // 내부 메서드 호출
    }

    public void innerMethod() {
        // 내부 메서드의 비즈니스 로직
    }
}
```

이 경우에도 outerMethod와 innerMethod 모두 트랜잭션이 적용됩니다. </br>
outerMethod가 @Transactional을 통해 트랜잭션을 시작하므로, 그 안에서 호출된 innerMethod는 같은 트랜잭션을 사용하게 됩니다.

3. 내부 메서드에만 @Transactional을 적용한 경우

```java
@Service
public class TransactionalService {

    public void outerMethod() {
        innerMethod(); // 내부 메서드 호출
    }

    @Transactional
    public void innerMethod() {
        // 내부 메서드의 비즈니스 로직
    }
}
```

이 경우, outerMethod와 innerMethod 모두 트랜잭션이 적용되지 않습니다. </br>
이유는 outerMethod에서 innerMethod를 직접 호출하기 때문에, 스프링의 프록시 객체를 거치지 않고 원본 메서드가 호출되기 때문입니다. </br>
@Transactional이 붙은 메서드는 프록시를 통해 호출되어야 트랜잭션 관리 로직이 적용됩니다.

그렇다면 내부 메서드에만 트랜잭션을 적용해야 하는 경우, 이 문제를 어떻게 해결해볼 수 있을까요? </br>
해당 메서드를 별도의 클래스로 분리하여 트랜잭션 관리가 제대로 작동하도록 해야 합니다.

```java

@Service
public class OuterService {

    private final InnerService innerService;

    public OuterService(InnerService innerService) {
        this.innerService = innerService;
    }

    public void outerMethod() {
        innerService.innerMethod();
    }
}

@Service
public class InnerService {

    @Transactional
    public void innerMethod() {
        // 내부 메서드의 비즈니스 로직
    }
}
```

이제 innerMethod는 InnerService라는 별도의 클래스에 정의되어 있으며, @Transactional이 붙어 있습니다. </br>
outerMethod에서 innerService.innerMethod()를 호출하면, 프록시를 통해 호출되므로 트랜잭션이 제대로 적용됩니다.

### @Transactional 전파 속성

스프링의 @Transactional 어노테이션은 트랜잭션의 전파 방식을 설정할 수 있는 여러 속성을 제공합니다. </br>
이 속성들은 트랜잭션이 다른 트랜잭션에 어떻게 영향을 미치는지를 정의합니다. 주요 전파 속성은 다음과 같습니다.

1. REQUIRED
- REQUIRED는 스프링에서 제공하는 기본 전파 속성입니다. 이 속성이 설정된 메서드가 호출되면, 기존의 트랜잭션이 존재하면 그 트랜잭션에 참여하고, 없으면 새로운 트랜잭션을 생성합니다.
- 예시: 외부 메서드가 @Transactional로 설정되어 있다면, 이 메서드에서 내부 메서드를 호출할 때, 내부 메서드는 외부 트랜잭션에 참여하게 됩니다.
2. REQUIRES_NEW
- REQUIRES_NEW는 외부 트랜잭션과 내부 트랜잭션을 완전히 분리합니다. 이 속성이 설정된 메서드는 항상 새로운 트랜잭션을 생성하며, 외부 트랜잭션이 존재하더라도 영향을 받지 않습니다.
- 예시: 외부 메서드에서 REQUIRES_NEW가 설정된 내부 메서드를 호출하면, 두 개의 물리적 트랜잭션이 생성되어 각각 독립적으로 커밋 및 롤백됩니다.
3. SUPPORTS
- SUPPORTS는 호출된 메서드가 트랜잭션이 존재하면 그 트랜잭션에 참여하고, 없으면 트랜잭션 없이 실행됩니다.
- 예시: 외부 메서드가 트랜잭션을 시작하면, SUPPORTS가 설정된 메서드는 그 트랜잭션에 참여하지만, 외부 트랜잭션이 없으면 그냥 실행됩니다.
4. MANDATORY
- MANDATORY는 호출된 메서드가 항상 기존 트랜잭션 내에서 실행되어야 함을 의미합니다. 트랜잭션이 없으면 예외가 발생합니다.
- 예시: 외부 메서드가 트랜잭션을 시작하지 않고 MANDATORY가 설정된 내부 메서드를 호출하면 예외가 발생합니다.
5. NOT_SUPPORTED
- NOT_SUPPORTED는 호출된 메서드가 트랜잭션을 사용하지 않도록 설정합니다. 만약 외부 트랜잭션이 존재하면, 이를 일시 중단하고 트랜잭션 없이 실행됩니다.
- 예시: 외부 메서드가 트랜잭션을 시작한 상태에서 NOT_SUPPORTED가 설정된 내부 메서드를 호출하면, 외부 트랜잭션이 중단되고 내부 메서드는 트랜잭션 없이 실행됩니다.
6. NEVER
- NEVER는 호출된 메서드가 트랜잭션 없이 실행되어야 함을 의미합니다. 트랜잭션이 존재하면 예외가 발생합니다.
- 예시: 외부 메서드가 트랜잭션을 시작한 상태에서 NEVER가 설정된 내부 메서드를 호출하면 예외가 발생합니다.
7. NESTED
- NESTED는 현재 트랜잭션 내에 중첩된 트랜잭션을 생성합니다. 이 속성이 설정된 메서드는 기존 트랜잭션이 있으면 그 트랜잭션에 중첩되어 실행됩니다. 중첩된 트랜잭션은 독립적으로 커밋 또는 롤백할 수 있습니다.
- 예시: 외부 메서드가 트랜잭션을 시작한 상태에서 NESTED가 설정된 내부 메서드를 호출하면, 중첩된 트랜잭션이 만들어져 해당 메서드에서의 변경 사항은 독립적으로 처리됩니다.


### 마무리하며

저도 실제 프로젝트를 진행하면서 @Transactional의 작동 원리를 잘 이해하지 못해 의도한 대로 실행되지 않았던 경험이 있습니다. </br> 
이러한 경험을 바탕으로 @Transactional의 기능에 대해 더 깊이 알아보자는 목적에서 이 글을 작성하게 되었습니다.

@Transactional을 사용할 때는 그 동작 원리에 대한 정확한 이해가 매우 중요합니다.
