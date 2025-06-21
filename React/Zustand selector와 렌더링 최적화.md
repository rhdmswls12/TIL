# Zustand selector와 렌더링 최적화

<br />

Zustand에서 전역 상태를 사용할 때 불필요한 리렌더링을 피하려면 **selector 패턴**을 사용해야 한다.

```tsx
const { username } = useUseStore();
```

위와 같이 구조 분해로 상태를 가져오면 `username`만 사용하는 것처럼 보여도 내부적으로는 **스토어 전체를 구독**하게 된다. 즉 `email` 같은 다른 값이 바뀌어도 해당 컴포넌트는 리렌더링이 발생한다.

<br />

## selector 사용

1. **개별 selector로 구독**

   ```tsx
   const username = useUserStore((state) => state.username);
   ```

   - `username`이 바뀔 때만 컴포넌트가 리렌더링된다.

2. **여러 상태를 묶을 경우: shallow 사용**

   ```tsx
   import { shallow } from "zustand/shallow";

   const { username, email } = useUserStore(
     (state) => ({
       username: state.username,
       email: state.email,
     }),
     shallow
   );
   ```

   - `username` 또는 `email` 중 하나라도 바뀌면 렌더링된다.
   - `shallow`는 객체 속성들을 얕은 비교하여 렌더링 여부를 판단한다.

<br />

## shallow 개념과 주의사항

```tsx
interface UserState {
  name: string;
  age: number;
  loginCount: number;
}
```

```tsx
const useUserStore = create<UserState>(() => ({
  name: "Alice",
  age: 25,
  loginCount: 0,
}));
```

- 예를 들어서 `user`라는 상태가 있고, 그 안에 여러 필드가 있다.

```tsx
const { name, age } = useUserStore((state) => ({
  name: state.name,
  age: state.age,
}));
```

- 이제 컴포넌트에서 두 개의 상태(`name`, `age`)를 동시에 가져오려 한다.

 <br />

**기본 동작: 매번 새로운 객체 생성**

상태가 바뀔 때마다 아래와 같이 새 객체를 반환한다.

```tsx
{ name: "Alice", age: 25 } // 렌더링 1
{ name: "Alice", age: 26 } // 렌더링 2
{ name: "Alice", age: 26 } // 렌더링 3 ← 값은 안 바뀌었지만 객체는 새로 만들어짐
```

- 상태 안의 값이 같아도 객체의 참조가 다르기 때문에 컴포넌트는 매번 렌더링되고, 이는 불필요한 리렌더링이다.

<br />

**shallow 사용**

`shalllow`를 사용하면 객체의 속성 값만 얕게 비교한다.

```tsx
import { shallow } from "zustand/shallow";

const { name, age } = useUserStore(
  (state) => ({ name: state.name, age: state.age }),
  shallow
);
```

```tsx
{ name: "Alice", age: 25 } // 렌더링 1
{ name: "Alice", age: 25 } // 렌더링 안 됨 ✅
{ name: "Alice", age: 26 } // age가 바뀌었으니 렌더링 됨
```

즉, 객체 속성 값이 변했는지만 비교하고, 객체 자체가 새로 만들어져도 값이 같으면 리렌더링하지 않는다.

하지만 다음과 같은 주의점이 있다.

<br />

**객체 자체를 구독하면 shallow가 무의미**

```tsx
const { user } = useUserStore(
  (state) => ({ user: state.user }), // user 객체 자체를 구독
  shallow
);
```

- 이 경우 `user.name`, `user.email` 중 어떤 값이 바뀌었는지 알 수 없고, `user` 객체 전체가 매번 새로 만들어지므로 `shallow`가 의미 없다.
- 객체 내부가 같더라도 참조가 다르면 렌더링이 발생한다.

<br />

**해결 방법: 필드를 평탄하게 꺼내기**

```tsx
const { name, email } = useUserStore(
  (state) => ({
    name: state.user.name,
    email: state.user.email,
  }),
  shallow
);
```

- 필드 단위로 꺼내고 묶어서 `shallow` 비교하면 값이 변했을 때만 리렌더링된다.

<br />

## 커스텀 훅으로 selector 캡슐화하기

```tsx
// store/selectors.ts
export const useUserInfo = () =>
  useUserStore(
    (state) => ({
      username: state.username,
      email: state.email,
    }),
    shallow
  );

// 사용처
const { username, email } = useUserInfo();
```

- 여러 곳에서 동일한 상태를 구독해야 한다면 커스텀 훅으로 selector를 재사용 가능하게 분리하면 된다.
