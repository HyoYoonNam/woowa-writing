# Immer을 활용한 서비스 내 코드 구조 개선기

## 서론

안녕하세요. 저는 `투룻`이라는 서비스를 만들고 있는 웹 프론트엔드 6기 지니 입니다. 투룻은 여행 계획을 세우는데 어려움을 겪는 분들을 위해 마음에 드는 여행기를 바탕으로 여행 계획을 세우도록 도와주는 서비스 입니다.

이번 글쓰기 주제는 Immer를 활용한 서비스 내 코드 구조 개선기 입니다.

서비스를 만들며 어려웠던 부분 중 하나는 중첩된 객체 구조의 도메인 객체를 핸들링 하는 부분이었습니다.

중첩 객체를 조금 더 편리하게 다루기 위해 Immer.js와 use-immer이라는 라이브러리를 도입하게 되었고, 커스텀 훅의 구조를 개선할 수 있었던 경험이 있었습니다.

> 이번 글의 경우 기본 수준의 JavaScript(Proxy & JavaScript), React, TypeScript 지식, 함수형 프로그래밍 개념이 포함되어있습니다.

### Immer의 필요성

투룻의 도메인은 크게 `여행기(여행을 다녀온 것에 대한 기록)`와 `여행 계획`으로 나눌 수 있습니다.

그 중 여행 계획을 등록하는 경우 제목, 시작일, Day 별 여행 계획 정보(Day 별 장소 정보(장소 이름, 장소 사진, 장소 설명, 장소 위치))를 작성해야합니다.

이를 타입스크립트의 interface로 표현하면 아래와 같이 표현할 수 있습니다.

```ts
export interface TravelPlanTodo {
  id: number | string;
  content: string;
  order?: number;
  checked?: boolean;
}
export interface TravelPlanPlace {
  id: string;
  placeName: string;
  todos?: TravelPlanTodo[];
  position: MapPosition;
}

export interface TravelPlanDay {
  id: string;
  places: TravelPlanPlace[];
}

export interface TravelPlanRegisterPayload {
  title: string;
  startDate: string;
  days: TravelPlanDay[];
}
```

interface로 구조화 시킨 정보에 대해 JSON 데이터로 다음과 같이 표현할 수 있습니다.

```json
{
  "title": "신나는 잠실 한강 여행",
  "startDate": "2024-11-16",
  "days": [
    {
      "places": [
        {
          "placeName": "잠실한강공원",
          "position": {
            "lat": "37.5175896",
            "lng": "127.0867236"
          },
          "todos": [
            {
              "content": "함덕 해수욕장 산책",
              "isChecked": true
            }
          ]
        }
      ]
    }
  ]
}
```

구조를 잘 살펴보면 payload → days → places → todos의 형태인 것을 알 수 있습니다.

여행 계획 등록 중 Day 별 여행 계획 정보를 `useTravelPlanDays`라는 hook으로 관리하고 있었습니다.

그 중 각 여행 장소의 to do를 작성하기위한 핸들러인 onChangeContent의 경우 아래와 같은 로직으로 작성되어있었습니다.

```ts
const [travelPlanDays, setTravelPlanDays] = useState<TravelPlanDay[]>();

const onChangeContent = ({
  content,
  dayIndex,
  placeIndex,
  todoId,
}: {
  content: string;
  dayIndex: number;
  placeIndex: number;
  todoId: string;
}) => {
  setTravelPlanDays((prevTravelPlansDays) => {
    // 1. 기존 여행 계획 데이터를 복사한다.
    const newTravelPlans = [...prevTravelPlansDays];

    // 2. dayIndex, placeIndex, todoId에 해당되는 todo를 찾는다.
    const todo = newTravelPlans[dayIndex]?.places[placeIndex]?.todos?.find(
      (todo) => todo.id === todoId
    );

    // 3. todo가 있다면 todo의 content에 유저가 입력한 content를 추가한다.
    if (todo) {
      todo.content = content.slice(
        FORM_VALIDATIONS_MAP.title.minLength,
        FORM_VALIDATIONS_MAP.title.maxLength
      );
    }

    // 4. 복사했던 여행 계획 데이터를 반환한다.
    return newTravelPlans;
  });
};
```

