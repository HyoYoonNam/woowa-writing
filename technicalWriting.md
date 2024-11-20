# 리액트에서의 클로저: 개념부터 활용까지

## 1. 클로저의 기본 개념과 리액트에서의 의미

### 1.1 JavaScript에서의 클로저 정의


클로저를 이해하기 위해서는 먼저 스코프(scope)의 개념을 파악해야 합니다. 스코프는 변수가 정의되고 접근될 수 있는 코드 영역을 나타냅니다. 변수는 이 스코프 내에서 선언되고 사용되는 데이터 저장소입니다. 각 프로그래밍 언어는 고유한 스코프 규칙을 가지고 있어, 변수가 어느 스코프에 속하는지를 결정합니다.

스코프 안에는 다른 스코프가 올 수도 있습니다. 이렇게 스코프가 중첩될 때 표현식이나 문(statement)는 해당 레벨의 스코프 혹은 더 높거나 바깥 레벨에 있는 변수에만 접근할 수 있고, 낮거나 안쪽 레벨 스코프에 있는 변수에는 접근할 수 없습니다. 대부분 프로그래밍 언어가 이런 작동 방식을 취하며 이를 렉시컬 스코프라고 합니다.

이러한 개념을 바탕으로, 클로저는 함수와 그 함수가 선언됐을 때의 렉시컬 환경과의 조합이라고 할 수 있습니다. 이는 함수가 생성될 때의 스코프를 기억하고 접근할 수 있게 해주는 메커니즘입니다.

### 1.2 리액트 컨텍스트에서의 클로저의 역할

리액트에서 클로저는 컴포넌트의 상태 관리와 이벤트 핸들링의 일관성 유지에 활용됩니다. 특히 함수형 컴포넌트와 함께 사용되는 훅(Hooks)에서 클로저의 역할은 더욱 중요해집니다.

예를 들어, `useState` 훅을 사용할 때 상태 업데이트 함수는 클로저를 형성합니다. 이를 통해 컴포넌트의 이전 상태 값을 참조하고 업데이트할 수 있습니다. 또한, `useEffect` 훅 내부에서 외부 스코프의 변수를 참조할 때 클로저가 생성되며, 이는 부수 효과를 관리하는 데 있어서 중요한 역할을 합니다.
클로저는 리액트에서 비동기 작업을 처리할 때도 유용하게 사용됩니다. 예를 들어, 비동기 요청의 결과를 처리하는 콜백 함수에서 컴포넌트의 상태나 `props`에 접근할 때 클로저가 활용됩니다.

### 1.3 클로저와 함수형 컴포넌트의 관계

함수형 컴포넌트에서 클로저는 상태 관리의 핵심 메커니즘입니다. 클래스형 컴포넌트와 달리 함수형 컴포넌트는 렌더링이 발생할 때마다 함수 자체가 다시 호출됩니다. 이때 이전 상태를 기억하고 있어야 하는데, 이 문제를 클로저를 통해 해결합니다.
`useState` 훅은 클로저를 활용하여 함수형 컴포넌트의 상태를 관리합니다. `useState`가 반환하는 상태 업데이트 함수는 클로저를 형성하여 컴포넌트의 최신 상태를 참조할 수 있게 합니다. 이를 통해 함수형 컴포넌트가 여러 번 렌더링되더라도 상태를 올바르게 유지하고 업데이트할 수 있습니다.
또한, `useEffect`, `useCallback`, `useMemo` 등의 다른 훅들도 클로저를 활용하여 이전 렌더링의 값들을 기억하고 접근합니다. 이를 통해 함수형 컴포넌트에서도 복잡한 상태 관리와 최적화가 가능해집니다.

## 2. 리액트 훅과 클로저

리액트 훅은 함수형 컴포넌트에서 상태 관리와 부수 효과를 처리하는 강력한 도구입니다. 이들은 내부적으로 클로저를 활용하여 효과적으로 작동합니다. 클로저의 특성을 이해하면 훅의 동작 원리를 더 깊이 이해하고 효과적으로 활용할 수 있습니다.

### 2.1 useState와 클로저

