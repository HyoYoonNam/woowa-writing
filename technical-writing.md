# Multi Step Form 구현기: 더 나은 UX를 위한 노력

# 1. **서론**

## **1.1. Multi Step Form을 개발한 배경**

<img src="/technical-writing/image1.png" width="200">

‘모우다: 모여봐요 우리의 다락방’은 사람들이 속한 그룹(다락방) 내에서 모임을 쉽게 만들고 참여할 수 있도록 돕기위해 제작된 서비스입니다. 모우다 서비스는 사용자가 자신이 속한 신뢰할 수 있는 집단에서 편하게 모임을 주선할 수 있는 공간을 제공하여, 새로운 인연을 맺을 수 있도록 돕는 것을 목표로 하고 있어요.

<img src="/technical-writing/image2.png" width="500">

이 목표 아래에서 저희 팀이 마주한 주요 문제는 복잡한 모임 여정이었어요. 다락방을 만들고, 사람을 초대하고, 다락방에서 모임을 만들고, 참여하고, 소통하는, 모임의 시작부터 마지막에 이르기까지의 절차가 길고 복잡했습니다.

<img src="/technical-writing/image3.png" width="200">

그 중에서 모임 생성 단계의 사용자 경험을 개선하기 위해 노력했던 이야기를 다루려고 해요. 프로젝트 초기, MVP로 시행한 UT에서 모임을 생성하는 사용자의 경험이 매우 좋지 않다는 것을 확인했어요. 모임을 만들기 위해 사용자는 날짜, 시간, 장소, 인원수, 모임 내용 등 많은 정보를 입력해야 했거든요. 이로인해 사용자가 폼을 작성하는 것에 스트레스를 받는다는 것을 알게됐고, 이를 개선하기 시작했어요.

최대한 빠르게 모임을 생성할 수 있도록 하기위해 꼭 필요한 정보만 필수로 두고, 나머지는 사용자가 원할 때 수정 페이지에서 추가로 입력하거나 모임이 이루어진 후에 채팅 과정에서 확정할 수 있도록 했어요.

또한 모임 생성 페이지에서 모든 입력을 한꺼번에 요청하는 것이 사용자의 부담을 가중시키고 있었고, 다른 입력란의 값에 따라 조건부로 입력해야 하는 정보가 있었어요. 이러한 문제들을 해결하기 위해 순차적으로 하나씩 입력을 요청하는 Multi Step Form으로 모임 생성 페이지의 UI를 전면 개편했어요. 사용자에게 보다 직관적이고 부드러운 모임 생성 경험을 제공하고, 폼 작성을 단계별로 나눔으로써 인지 부하를 최소화하고자 했어요.

이 글은 Multi Step Form을 개발하면서 겪은 문제와 그걸 어떻게 해결했는지를 공유하고자 작성한 글이에요. 어딘가에서 저희 팀과 유사한 문제를 겪을 사람들에게 조금이나마 도움이 되길 바랍니다.

## **1.2. Multi Step Form?**

본격적인 이야기를 하기 전에 Multi Step Form 용어와 특징을 먼저 짚어보겠습니다.

Multi Step Form이라는 용어가 생소할 수 있는데, 사실 굉장히 직관적으로 이해할 수 있는 단순한 단어의 조합일 뿐이에요. `Multi Step Form = Multi Step(여러 단계인) + Form(입력 양식)`입니다. 말 그대로 여러 단계로 나누어진 폼이죠. 여러 입력 요소를 한 번에 하나씩만 노출하고, 사용자가 하나의 입력에만 집중할 수 있도록 하여 자연스럽게 Form 완성하도록 유도하는 게 핵심 목적인 UI에요.

그럼 일반적인 Form과 비교를 통해 Multi Step Form의 UX적인 가치를 알아보도록 할게요. 일반적인 정적인 Form을 Single Step Form이라고 칭하도록 하겠습니다.

### **1.2.1. Single Step Form**

<img src="/technical-writing/image4.png" width="500">

**장점:**

1. **(DX) 개발 및 유지보수 용이**: 구조가 단순하여 구현이 쉽고, 유지보수도 상대적으로 간단합니다.
2. **(UX) 빠른 입력 완료**: 모든 정보를 한 번에 입력할 수 있어, 단순한 입력의 경우 단시간에 폼 작성이 가능합니다.

**단점:**

1. **(UX) 인지적 부담**: 정보가 한 페이지에 집중되기 때문에, 질문이 많거나 복잡할 경우 사용자가 부담을 느끼고 이탈할 가능성이 높습니다.
2. **(UX) 모바일 친화성 부족**: 화면이 작은 모바일 환경에서 모든 정보를 한 번에 보여주면 화면에 다 들어오지 않을 수 있습니다.
3. **(UX) 조건부 질문에 불리**: 사용자의 이전 답변에 따라 질문 내용 구성이 달라지는 경우, 개발자의 의도를 사용자가 이해하기 어렵습니다.