잘 살펴보면 `travelPlanDays` 상태를 업데이트 하기 위해 별도의 복사 과정을 필요로 합니다.

이는 리액트가 상태 업데이트를 트리거 후 리렌더링하는 과정에서 아래와 같은 과정을 별도로 수행하기 때문입니다.

1. 기존 데이터에 대해 얕은 비교를 진행한다.
2. 참조 값이 동일하다면, 값이 변경되지 않았다고 판단해 리렌더링을 하지 않는다.
3. 참조 값이 다른 경우 값이 변경되었다고 판단해 리렌더링한다.

하나의 핸들러만 보았을 땐 크게 문제 될 부분이 없을 수도 있을거 같습니다.

```ts
return {
  onChangeTravelPlanDays,
  onAddDay,
  onDeleteDay,
  onAddPlace,
  onDeletePlace,
  onChangeContent,
  onAddPlaceTodo,
  onDeletePlaceTodo,
};
```

하지만 커스텀 훅이 반환하는 핸들러는 총 8개입니다.

여행기에 대한 Day와 관련된 커스텀 훅까지 합치면 총 16개의 핸들러에 대해 불변성을 보장하는 로직을 개발자가 관리하게 됩니다.

투룻팀 프론트엔드 개발자 내에서도 매번 16개의 핸들러에 대해 불변성을 신경써야하는 부분, 로직을 읽을 때의 불편함으로 인해 개선이 필요하다고 생각했습니다.

```ts
const transformTravelPlanDays = (days: TravelTransformPlaces[]) => {
  return [...days].map((day) => ({
    ...day,
    places: day.places.map((place) => {
      return {
        ...place,
        todos: [],
      };
    }),
  }));
};
```

`transformTravelPlanDays`의 경우 여행기를 여행 계획 데이터로 변환하는 과정에서 필요한 데이터를 채우기 위한 함수입니다.

이 함수의 구조를 살펴보면, days와 places를 복사하고 나서야 todos에 각 빈 배열을 추가하는 것을 알 수 있습니다.

위 로직 또한 가독성 측면에서 개선이 필요하다고 생각했습니다.

## Immer.js

해당 라이브러리는 아래와 같이 소개됩니다.

> javascript에서 data의 불변성을 보장해주는 라이브러리

immer.js의 경우 내부에서 불변성을 유지해주기 때문에, 개발자가 따로 객체를 복사하지 않아도 됩니다.

예시 코드를 통해 한번 살펴보겠습니다.

```ts
interface User {
  name: string;
  age: number;
  hobbies: string[];
}

const user1: User = {
  name: '김철수',
  age: 30,
  hobbies: ['독서', '등산'],
};

const user2 = user1;
console.log(user1 === user2); // true

user2.age = 31;
user2.hobbies.push('요리');

console.log(user1.age); // 31
console.log(user1.hobbies); // ['독서', '등산', '요리']
```

user2의 경우 user1을 복사한 형태입니다.

자바스크립트의 `참조에 의한 전달` 특성으로 인해, user1과 user2는 서로 같은 참조 값을 바라보게 됩니다.

이로 인해 user1과 user2를 서로 비교해보면 true가 출력되는 것을 알 수 있습니다.

또한, user2의 age와 hobbies를 변경하게 되면 user1에도 영향을 받아 age는 31로, hobbies는 `요리`가 추가 된 것을 알 수 있습니다.

```ts
import { produce } from 'immer';
// ...

const user2 = produce(user1, (draft) => {
  draft.age = 31;
  draft.hobbies.push('요리');
});

console.log(user1 === user2); // false

console.log(user1.age); // 30
console.log(user1.hobbies); // ['독서', '등산']

console.log(user2.age); // 31
console.log(user2.hobbies); // ['독서', '등산', '요리']

// 원본 객체의 불변성 확인
console.log(user1.hobbies === user2.hobbies); // false
```