useState 훅은 클로저를 활용하여 컴포넌트의 상태를 관리합니다. 이는 함수형 컴포넌트가 여러 번 호출되더라도 상태를 올바르게 유지할 수 있게 해줍니다.

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount((prevCount) => prevCount + 1);
  };

  const incrementByTen = () => {
    for (let i = 0; i < 10; i++) {
      setCount((prevCount) => prevCount + 1);
    }
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={incrementByTen}>Increment by 10</button>
    </div>
  );
}
```

이 예제에서 `setCount` 함수는 클로저를 형성하여 `count` 상태에 접근합니다. `increment` 함수에서는 단순히 `count + 1`을 사용하지 않고 `prevCount => prevCount + 1`와 같은 함수 형태를 사용합니다. 이는 클로저를 통해 최신의 상태 값을 참조할 수 있게 해줍니다.
특히 `incrementByTen` 함수에서 이 방식의 중요성이 드러납니다. 만약 `setCount(count + 1)`을 사용했다면, 루프가 실행되는 동안 `count` 값이 변경되지 않아 결과적으로 1만 증가하게 됩니다. 하지만 클로저를 활용한 함수 형태를 사용함으로써, 각 호출마다 최신의 상태를 참조하여 정확히 10씩 증가시킬 수 있습니다.

### 2.2 useEffect에서의 클로저 활용

`useEffect` 훅은 클로저를 사용하여 이전 렌더의 값을 기억하고 정리(cleanup) 함수를 구현합니다. 이는 비동기 작업이나 구독 관리에 특히 유용합니다.

```javascript
function DataFetcher({ id }) {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    let isCancelled = false;
    setIsLoading(true);
    setError(null);

    const fetchData = async () => {
      try {
        const result = await fetchDataFromAPI(id);
        if (!isCancelled) {
          setData(result);
          setIsLoading(false);
        }
      } catch (err) {
        if (!isCancelled) {
          setError(err.message);
          setIsLoading(false);
        }
      }
    };

    fetchData();

    return () => {
      isCancelled = true;
    };
  }, [id]);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return <div>{data ? data.name : "No data"}</div>;
}
```

이 예제에서 `useEffect`의 정리 함수는 클로저를 통해 `isCancelled` 변수에 접근합니다. 이를 통해 컴포넌트가 언마운트되거나 `id`가 변경될 때 비동기 작업을 안전하게 취소할 수 있습니다. 클로저를 사용함으로써 `isCancelled` 변수는 비동기 작업이 완료될 때까지 유지되며, 컴포넌트의 생명주기와 무관하게 정확한 상태를 반영할 수 있습니다.
또한, `setData`와 `setError` 함수 호출 전에 `isCancelled`를 확인함으로써, 컴포넌트가 이미 언마운트된 후에 상태를 업데이트하는 것을 방지합니다. 이는 메모리 누수와 "Can't perform a React state update on an unmounted component" 경고를 예방하는 데 도움이 됩니다.

### 2.3 useCallback과 클로저의 상호작용

`useCallback` 훅은 메모이제이션된 콜백 함수를 생성하고 의존성을 관리하는 데 클로저를 활용합니다. 이는 불필요한 리렌더링을 방지하고 성능을 최적화하는 데 중요한 역할을 합니다.

```javascript
function SearchComponent({ query, onResultSelect }) {
  const [results, setResults] = useState([]);
  const [searchHistory, setSearchHistory] = useState([]);

  const searchAPI = useCallback((searchQuery) => {
    setSearchHistory((prev) => [...prev, searchQuery]);
    return fetch(`/api/search?q=${searchQuery}`)
      .then((response) => response.json())
      .then((data) => setResults(data));
  }, []);

  const handleResultSelect = useCallback(
    (result) => {
      onResultSelect(result);
      setSearchHistory((prev) => [...prev, result.title]);
    },
    [onResultSelect]
  );

  useEffect(() => {
    searchAPI(query);
  }, [query, searchAPI]);

  return (
    <div>
      <ResultsList results={results} onSelect={handleResultSelect} />
      <SearchHistory history={searchHistory} />
    </div>
  );
}
```

이 예제에서 `useCallback`은 클로저를 사용하여 `searchAPI`와 `handleResultSelect` 함수를 메모이제이션합니다.
`searchAPI` 함수는 빈 의존성 배열 `[]`을 가지고 있어, 컴포넌트가 리렌더링되어도 항상 동일한 함수 인스턴스를 반환합니다. 이 함수는 클로저를 통해 `setSearchHistory`와 `setResults`에 접근하여 상태를 업데이트합니다.

`handleResultSelect` 함수는 `onResultSelect` prop을 의존성으로 가집니다. 이 함수는 클로저를 통해 `onResultSelect` prop과 `setSearchHistory` 함수에 접근합니다. `onResultSelect` prop이 변경될 때만 새로운 함수 인스턴스가 생성되므로, 불필요한 리렌더링을 방지할 수 있습니다.

이러한 방식으로 `useCallback`과 클로저를 함께 사용하면, 컴포넌트의 성능을 최적화하면서도 필요한 상태와 props에 안전하게 접근할 수 있습니다. 특히 자식 컴포넌트에 콜백을 prop으로 전달할 때 유용하며, `React.memo`와 함께 사용하면 더욱 효과적인 최적화를 달성할 수 있습니다.

이처럼 리액트 훅은 클로저를 활용하여 상태 관리, 부수 효과 처리, 그리고 성능 최적화를 효과적으로 수행합니다. 클로저의 특성을 잘 이해하고 활용함으로써, 더 안정적이고 효율적인 리액트 애플리케이션을 개발할 수 있습니다.

## 3. 클로저를 활용한 상태 관리

클로저는 리액트에서 복잡한 상태 로직을 캡슐화하고 비동기 상황에서도 안전하게 상태를 관리할 수 있게 해주는 강력한 도구입니다. 클로저의 특성을 이해하고 활용함으로써, 더 안정적이고 유지보수가 용이한 리액트 애플리케이션을 개발할 수 있습니다.

### 3.1 함수형 업데이트와 클로저

함수형 업데이트는 이전 상태를 안전하게 참조하여 업데이트하는 방법입니다. 클로저를 활용하여 구현되며, 특히 비동기 상황이나 여러 상태 업데이트가 연속적으로 일어나는 경우에 유용합니다.

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount((prevCount) => prevCount + 1);
  };

  const incrementAsync = () => {
    setTimeout(() => {
      setCount((prevCount) => prevCount + 1);
    }, 1000);
  };

  const incrementMultiple = () => {
    setCount((prevCount) => prevCount + 1);
    setCount((prevCount) => prevCount + 1);
    setCount((prevCount) => prevCount + 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={incrementAsync}>Increment Async</button>
      <button onClick={incrementMultiple}>Increment by 3</button>
    </div>
  );
}
```

