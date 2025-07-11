닉네임 설정 페이지에서 폼 입력값을 실시간으로 검증하던 중, 입력값이 변경되지 않아도 유효성 검사 로직이 매 렌더링마다 실행되는 비효율이 있음을 발견했다. 이를 해결하기 위해 `useMemo`를 활용해 유효성 검사 결과(`message`, `isValid`)를 메모이제이션했다.

<br />

## 닉네임 유효성 검사 기준

닉네임 유효성 검사를 할 때 다음 조건들을 점검했다:

- 최소 글자 수 (2글자 이상)
- 특수문자 포함 여부
- 공백 포함 여부
- 중복된 닉네임인지 여부
- 위 조건들을 모두 통과하면 사용 가능한 닉네임으로 판단

이를 `useMemo`를 사용하여 최적화했다.

<br />

## **useMemo란?**

```tsx
useMemo(() => {
  // 계산된 결과
  return something;
}, [dependency1, dependency2]);
```

- `useMemo`는 **특정 값의 계산 결과를 기억(메모이제이션)** 해서 의존성 배열 안의 값이 변경되지 않으면 **이전 결과를 재사용**한다.
- 즉, 불필요한 재계산을 피해서 성능을 최적화한다.

<br />

## **아래 코드가 매 렌더링마다 호출된다면?**

```tsx
const { message, isValid } = (() => {
  // 유효성 검사 로직 (공백, 특수문자, 중복 등)
  ...
})();
```

- 위와 같은 방식은 입력값이 변하지 않아도 유효성 검사 로직이 매번 실행되며, 이로 인해 성능 낭비가 발생할 수 있다.
- 특히 닉네임 외의 다른 이유로 컴포넌트가 렌더링될 때도 검사가 실행되므로 비효율적이다.

<br />

## **닉네임이 바뀌었을 때만 검사하자!**

```tsx
useMemo(() => {
  // trimmedNickname이나 nicknames가 바뀔 때만 실행
  return {
    message: JSX.Element,
    isValid: boolean,
  };
}, [trimmedNickname, nicknames]);
```

- `trimmedNickname` 또는 `nicknames`가 변경될 때만 유효성 검사 다시 수행
- 그 외의 렌더링에서는 기존 계산 결과 재사용
- 복잡한 검증 로직(정규식 검사, 배열 비교 등)을 불필요하게 반복하지 않는다.

<br />

**\*useCallback과의 차이**

- `useCallback`은 함수 자체를 메모이제이션할 때 사용
- `useMemo`는 계산된 값 자체(객체, JSX, 숫자 등)를 메모이제이션할 때 사용