### **1.2.2. Multi Step Form**

<img src="/technical-writing/image5.png" width="800">

**장점:**

1. **(UX) 화면 공간 최적화**: 모바일 기기에서 각 단계별로 필요한 정보만 표시해 화면 복잡도를 줄일 수 있습니다.
2. **(UX) 인지 부담 감소**: 정보를 단계별로 분할해 제공함으로써 사용자의 인지 부담을 줄일 수 있습니다.
3. **(UX) 집중도 향상**: 질문을 더 구체적으로 할 수 있고, 사용자도 한 번에 하나의 질문에만 집중할 수 있기 때문에 적확한 정보 입력을 유도하기 좋습니다.
4. **(UX) 조건부 질문에 유리**: 이전 단계의 답변에 따라 다음 단계를 맞춤형으로 조정할 수 있어 유연한 폼 구성이 가능합니다.

**단점:**

1. **(DX) 구현 복잡성 증가**: 여러 단계를 관리해야 하기 때문에 개발 및 유지보수가 상대적으로 더 복잡합니다.
2. **(DX) 복잡한 네비게이션**: 사용자가 이전 단계로 돌아가 정보를 수정하려 할 때 복잡한 네비게이션이 문제가 될 수 있기 때문에 이를 처리해야 합니다.
3. **(UX) 전체 프로세스 파악 어려움**: 사용자가 전체 폼의 길이를 한눈에 파악하기 어려워 도중에 포기할 수 있습니다.
4. **(UX) 컨텍스트 손실**: 이전 단계의 정보를 쉽게 참조할 수 없어 사용자가 헷갈릴 수 있습니다.

정리하면,

- 빠르게 개발을 해야하는 경우
- 질문 수가 적은 경우
- 화면 공간의 여유가 있는 경우

에는 Single Step Form이 적절한 선택일 것이고,

- 질문 수가 많은 경우
- 화면 공간의 여유가 없는 경우
- 질문의 순서가 중요하거나 이전 단계의 답변에 따라 다른 폼 절차를 유도하고 싶은 경우

에는 Multi Step Form을 추천합니다!

---

# 2. **Multi Step Form 개발 과정 중 겪은 문제**

그럼 이제 Multi Step Form을 개발하면서 겪은 문제와 그걸 어떻게 해결했는지 공유해볼게요.

Multi Step Form을 구현하면서 겪은 문제는 크게 단계 관리, 상태 관리, UX 개선 및 모바일 관련 문제로 총 세 가지에요. 이 세 가지 문제는 대략적으로 아래와 같아요.

## **2.1. 단계 관리 문제**

Multi Step Form은 Form을 분리한다는 것이니까, 단계별로 화면이 분리되겠죠. 그래서 가장 먼저 해야했던 고민은 ‘사용자가 여러 단계에 걸쳐 데이터를 입력할 때, 각 단계를 어떻게 관리하지?’였어요.

그리고 모우다 서비스는 브라우저 환경에서 구동되기 때문에 브라우저 UI에 대응하는 것도 중요했어요. 예를 들어 페이지 새로고침이나 뒤로/앞으로 가기 버튼을 눌렀을 때 사용자가 예상하지 못한 동작이 발생하지 않도록 하기위한 방법을 고민했어요.

## **2.2. 상태 관리 문제**

Multi Step Form을 개발하면서 겪은 가장 어려운 문제였어요. 아마 Multi Step Form을 구현하신/하실 다른 분들도 이 지점에서 많은 고민을 하셨을/하실거라 생각해요.

사용자가 여러 단계에 걸쳐 입력한 정보를 안전하게 잘 유지하는 게 중요했어요. 특히 새로고침, 페이지 이탈 및 접근 등 앱 외부에서 일어나는 일들에 대해 사용자 입장에서 최대한 자연스럽게 처리되도록 구현하는 데 많은 노력을 들였어요.

## **2.3. UX 개선 및 모바일 환경 최적화**

Multi Step Form의 UX 단점을 어떻게 극복할지, 모바일 환경의 가상 키보드로 인한 UX 저하 문제를 어떻게 해결할지 등, (해결 방법은) 사소하지만 (사용자 경험에는) 중요한 문제들이었어요.

이 세 가지 문제를 각각 어떻게 해결했는지 하나씩 살펴봅시다.

---

# 3. 단계 **관리 문제 해결**

브라우저 환경에서 여러 단계를 적절히 관리하기 위해 고려한 접근 방식은 아래와 같이 총 네 가지인데요,

1. useState를 이용한 방법
2. Path Parameter를 이용한 방법
3. Query Parameter를 이용한 방법
4. History state를 이용한 방법

