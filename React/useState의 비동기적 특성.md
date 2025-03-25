# useState의 비동기적 특성

<br />

## useState의 비동기적 특성

리액트의 `useState`는 상태를 즉시 변경하지 않고 **비동기적으로 업데이트**하며 **다음 렌더링에서 변경된 값이 반영**된다. 따라서 상태를 업데이트한 직후에는 이전 값이 유지될 수 있다.

<br />

## 비동기적 특성 예시

1. **상태 업데이트가 즉시 반영되지 않는 예시**

```jsx
import { useState } from "react";

export default function FeedbackForm() {
  const [name, setName] = useState("");

  function handleClick() {
    setName(prompt("What is your name?")); // 상태 업데이트 (비동기)
    alert(`Hello, ${name}!`); // 이전 상태를 사용
  }

  return <button onClick={handleClick}>Greet</button>;
}
```

- `name`이 즉시 업데이트 되지 않고 이름 입력 시 ‘Hello, !’만 출력된다.
- `setName`을 호출하면 리액트가 상태 업데이트를 예약하지만 즉시 적용되지는 않는다.
- `alert(Hello, ${name}!);`가 실행되는 시점에서는 `name`이 여전히 업데이트 이전 값을 가진다.
- 즉 `setState`가 적용되기 전에 `alert`가 실행되므로 사용자가 입력한 값이 반영되지 않는다.

```jsx
function handleClick() {
  const newName = prompt("What is your name?"); // 입력값을 변수에 저장
  setName(newName); // 상태 업데이트 (비동기)
  alert(`Hello, ${newName}!`); // 즉시 반영된 값 사용
}
```

- 위와 같이 수정하면 상태 업데이트와 관계없이 `newName`이 즉시 반영된다.

<br />

2. **`setState` 호출 후에도 콘솔에 이전 값이 출력되는 예시**

```jsx
import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    console.log(count); // 여전히 이전 값이 출력됨!
  }

  return <button onClick={handleClick}>Count: {count}</button>;
}
```

- 이 예시 역시 `setCount(count + 1)`을 호출하면 리액트가 렌더링을 예약하지만, `console.log(count)`가 실행될 때는 아직 상태가 변경되지 않았다. 즉 콘솔에 출력되는 값은 **변경되기 전의 값**이다.

```jsx
import { useState, useEffect } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log(`Updated count: ${count}`);
  }, [count]); // count가 변경될 때마다 실행

  function handleClick() {
    setCount(count + 1);
  }

  return <button onClick={handleClick}>Count: {count}</button>;
}
```

- `useEffect`를 사용하여 상태 변경 후에 동작을 실행하도록 한다.
- `count`가 변경될 때마다 `console.log`가 실행되므로 변경된 값이 출력된다.

<br />

3. **`setState`가 여러 번 호출될 때 마지막 값만 반영되는 예시**

```jsx
import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  }

  return <button onClick={handleClick}>Count: {count}</button>;
}
```

- `setCount`를 여러 번 호출했는데도 1만 증가한다.
- 여러 개의 `setCount` 호출을 한 번의 렌더링에서 처리하려 하므로 `setCount(count + 1)`이 실행될 때마다 최신 값이 아닌 기존 값을 기반으로 계산한다. 이는 리액트의 **배치 처리**로 인해 일어나는 현상이다.
  > **배치(batch) 처리** <br /> <br />
  > **배치(batch)** 란 리액트가 더 나은 성능을 위해 같은 이벤트 핸들러 내에서 발생하는 여러개의 state 업데이트를 하나의 리랜더링으로 묶는 것을 의미한다.
- 결과적으로 `count`는 1만 증가한다.

```jsx
function handleClick() {
  setCount((prevCount) => prevCount + 1);
  setCount((prevCount) => prevCount + 1);
  setCount((prevCount) => prevCount + 1);
}
```

- `setState`에 **함수**를 전달하면 최신 상태를 기반으로 값을 업데이트할 수 있다.
- `prevCount`는 항상 최신 상태 값을 참조하므로 클릭할 때마다 `count`가 3씩 증가한다.

<br />

<aside>
💡

- `useState`는 **비동기적**으로 상태를 업데이트한다.
- 리액트는 여러 개의 state 업데이트를 하나로 묶어 **배치 처리**한다.
- `useState`를 **동기적**으로 처리하는 방법
  - `useEffect`의 의존성 배열 이용
  - `setState`의 인자로 함수 사용
  </aside>
