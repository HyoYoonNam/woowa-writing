# 스프링을 버리고 객체지향으로 나아가기

> 이번 글은 우아한테크코스 6기 모우다팀의 서비스를 바탕으로 작성된 글입니다.

자바,스프링으로 프로젝트를 진행하다 보면 자연스럽게 접하게 되는 개념들이 많습니다. **레이어드 아키텍처, MVC, JPA** 등이 그것이죠. 대부분은 별다른 고민 없이 개념을 받아들입니다. 다른 사람들이 어떻게 작성했는지 찾아보고 따라하는데 급한 경우가 많죠. 그래서 우리는 다음과 같은 코드를 많이 마주치게 됩니다.

```java
public MoimService {

    private final ChamyoRepository chamyoRepository;
    private final MoimRepository moimRepository;
    private final NotificationService notificationService;

    ...

    public CreateChamyoMoimResponse chamyoMoim(Long darakbangId, Long moimId, DarakbangMember darakbangMember) {
        Moim moim = moimRepository.findByIdForUpdate(moimId)
            .orElseThrow(() -> new ChamyoException(HttpStatus.NOT_FOUND, ChamyoErrorMessage.MOIM_NOT_FOUND));
        if (moim.isNotInDarakbang(darakbangId)) {
            throw new ChamyoException(HttpStatus.BAD_REQUEST, ChamyoErrorMessage.MOIM_NOT_FOUND);
        }
        validateCanChamyoMoim(moim, darakbangMember);

        Chamyo chamyo = Chamyo.builder()
            .moim(moim)
            .darakbangMember(darakbangMember)
            .moimRole(MoimRole.MOIMEE)
            .build();
        try {
            Chamyo chamyo = chamyoRepository.save(chamyo);
            return CreateChamyoMoimResponse.from(chamyo);
        } catch (DataIntegrityViolationException exception) {
            throw new ChamyoException(HttpStatus.BAD_REQUEST, ChamyoErrorMessage.MOIM_ALREADY_JOINED);
        }

        int currentPeople = chamyoRepository.countByMoim(moim);
        if (currentPeople >= moim.getMaxPeople()) {
            moimRepository.updateMoimStatusById(moim.getId(), MoimStatus.COMPLETED);
        }

        notificationService.notifyToMembers(NotificationType.NEW_MOIMEE_JOINED, darakbangId, moim, darakbangMember);
    }

    private void validateCanChamyoMoim(Moim moim, DarakbangMember darakbangMember) {
        int currentPeople = chamyoRepository.countByMoim(moim);
        if (currentPeople >= moim.getMaxPeople()) {
            throw new ChamyoException(HttpStatus.BAD_REQUEST, ChamyoErrorMessage.MOIM_FULL);
        }
        if (moim.getMoimStatus() == MoimStatus.CANCELED) {
            throw new ChamyoException(HttpStatus.BAD_REQUEST, ChamyoErrorMessage.MOIMING_CANCELED);
        }
        if (moim.getMoimStatus() == MoimStatus.COMPLETED) {
            throw new ChamyoException(HttpStatus.BAD_REQUEST, ChamyoErrorMessage.MOIMING_COMPLETE);
        }
        if (chamyoRepository.existsByMoimIdAndDarakbangMemberId(moim.getId(), darakbangMember.getId())) {
            throw new ChamyoException(HttpStatus.BAD_REQUEST, ChamyoErrorMessage.MOIM_ALREADY_JOINED);
        }
    }

    ...
```

네 맞습니다. 사실 이 코드는 모우다팀의 결과물입니다. 여러분은 위 코드를 보면 어떤 생각이 드시나요? '코드가 왜 이렇게 복잡하지?'라고 생각하신 분도 있을 것입니다. 만약 위 코드에서 이상함을 느끼지 못하셨다면 끝까지 글을 읽어보시길 추천드립니다.

모우다팀은 두 가지 의문을 가지게 되었습니다.

1. 스프링은 객체지향의 강점을 극대화하는 프레임워크입니다. **우리의 코드는 정말 객체지향이라고 말할 수 있을까요?**
2. 서비스는 비즈니스 로직을 수행하는 객체입니다. **우리의 서비스 객체는 정말 비즈니스 로직을 잘 드러내고 있을까요?**