immer의 produce를 사용하게 되면 이전과 달리 user2의 불변성도 보장되며 변이를 쉽게 할 수 있습니다.

또한, 특정 객체의 변경 로직을 함수 내부에서 관리함으로써 보기 쉬운 코드를 만들 수 있습니다.

그렇다면 이런 의문을 가져볼 수 있을거 같습니다.

> immer는 내부적으로 어떻게 불변성을 보장해주는걸까?

본격적으로 immer의 불변성을 보장해주는 방법에 대해 살펴보겠습니다.

## immer에서 불변성을 보장해주는 방법

> The basic idea is that with Immer you will apply all your changes to a temporary draft, which is a proxy of the currentState. Once all your mutations are completed, Immer will produce the nextState based on the mutations to the draft state. This means that you can interact with your data by simply modifying it while keeping all the benefits of immutable data. - immer.js docs -

immer.js에서 내부적으로 불변성을 보장해주는 방법에 대해 위와 같이 소개합니다.

이를 요약하면 아래와 같습니다.

<div align="center">
<img width="400" src="https://github.com/user-attachments/assets/671cc29a-5960-4c84-81e1-900f94fe021b"/>
</div>

1. 기존 객체에 대한 복사본(draft)을 프록시를 통해 생성
2. 복사본(draft)에 대한 모든 변경 사항을 적용
3. 모든 변경이 완료되면 복사본을 바탕으로 다음 상태를 생성

즉, 프록시를 사용하여 원본 객체의 가상 복사본을 만들고, 사용자가 이를 수정하는 듯한 코드를 작성할 수 있게 하되, 실제로는 변경된 부분만 반영한 새로운 불변 객체를 생성하는 방식입니다.

이는 마치 누군가가 문서를 제공해주면 그 문서를 마음대로 수정할 때, 또 다른 누군가가 그 문서의 변경 사항을 적용해 최종 문서를 만드는 것과 같습니다.

```js
const nextState = produce(baseState, (draft) => {
  // ...
});
```

Immer에서 제공하는 대표적인 함수 `produce`는 총 3개의 매개변수를 필요로 합니다.

- base : 초기 상태의 값
- recipe : base를 수정할 수 있도록 제공하는 함수
  - draft : recipe의 첫번째 인자로, 안전하게 변경할 수 있는 기본 상태의 proxy 객체

그렇다면 produce는 어떻게 로직이 구성되어있을까요?

## produce deep dive

```ts
produce: IProduce = (base: any, recipe?: any, patchListener?: any) => {
  let result;

  if (isDraftable(base)) {
    // 1. 스코프 생성
    const scope = enterScope(this);

    // 2. base 상태에 대한 프록시 생성
    const proxy = createProxy(base, undefined);

    let hasError = true;
    try {
      // 3. recipe 함수를 실행하고, 프록시를 통해 result를 수정
      result = recipe(proxy);
      hasError = false;
    } finally {
      // 4. 스코프 종료: 에러 발생 시 revokeScope, 아니면 leaveScope
      if (hasError) revokeScope(scope);
      else leaveScope(scope);
    }

    // 5. 최종 결과 처리 및 반환
    return processResult(result, scope);
  }
};
```

로직을 살펴보면 크게 5가지로 나눌 수 있습니다.

1. 스코프 생성
2. base 상태의 프록시 객체 생성
3. recipe 함수를 실행하고 프록시를 통해 상태 수정
4. 스코프 종료 - 에러 발생시 revokeScope를, 그렇지 않다면 leaveScope 실행
5. 최종 결과 처리 및 반환

2번과 3번은 충분히 이해가 가능한 상황이지만 1, 4, 5에 대한 추가적인 이해가 필요한 상황입니다.

하나씩 살펴보겠습니다.

### 스코프 생성

immer에서 스코프를 아래와 같이 정의하고 있습니다.

> immer 전역에서 사용할 정보들을 저장하는 객체