이 네 가지 방법이 뭔지, 결정을 위해 어떤 고민을 했고, 최종적으로 어떤 방법을 선택했는지 공유해볼게요.

## 3.1. 다양한 접근 방식 검토

### 3.1.1. **useState를 이용한 방법**

```tsx
function MultiStepForm() {
  const [step, setStep] = useState("title");

  return (
    <>
      {step === "title" && <TitleStep />}
      {step === "place" && <PlaceStep />}
    </>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/add-moim" element={<MultiStepForm />} />
    </Routes>
  );
}
```

폼의 단계를 React의 `useState` 훅을 사용해 상태로 관리하는 방법이에요. 폼 내에서 단계를 이동할 때마다 상태를 업데이트하여 현재 단계에 맞는 컴포넌트를 보여줄 수 있어요.

- **장점**: 외부 의존성이 없고, 구현이 매우 간단합니다.
- **단점**: 단계가 리액트 메모리 상에서 관리되기 때문에 브라우저 UI로 새로고침, 뒤로/앞으로가기 시 폼 프로세스가 중단됩니다.

### 3.1.2. **Path Parameter를 이용한 방법**

```tsx
function MultiStepForm() {
  const { step } = useParams();

  return (
    <>
      {step === "title" && <TitleStep />}
      {step === "place" && <PlaceStep />}
    </>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/add-moim/:step" element={<MultiStepForm />} />
    </Routes>
  );
}
```

폼의 단계를 URL의 Path Parameter로 관리하는 방법이에요. `mouda.site/add-moim/title`, `mouda.site/add-moim/place`와 같이 URL 경로에 현재 단계를 포함하고, 그 단계를 기준으로 적절한 컴포넌트를 보여줄 수 있어요.

- **장점**: 각 단계를 별도의 URL로 관리하기 때문에 각 단계를 별도의 히스토리 스택에 쌓을 수 있어 단계 관리에 용이합니다.
- **단점**: 단계가 URL에 노출되기 때문에 사용자가 URL을 직접 입력하여 폼 프로세스 중간에 접근할 위험이 있습니다.

### 3.1.3. **Query Parameter를 이용한 방법**

```tsx
function MultiStepForm() {
  const [searchParams] = useSearchParams();
  const step = searchParams.get("step");

  return (
    <>
      {step === "title" && <TitleStep />}
      {step === "place" && <PlaceStep />}
    </>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/add-moim" element={<MultiStepForm />} />
    </Routes>
  );
}
```

폼의 단계를 URL의 Query Parameter로 관리하는 방법이에요. `mouda.site/add-moim?step=title`, `mouda.site/add-moim?step=place`와 같이 URL에 현재 단계를 포함하고, 그 단계를 기준으로 적절한 컴포넌트를 보여줄 수 있어요.

Path Parameter와 유사한 방식인 만큼, 장/단점도 동일해요.

### 3.1.4. History state**를 이용한 방법**

```tsx
function MultiStepForm() {
  const location = useLocation();
  const { step } = location.state || { step: "title" };

  return (
    <>
      {step === "title" && <TitleStep />}
      {step === "place" && <PlaceStep />}
    </>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/add-moim" element={<MultiStepForm />} />
    </Routes>
  );
}
```

폼의 단계를 React Router의 `state`를 사용해 단계를 관리하는 방법이에요. 단계를 이동할 때 동일한 주소로 페이지 이동을 하고, History state 내에 각 단계를 같이 주입해서 보내줍니다. 이 정보를 기준으로 단계에 맞는 컴포넌트를 보여줄 수 있어요.

- **장점**: 모든 단계를 동일한 URL로 관리 하면서도, 각 단계를 히스토리 스택에 별도로 쌓을 수 있어 단계 관리에 용이합니다.
- **단점**: React Router의 `state`는 브라우저의 `history` 객체에 저장됩니다. 사용자가 브라우저의 개발자 도구를 열어 history를 조작하면 단계 정보가 유실될 수 있는 가능성이 있습니다.

### 3.1.5. 최종 결정: History state 방식

여러 방식을 검토하고, 최종적으로 History state 방식을 선택했어요. 그 이유는 크게 UX의 일관성, Form의 완결성, 히스토리 스택 연동이었습니다.

개별 단계를 URL로 직접 접근하지 못하게 하여 사용자가 일관된 프로세스를 거치도록 하고, Form의 완결성을 보장할 수 있어요. 근본적으로 사용자가 폼 프로세스 중간에 접근할 수 있는 방법이 없기 때문에 때문에 예상치 못한 변수 케이스를 차단할 수 있다는 점이 매우 큰 장점이었어요.