이 예제에서 `increment` 함수는 기본적인 함수형 업데이트를 보여줍니다. `incrementAsync` 함수는 비동기 상황에서도 클로저를 통해 최신 상태를 참조할 수 있음을 보여줍니다. `incrementMultiple` 함수는 연속적인 상태 업데이트에서 클로저의 중요성을 보여줍니다. 각 `setCount` 호출은 이전 상태를 기반으로 새로운 상태를 계산하므로, 결과적으로 카운트가 3 증가합니다.

함수형 업데이트를 사용하지 않고 `setCount(count + 1)`을 사용했다면, `incrementMultiple` 함수에서 카운트가 1만 증가했을 것입니다. 클로저를 활용한 함수형 업데이트는 이러한 문제를 해결하고 예측 가능한 상태 업데이트를 가능하게 합니다.

### 3.2 클로저를 이용한 비동기 상태 관리

클로저는 비동기 작업에서 최신 상태를 올바르게 참조하는 데 유용합니다. 특히 `useRef`와 함께 사용하면 비동기 콜백에서도 항상 최신 상태를 참조할 수 있습니다.

```javascript
function AsyncCounter() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);

  useEffect(() => {
    countRef.current = count;
  }, [count]);

  const incrementAsync = () => {
    setTimeout(() => {
      setCount(countRef.current + 1);
    }, 1000);
  };

  const incrementAsyncMultiple = () => {
    for (let i = 0; i < 3; i++) {
      setTimeout(() => {
        setCount((prevCount) => prevCount + 1);
      }, 1000 * (i + 1));
    }
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementAsync}>Increment Async</button>
      <button onClick={incrementAsyncMultiple}>Increment Async Multiple</button>
    </div>
  );
}
```

이 예제에서 `countRef`는 클로저를 통해 항상 최신의 `count` 값을 참조합니다. `incrementAsync` 함수는 countRef를 사용하여 비동기 작업에서도 최신 상태를 참조합니다. `incrementAsyncMultiple` 함수는 함수형 업데이트를 사용하여 여러 비동기 업데이트를 올바르게 처리합니다.
이러한 방식으로 클로저를 활용하면, 복잡한 비동기 상황에서도 상태를 안전하게 관리할 수 있습니다.

### 3.3 커스텀 훅에서의 클로저 활용

클로저를 활용하여 상태 로직을 재사용 가능한 형태로 추상화할 수 있습니다. 커스텀 훅을 만들어 여러 컴포넌트에서 동일한 상태 로직을 재사용할 수 있습니다.

```javascript
function useCounter(initialCount = 0, step = 1) {
  const [count, setCount] = useState(initialCount);

  const increment = useCallback(() => {
    setCount((prevCount) => prevCount + step);
  }, [step]);

  const decrement = useCallback(() => {
    setCount((prevCount) => prevCount - step);
  }, [step]);

  const reset = useCallback(() => {
    setCount(initialCount);
  }, [initialCount]);

  const setCountWithLimit = useCallback((newCount) => {
    setCount((prevCount) => {
      const limitedCount = Math.max(0, Math.min(newCount, 100));
      return limitedCount;
    });
  }, []);

  return { count, increment, decrement, reset, setCountWithLimit };
}

function CounterComponent() {
  const { count, increment, decrement, reset, setCountWithLimit } = useCounter(
    0,
    2
  );

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+2</button>
      <button onClick={decrement}>-2</button>
      <button onClick={reset}>Reset</button>
      <button onClick={() => setCountWithLimit(count + 10)}>
        +10 (with limit)
      </button>
    </div>
  );
}
```

