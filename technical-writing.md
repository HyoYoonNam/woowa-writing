# 좋은 도메인 설계를 위한 과정

> 프로젝트 구현을 위한 설계 과정에서 발생한 문제와 이를 해결하기 위한 과정을 기록했습니다. 이 글의 예시 코드는 당시 상황을 이해하기 쉽게 간결하게 구성되었습니다.

## 도메인 설계를 위한 ERD 작성

리뷰미 서비스는 프로젝트를 함께한 팀원끼리 서로 리뷰를 주고받을 수 있는 상호 리뷰 플랫폼입니다.
리뷰를 받고자 하는 사용자는 자신이 참여한 프로젝트에 대한 리뷰를 받기 위해 프로젝트명(`group_name`)을 입력하여 리뷰 그룹을 생성하고, 리뷰를 받고 싶은 사람들에게 리뷰 요청을 보냅니다.
이후 리뷰 요청을 받은 팀원들은 해당 리뷰 그룹에 리뷰를 작성하고, 작성된 리뷰는 리뷰 요청자가 확인할 수 있는 구조로 되어 있습니다.

서비스 구현을 시작하며 가장 먼저 진행한 작업은 ERD 작성이었습니다.
JPA를 활용해 도메인을 설계하기에 앞서, 데이터 구조를 명확히 하기 위해 ERD를 작성했습니다.


![erd.png](./image/erd.png)

하지만 JPA를 위한 ERD가 먼저 작성된 후 도메인을 설계하니, 도메인 관점과 맞지 않는 부분이 생겼습니다.
=======
JPA를 위한 ERD가 먼저 작성된 후 도메인을 설계하니, 도메인 관점과 맞지 않는 부분이 생겼습니다.

특정 멤버가 자신이 참여한 프로젝트에 대한 리뷰를 받기 위해 프로젝트명(`group_name`)을 설정하고, 리뷰 그룹을 생성합니다.
이후 리뷰를 받고 싶은 사람들에게 리뷰 요청을 보냅니다. 리뷰 요청을 받은 사람들이 해당 리뷰 그룹에 리뷰를 작성하면,
멤버는 자신의 리뷰 그룹을 통해 받은 리뷰를 확인할 수 있어야 합니다.

```java
public class Review {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "reviewer_group_id", nullable = false)
    private ReviewGroup reviewGroup;
}
```

- 여기서 리뷰와 리뷰 그룹의 관계뿐만 아니라, 리뷰에 포함되는 리뷰 질문과 답변에 대한 경우도 마찬가지이나, 이 글에서는 리뷰와 리뷰 그룹 간의 관계에 대해서만 설명하겠습니다.

## 도메인 설계 관점과 DB 설계 관점의 비교

도메인 설계 관점에서 `Review` 와 `ReviewGroup` 은 스스로 자신의 상태를 책임집니다.
`Review`는 생성될 때 스스로 리뷰 내용을 검증한 후 생성됩니다.
이후 `ReviewGroup`에 포함될 때에는 그 자체로 검증된 객체로 제공되며,
`ReviewGroup` 은 리뷰 작성자가 리뷰 그룹의 소유자와 일치하는지 등 규칙이나 정책에 따라 `Review`가 자신에게 속할 수 있는지 만을 확인합니다.

```java
    // 리뷰 생성자
    public Review(Member reviewer, String content) {
        validateContent(content);
        this.reviewer = reviewer;
        this.content = content;
    }
```

```java
public class ReviewGroup {

	private List<Review> reviews;

    public void addReview(Review review) {
        if (canAddReview(review)) {
            // 리뷰 그룹 포함 검증 실패 예외
        }
        reviews.add(review);
    }

    private boolean canAddReview(Review review) {
        // Review를 그룹에 추가할 수 있는지 검증하는 로직
        return true;
    }
}
```

하지만 DB 설계를 따라 `Review`가 `ReviewGroup`을 참조하는 방향으로 설계되면, 리뷰가 생성되는 순간에 리뷰 그룹이 정해집니다.
따라서 `Review`는 생성될 때 리뷰 내용을 검증하는 것뿐만 아니라, `ReviewGroup`과 협력해 자신이 해당 그룹에 속할 수 있는지도 함께 검증해야 합니다.

```java
public class Review {

		// 다른 필드
    private ReviewGroup reviewGroup;

    public Review(Member reviewer, ReviewGroup reviewGroup, String content) {
        validateContent(content);
        validateInclusionInReviewGroup(reviewGroup);
        this.reviewer = reviewer;
        this.reviewGroup = reviewGroup;
        this.content = content;
    }

    private void validateInclusionInReviewGroup(ReviewGroup reviewGroup) {
        if (!reviewGroup.canAddReview(this)) {
            // 리뷰 그룹 포함 검증 실패 예외
        }
    }
}
```