또 URL 주소는 그대로 유지하면서도 각 단계를 브라우저 히스토리 스택에 쌓을 수 있기 때문에 브라우저 UI로 인한 폼 절차 중단을 예방할 수 있는 점이 큰 장점이었어요. 새로고침 시에도 동일한 단계를 유지할 수 있고, 뒤로/앞으로가기 시에도 히스토리 스택에 전/후 단계가 있기 때문에 자연스럽게 대응할 수 있다는 가치가 컸습니다.

## 3.2. useFunnel 커스텀 훅 개발

### 3.2.1. useFunnel 훅의 목적과 기능

모임 생성 페이지를 Multi Step Form으로 전환하고 나서, 다락방 또는 다른 생성 폼에서도 단계를 안정적으로 관리해줄 수 있는 커스텀 훅을 개발했어요. 복잡한 폼 관리 로직에서 단계 관리 로직을 분리함으로써, 폼의 제어나 데이터 유효성 검사 등 핵심 비즈니스 로직에 집중할 수 있어요.

```tsx
import { ReactNode, useCallback } from "react";
import { useLocation, useNavigate } from "react-router-dom";

export default function useFunnel<Step extends string | number>(
  firstStep: Step
) {
  const location = useLocation(); // 현재 URL의 상태를 가져옴
  const navigate = useNavigate(); // 스텝 간 이동을 처리

  // 현재 스텝을 추적. firstStep을 초기값으로 설정
  const currentStep: Step = location.state?.step || firstStep;

  const goBack = () => {
    navigate(-1);
  };

  // 동일한 path로 navigate. state로 step을 관리
  const goNextStep = (nextStep: Step) => {
    navigate(location.pathname, {
      state: { step: nextStep },
    });
  };

  interface FunnelProps {
    step: Record<Step, ReactNode>;
  }

  // 현재 스텝에 맞는 컴포넌트를 렌더링
  const Funnel = useCallback(
    (props: FunnelProps) => {
      const { step } = props;
      return step[currentStep];
    },
    [currentStep]
  );

  return { currentStep, goBack, goNextStep, Funnel };
}
```

**주요 기능**:

- **현재 스텝 추적**: `useLocation`을 이용해 React Router의 `state`에서 현재 단계 정보를 가져오며, 이 정보를 `currentStep` 변수로 관리합니다.
- **단계 이동**: `goNextStep` 함수를 통해 다음 단계로 이동하고, `goBack` 함수를 통해 이전 단계로 돌아갑니다. 이 이동은 `useNavigate`를 통해 이루어집니다.
- **렌더링**: `Funnel` 컴포넌트를 사용하여, 현재 스텝에 해당하는 컴포넌트를 동적으로 렌더링할 수 있습니다. `Funnel` 컴포넌트는 단계별로 어떤 컴포넌트를 렌더링할지를 정의하는 객체를 받아, 현재 단계에 맞는 컴포넌트를 화면에 표시합니다.

### 3.2.2. 사용 예시 및 이점

```tsx
export type MoimCreationStep =
  | "이름입력"
  | "장소선택"
  | "날짜/시간설정"
  | "최대인원설정"
  | "설명입력";

export default function MoimCreationPage() {
  const { Funnel, currentStep, goBack, goNextStep } =
    useFunnel<MoimCreationStep>("이름입력");

  const createMoim = (moimFormData: MoimFormData) => {
    // do something...
  };

  return (
    <Funnel
      step={{
        이름입력: <TitleStep onButtonClick={() => goNextStep("장소선택")} />,
        장소선택: (
          <PlaceStep onButtonClick={() => goNextStep("날짜/시간설정")} />
        ),
        "날짜/시간설정": (
          <DateAndTimeStep onButtonClick={() => goNextStep("최대인원설정")} />
        ),
        최대인원설정: (
          <MaxPeopleStep onButtonClick={() => goNextStep("설명입력")} />
        ),
        설명입력: (
          <DescriptionStep onButtonClick={() => createMoim(formData)} />
        ),
      }}
    />
  );
}
```

**이점**:

- **책임 분리**: 단계 관리 로직을 `useFunnel`에게 맡기고, 폼 관리와 같은 핵심 비즈니스 로직에 집중할 수 있습니다.
- **재사용성**: 다양한 Multi Step Form에서 동일한 방식으로 단계를 관리할 수 있습니다.

---

# 4. **상태 관리 문제 해결**

이번엔 Multi Step Form 구현 중 새로고침, 페이지 이탈 및 접근과 관련한 문제를 어떻게 해결했는지 공유해볼게요.

## 4.1. 새로고침 대응

사용자가 여러 단계를 거치며 정보를 입력하는 도중, 새로고침이 발생할 경우 어떻게 대응할지 고민했어요. 먼저 새로고침 발생시 가능한 대응 시나리오는 아래 세가지였어요.

1. 기존 상태 유지
2. 지금 단계만 리셋
3. 전부 리셋 & 처음 단계로 이동