immer의 기본 동작에는 크게 사용되지 않지만 어떤 데이터를 저장하고 있는지 주목할 필요가 있다.

```ts
export function enterScope(immer: Immer) {
  return (currentScope = createScope(currentScope, immer));
}

function createScope(parent_: ImmerScope | undefined, immer_: Immer): ImmerScope {
  return {
    drafts_: [],
    parent_,
    immer_,
    canAutoFreeze_: true,
    unfinalizedDrafts_: 0,
  };
}
```

스코프를 통해 draft(불변 객체의 임시 수정 가능한 복사본) 상태를 관리하고, 상태가 변경되는 동안 발생하는 정보를 추적할 수 있습니다.

즉, produce가 호출 되었을 때의 범위 내에서 draft가 변경되는 모든 것을 옵저버 처럼 관찰하며 변경 후 반환까지 스코프가 책임지게 됩니다.

비유하자면 스코프는 작업 공간으로써 이해할 수 있습니다. 새로운 공간이 열리면 그 내에서 모든 작업(데이터 변경)이 이뤄집니다.

### 스코프 종료 - 에러 발생시 revokeScope를, 그렇지 않다면 leaveScope 실행

만약 recipe 함수를 실행하는 과정에서 에러가 발생한다면 revokeScope를 실행하게 됩니다.

```ts
export function revokeScope(scope: ImmerScope) {
  leaveScope(scope);
  scope.drafts_.forEach(revokeDraft);
  scope.drafts_ = null;
}
```

이 함수의 경우 스코프를 무효화 시킨 후 그 안에서 변경된 모든 draft 객체들을 해제하는 역할을 수행합니다.

요약하면 2가지 작업을 수행합니다.

1. 현재 활성화 된 스코프에서 벗어나 부모 스코프로 돌아간다.
2. revokeDraft 호출 및 `drafts_` 배열을 null로 설정하여 모든 draft 들을 정리

```ts
export function leaveScope(scope: ImmerScope) {
  if (scope === currentScope) {
    currentScope = scope.parent_;
  }
}
```

leaveScope는 더 간단합니다.

만약 종료하려는 스코프가 currentScope와 동일하다면, currentScope를 부모 스코프로 전환합니다.

즉, 현재 작업 세션을 종료하고 상위 작업 세션으로 돌아가게 됩니다.

### 최종 결과 처리 및 반환

```ts
export function processResult(result: any, scope: ImmerScope) {
  const baseDraft = scope.drafts_![0];
  const isReplaced = result !== undefined && result !== baseDraft;

  if (isReplaced) {
    result = isDraftable(result) ? finalize(scope, result) : result;
  } else {
    result = finalize(scope, baseDraft, []);
  }

  revokeScope(scope);

  return result !== NOTHING ? result : undefined;
}
```

해당 역할을 수행하는 `processResult` 함수에서는 produce 내 존재하는 scope와 result를 인자로 받습니다.

baseDraft와 result와 비교하여 변경 사항이 있었는지 확인합니다.

변경 사항이 있었다면 최종화 된 작업을 result에 추가합니다.

그렇지 않다면 기본 드래프를 최종화 시켜 result에 할당합니다.

그 후 스코프를 해제 후 최종 결과를 반환하는 형태입니다.

### 정리

이제까지 학습한 내용을 정리하면 아래와 같이 요약할 수 있습니다.

1. enterScope를 통해 새로운 스코프를 생성하며, 이 스코프는 현재 작업 세션에서 필요한 정보를 저장하고 관리하는 역할을 수행

2. base 상태의 프록시 객체를 생성하여 draft 생성

3. 레시피 함수(recipe)를 실행하여 프록시를 통해 상태를 안전하게 수정

4. 스코프 종료

5. 드래프트 객체를 최종화 후 결과 반환

## 프로젝트 내 immer 적용