이 커스텀 훅에서 `increment`, `decrement`, `reset`, `setCountWithLimit` 함수들은 클로저를 통해 `count` 상태에 접근합니다.
`useCallback`을 사용하여 함수들을 메모이제이션하고, 의존성 배열을 통해 필요할 때만 새로운 함수 인스턴스를 생성합니다. `setCountWithLimit` 함수는 클로저를 활용하여 상태 업데이트 로직을 캡슐화하고, 0에서 100 사이의 값으로 제한합니다.

이러한 방식으로 클로저를 활용하면 상태 로직을 효과적으로 추상화하고 재사용할 수 있으며, 컴포넌트의 로직을 더 깔끔하게 유지할 수 있습니다.
클로저를 활용한 상태 관리는 리액트 애플리케이션에서 복잡한 상태 로직을 효과적으로 다룰 수 있게 해주며, 특히 비동기 상황에서 안전하고 예측 가능한 상태 업데이트를 가능하게 합니다. 또한, 재사용 가능한 로직을 만들어 코드의 중복을 줄이고 유지보수성을 향상시킬 수 있습니다.

## 4. 클로저 관련 흔한 실수와 해결 방법

리액트 개발에서 클로저는 강력한 도구이지만, 잘못 사용하면 예기치 않은 버그를 유발할 수 있습니다. 이 글에서는 클로저 사용 시 자주 발생하는 문제들과 그 해결 방법을 상세히 살펴보겠습니다.

### 4.1 오래된 클로저(Stale Closure) 문제

오래된 클로저 문제는 클로저가 생성 시점의 값을 계속 참조하여 발생하는 이슈입니다. 이는 특히 비동기 작업이나 타이머 함수에서 자주 나타납니다. 문제 예시 코드를 한 번 살펴 보겠습니다.

```javascript
function WatchCount() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      console.log(`Count is: ${count}`);
    }, 1000);

    return () => clearInterval(timer);
  }, []); // 빈 의존성 배열

  return (
    <div>
      Count: {count}
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

이 컴포넌트에서 `setInterval` 내부의 콜백 함수는 초기 `count` 값(0)을 계속 참조합니다. 따라서 버튼을 클릭하여 `count`를 증가시켜도 콘솔에는 항상 `"Count is: 0"`이 출력됩니다.

**해결 방법:**

1. 의존성 배열에 count 추가:

```javascript
useEffect(() => {
  const timer = setInterval(() => {
    console.log(`Count is: ${count}`);
  }, 1000);

  return () => clearInterval(timer);
}, [count]); // count를 의존성 배열에 추가
```

이 방법은 `count`가 변경될 때마다 `effect`를 재실행하여 최신 값을 참조하게 합니다. 하지만 `count`가 자주 변경되면 불필요한 `effect` 재실행이 발생할 수 있습니다.

2. 함수형 업데이트 사용:

```javascript
useEffect(() => {
  const timer = setInterval(() => {
    setCount((prevCount) => {
      console.log(`Count is: ${prevCount}`);
      return prevCount;
    });
  }, 1000);

  return () => clearInterval(timer);
}, []); // 빈 의존성 배열 유지 가능
```

이 방법은 `setCount` 내부에서 최신 상태를 참조하므로, `effect`를 한 번만 실행해도 됩니다. 성능상 이점이 있지만, 로직이 복잡해질 수 있습니다.

### 4.2 의존성 배열 관련 이슈

`useEffect`, `useCallback` 등의 훅에서 의존성 배열을 잘못 설정하면 예상치 못한 동작이 발생할 수 있습니다.
문제 예시 코드를 살펴보겠습니다.

```javascript
function SearchComponent({ query }) {
  const [results, setResults] = useState([]);

  const searchAPI = useCallback(() => {
    fetch(`/api/search?q=${query}`)
      .then((response) => response.json())
      .then((data) => setResults(data));
  }, []); // 빈 의존성 배열

  useEffect(() => {
    searchAPI();
  }, [query]); // searchAPI가 누락됨

  return <ResultsList results={results} />;
}
```

이 예제에서 `searchAPI` 함수는 `query`의 변경을 감지하지 못하고, 항상 초기 `query` 값으로 검색을 수행합니다.

**해결 방법:**

1. 의존성 배열에 필요한 모든 값 추가:

```javascript
const searchAPI = useCallback(() => {
  fetch(`/api/search?q=${query}`)
    .then((response) => response.json())
    .then((data) => setResults(data));
}, [query]); // query를 의존성 배열에 추가