그리고 새로고침이 트리거됐을 때 사용자의 의도가 무엇일지도 고민해봤는데요, 세 가지로 정리가 됐어요.

1. 폼 절차 재시작: 지금까지 입력한 거 다 취소하고 처음부터 작성하고 싶어!
2. 해당 단계만 리셋: 지금 단계에서 입력한 것만 다시 입력하고 싶어!
3. 실수: 아.. 실수로 새로고침 됐다..ㅜ

이 사용자 의도와 시나리오를 바탕으로 시뮬레이션을 구성해봤어요.

폼 절차를 재시작하길 원하는 사용자에게는 ‘전부 리셋 & 처음 단계로 이동’ 시나리오가 베스트겠죠. 반면 ‘기존 상태 유지’, ‘지금 단계 리셋’의 시나리오로 진행된다면 당황하거나 의아할 수 있어요.

<img src="/technical-writing/image6.png" width="700">

두 번째로 해당 단계만 리셋되길 원하는 사용자 입장에서는 ‘지금 단계만 리셋’되는 시나리오가 가장 이상적일 겁니다. ‘기존 상태 유지’ 시나리오에서는 의도와 다르니 의아할 것이고요. 문제는 ‘전부 리셋 & 처음 단계로 이동’ 시나리오입니다. 이때는 정말 경험이 안좋을 거에요.

<img src="/technical-writing/image7.png" width="700">

마지막은 실수로 새로고침을 트리거한 사용자에요. 시나리오별로 가장 드라마틱한 사용자 경험을 만들어낼 수 있는 케이스였어요. ‘기존 상태 유지’ 시나리오에서는 최소한 안심, 최대는 감동까지 받을 수 있다고 봅니다. 반면 ‘지금 단계만 리셋’, ‘전부 리셋 & 처음 단계로 이동’ 시나리오에서는 정말 큰 불만을 느낄 수 있어요. 경우에 따라서는 욕까지 나올 정도로 화가날 수도 있겠죠.

<img src="/technical-writing/image8.png" width="700">

그래서 결론은, 새로고침이 됐을 때 기존 상태를 유지하기로 했어요. 모든 사용자 의도 케이스에서 사용자 경험의 하방과 상방이 모두 높다고 판단했기 때문이에요.

<img src="/technical-writing/image9.png" width="700">

그럼 이제 문제는 이걸 어떻게 구현하나는 거겠죠. 이 부분을 이제 다뤄보도록 할게요.

### 4.1.1. How?

먼저 사용자 입력 정보를 저장할 곳을 결정해야 했어요. 저는 Web Storage API를 사용해 간단히 브라우저 저장소에 저장하기로 했습니다.

- 로컬 스토리지: 영구 저장소로서, 탭을 닫거나 브라우저를 종료해도 데이터가 유지되고, 모든 탭과 창에서 공유됩니다.
- 세션 스토리지: 임시 저장소로서, 탭 단위로 데이터를 저장하고 탭을 닫으면 데이터가 삭제됩니다.

이중 세션 스토리지를 사용하기로 했어요. 새로고침에 대응하기 위한 임시 저장소가 필요했기 때문입니다.

### 4.1.2. When?

그럼 이 정보를 언제 세션 스토리지에 저장하고 가져오는 지가 중요하겠죠.

먼저 저장하는 시점에 대한 고민부터 공유해볼게요. 총 두 가지 옵션이 있었어요.

1. 상태 업데이트: 상태가 업데이트될 때마다 스토리지도 업데이트!

   ```tsx
   const [formData, setFormData] = useState(initialState);
   useEffect(() => {
     sessionStorage.setItem("formData", formData);
   }, [formData]);
   ```

2. 새로고침: 새로고침이 발생하기 직전에만 스토리지에 저장한다!

   ```tsx
   const [formData, setFormData] = useState(initialState);
   // custom hook from 'react-router-dom'
   useBeforeUnload(() => {
     sessionStorage.setItem("key", formData.toString());
   });
   useEffect(() => {
     sessionStorage.removeItem("key");
   }, []);
   ```

이 중에서 아래와 같은 이유로 두 번째 옵션인 새로고침 시에만 저장하는 것으로 결정했어요.

1. Storage I/O 빈도: 새로고침이 발생했을 때만 스토리지 업데이트를 하면 스토리지 입출력 빈도를 줄일 수 있습니다.
2. 데이터 노출: 사용자가 입력한 데이터가 앱 외부에 노출되는 시간을 줄일 수 있습니다.
3. 지역 상태로서의 의의: 사용자 입력 데이터를 지역 상태로 관리하고 있는데, 이 데이터를 스토리지에 지속적으로 동기화한다는 것은 지역 상태로 관리하는 의도와 상충됩니다.