이렇게 도메인 설계와 DB 설계 간의 **연관 관계 방향의 불일치**가 있을 때 발생할 수 있는 문제가 있습니다.

## 도메인과 DB 연관 관계 방향의 불일치로 인한 문제

`ReviewGroup` 은 `Review`를 그룹으로 관리하기 위해 만들어진 도메인으로, 재사용보단 관리의 목적이 큽니다.
반면 `Review`는 `ReviewGroup`없이도 독립적으로 재사용될 수 있어야 합니다.
즉, `ReviewGroup`에 의해 관리되지 않더라도, `Review` 자체에는 동작에 문제가 없어야 합니다.
하지만 DB 설계를 따라 `Review`가 `ReviewGroup`을 참조하는 방향으로 설계되면, `Review`에는 리뷰 그룹 포함과 관련된 검증이 포함되게 됩니다.

```java
 private void validateInclusionInReviewGroup(ReviewGroup reviewGroup) {
        if (!reviewGroup.canAddReview(this)) {
            // 리뷰 그룹 포함 검증 실패 예외
        }
    }
```

이는 `Review` 가 `Review` 자체에 대한 책임뿐만 아니라, `ReviewGroup`의 책임도 지니게 하면서 **단일 책임 원칙을 위반**하게 됩니다.

- `Review`는 `ReviewGroup` 이 없는 상황에서 독립적으로 재사용할 수 없게 됩니다.
- `Review`가 `ReviewGroup`의 기능을 직접 사용하게 되면서 `ReviewGroup`이 관리해야 할 책임이 `Review`로 넘어가게 됩니다.
- `Review`는 `ReviewGroup`의 변경에 영향을 받게 됩니다.

단일 책임 원칙에 따르면, 모듈은 한 가지 이유로만 변경되어야 하지만,
`Review`는 자체 변경뿐만 아니라 `ReviewGroup`의 변경에도 영향을 받습니다.
이는 `Review`와 `ReviewGroup` 간의 결합도를 높여 변경에 유연하지 못하는 상태를 초래하며, 유지 보수를 어렵게 만듭니다.

## 양방향 관계가 해결책이 될 수 있을까?

처음에는 도메인 관점과 DB 관점의 연관 관계 방향 모두 충족시키기 위해 양방향 관계를 사용했습니다.

```java
public class Review {
    
    // 다른 필드

    @ManyToOne
    @JoinColumn(name = "reviewer_group_id", nullable = false)
    private ReviewerGroup reviewerGroup;
    
    public Review(Member reviewer, ReviewGroup reviewGroup, String content) {
        validateContent(content);
        this.reviewer = reviewer;
        this.reviewGroup = reviewGroup;
        this.content = content;
        reviewGroup.addReview(this);
    }
}
```

```java
public class ReviewerGroup {
    
    // 다른 필드

    @OneToMany(mappedBy = "reviewerGroup")
    private List<Review> reviews;

    public void addReview(Review review) {
        // 리뷰 검증 로직
        
        reviews.add(review);
    }

}
```

하지만 책임 분리가 명확하지 않고, 초기화되지 않은 상태에서의 참조로 인해 의도치 않은 문제가 발생했습니다.
양방향 관계에서 발생하는 문제를 해결하기 위해 추가적인 장치들을 도입했지만, 이는 오히려 로직을 복잡하게 만들었습니다.

결과적으로 개발자가 관리해야 할 부분을 증가 시키기 때문에 문제를 해결하기보단, 새로운 문제를 야기하게 되어 해결책이 되지 못했습니다.

## 어떤 설계가 좋은 설계일까?

우리는 어떤 설계가 좋은 설계인지 계속해서 고민했습니다.

- 로직을 작성하기 쉬운 설계
- 성능을 고려한 설계
- 변경에 유연하고 유지 보수에 용이한 설계

이러한 목표를 달성하기 위해 서비스의 요구사항과 변화 가능성을 고려해야 했습니다.
리뷰미는 리뷰 작성 폼의 발전과 확장 가능성을 염두에 두고 있었기 때문에, 변경에 유연하게 대처할 수 있는 설계를 추구했습니다.


## 연관 관계를 다시 생각해보자

객체지향 설계에서 객체들은 서로 연관 관계를 맺음으로써 메시지를 주고받으며 협력합니다.
그러나 불필요한 연관 관계가 생기면 불필요한 책임이 생기고, 이는 변경에 유연하지 못하게 됩니다.

