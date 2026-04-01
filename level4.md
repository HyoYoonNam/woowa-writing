토스트 상태관리를 통해 배우는 선언적 프로그래밍

## 들어가며

사용자에게 "저장되었습니다", "복사되었습니다"와 같은 간단한 피드백을 전달할 때 토스트 알림은 매우 효과적입니다. 이 글에서는 토스트 상태관리를 선언적으로 구현하는 방법과 실제 적용 사례를 공유합니다.

## 선언적 프로그래밍에 대한 흔한 오해

많은 개발자들이 "함수로 추상화하면 선언적"이라고 생각하지만, 이것은 정확하지 않습니다. 함수 사용 자체가 선언적 프로그래밍을 보장하지는 않습니다.

### 절차적 접근의 한계

다음은 토스트를 절차적으로 구현한 예시입니다:

```tsx
function showToast(message: string) {
  const toastElement = document.createElement('div');
  toastElement.textContent = message;
  toastElement.style.position = 'fixed';
  toastElement.style.top = '20px';
  // ... 스타일 설정
  
  document.body.appendChild(toastElement);
  
  setTimeout(() => {
    toastElement.style.opacity = '0';
    setTimeout(() => {
      document.body.removeChild(toastElement);
    }, 300);
  }, 3000);
}
```

이 코드는 "DOM 요소 생성 → 스타일 적용 → 타이머 설정"이라는 시간적 순서에 집중합니다. 함수를 사용하고 있지만 여전히 절차적 사고방식입니다.

### How vs What: 핵심 차이점

**절차적 코드 - "어떻게(How)"에 집중:**
```tsx
function handleSave() {
  saveData();
  setToastMessage('저장되었습니다!');
  setIsToastVisible(true);
  setTimeout(() => {
    setIsToastVisible(false);
  }, 3000);
}
```

**선언적 코드 - "무엇을(What)"에 집중:**
```tsx
function handleSave() {
  saveData();
  showToast('저장되었습니다!');
}
```

선언적 접근은 "저장 완료 시 토스트 표시"라는 관계를 명확히 선언합니다.

### 추상화의 계층

선언적 프로그래밍은 여러 계층에서 작동합니다:

```tsx
const showToast = useCallback((newMessage: string) => {
  setMessage(newMessage);
  setIsVisible(true);
}, []);
```

이 코드도 메시지 설정과 가시성 변경을 하나의 의도로 추상화한 선언적 접근입니다.

React의 `setState`도 마찬가지입니다. 내부적으로는 상태 비교, 변경 감지, 리렌더링 스케줄링, DOM 조작 등 복잡한 과정을 거치지만, 개발자는 단순히 "상태를 업데이트한다"는 의도만 표현합니다.

흥미로운 예시로, "토스트를 3초간 보여라"라는 문장도 다음의 복잡한 과정을 추상화한 선언적 표현입니다:
- "3초" = 1초의 3배 시간
- "보여라" = 가시성 설정, UI 렌더링, 애니메이션 실행
- "토스트" = 특정 위치와 스타일의 UI 컴포넌트

## 구현 방식 비교: Before & After

### Before: 명령적 구현

토스트 상태관리 도입 전에는 각 컴포넌트에서 모든 것을 직접 관리했습니다:

```tsx
function SaveButton() {
  const [isToastVisible, setIsToastVisible] = useState(false);
  const [toastMessage, setToastMessage] = useState('');

  const handleSave = () => {
    saveData();
    setToastMessage('저장되었습니다!');
    setIsToastVisible(true);
    
    setTimeout(() => {
      setIsToastVisible(false);
    }, 3000);
  };

  return (
    <>
      <button onClick={handleSave}>저장</button>
      {isToastVisible && (
        <div style={{
          position: 'fixed',
          top: '20px',
          // ... 스타일들
        }}>
          {toastMessage}
        </div>
      )}
    </>
  );
}
```

**문제점:**
- 코드 중복 발생
- 컴포넌트마다 스타일과 동작이 다를 수 있음
- 변경사항 반영 시 모든 컴포넌트 수정 필요
- 컴포넌트 복잡도 증가

### After: 선언적 구현

커스텀 훅과 Context를 활용한 중앙화된 관리:

```tsx
// 토스트 로직 중앙화
function useToastState() {
  const [isVisible, setIsVisible] = useState(false);
  const [message, setMessage] = useState('');

  const showToast = useCallback((newMessage: string) => {
    setMessage(newMessage);
    setIsVisible(true);
  }, []);

  const hideToast = useCallback(() => {
    setIsVisible(false);
  }, []);

  useEffect(() => {
    if (isVisible) {
      const timer = setTimeout(hideToast, 3000);
      return () => clearTimeout(timer);
    }
  }, [isVisible, hideToast]);

  return { isVisible, message, showToast, hideToast };
}

// 사용하는 곳은 매우 간단
function SaveButton() {
  const { showToast } = useToastActionsContext();

  const handleSave = () => {
    saveData();
    showToast('저장되었습니다!');
  };

  return <button onClick={handleSave}>저장</button>;
}
```

**개선사항:**
- 중복 제거 및 로직 중앙화
- 일관된 스타일과 동작 보장
- 한 곳만 수정하면 전체 반영
- 간결한 사용법 (`showToast('메시지')` 한 줄)