다음으로 저장소로부터 데이터를 가져오는 시점에 대한 고민도 공유해볼게요. 여기서도 두 가지 옵션이 있었어요.

1. 렌더링 이후: 초기 렌더링 이후 setter로 업데이트!

   ```tsx
   const [formData, setFormData] = useState(initialState);

   useBeforeUnload(() => {
     sessionStorage.setItem("key", formData.toString());
   });

   useEffect(() => {
     const item = sessionStorage.getItem("key");
     if (item !== null) {
       setFormData(JSON.parse(item));
       sessionStorage.removeItem("key");
     }
   }, []);
   ```

2. 렌더링 과정 중: 초기 렌더링 과정 중 initializer로 초기화!

   ```tsx
   const [formData, setFormData] = useState(() => {
     const item = sessionStorage.getItem("key");
     if (item !== null) {
       return JSON.parse(item);
     }
     return initialState;
   });

   useBeforeUnload(() => {
     sessionStorage.setItem("key", formData.toString());
   });

   useEffect(() => {
     sessionStorage.removeItem("key");
   }, []);
   ```

이 중에서 렌더링 과정 중 initializer로 초기화하는 방식을 선택했어요. 이유는 아래와 같아요.

1. 렌더링 낭비
   - 렌더링 이후 데이터를 가져오면, 총 두번의 렌더링이 발생합니다.
   - 렌더링 과정에 데이터를 가져오면, 렌더링이 한 번만 발생합니다.
2. UX
   - 렌더링 이후 데이터를 가져와 상태를 업데이트하면, 초기 렌더링 시 빈 값으로 렌더링되고 리렌더링이 될 때 값이 채워집니다.
   - 렌더링 과정에서 상태를 초기화하면, 처음부터 값이 채워진 화면을 볼 수 있습니다.

새로고침 시 상태를 유지하는 방법을 정리해볼게요. 새로고침이 트리거될 때 세션 스토리지에 정보를 저장하고, 페이지가 다시 로딩될 때 기존 데이터로 상태를 초기화하고 스토리지를 비웁니다.

<img src="/technical-writing/image10.png" width="700">

## 4.2. useStatePersist 커스텀 훅 개발

각 단계에서 입력받은 정보를 새로고침 시에도 유지할 수 있도록 스토리지로 관리하는 로직을 커스텀 훅 `useStatePersist`으로 분리했어요.

### 4.2.1. useStatePersist 훅의 목적과 기능

상태 유지 로직을 추상화함으로써, 상태 관리 및 핵심 비즈니스 로직에 집중할 수 있도록 했어요.

```tsx
import { useCallback, useEffect, useState } from "react";
import { useBeforeUnload } from "react-router-dom";

interface UseStatePersistParams<StateType> {
  key: string;
  initialState: StateType | (() => StateType);
}

export default function useStatePersist<StateType>({
  key,
  initialState,
}: UseStatePersistParams<StateType>): [
  StateType,
  React.Dispatch<React.SetStateAction<StateType>>
] {
  // 세션 스토리지에서 지정된 key로 저장된 데이터를 가져옴
  // 만약 스토리지에 값이 있으면 JSON으로 파싱하여 반환하고, 없으면 initialState 값을 반환
  const getStoredValue = (): StateType => {
    try {
      const item = sessionStorage.getItem(key);
      if (item !== null) {
        return JSON.parse(item);
      }
    } catch (error) {
      console.error(`Error reading from session storage`, error);
    }

    return typeof initialState === "function"
      ? (initialState as () => StateType)()
      : initialState;
  };

  const setStoredValue = (value: StateType) => {
    try {
      sessionStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(`Error writing to session storage`, error);
    }
  };

  const removeStoredValue = useCallback(() => {
    try {
      sessionStorage.removeItem(key);
    } catch (error) {
      console.error(`Error removing item from  session storage`, error);
    }
  }, [key]);

  const [state, setState] = useState<StateType>(getStoredValue);

  // 페이지를 떠나거나 새로고침할 때, 상태를 세션 스토리지에 저장
  useBeforeUnload(() => {
    setStoredValue(state);
  });

  // 컴포넌트가 처음 렌더링된 후, 세션 스토리지에 저장된 값을 삭제
  useEffect(() => {
    removeStoredValue();
  }, [removeStoredValue]);

  return [state, setState];
}
```

**주요 기능:**

1. **상태 초기화 및 복원**: 훅이 처음 실행될 때, `sessionStorage`에 저장된 값이 있으면 이를 불러와서 초기 상태로 설정합니다. 만약 저장된 값이 없으면 전달된 `initialState`로 상태를 초기화합니다.
2. **상태 저장**: 페이지가 새로고침되거나 페이지를 이탈하기 전(`beforeunload` 이벤트 발생 시), 현재 상태를 `sessionStorage`에 저장합니다. 이를 통해 리액트 상태를 유지할 수 있습니다.
3. **스토리지 비우기**: 컴포넌트가 렌더링된 이후에는 세션 스토리지에 저장된 데이터를 제거하여, 해당 데이터를 다른 곳에서 접근할 수 없도록 합니다.