이번 글에서는 이런 의문들에 대한 고찰에 대해서 나누도록 하겠습니다.

## 스프링과 객체지향

객체지향에 대해 조금이라도 관심이 있는 분들은 조영호님에 대해서 아실 겁니다. 조영호님은 [우아한 객체지향 세미나](https://youtu.be/dJ5C4qRqAgA?feature=shared)에서 다음과 같이 이야기합니다.

> **도메인 객체 간의 협력을 먼저 설계한다. 그리고 완료된 도메인 설계 위에 서비스를 붙인다.**

돌이켜보면, 모우다팀이 코드를 작성하기 앞서 가장 먼저 설계한 것은 도메인 설계가 아니라 데이터베이스 테이블 설계였습니다. 그리고 API 스펙에 맞추어 컨트롤러와 요청/응답 객체부터 개발했습니다. 그런 후에 자연스럽게 서비스 클래스를 만들고 JPA 레포지토리를 의존하여 절차지향적으로 도메인 로직을 수행하고 결과를 반환했습니다.

여기서 다음과 같은 질문을 할 수 있습니다.

> **테이블 설계가 곧 도메인 설계 아닌가요? JPA 엔티티(@Entity 어노테이션이 붙은 클래스)가 곧 도메인이 아닌가요?**

**아닙니다.** JPA의 엔티티는 사실 데이터베이스 관점의 엔티티에 가깝기 때문입니다.

이를 제대로 이해하려면 도메인 엔티티와 데이터베이스 엔티티에 대해 이해할 필요가 있습니다. 객체지향에서 협력의 주체가 되고 도메인 로직을 수행하는 객체를 도메인 엔티티라고 부르며, 관계형 데이터베이스에서는 테이블을 하나의 엔티티라고 부릅니다.

객체지향과 데이터베이스는 추구하는 바가 다릅니다. **객체지향은 '객체 간의 협력을 통한 유지보수하기 좋은 구조 설계'** 라는 방향으로 발전했고, 데이터베이스는 **'데이터 간의 중복성 문제 해결과 데이터 정합성'** 을 향해 발전했습니다. 이 과정에서 현실의 개념을 데이터로 표현하는데 양측 모두 '엔티티'라는 용어를 사용하게 된 것입니다.

따라서 데이터베이스 설계는 도메인 엔티티 설계라고 할 수 없습니다. 일정 부분 연관은 있을 수 있지만 데이터베이스 구조가 그대로 서비스에 투영될 수는 없습니다. 대표적인 사례는 다음과 같습니다.

모우다에는 회원가입한 멤버가 만들 수 있는 '모임'이라는 개념이 있습니다. 멤버는 여러 모임에 동시 참여할 수 있으며 하나의 모임에도 여러 명의 멤버가 참여할 수 있습니다. 전형적인 **'다대다 관계'** 입니다. 이 개념을 객체지향과 관계형 데이터베이스에서는 각각 어떻게 표현할 수 있을까요?

![다이어그램](./image/mouda.drawio.png)

먼저 데이터베이스입니다. 다대다 관계를 해결하기 위해 매핑 테이블을 사용하는 것이 일반적입니다. 그런데 객체지향에서는 어떻게 표현할까요?

```java
class Moim {
    List<Member> participants;
}
```

모임에 여러 명의 참가자가 참여할 수 있다는 관계는 위와 같이 간단하게 표현됩니다. 복잡하게 생각할 필요가 없죠. 바로 이 지점이 패러다임의 불일치입니다.

```java
@Entity
@Table(name = "moim_member")
class MoimMember {
    private Moim moim;
    private Member member;
    ...
}

// 실제 사용례
long moimId = 1L;
List<MoimMember> moimMembers = moimMemberRepository.findAllByMoimId(moimId);
List<Member> members = new ArrayList();
for (MoimMember moimMember : moimMembers) {
    members.add(moimMember.getMember());
}
```

별다른 고민 없이 JPA 레포지토리를 사용한다면 특정 모임에 참여하는 멤버들을 조회하기 위해 위와 같이 구현해야 합니다. 직접 비교해보니 뭔가 위화감을 느끼시나요? 위 흐름이 자연스럽다고 느껴지신다면 반박할 수는 없지만, 언젠가 패러다임의 불일치로 인해 개발이 정체될 수 있다는 점은 자신 있게 이야기할 수 있습니다.

상상해봅시다. 어느 날 스프링 프레임워크와 하이버네이트가 갑자기 유료로 전환되고, 우리 회사는 이를 감당할 수 없다면? 우리는 쉽게 다른 프레임워크나 ORM, 혹은 영속성 기술로 이전할 수 있을까요? 위와 같이 작성된 도메인 로직은 과연 안전할까요?

그럼 어떻게 해야 할까요? 정답은 **라이브러리, API, 프레임워크에 의존하지 않고 도메인을 설계**하는 것입니다. 스프링, JPA 없이 오직 자바 코드로만 말이죠. 태초로 돌아가는 것입니다. 그리고 완성된 도메인 설계에 역으로 스프링과 JPA를 입히는 것이죠. 그렇게 하면 우리는 어느 기술에도 종속되지 않고 비즈니스의 핵심 로직을 표현할 수 있습니다.

말뿐인 주장은 설득력이 없죠. 함께 모우다를 다시 설계하러 가봅시다.

## 메시지를 중심으로 객체 간 협력 설계하기

모우다의 모든 기능 설계를 이 글에서 모두 설명할 수는 없기에 **‘모임 생성’, ‘모임 참여’** 두 가지 사용자 플로우에만 집중해보겠습니다. 중요한 것은 데이터가 아닌 어떤 메시지를 보내야 하는가를 중심으로 한 설계입니다. 이것이 객체지향으로 나아가는 가장 중요한 원칙이기 때문입니다.

모우다의 도메인 요구사항은 다음과 같습니다(간단한 구현을 위해 실제보다 간소화된 형태입니다).

1. 모임을 생성할 수 있다.
    1. 모임을 만든 멤버는 모임의 호스트가 되며 자동으로 참여 처리가 된다.
    2. 모임 날짜와 시간은 현재 시각보다 더 빠르게 설정할 수 없다.
    3. 모임 이름은 중복할 수 없다.
2. 모임에 참여할 수 있다.
    1. 한 모임에는 여러 멤버가 참여할 수 있다.
    2. 한 번 참여한 모임에는 다시 참여할 수 없다.

요구사항을 읽어보셨다면 요구사항을 준수하는 관계형 데이터베이스의 ERD를 한번 상상하고 밑으로 내려가 보시길 추천드립니다.

위 요구사항을 반영한 도메인 객체 간 관계 다이어그램은 다음과 같습니다.

![mouda.domain.drawio.png](./image/mouda.domain.drawio.png)


관계형 데이터베이스의 ERD를 상상해보셨다면 위 클래스 다이어그램이 얼마나 직관적이며 현실에 더 가까운지 감이 오실 것입니다. 다이어그램만 보더라도 객체 간 관계가 쉽게 이해되며 요구사항의 대부분이 표현되어 있음을 알 수 있습니다. (물론 실제 규모의 비즈니스는 이보다 훨씬 복잡합니다)

### 모임 생성 플로우 분석

![moim.drawio.png](./image/moim.drawio.png)

먼저 `Moim`에게 **검증하라**는 메시지를 보냅니다. 이때 `Moim`이 검증해야 하는 내용은 모임의 날짜 및 시간입니다. 모임의 날짜와 시간의 전문가는 `MoimDetail`이기 때문에 `Moim`은 `MoimDetail`에게 **모임 날짜와 시간을 검증하라**는 메시지를 보냅니다. 모임 생성의 요구사항은 이것으로 끝인가요? 아닙니다. **모임 이름은 중복할 수 없다**는 요구사항을 만족하기 위해 다음과 같이 추가적인 설계가 필요합니다.

![moims.drawio.png](./image/moims.drawio.png)

현재 존재하는 모임들을 지니는 모임 목록의 전문가인 `Moims` 객체(`Moim`의 일급 컬렉션)를 만들었습니다. 이제 `Moims`에게 `Moim` 객체를 던지며 **중복 이름을 검증하라**는 메시지를 보낼 수 있습니다. 이때 `Moims`는 자신이 지닌 각각의 `Moim`에게 새로운 `Moim`과 **같은 이름이 있는지 확인하라**는 메시지를 보냅니다.

이로써 우리는 모임 생성 플로우의 모든 요구사항을 설계했습니다. 이제 이를 코드로 옮겨보겠습니다.

#### Moim

```java
public class Moim {

    private final MoimDetail moimDetail;

    // 생성자와 정적 팩토리 메서드들 ...
    
    public void validate() {
        moimDetail.validateDateTime();
    }

    public boolean hasSameNameWith(Moim newMoim) {
        return moimDetail.hasSameNameWith(newMoim.moimDetail);
    }
    
    // Getter...
}
```

#### MoimDetail

```java
public class MoimDetail {

    private final String name;
    private final LocalDate date;
    private final LocalTime time;
    private final String description;
    private final int maxParticipants;

    // ...

    public boolean hasSameNameWith(MoimDetail newDetail) {
        return this.name.equals(newDetail.name);
    }

    public void validateDateTime() {
        if (LocalDateTime.of(date, time).isBefore(LocalDateTime.now())) {
            throw new IllegalArgumentException("Moim dateTime is before now");
        }
    }
}
```

#### Moims

```java
public class Moims {

    private final List<Moim> moims;

    // ...
    
    public void validateExistMoimName(Moim newMoim) {
        for (Moim moim : moims) {
            if (moim.hasSameNameWith(newMoim)) {
                throw new IllegalArgumentException("Moim name already exist");
            }
        }
    }
}
```

어떤가요? 각 도메인 객체들은 자신들의 일을 충실히 잘하고 있습니다. 유기적인 협력 관계 속에서 말이죠. 코드가 크게 어렵지 않음에도 요구사항을 잘 반영하고 있습니다.

그런데 고민이 생깁니다. 위 다이어그램에서 `Moim`과 `Moims`에게 메시지를 보내는 주어가 표현되지 않았습니다. 그렇다면 저 메시지는 누가 보내주어야 할까요? 이때 등장해야 하는 것이 바로 서비스 클래스입니다. 위 도메인들의 동작을 조율해주는 역할이 서비스였던 것이죠. 따라서 서비스를 Orchestrator, 즉 비즈니스 로직의 조율자라고 부르기도 합니다. 

최종적으로 서비스 코드로 모임을 생성하는 흐름을 구현해보겠습니다.

#### MoimService

```java
public class MoimService {

    public void createMoim() {
        Moim moim = new Moim(); // 자세한 초기화 과정은 생략했습니다.
        Moims moims = new Moims();
        moim.validate();
        moims.validateExistMoimName(moim);
    }
}
```

이제 서비스의 역할이 눈에 들어오시나요? **서비스는 각 도메인들에게 적절한 메시지를 보내어 하나의 비즈니스 흐름(모임을 생성한다)는 역할에 충실**할 뿐 각 도메인 규칙이 어떻게 구현되어있는지는 관심이 없다는 것을 알 수 있습니다.

그런데 우리가 반영하지 않은 요구사항이 한 가지 있습니다.

> 모임을 만든 멤버는 모임의 호스트가 되며 자동으로 참여 처리가 된다.

위 요구사항은 모임에 참여 로직을 구현해보며 추가해보도록 하겠습니다.

### 모임 참여 플로우 분석

![enter.drawio.png](./image/enter.drawio.png)

먼저 새로운 참가자인 `Participant`를 보내며 `Moim`에게 **참여한다**는 메시지를 보냅니다. 그 후 `Moim`은 참가자 목록의 전문가인 `Participants`에게 **참가자를 추가하라**는 메시지를 보냅니다. 이후 `Participants`는 자신이 지닌 `Participant` 목록을 순회하며 새 `Participant`와 **같은 참가자인지 확인하라**는 메시지를 보내며 검증을 수행한 후 참여 처리를 합니다.

설계를 마쳤습니다. 마찬가지로 코드로 옮겨보겠습니다.

#### Moim

```java
public class Moim {

    private final MoimDetail moimDetail;
    private final Participants participants; // 참가자 목록 일급 컬렉션 추가

    // ...

    // 참여한다 메시지 구현
    public void enter(Participant newParticipant) {
        participants.add(newParticipant);
    }
}
```

#### Participants

```java
public class Participants {

    List<Participant> participants;
    
    // ...

    // 참가자를 추가하라 메시지 구현
    public void add(Participant newParticipant) {
        validateAlreadyEnteredMember(newParticipant);
        participants.add(newParticipant);
    }

    // participants 순회하며 같은 참가자인지 확인하라는 메시지 보내기
    private void validateAlreadyEnteredMember(Participant newParticipant) {
        for (Participant participant : participants) {
            if (participant.isSameAs(newParticipant)) {
                throw new IllegalArgumentException("member already entered (id : " + newParticipant.getId() + ")");
            }
        }
    }
}
```

#### Participant

```java
public class Participant {

    private final Member member;

    // ...
    
    // 같은 참가자인지 확인하라 메시지 구현
    public boolean isSameAs(Participant newParticipant) {
        return this.equals(newParticipant);
    }
    
    // equals & hashCode 재정의
}
```

그리고 이 도메인들을 조율해줄 서비스 클래스에 참여 기능을 구현해보겠습니다.

#### MoimService

```java
public class MoimService {
    // ...
    
    public void enterMoim() {
        Moim moim = new Moim(); // 마찬가지로 복잡한 생성과정은 생략합니다.
        Participant newParticipant = new Participant();
        moim.enter(newParticipant);
    }
}
```

모임에 참여한다는 요구사항도 순조롭게 완성했습니다. 이제 마지막으로 참가자를 모임 호스트와 모임 참가자로 구분시켜 줄 수 있도록 `MoimRole`을 구현하여 남겨둔 요구사항을 처리해보겠습니다.

#### MoimRole

```java
public enum MoimRole {
    HOST, ATTENDANT;
}
```

모임 역할을 나타내는 도메인인 `MoimRole`을 참가자 객체인 `Participant`에 추가한 후, 원하는 역할로 생성할 수 있도록 기능을 추가하겠습니다.

```java
public class Participant {

    private final Member member;
    private final MoimRole moimRole;
    
    // ...
    
    public static Participant toHost(Member member) {
        return new Participant(member, MoimRole.HOST);
    }
    
    public static Participant toAttendant(Member member) {
        return new Participant(member, MoimRole.ATTENDANT);
    }
}
```

이제 우리는 참가자의 유형을 구분할 수 있게 되었습니다. 이제 각 서비스에서 `Participant`를 생성하는 과정을 추가하면 완성입니다.

#### MoimService

```java
public class MoimService {

    public void createMoim() {
        // Moim, Moims, Member 생성
    
        moim.validate();
        moims.validateExistMoimName(moim);
        
        Participant host = Participant.toHost(member);
        moim.enter(host);
    }

    public void enterMoim() {
        // Moim, Member 생성
        Participant attendant = Participant.toAttendant(member);
        moim.enter(attendant);
    }
}
```

이로써 모든 요구사항을 만족시켰습니다. 하지만 위 로직은 실제와 조금 다르기 때문에 조금 더 현실적으로 수정해보겠습니다.

```java
public class MoimService {

    private final MoimRepository moimRepository;

    public MoimService(MoimRepository moimRepository) {
        this.moimRepository = moimRepository;
    }

    public long createMoim(Moim moim, Member member) {
        Moims moims = moimRepository.findAll();

        moim.validate();
        moims.validateExistMoimName(moim);
        
        Participant host = Participant.toHost(member);
        moim.enter(host);

        return moimRepository.append(moim, member);
    }

    public long enterMoim(long moimId, Member member) {
        Moim moim = moimRepository.findById(moimId);

        Participant attendant = Participant.toAttendant(member);
        moim.enter(attendant);
        
        return moimRepository.modify(moim, member);
    }
}
```

이제 컨트롤러 없이도 개발이 가능하다는 것을 보여드리기 위해 컨트롤러 역할을 서비스 메서드 시그니처에 반영했습니다. 또한 레포지토리를 추가하여 도메인 로직을 수행할 주체를 가져오는 방식으로 바꾸었습니다.

```java
public interface MoimRepository {

    long append(Moim moim, Member member);

    long modify(Moim moim, Member member);

    Moims findAll();

    Moim findById(long moimId);
}
```

여기서 주목해야 할 점은 `MoimRepository`가 `JpaRepository`를 상속하지 않는다는 것입니다.

우리는 지금까지 JPA뿐 아니라 스프링과 관련된 그 어떠한 의존성도 사용하지 않았습니다. 순수한 자바 코드로 비즈니스 요구사항을 만족시킨 것입니다. 이는 놀라운 결과입니다. 스프링을 배운 지 얼마 안 된 개발자들은 다음과 같은 질문을 자주하기 때문입니다.

- 데이터베이스 테이블 설계 없이 개발하는 게 가능한가요?
- API 스펙을 먼저 작성하지 않고 개발하는 게 가능한가요?

네, 가능합니다. 메서드 시그니처와 레포지토리 인터페이스라는 규약을 통해 스프링과 JPA 없이도 개발할 수 있습니다. 우리는 능력이 있었음에도 스프링과 JPA에 갇혀 스프링의 본질을 잊어버렸을지 모릅니다.



## 스프링의 본질

> A key element of Spring is infrastructural support at the application level: Spring focuses on the "plumbing" of enterprise applications so that **teams can focus on application-level business logic**, without unnecessary ties to specific deployment environments.  
[스프링 공식 문서 중](https://spring.io/projects/spring-framework)

스프링 공식 문서에는 팀이 애플리케이션 레벨의 비즈니스 로직에 집중할 수 있도록 돕는다고 합니다. 이게 무슨 뜻일까요?

웹 애플리케이션 개발에는 다양한 기술들이 필요합니다. HTTP 웹 통신, 웹 서버, WAS, 데이터베이스와 같은 것들이죠. 스프링 프레임워크는 웹 개발을 위한 거의 모든 기능을 제공합니다. 단 하나, 실제 애플리케이션이 어떻게 동작해야 하는지를 나타내는 도메인 로직만을 제외하고 말이죠.

우리는 습관처럼 `Controller`, `Service`, `JPA Entity`, `JpaRepository` 등을 먼저 구현합니다. 그러나 이들은 스프링이 개발자들을 도와주기 위한 도구에 불과합니다. 내장 톰캣은 WAS 기능을 제공하고, `@Controller`는 개발자들이 도메인 로직을 제공할 인터페이스를 쉽게 정의할 수 있도록 돕고, JPA는 복잡한 데이터베이스 연결 과정을 추상화하여 사용하기 쉽게 만듭니다. 하지만 가장 중요한 도메인 로직은 `@Service` 어노테이션을 붙인 클래스에 덕지덕지 나열되는 경우가 많습니다.

스프링의 본질은 도메인 로직 설계를 먼저 고민하고, 나머지 귀찮은 일들은 스프링에게 맡기는 것입니다. 그러나 우리는 결과물에 해당하는 부분부터 개발을 시작하면서 쓸데없이 힘을 빼고, 정작 심혈을 기울여야 할 도메인에는 크게 고민하지 않는 경우가 많습니다.

## 스프링과 도메인 로직의 분리

이제 우리가 만든 도메인 로직에 스프링을 입혀보겠습니다. 아주 간단합니다. 먼저 서비스 클래스에 `@Service` 어노테이션을 붙여줍니다. 그 후 API 스펙에 맞추어 컨트롤러 클래스를 만들고 서비스와 협력하도록 합니다. 마지막으로 데이터베이스 테이블과 매핑할 `JPA Entity`와 해당하는 `JpaRepository`를 사용하여 `MoimRepository`의 구현체를 만들면 됩니다.

여기서 중요한 점은 우리의 소스코드가 스프링과 JPA에 종속되지 않았다는 것입니다. 만약 JPA가 아닌 다른 ORM을 사용하게 된다면? 혹은 아예 ORM 없이 `JdbcTemplate`을 사용하기로 결정했다면? 관계형 데이터베이스 대신 NoSQL을 사용하기로 했다면? 이 경우에도 서비스와 도메인 코드는 건드리지 않고 `MoimRepository`의 구현체만 새로 만들면 됩니다. 또한 자바를 사용하는 다른 프레임워크로 전환하는 것도 가능합니다. 심지어 다른 언어의 프레임워크로도 문법만 수정한 후 재사용할 수 있습니다. 

## 완벽한 설계?

그렇다면 위 코드는 과연 완벽한 설계일까요? 당연히 아닙니다. 객체지향은 분명 장점이 많은 패러다임입니다. 잘 사용하면 코드의 응집도를 높이고 결합도를 낮춰 유지보수하기 좋은 코드를 만들 수 있습니다. 하지만 만능은 아닙니다.

> ***때로는 절차지향이 객체지향보다 좋다 - 조영호 (우아한 객체지향 강의 중)***

객체지향이 항상 최선은 아닙니다. 객체지향으로 작성된 코드는 유지보수에 용이하지만 여러 객체에 흩어진 로직으로 인해 이해하기 어려운 경우가 많습니다. 따라서 때로는 절차지향으로 작성하는 것이 나을 수 있습니다. 대표적인 예시로는 연쇄적인 검증 로직이 있습니다. 여러 클래스를 돌아다니며 검증 로직을 파악하기보다는, `Validator` 클래스 하나에 여러 검증 로직을 배치하여 해당 파일 하나만 보면 이해할 수 있도록 하는 것이 나을 수 있습니다. 뿐만 아니라 객체 간, 패키지 간 의존성 측면에서도 개선할 여지는 많이 남아있습니다.

그럼에도 불구하고 객체지향적인 설계로 도메인 로직을 고민해야 하는 이유는 무엇일까요?

## 도메인: 웹 애플리케이션의 본질

**도메인이야말로 웹 애플리케이션의 본질**이기 때문입니다. 개발자는 문제를 프로그래밍으로 해결하는 사람입니다. 스프링, JPA, 자바는 기술이지 개발의 본질이 아닙니다. 모든 기술은 결국 문제를 풀기 위한 수단입니다. 개발자가 비즈니스 문제에 더욱 집중할 수 있도록 도와주는 도구인 것이죠. 이러한 철학에서 출발한 개발 이론이 바로 ***도메인 주도 설계 (Domain-Driven Design)*** 입니다.

소프트웨어가 풀어야 하는 문제가 시장이 해결하기 원하는 문제가 아니라면 유연한 설계가 필요할까요? 아닙니다. 사용자가 없는 프로젝트는 유지보수할 필요가 없습니다. 그러나 우리는 미래를 모릅니다. 우리의 프로젝트가 성공할지, 실패할지는 열어봐야 알 수 있습니다. 그래서 저는 **최소한의 유연성을 포기하지 않는 설계**를 항상 목표로 개발하고 있습니다.

## 마무리하며

하고 싶은 이야기는 너무 많지만 글이 지나치게 길어지는 것을 경계하며 이만 줄이겠습니다. 이 글을 통해 저와 같은 고민을 하는 초보 개발자들에게 많은 도움이 되기를 바랍니다. 모든 소스 코드는 [깃허브 레포지토리](https://github.com/ksk0605/Mouda-DDD)에 공유되어 있습니다.

## 레퍼런스

1. [우아한 객체지향 by 우아한테크](https://www.youtube.com/watch?v=dJ5C4qRqAgA&ab_channel=%EC%9A%B0%EC%95%84%ED%95%9C%ED%85%8C%ED%81%AC)
2. 조영호 저, [오브젝트: 코드로 이해하는 객체지향 설계], 2019
3. 김우근 저, [자바/스프링 개발자를 위한 실용주의 프로그래밍], 2024

#### 깃허브 레포지토리 주소

https://github.com/ksk0605/Mouda-DDD