## Context를 통한 상태와 액션 분리

상태와 액션을 별도 Context로 분리하여 관심사를 명확히 구분합니다:

```tsx
// 상태 Context (읽기 전용)
const ToastStateContext = createContext({
  isVisible: boolean;
  message: string;
});

// 액션 Context (함수들)
const ToastActionsContext = createContext({
  showToast: (message: string) => void;
  hideToast: () => void;
});
```

**분리의 장점:**
- 재사용성 향상
- 한 곳에서 수정하면 전체 반영
- 명확한 책임 분리

**주의할 점:**

과도한 추상화는 오히려 복잡성을 증가시킬 수 있습니다:

```tsx
<ToastProvider
  duration={5000}
  position="top-right"
  animation="slide-in"
  backgroundColor="#4CAF50"
  textColor="#FFFFFF"
  /* 옵션이 너무 많으면... */
>
  {children}
</ToastProvider>
```

화면마다 토스트가 크게 다르다면 공통화가 오히려 부담이 될 수 있습니다. 비즈니스 요구사항과 향후 변경 가능성을 고려하여 적절한 추상화 레벨을 선택해야 합니다.

## 점진적 개선 방향

### 1단계: 타입 안전성 강화

```tsx
interface ToastState {
  isVisible: boolean;
  message: string;
}

const useToastState = () => {
  const [toastState, setToastState] = useState<ToastState>({
    isVisible: false,
    message: '',
  });

  const showToast = useCallback((message: string) => {
    setToastState({ isVisible: true, message });
  }, []);

  const hideToast = useCallback(() => {
    setToastState({ isVisible: false, message: '' });
  }, []);

  useEffect(() => {
    if (toastState.isVisible) {
      const timer = setTimeout(hideToast, 3000);
      return () => clearTimeout(timer);
    }
  }, [toastState.isVisible, hideToast]);

  return { ...toastState, showToast, hideToast };
};
```

### 2단계: 비즈니스 의도 명확화

```tsx
const useToast = () => {
  const { showToast } = useToastActionsContext();
  
  return {
    showSuccess: (message: string) => showToast(message),
    showError: (message: string) => showToast(message),
    showInfo: (message: string) => showToast(message),
  };
};

// 사용 시 의도가 명확
function SaveButton() {
  const { showSuccess, showError } = useToast();

  const handleSave = async () => {
    try {
      await saveData();
      showSuccess('저장되었습니다!');
    } catch (error) {
      showError('저장에 실패했습니다.');
    }
  };

  return <button onClick={handleSave}>저장</button>;
}
```

### 3단계: 엄격한 상태 모델링 (필요시)

불가능한 상태를 타입 시스템으로 차단:

```tsx
type StrictToastState = 
  | { type: 'hidden' }
  | { type: 'visible'; message: string };

const useStrictToastState = () => {
  const [state, setState] = useState<StrictToastState>({ type: 'hidden' });

  const showToast = useCallback((message: string) => {
    setState({ type: 'visible', message });
  }, []);

  const hideToast = useCallback(() => {
    setState({ type: 'hidden' });
  }, []);

  return {
    isVisible: state.type === 'visible',
    message: state.type === 'visible' ? state.message : '',
    showToast,
    hideToast,
  };
};
```

## 선언적 사고의 본질

선언적 프로그래밍의 핵심은 "시간적 순서에서 관계로"의 사고 전환입니다.

### 관계적 사고 예시

```tsx
// 절차적: 시간적 순서
const handleUserAction = () => {
  performAction();
  if (actionSucceeded) {
    showToast('성공했습니다!');
  }
};

// 선언적: 상태 간 관계
const handleUserAction = () => {
  performAction();
  showToast('성공했습니다!');
};
```

### 상태 모델링 철학

1. **가능한 상태 명시**: 토스트는 "숨김" 또는 "표시됨"만 존재
2. **불가능한 상태 차단**: "표시됨이면서 메시지 없음"은 불가능
3. **상태 전환을 관계로 표현**: "표시됨"은 3초 후 자동으로 "숨김"으로 전환

```tsx
const useToastState = () => {
  const [state, setState] = useState<'hidden' | { type: 'visible'; message: string }>('hidden');

  const showToast = useCallback((message: string) => {
    setState({ type: 'visible', message });
  }, []);

  const hideToast = useCallback(() => {
    setState('hidden');
  }, []);

  useEffect(() => {
    if (state !== 'hidden') {
      const timer = setTimeout(() => setState('hidden'), 3000);
      return () => clearTimeout(timer);
    }
  }, [state]);

  return {
    isVisible: state !== 'hidden',
    message: state !== 'hidden' ? state.message : '',
    showToast,
    hideToast,
  };
};
```

## 마치며

선언적 프로그래밍은 단순한 문법이나 도구가 아니라 사고방식이자 철학입니다. 토스트 상태관리를 통해 이러한 철학을 이해하고 적용하면, 개발자는 복잡한 구현 세부사항 대신 "무엇을 보여줄지"라는 본질에 집중할 수 있습니다.

중요한 것은 무조건적인 추상화가 아니라, 비즈니스 요구사항과 변경 가능성을 고려하여 적절한 수준의 선언적 설계를 적용하는 것입니다.