## 4.3. 페이지 이탈/접근 대응

사용자가 페이지를 이탈하는 케이스는 총 세가지에요.

1. 앱 내 routing: 앱 내의 navigation을 이용한 페이지 이탈
2. 브라우저 UI 뒤로가기: 브라우저의 뒤로가기 버튼으로 페이지 이탈
3. URL 직접 입력: 브라우저 URL 입력란에 다른 주소를 입력해 페이지 이탈

이 중에서 세 번째 케이스가 문제가 되었어요. 그 이유는 새로고침 대응을 위한 처리로 인한 사이드 이펙트로 인한 문제였어요. 아까 새로고침이 트리거될 때 세션 스토리지에 데이터를 저장한다고 했는데요, 그 새로고침을 감지하는 방식이 `beforeunload` 이벤트의 발생이었잖아요. 근데 이 이벤트가 발생하는 시점이 정확히는 새로고침이 트리거될 때가 아니라, 문서가 언로드되기 직전이기 때문에 사용자가 URL로 다른 페이지를 접근하면 `beforeunload` 이벤트가 발생하게 되어 데이터가 세션 스토리지에 저장된 채로 떠나게 되는 문제가 있었어요.

이로 인한 파생되는 문제는 총 두 가지였어요.

1. 데이터 노출: 사용자가 입력한 데이터가 스토리지에 저장되어 외부에 노출된다.
2. 페이지 재접근 시 초기화: 페이지에 다시 접근할 때, 스토리지의 데이터로 상태가 초기화된다.

안타깝게도 새로고침과 페이지 이탈을 구분할 수 있는 방법이 없어 데이터가 노출되는 문제는 해결할 수 없었어요. 대신, 서비스에 정말 치명적인 결함을 유발하는 페이지 재접근 시의 문제에 집중했습니다.

아이디어는 이랬어요. ‘모든 단계가 동일한 페이지니까, 현재 단계와 navigation type만 알면 페이지 접근을 감지할 수 있겠다.’ 현재 단계가 첫 번째 단계이면서 히스토리 스택이 PUSH라면 페이지에 새로 접근한 경우로 판정할 수 있고, 이때 스토리지 데이터를 삭제하면 되는 거였어요.

<img src="/technical-writing/image11.png" width="700">

그럼 여기서 navigation type을 알 수 있는 방법을 찾아야 했어요.

첫 번째로 고려한 방법은 NavigateEvent의 `navigationType`이었어요. 그러나 아직 실험 단계에 있는 기술이고, 브라우저 호환성 이슈가 있어 이 방법은 사용하지 않기로 했습니다.

<img src="/technical-writing/image12.png" width="700">

그래서 다음으로 고려한 방법은 React Router의 `useNavigationType` 훅이었어요. 이 훅으로 히스토리 스택이 새로 쌓이는 경우를 감지하고 전에 삭제하지 못했던 데이터를 삭제할 수 있게 됐어요.

<img src="/technical-writing/image13.png" width="700">

```tsx
const navigationType = useNavigationType();

const isNewMoimCreation =
  currentStep === "first-step" && navigationType === "PUSH";

if (isNewMoimCreation) {
  sessionStorage.removeItem("key");
}
```

이 훅은 꽤 괜찮은 방법이었지만 아쉽게도 완벽한 해결책은 아니었어요. 이 훅을 통해 앱 내의 routing을 이용한 페이지 접근은 감지할 수 있었지만, URL 직접 입력으로 페이지를 접근한 경우에는 이 훅이 감지할 수 없었습니다.

`useNavigationType`은 React Router 훅이고, React Router는 React에서 동작합니다. 그리고 URL 직접 입력으로 페이지를 접근한 시점은 React가 동작하기 전이죠. 그렇기 때문에 URL 직접 입력을 감지할 수 있는 브라우저의 API가 필요했습니다.

다행히도 Performance API에 navigation type을 알 수 있는 프로퍼티가 있었어요. 이 프로퍼티를 사용해 페이지 접근의 타입을 판단할 수 있게 됐어요. URL 입력으로 페이지를 접근한 경우, navigationType의 값이 `"navigate"`가 됩니다.

<img src="/technical-writing/image14.png" width="700">

```tsx
const getBrowserNavigationType = ():
  | "navigate"
  | "reload"
  | "back_forward"
  | "prerender"
  | undefined => {
  const navEntries = performance.getEntriesByType("navigation");
  if (navEntries.length > 0) {
    const navEntry = navEntries[0] as PerformanceNavigationTiming;
    return navEntry.type;
  }
};

export const isApprochedByUrl = () => {
  return getBrowserNavigationType() === "navigate";
};
```