따라서 우리는 불필요한 연관 관계를 제거하고, 정말 필요한 곳만 연관 관계를 가지도록 했습니다.
이 과정에서 조영호 님의 [우아한객체지향](https://www.youtube.com/watch?v=dJ5C4qRqAgA)을 공부하고 참고했습니다.

### 객체를 묶는 기준

우리는 의존 받는 객체의 변경이 의존하는 객체에 영향을 미치는 것을 경험했습니다.
모든 의존 관계를 완전히 제거하는 것이 아니라, **같이 변경되어야 하는 객체들**만 묶는다면 다른 객체들은 변경에 영향을 미치지 않을 것이라고 생각했습니다.

같이 변경되어야 하는 객체들은 본질적으로 결합도가 높아 같이 변경되어도 무방하여, 도메인적인 관점에서 같이 처리되어야 할 객체들입니다.
우리는 이 기준을 생명 주기를 함께하는 객체로 설정했습니다.

현재 구현의 예시로는 리뷰(`Review`)와 리뷰에 작성된 답변(`TextAnswer`)이 있습니다.

- 리뷰의 답변은 리뷰 없이 생성될 수 없고, 리뷰가 작성되는 시점에 항상 같이 생성됩니다.
- 리뷰 없이 답변만 있는 것이 불가능하기에, 리뷰가 삭제되면 답변도 항상 같이 생성됩니다.

이러한 특성 때문에 `Review`와 `TextAnswer`는 생명 주기가 일치하며, 두 객체는 강한 결합 관계를 가집니다.
따라서 두 객체를 하나의 단위로 묶어 관리하는 것이 적절하다고 생각했습니다.

```java
public class Review {

	// 다른 필드

    @OneToMany(fetch = FetchType.EAGER, cascade = CascadeType.PERSIST)
    @JoinColumn(name = "review_id", nullable = false, updatable = false)
    private List<TextAnswer> textAnswers;
    
}
```

### 논리적인 연관 관계의 장점

논리적으로 연관 관계를 재설정하면서 느꼈던 장점 입니다.

1. 객체 관계를 명확하게 정의함으로써, 설계 의도를 더 명확하게 드러낼 수 있습니다.
2. 변경이 발생할 때, 그 영향에 대한 범위가 명확함으로 예상 가능하며 다른 부분에 미치는 영향을 최소화할 수 있습니다. 이로써 코드 재사용을 높이고, 일관성을 유지할 수 있습니다.
3. 데이터베이스의 연관 관계에 따라 `CascadeType`과 `FetchType` 설정을 고민해야 했지만, 논리적인 기준을 세우고 객체를 묶음으로써 연관 관계 설정의 기준을 명확히 할 수 있습니다.
    - 두 객체는 함께 생성되고 관리되므로 `CascadeType.PERSIST` 설정을 주었습니다.
    - 리뷰를 조회할 때, 답변도 함께 조회할 수 있도록 `FetchType.EAGER`를 사용했습니다.

### 간접 참조의 활용

위에서 세운 기준으로 강한 결합 관계를 가지는 객체들을 묶어 관리하고 불필요한 연관 관계를 끊었습니다.
여기서 강한 결합 관계를 가지는 객체들이 아닌 객체들의 연관 관계를 모두 완전히 끊을 수는 없기에,
직접적인 객체 참조 대신 `id`를 사용하여 객체 간의 결합을 느슨하게 유지할 수 있게 했습니다.

```java
public class Review {

    @Column(name = "review_group_id", nullable = false)
    private long reviewGroupId;
    
}
```

## 현재 구현 방식의 단점
이번 설계를 바탕으로 객체들 간의 관계를 정의하는 기준을 재정립하고, 연관 관계가 불필요한 객체들은 id로 간접적으로 관리하는 방식을 적용했습니다.
이러한 방식은 여러 이점도 있었지만 단점도 존재했습니다.

- 필요할 때만 DB에서 연관된 객체를 조회하기 때문에 불필요한 데이터를 가져오지 않도록 할 수 있지만, 이는 다르게 보면 연관된 객체를 필요로 할 때마다 별도의 별도의 쿼리를 사용해야 하므로 성능 저하가 발생할 수 있습니다.
- 간접 참조를 사용함으로써 책임이 명확하게 분리되고 독립적으로 관리되지만, 객체 간의 관계를 수동으로 처리해야 하는 부담이 생깁니다. 또 데이터 정합성을 유지하기 위한 추가적인 로직이 필요할 수 있습니다.

결과적으로 연관관계를 분리함으로써 `findById()`를 통하여 데이터를 조회하기 위해 DB 접근 횟수가 많아진다는 점이고, 이는 성능 저하로 이어질 수 있습니다.

현재 리뷰미의 리뷰 템플릿 구조는 `템플릿 > 섹션 > 질문 > 질문 항목`으로 되어있습니다. 
리뷰를 조회하기 위해서 템플릿에 맞는 형태로 리뷰의 내용들을 가져와야 하는데, 이 과정에서 리뷰 템플릿 구조마다 id를 사용해 DB에 반복적으로 접근하게 됩니다.

```java
    // 전체 템플릿 응답 객체 매핑
    public ReviewDetailResponse mapToReviewDetailResponse(Review review, ReviewGroup reviewGroup) {
        long templateId = review.getTemplateId();
        List<Section> sections = sectionRepository.findAllByTemplateId(templateId);
        List<SectionAnswerResponse> sectionResponses = sections.stream()
                .map(section -> mapToSectionResponse(review, reviewGroup, section))
                // 기타 로직
                .toList();
        
        return new ReviewDetailResponse(
                templateId,
                sectionResponses
        );
    }
    
    // 섹션 응답 객체 매핑
    private SectionAnswerResponse mapToSectionResponse(Review review, ReviewGroup reviewGroup, Section section) {
        List<QuestionAnswerResponse> questionResponses = questionRepository.findAllBySectionId(section.getId())
                .stream()
                // 기타 로직
                .map(question -> mapToQuestionResponse(review, reviewGroup, question))
                .toList();
        return new SectionAnswerResponse(
                section.getId(),
                questionResponses
        );
    }
    
    // 질문 응답 객체 매핑
    private QuestionAnswerResponse mapToQuestionResponse(Review review, ReviewGroup reviewGroup, Question question) {
        OptionGroup optionGroup = optionGroupRepository.findByQuestionId(question.getId())
                .orElseThrow();
        
        // 기타 로직
    }
```

각 템플릿의 섹션, 섹션의 질문, 그리고 질문의 항목마다 계속해서 데이터를 찾기 위해 DB에 접근합니다.

또한 각 리뷰 그룹에는 작성될 리뷰의 템플릿이 정해져 있습니다.

```java
public class ReviewGroup {

    // 다른 필드

    @Column(name = "template_id", nullable = false)
    private long templateId = 1L;
}
```

리뷰 그룹에 리뷰를 등록하기 위해, 리뷰 그룹에 있는 템플릿을 가져올 때에도 id로 연관되어 있기 때문에 DB에 실제로 존재하는 정보인지 확인하는 과정을 거치게 됩니다.
```java
    private Template findTemplateByReviewGroupOrThrow(ReviewGroup reviewGroup) {
        return templateRepository.findById(reviewGroup.getTemplateId())
                .orElseThrow(() -> new TemplateNotFoundByReviewGroupException(
                        reviewGroup.getId(), reviewGroup.getTemplateId()));
    }
```

현재 리뷰미 팀은 이러한 문제의 해결방안으로 캐싱을 사용하려 합니다.
템플릿, 섹션, 질문 등의 데이터는 초기 설정된 고정 데이터로, 앞으로 크게 추가될 가능성이 적습니다.
또한 이 데이터들은 리뷰를 추가하거나 조회할 때 필수적으로 참조됩니다.
따라서 템플릿 구조와 관련된 데이터는 자주 조회되지만 자주 변경되지 않기 때문에, 캐싱을 활용하면 반복적인 DB 접근을 줄여 성능을 크게 개선할 수 있을 것이라고 기대하고 있습니다.

## 결론

불필요한 연관관계를 제거함에 있어서 오는 단점은 DB 접근 횟수 증가 외에도 더 많은 구현을 해보면 직접적으로 느끼는 것이 많을 거라 생각됩니다.

하지만 우리가 설계에서 고려했던 핵심 중 하나는 서비스 요구사항이 계속해서 변화할 가능성이 크다는 점이었습니다. 
초기 기획이 계속해서 변경되었고, 향후에도 서비스 요구사항은 계속해서 변경할 가능성이 있습니다. 
이러한 변화에 대비하기 위해, 우리는 확장성을 예측해서 구현하기보다는 확장성을 열어두는 방향으로 설계를 하는 것이 중요하다 생각했습니다.
우리는 이번 설계 과정에서 여러 가지 새로운 시도를 해보았고, 특정 상황에서 이론적으로 좋은 설계라고 여겨지던 부분들이 실제 구현에서는 예상치 못한 문제를 야기할 수 있음을 깨달았습니다.
이와 같은 경험들은 앞으로의 설계와 구현에서 더 나은 결정을 내릴 수 있는 기반이 될 것이라고 생각합니다.

