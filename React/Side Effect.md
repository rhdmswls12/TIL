# Side Effect

<br />

## 순수 함수(pure function)

순수 함수는 **입력값이 같으면 항상 같은 결과를 반환**하며, **외부 상태를 변경하지 않는 함수**를 의미한다.
즉 함수 내부에서 함수 밖에 있는 값을 직접 수정하면 **사이드 이펙트(side effect)** 가 발생해서 순수 함수가 아니게 된다.

**나쁜 예시 (순수 함수가 아님)**

```jsx
let count = 0; // 함수 외부의 변수

function increment() {
  count += 1; // 외부 변수 변경 (부작용)
  return count;
}
```

**좋은 예시 (순수 함수)**

```jsx
function increment(count) {
  return count + 1; // 외부 변수 변경 없이 새로운 값 반환
}
```

리액트의 렌더링 과정도 순수 함수처럼 동작한다. 즉 input을 받아 예측 가능한 JSX 리턴 값을 만들어낸다. 자바스크립트 함수에 대한 input은 인수, 리액트 컴포넌트에 대한 입력은 props이다.

순수 함수는 예측 가능하고, 신뢰할 수 있고, 테스트하기 쉽다. 반대로 함수 내의 구현 내용이 함수 외부에 영향을 끼치는 경우 해당 함수는 side effect가 있다고 얘기한다.

<br />

## side effect

**사이드 이펙트(side effect)** 란 렌더링 외부에서 발생하는 상태 변화를 의미한다. 일반적으로 다음과 같은 경우들이 해당된다.

- API 요청 (데이터 가져오기, 저장하기)
- DOM 조작 (스크롤 위치 변경, 애니메이션 실행)
- 타이머 설정 (`setTimeout`, `setInterval`)
- 브라우저 스토리지 사용 (`localStorage`, `sessionStorage`)

이러한 작업들은 리엑트의 렌더링 흐름과 분리해서 실행해야 한다.

```jsx
function User({ name }) {
  document.title = name;
  // This is a side effect. Don't do this in the component body!

  return <h1>{name}</h1>;
}
```

브라우저 탭의 title을 변경하려는 경우 위와 같이 컴포넌트 내에서 직접 변경할 수도 있지만 이렇게 작성해서는 안된다. `document.title = name;`이 렌더링 중에 실행되므로 순수성이 깨진다. 즉 컴포넌트가 렌더링될 때마다 브라우저의 `document.title`이 강제로 변경된다.

\*`document.title`은 컴포넌트 내부의 지역 변수가 아닌, 외부 환경(BOM, 즉 `window.document` 객체)에 속하는 값이므로 순수성이 깨진다.

```jsx
import { useEffect } from "react";

function User({ name }) {
  useEffect(() => {
    document.title = name; // 컴포넌트가 렌더링된 후 실행됨
  }, [name]); // name이 변경될 때만 실행됨

  return <h1>{name}</h1>;
}
```

위와 같이 렌더링이 끝난 후에 실행되도록 `useEffect`를 사용하는 것이 좋다.

<br />

## 사이드 이펙트와 이벤트 핸들러

리액트에서는 보통 이벤트 핸들러에서 사이드 이펙트를 실행한다. 이벤트 핸들러는 사용자의 동작(버튼 클릭, 입력 등)에 따라 실행되므로 렌더링과 분리된다.

**잘못된 예시 (렌더링 중 사이드 이펙트 발생)**

```jsx
function MyComponent() {
  console.log("렌더링 중...");

  fetch("https://api.example.com/data") // 잘못된 코드! 렌더링 중 API 요청 발생
    .then((response) => response.json())
    .then((data) => console.log(data));

  return <div>Hello</div>;
}
```

- 렌더링할 때마다 fetch 요청이 실행되어 불필요한 API 호출이 반복된다.

**올바른 예시 (이벤트 핸들러에서 실행)**

```jsx
function MyComponent() {
  const handleClick = () => {
    fetch("https://api.example.com/data") // 버튼 클릭 시 API 요청
      .then((response) => response.json())
      .then((data) => console.log(data));
  };

  return <button onClick={handleClick}>데이터 가져오기</button>;
}
```

- 필요할 때만 실행되므로 불필요한 API 호출을 방지할 수 있다.