useEffect(() => {
  searchAPI();
}, [query, searchAPI]); // searchAPI도 의존성 배열에 추가
```

이 방법은 모든 의존성을 명시적으로 선언하여 예상치 못한 동작을 방지합니다.

2. ESLint 플러그인 사용:
   `react-hooks/exhaustive-deps ESLint` 규칙을 활성화하여 누락된 의존성을 자동으로 감지하고 수정할 수 있습니다.

```javascript
{
  "plugins": [
    "react-hooks"
  ],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

이 ESLint 설정은 의존성 배열 관련 문제를 사전에 방지하는 데 매우 효과적입니다.

클로저 관련 문제를 해결하기 위해서는 React의 렌더링 사이클과 클로저의 동작 방식을 깊이 이해해야 합니다. 의존성 배열을 올바르게 설정하고, 필요한 경우 함수형 업데이트를 사용하며, `ESLint` 플러그인을 활용하면 대부분의 클로저 관련 문제를 예방할 수 있습니다.
클로저는 강력한 도구이지만, 신중하게 사용해야 합니다. 이 글에서 소개한 패턴과 해결 방법을 적용하면 클로저로 인한 버그를 최소화하고 더 안정적인 리액트 애플리케이션을 개발할 수 있습니다.

# 5. 클로저를 활용한 성능 최적화

리액트 애플리케이션에서 클로저는 성능 최적화를 위한 강력한 도구입니다. 적절히 활용하면 불필요한 리렌더링을 방지하고, 계산 비용이 큰 작업을 최적화할 수 있습니다. 이 장에서는 클로저를 사용한 다양한 성능 최적화 기법을 살펴보겠습니다.

## 5.1 불필요한 리렌더링 방지

클로저를 이용하여 컴포넌트의 불필요한 리렌더링을 방지할 수 있습니다. 특히 useCallback 훅과 React.memo를 함께 사용하면 효과적입니다.

```javascript
function ParentComponent() {
  const [count, setCount] = useState(0);

  const increment = useCallback(() => {
    setCount((prevCount) => prevCount + 1);
  }, []);

  return (
    <div>
      <ChildComponent onClick={increment} />
      <p>Count: {count}</p>
    </div>
  );
}

const ChildComponent = React.memo(({ onClick }) => {
  console.log("ChildComponent rendered");
  return <button onClick={onClick}>Increment</button>;
});
```

이 예제에서 `useCallback`은 클로저를 사용하여 `increment` 함수를 메모이제이션합니다. 빈 의존성 배열 `[]`로 인해 `increment` 함수는 컴포넌트의 생명주기 동안 동일한 참조를 유지합니다. `React.memo`는 `ChildComponent`의 `props`가 변경되지 않으면 리렌더링을 방지합니다.
결과적으로 `ParentComponent`가 리렌더링되어도 `ChildComponent`는 불필요하게 리렌더링되지 않습니다. 이는 특히 자주 업데이트되는 부모 컴포넌트 내부에 무거운 자식 컴포넌트가 있을 때 유용합니다.

## 5.2 메모이제이션과 클로저

`useMem`o와 `useCallback` 훅은 클로저를 활용하여 계산 비용이 큰 작업을 최적화합니다. 이들 훅은 특정 의존성이 변경될 때만 값을 재계산하거나 함수를 재생성합니다.

```javascript
function ExpensiveComponent({ data, filter }) {
  const expensiveCalculation = useMemo(() => {
    return data
      .filter((item) => item.category === filter)
      .sort((a, b) => b.value - a.value);
  }, [data, filter]);

  return (
    <ul>
      {expensiveCalculation.map((item) => (
        <li key={item.id}>
          {item.name}: {item.value}
        </li>
      ))}
    </ul>
  );
}
```

이 예제에서 `useMemo`는 클로저를 사용하여 `expensiveCalculation`의 결과를 캐시합니다. `data`나 `filter`가 변경되지 않으면, 이전에 계산된 결과를 재사용합니다. 이는 특히 대량의 데이터를 처리하거나 복잡한 계산을 수행할 때 성능을 크게 향상시킬 수 있습니다.

## 5.3 클로저를 이용한 계산 최적화

복잡한 계산을 클로저로 캐싱하여 성능을 최적화할 수 있습니다. 이는 특히 재귀적인 계산이나 동적 프로그래밍 문제에 유용합니다.

```javascript
function useFibonacci() {
  const cache = useRef({});

  const fibonacci = useCallback((n) => {
    if (n in cache.current) {
      return cache.current[n];
    }
    if (n <= 1) return n;
    const result = fibonacci(n - 1) + fibonacci(n - 2);
    cache.current[n] = result;
    return result;
  }, []);

  return fibonacci;
}

function FibonacciComponent() {
  const [num, setNum] = useState(10);
  const fibonacci = useFibonacci();

  const result = useMemo(() => fibonacci(num), [fibonacci, num]);

  return (
    <div>
      <input
        type="number"
        value={num}
        onChange={(e) => setNum(parseInt(e.target.value))}
      />
      <p>
        Fibonacci({num}) = {result}
      </p>
    </div>
  );
}
```

이 예제에서 `useFibonacci` 커스텀 훅은 클로저를 사용하여 피보나치 수열 계산 결과를 캐시합니다.
`useRef`를 사용하여 컴포넌트 리렌더링 간에 캐시를 유지합니다.
`useCallback`은 `fibonacci` 함수가 항상 동일한 참조를 유지하도록 보장합니다. `useMemo`는 `num`이 변경될 때만 피보나치 수를 재계산합니다.
이 접근 방식은 동일한 입력에 대해 반복적인 계산을 피하고 성능을 크게 향상시킵니다.

클로저를 활용한 성능 최적화는 리액트 애플리케이션에서 매우 중요합니다. 불필요한 리렌더링을 방지하고, 계산 비용이 큰 작업을 최적화함으로써 애플리케이션의 반응성과 효율성을 크게 향상시킬 수 있습니다.
그러나 모든 최적화 기법과 마찬가지로, 과도한 사용은 코드의 복잡성을 증가시킬 수 있습니다. 따라서 실제로 성능 문제가 있는 부분을 식별하고, 그 부분에 집중하여 최적화를 적용하는 것이 중요합니다. 성능 프로파일링 도구를 사용하여 병목 지점을 찾고, 필요한 곳에 적절히 클로저 기반 최적화를 적용하는 것이 바람직합니다.
클로저를 이용한 성능 최적화는 리액트 개발자의 필수 도구 중 하나입니다. 이를 효과적으로 활용하면 더 빠르고 효율적인 리액트 애플리케이션을 구축할 수 있습니다.

## 6. 클로저의 장단점

클로저는 `JavaScript`의 강력한 기능 중 하나로, 리액트 개발에서도 중요한 역할을 합니다. 그러나 모든 도구와 마찬가지로 클로저에도 장단점이 있습니다. 이 섹션에서는 클로저의 주요 장단점을 자세히 살펴보고, 리액트 개발 맥락에서 이를 어떻게 활용하거나 주의해야 하는지 알아보겠습니다.

### 6.1 장점: 캡슐화, 데이터 은닉

클로저의 주요 장점은 다음과 같습니다:

1. 캡슐화

클로저를 사용하면 관련된 함수와 변수를 하나의 단위로 묶을 수 있습니다. 이는 코드의 구조화와 모듈화에 도움을 줍니다.

```javascript
function createCounter() {
  let count = 0;

  return {
    increment: function () {
      count++;
      return count;
    },
    decrement: function () {
      count--;
      return count;
    },
    getCount: function () {
      return count;
    },
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.decrement()); // 1
console.log(counter.getCount()); // 1
```

이 예제에서 `count` 변수와 관련 함수들이 하나의 단위로 캡슐화되어 있습니다. 리액트에서 이러한 패턴은 커스텀 훅을 만들 때 유용하게 사용될 수 있습니다.

2. 데이터 은닉

클로저를 통해 외부에서 직접 접근할 수 없는 private 변수를 만들 수 있습니다.

```javascript
function createPerson(name) {
  let age = 0;

  return {
    getName: function () {
      return name;
    },
    getAge: function () {
      return age;
    },
    growOlder: function () {
      age++;
    },
  };
}

const person = createPerson("John");
console.log(person.getName()); // "John"
console.log(person.getAge()); // 0
person.growOlder();
console.log(person.getAge()); // 1
// console.log(person.age); // undefined
```

이 예제에서 `age` 변수는 외부에서 직접 접근할 수 없습니다. 리액트 컴포넌트에서 이러한 패턴을 사용하면, 컴포넌트의 내부 상태를 보호하고 제어할 수 있습니다.

3. 상태 유지

클로저를 사용하면 함수 실행이 끝난 후에도 내부 상태를 유지할 수 있습니다.

```javascript
function createGenerator() {
  let id = 0;
  return function () {
    return ++id;
  };
}

const generateId = createGenerator();
console.log(generateId()); // 1
console.log(generateId()); // 2
console.log(generateId()); // 3
```

이 예제에서 `id` 변수의 상태가 함수 호출 사이에 유지됩니다. 리액트에서 이러한 패턴은 `useCallback`이나 `useMemo` 훅과 함께 사용되어 컴포넌트의 상태를 효율적으로 관리하는 데 도움을 줄 수 있습니다.

### 6.2 단점: 메모리 사용, 디버깅의 어려움

클로저의 주요 단점은 다음과 같습니다:

1. 메모리 사용

클로저는 외부 함수의 변수를 참조하므로, 이 변수들이 메모리에 계속 남아있게 됩니다. 이는 메모리 누수로 이어질 수 있습니다.

```javascript
function createLargeArray() {
  const largeArray = new Array(1000000).fill("data");
  return function () {
    return largeArray[0];
  };
}

const getFirstElement = createLargeArray();
console.log(getFirstElement()); // 'data'
// largeArray는 여전히 메모리에 남아있음
```

이 예제에서 `largeArray`는 `getFirstElement` 함수가 존재하는 한 계속 메모리에 남아있게 됩니다. 리액트 애플리케이션에서 이러한 패턴을 과도하게 사용하면 메모리 사용량이 증가하여 성능 문제가 발생할 수 있습니다.

2. 디버깅의 어려움

클로저로 인해 스코프 체인이 복잡해지면 디버깅이 어려워질 수 있습니다.

```javascript
function outer(x) {
  return function middle(y) {
    return function inner(z) {
      return x + y + z;
    };
  };
}

const add5 = outer(5);
const add5and10 = add5(10);
console.log(add5and10(15)); // 30
```

이런 중첩된 클로저는 디버깅 시 각 스코프의 변수 값을 추적하기 어렵게 만듭니다. 리액트 컴포넌트에서 복잡한 클로저를 사용할 때는 이점을 주의해야 합니다.

3. 성능 영향

클로저의 과도한 사용은 성능에 부정적인 영향을 줄 수 있습니다. 특히 클로저가 큰 스코프를 가지고 있을 때 그렇습니다.

```javascript
function createFunctions() {
  const functions = [];

  for (var i = 0; i < 10000; i++) {
    functions.push(function () {
      return i;
    });
  }

  return functions;
}

const functions = createFunctions();
console.log(functions[0]()); // 10000
console.log(functions[9999]()); // 10000
```

이 예제에서 모든 함수가 동일한 `i` 변수를 참조하므로, 메모리 사용량이 증가하고 성능이 저하될 수 있습니다. 리액트 애플리케이션에서 이러한 패턴을 사용할 때는 성능에 미치는 영향을 고려해야 합니다.

클로저는 강력한 도구이지만, 이러한 장단점을 잘 이해하고 적절히 사용해야 합니다. 리액트 개발에서 클로저를 활용할 때는 다음 사항을 고려해야 합니다:

- 커스텀 훅을 만들 때 클로저를 사용하여 로직을 캡슐화하고 재사용성을 높일 수 있습니다.
- `useCallback`과 `useMemo` 훅을 사용할 때 클로저의 특성을 이해하고 의존성 배열을 올바르게 설정해야 합니다.
- 대규모 데이터 구조를 클로저 내부에 유지할 때는 메모리 사용량에 주의해야 합니다.
- 복잡한 중첩 클로저는 가능한 피하고, 필요한 경우 코드의 가독성과 유지보수성을 고려하여 설계해야 합니다.

클로저를 적절히 활용하면 리액트 애플리케이션의 구조를 개선하고 성능을 최적화할 수 있지만, 과도한 사용은 오히려 문제를 야기할 수 있습니다. 따라서 클로저의 장단점을 충분히 이해하고, 상황에 맞게 적절히 사용하는 것이 중요합니다.

## 7. 클로저와 관련된 리액트 패턴

리액트 개발에서 클로저는 컴포넌트 간의 상태와 메서드를 공유하는 데 중요한 역할을 합니다. 특히, 고차 컴포넌트(HOC)와 Render Props 패턴은 클로저를 활용하여 컴포넌트의 재사용성과 유연성을 크게 향상시킵니다. 이 섹션에서는 이 두 가지 주요 패턴을 자세히 살펴보고, 각 패턴이 클로저를 어떻게 활용하는지 알아보겠습니다.

### 7.1 고차 컴포넌트 (HOC)

고차 컴포넌트(Higher-Order Component, HOC)는 컴포넌트를 인자로 받아 새로운 컴포넌트를 반환하는 함수입니다. 이 패턴은 클로저를 사용하여 상위 컴포넌트의 상태나 메서드를 하위 컴포넌트에 전달합니다.

**_예제: 로깅 기능 추가하기_**

```javascript
function withLogger(WrappedComponent) {
  return class extends React.Component {
    componentDidMount() {
      console.log(`Component ${WrappedComponent.name} mounted`);
    }

    render() {
      return <WrappedComponent {...this.props} />;
    }
  };
}

class MyComponent extends React.Component {
  render() {
    return <div>Hello, World!</div>;
  }
}

const MyComponentWithLogger = withLogger(MyComponent);
```

`withLogger` 함수는 `WrappedComponent`를 클로저로 캡처합니다. 이를 통해 반환된 새 컴포넌트 내에서 원본 컴포넌트에 접근할 수 있습니다.
HOC 패턴을 사용하면 로깅과 같은 공통 기능을 여러 컴포넌트에 쉽게 추가할 수 있습니다. 예를 들어, `withLogger`를 다른 컴포넌트에도 적용할 수 있습니다

```javascript
const Button = withLogger(({ onClick, label }) => (
  <button onClick={onClick}>{label}</button>
));
const Input = withLogger(({ value, onChange }) => (
  <input value={value} onChange={onChange} />
));
```

HOC를 사용하면 로깅 로직을 별도의 모듈로 분리할 수 있어, 컴포넌트의 주요 로직에 집중할 수 있습니다.
그리고 여러 HOC를 조합하여 복잡한 기능을 구현할 수 있습니다. 예를 들어

```javascript
const EnhancedComponent = withAuth(withLogger(withStyles(MyComponent)));
```

### 7.2 Render Props

Render Props 패턴은 props로 함수를 전달받아 그 함수를 통해 컴포넌트를 렌더링하는 방식입니다. 이 패턴은 클로저를 활용하여 상위 컴포넌트의 상태나 메서드를 하위 컴포넌트에 전달합니다.

**_예제: 마우스 위치 추적기_**

```javascript
class MouseTracker extends React.Component {
  state = { x: 0, y: 0 };

  handleMouseMove = (event) => {
    this.setState({
      x: event.clientX,
      y: event.clientY,
    });
  };

  render() {
    return (
      <div style={{ height: "100vh" }} onMouseMove={this.handleMouseMove}>
        {this.props.render(this.state)}
      </div>
    );
  }
}

function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <h1>
          The mouse position is ({x}, {y})
        </h1>
      )}
    />
  );
}
```

render prop으로 전달된 함수는 `MouseTracker`의 상태를 클로저로 캡처합니다. 이를 통해 마우스 위치 데이터를 자식 컴포넌트에 동적으로 전달할 수 있습니다. Render Props 패턴은 다양한 렌더링 요구사항에 유연하게 대응할 수 있습니다. 예를 들어:

```javascript
<MouseTracker
  render={({ x, y }) => (
    <div>
      <h1>Mouse position:</h1>
      <p>
        X: {x}, Y: {y}
      </p>
      <CustomComponent mouseX={x} mouseY={y} />
    </div>
  )}
/>
```

여러 컴포넌트 간에 상태를 쉽게 공유할 수 있습니다. 예를 들어, 마우스 위치를 여러 컴포넌트에서 사용해야 할 경우 유용합니다.
Render Props 패턴을 사용하면 로직(`MouseTracker`)과 표현(render prop)을 분리할 수 있어, 각 부분을 독립적으로 테스트하기 쉽습니다.

이러한 패턴들을 사용할 때는 몇 가지 주의사항과 고려사항이 있습니다. 먼저, HOC와 Render Props 모두 불필요한 리렌더링을 야기할 수 있으므로, `React.memo`나 `shouldComponentUpdate`를 사용하여 성능을 최적화해야 합니다. 또한, 여러 HOC를 중첩하거나 Render Props를 과도하게 사용하면 "래퍼 지옥"이 발생할 수 있으므로, 컴포넌트 구조를 신중히 설계해야 합니다. HOC를 사용할 때는 원본 컴포넌트의 이름을 유지하는 것이 좋으며(예: `withLogger`(`MyComponent`) → `LoggedMyComponent`), props를 추가할 때 기존 props와 충돌하지 않도록 주의해야 합니다.

고차 컴포넌트와 Render Props는 리액트에서 클로저를 활용하여 상태와 메서드를 공유하는 강력한 패턴입니다. 이 패턴들은 코드의 재사용성과 유연성을 크게 향상시키며, 복잡한 애플리케이션 구조에서도 유지보수성을 개선합니다. 그러나 이러한 패턴을 사용할 때는 앞서 언급한 성능과 코드 복잡성을 고려하여 적절히 활용해야 합니다. 최근에는 React Hooks의 도입으로 이러한 패턴들의 사용이 줄어들고 있지만, 여전히 많은 레거시 코드와 라이브러리에서 사용되고 있어 이해하고 있어야 합니다.