```ts
import { produce } from 'immer';

// before
const transformTravelPlanDays = (travelTransformDays: TravelTransformDays[]) => {
  return [...travelTransformDays].map((day) => ({
    ...day,
    places: day.places.map((place) => {
      return {
        ...place,
        todos: [],
      };
    }),
  }));
};

// after
const transformTravelPlanDays = (travelTransformDays: TravelTransformDays[]) =>
  produce(travelTransformDays, (newTransformDays) => {
    newTransformDays.forEach((day) => {
      day.places = day.places.map((place) => ({ ...place, todos: [] }));
    });
  });
```

이전에 비해 복사하는 로직을 최소화 하고, 개발자는 변경되는 로직에 집중할 수 있게 되었습니다.

before와 동일하게 불변성도 보장하며, 코드 구조도 많이 개선된 것을 알 수 있습니다.

### use-immer 적용

react에서는 use-immer이라는 라이브러리를 통해 produce 함수를 사용할 수 있습니다.

```ts
export function useImmer(initialValue: unknown) {
  const [val, updateValue] = useState(() =>
    freeze(typeof initialValue === 'function' ? initialValue() : initialValue, true)
  );
  return [
    val,
    useCallback((updater) => {
      if (typeof updater === 'function') updateValue(produce(updater));
      else updateValue(freeze(updater));
    }, []),
  ];
}
```

useImmer는 immer의 freeze와 produce 함수를 사용하며, 상태 업데이트 과정에서 produce를 통해 리액트의 불변성을 어겨도 immer 내부에서 불변성을 관리해주어 상태 업데이트를 가능하게 만들 수 있습니다.

```ts
export const useTravelPlanDays = (days: TravelTransformPlaces[]) => {
  const [travelPlanDays, setTravelPlanDays] = useImmer<TravelPlanDay[]>(() =>
    transformTravelPlanDays(days)
  );

  // ...

  const onDeleteDay = (targetDayIndex: number) => {
    setTravelPlanDays((previousTravelPlanDays) => {
      previousTravelPlanDays.splice(targetDayIndex, 1);
    });
  };

  return {
    // ...
    onDeleteDay,
  };
};
```

useImmer은 useState와 동일하게 사용하며, onDeleteDay를 살펴볼 때 별도의 복사본 없이 개발자는 내부 로직에만 집중할 수 있게 되었습니다.

## 끝으로

이번 글에서는 Immer.js와 use-immer를 활용하여 프로젝트 내의 복잡한 상태 관리 코드를 어떻게 개선할 수 있는지 살펴보았습니다.

중첩된 객체 구조를 가진 상태를 다룰 때 불변성을 유지하는 과정은 개발자에게 많은 부담을 줄 수 있습니다.

Immer를 도입함으로써 얻은 주요 이점은 다음과 같습니다.

    •	코드 간결화: 복잡한 복사 로직을 작성할 필요 없이, 마치 가변 객체를 다루듯이 상태를 업데이트할 수 있습니다.
    •	가독성 향상: 상태 변경 로직에 집중할 수 있어 코드의 가독성이 높아집니다.
    •	오류 감소: 불변성을 수동으로 관리할 때 발생할 수 있는 실수를 줄여줍니다.
    •	생산성 향상: 개발자가 비즈니스 로직에 더욱 집중할 수 있어 개발 생산성이 향상됩니다.

또한, React 환경에서 use-immer를 활용하여 상태 관리 훅을 더욱 효율적으로 구현할 수 있었습니다.

useState를 대체하여 useImmer를 사용함으로써 상태 업데이트 시 불변성에 신경 쓰지 않고도 안전하게 상태를 변경할 수 있었습니다.

이러한 개선을 통해 팀 내에서도 코드의 유지 보수성과 확장성이 향상되었으며, 새로운 기능을 추가하거나 버그를 수정하는 데 드는 시간과 노력을 절약할 수 있었습니다.

# 레퍼런스

- https://immerjs.github.io/immer/
- https://developer.mozilla.org/en-US/docs/Web/API/Window/structuredClone
- https://hmos.dev/deep-dive-to-immer#scope
- https://github.dev/immerjs/immer/blob/main/src/core/immerClass.ts
- https://github.com/immerjs/use-immer/blob/master/src/index.ts