정리하면, `useNavigationType` 훅과 Performance API의 `PerformanceNavigationTiming.type`를 활용해 페이지 접근을 감지할 수 있었어요. 궁극적으로는 URL로 페이지를 이탈하는 경우 발생하는 ‘페이지 재접근 시 초기화’되는 문제를 해결한 것이었습니다.

<img src="/technical-writing/image15.png" width="700">

```tsx
const isNewMoimCreation =
  currentStep === "first-step" &&
  (navigationType === "PUSH" || isApprochedByUrl());

if (isNewMoimCreation) {
  sessionStorage.removeItem("moimCreationInfo");
}
```

---

# 5. **UX 개선 및 모바일 환경 최적화**

이 글의 서론에서 다룬 것 처럼, Multi Step Form가 UX적으로 매우 큰 가치가 있지만 UX 단점도 있어요. 사용자가 프로세스를 파악하기 어려워 입력 도중에 답답함을 느끼고 절차를 포기할 수 있다는 문제가 있었어요. 이를 개선하기 위해 Step Indicator를 개발해 사용자가 전체 단계와 현재 단계를 시각적으로 인지할 수 있도록 했어요.

다음으로는 모바일 환경에서의 UX 최적화를 고민했어요. 다음 단계로 넘어가기 위한 CTA 버튼을 화면 하단에 배치했는데, 모바일 가상 키보드 UI로 인해 하단 버튼이 가려져 다음 단계로 넘어가는 과정이 번거롭다는 문제가 있었어요. 이 문제를 `visualViewport` API를 통해 해결했어요. `innerHeight`와 `visualViewport.height`로 키보드 영역을 계산할 수 있기 때문에 이 키보드 영역 위로 버튼이 올라오도록 했어요.

```tsx
const [keyboardHeight, setKeyboardHeight] = useState(0);

useEffect(() => {
  const handleResize = () => {
    if (window.visualViewport) {
      const keyboardHeight = window.innerHeight - window.visualViewport.height;
      setKeyboardHeight(Math.max(0, keyboardHeight));
    }
  };

  window.visualViewport?.addEventListener("resize", handleResize);
  return () =>
    window.visualViewport?.removeEventListener("resize", handleResize);
}, []);
```

- `innerHeight`는 전체 뷰포트 높이를 나타냅니다.
- `visualViewport.height`는 실제로 보이는 화면 높이를 의미합니다.

마지막으로 해결한 문제는 iOS 환경에서 인풋에 포커스 시 화면이 확대되는 문제였어요. 단계를 넘어갈 때마다 인풋에 자동으로 포커스 해주고, 사용자가 빠르게 입력하고 넘어갈 수 있도록 했는데, 화면이 확대되어 하단 버튼이 보이지 않아 사용자가 화면을 다시 조정해야 했어요. UX적으로는 큰 문제였지만, 사실 정말 해결 방법은 정말 사소했어요. iOS 환경에서 화면이 확대되는 원인이 font-size가 16px보다 작기 때문이었어요. 이 인풋의 font-size를 16px로 높여주어 화면이 확대되지 않도록 조치해 UX가 저하되는 문제를 해결할 수 있었어요.

---

# 6. **결론**

모우다 팀이 모임 생성 UI를 Multi Step Form로 전환하면서 겪은 문제와 해결 과정을 공유했는데, 이런 질문이 있을 수 있다고 생각해요.

> “모임 생성 UI가 이렇게 Multi Step Form까지 도입할 정도로 복잡한 Form인가요?”

물론 모임 생성 Form이 금융 서비스나 어떤 행정 서류처럼 엄청 복잡하진 않아요. 하지만 우리 모우다의 핵심 가치는 ‘모임을 쉽게’입니다. 사용자가 모임을 쉽게 만들고 참여할 수 있도록 돕는 게 우리 팀의 목표에요. 복잡한 모임 여정 속에서, 모임 생성을 부담 없이 쉽게 할 수 있도록 최대한의 노력을 기울인 결과가 Multi Step Form이었습니다. 문제를 해결하기 위해, 가치를 만들어내기 위해 고민하고 구현해내는 것, 그게 개발자 아닐까요?

---

# Refs

https://www.giosg.com/blog/single-step-forms-vs-multi-step-forms

https://reactrouter.com/en/main

https://www.slash.page/ko/libraries/react/use-funnel/README.i18n

https://use-funnel.slash.page/ko

https://developer.mozilla.org/ko/docs/Web/API/Web_Storage_API

https://developer.mozilla.org/en-US/docs/Web/API/Performance

https://developer.mozilla.org/en-US/docs/Web/API/VisualViewport
