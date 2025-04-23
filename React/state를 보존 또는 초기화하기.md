# state를 보존 또는 초기화하기

<br />

각 컴포넌트는 독립된 state를 가진다. 리액트는 UI 트리에서의 위치를 통해 각 state가 어떤 컴포넌트에 속하는지 추적한다. 리렌더링마다 언제 state를 보존하고 또 state를 초기화할지 컨트롤할 수 있다.

<br />

## state는 렌더트리의 위치에 연결된다

리액트는 UI 안에 있는 컴포넌트 구조로 렌더 트리를 만든다.

컴포넌트에 state를 설정할 때 state가 해당 컴포넌트 내부에 존재한다고 생각할 수 있지만 사실 리액트 안에 저장되어 있다. **리액트는 컴포넌트가 UI 트리 상에 존재하는 위치를 기준으로 각 state를 추적하고, 해당 위치에 있는 컴포넌트와 state를 연결한다.**

```jsx
import { useState } from "react";

export default function App() {
  const counter = <Counter />;
  return (
    <div>
      {counter}
      {counter}
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = "counter";
  if (hover) {
    className += " hover";
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

동일한 `Counter`가 다른 두 군데에서 렌더링되고 있다.

![image.png](attachment:5683a311-5154-47dd-a49c-ea8a0af364e0:image.png)

**이 둘은 트리에서 각각 자기 고유의 위치에 렌더링되어 있으므로 분리되어 있는 카운터다.**

리액트에서 화면의 각 컴포넌트는 완전히 분리된 state를 가진다. 두 `Counter` 컴포넌트를 나란히 렌더링하면 각각 자신만의 독립된 `score`과 `hover` state를 가지게 된다.

![image.png](attachment:3292dd3e-6c05-481f-ae4e-ac537834baa5:image.png)

특정 카운터가 갱신되면 해당 컴포넌트의 상태만 갱신된다.

리액트는 트리의 동일한 컴포넌트를 동일한 위치에 렌더링하는 동안 상태를 유지한다.

```jsx
import { useState } from "react";

export default function App() {
  const [showB, setShowB] = useState(true);
  return (
    <div>
      <Counter />
      {showB && <Counter />}
      <label>
        <input
          type="checkbox"
          checked={showB}
          onChange={(e) => {
            setShowB(e.target.checked);
          }}
        />
        Render the second counter
      </label>
    </div>
  );
}
```

두 카운터를 모두 증가시키고 ‘Render the second counter’ 체크 박스의 체크를 해제하여 두 번째 컴포넌트를 제거한다. 그리고 다시 체크박스를 눌러 두 번째 컴포넌트를 추가한다. 그럼 두 번째 컴포넌트의 `count`가 다시 0으로 초기화되어 있는 것을 확인할 수 있다.

![컴포넌트 삭제](attachment:0d459ae7-6476-4e0f-a7c5-720e4bf99cbf:image.png)

컴포넌트 삭제

![컴포넌트 추가](attachment:6a710af5-57fe-42d6-adc4-b4bf0be6d672:image.png)

컴포넌트 추가

두 번째 카운터를 렌더링하지 않을 때 state 역시 완전히 사라진다. 이는 리액트가 컴포넌트를 제거할 때 state도 같이 제거하기 때문이다.

**리액트는 컴포넌트가 UI 트리에서 그 자리에 렌더링되는 한 state를 유지한다.** 만약 이를 제거하거나 같은 자리에 다른 컴포넌트가 렌더링되면 리액트는 해당 state를 버린다.

<br />

### 같은 자리의 같은 컴포넌트는 state를 보존한다

---

```jsx
import { useState } from "react";

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? <Counter isFancy={true} /> : <Counter isFancy={false} />}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={(e) => {
            setIsFancy(e.target.checked);
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = "counter";
  if (hover) {
    className += " hover";
  }
  if (isFancy) {
    className += " fancy";
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

![image.png](attachment:7e465a49-69aa-4410-829d-2c3fec2cc601:image.png)

![image.png](attachment:7ac85c27-ed47-4c6e-b649-913201132b22:image.png)

체크 박스를 선택하거나 해제할 때 카운터 state는 초기화되지 않는다. `isFancy`가 `true`이든 `false`이든 `Counter`는 같은 자리에 있기 때문이다. 즉 루트인 `App` 컴포넌트가 반환하는 `div`의 첫 번째 자식으로 존재한다.

![image.png](attachment:7f75139e-4723-40cf-a3d1-e3ce5531f49c:image.png)

카운터는 같은 자리에 있으므로 `App`의 상태 갱신은 카운터를 초기화시키지 않는다.

```jsx
export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  if (isFancy) {
    return (
      <div>
        <Counter isFancy={true} />
        <label>
          <input
            type="checkbox"
            checked={isFancy}
            onChange={(e) => {
              setIsFancy(e.target.checked);
            }}
          />
          Use fancy styling
        </label>
      </div>
    );
  }
  return (
    <div>
      <Counter isFancy={false} />
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={(e) => {
            setIsFancy(e.target.checked);
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}
```

위 컴포너트는 `if` 안과 밖에 다른 `Counter`를 가진 return문을 두 개 가지고 있다. 체크박스를 선택할 때 state가 초기화될 거라고 예상할 수 있지만 그렇지 않다. 이는 두 `Counter` 태그가 **같은 위치에 렌더링**되기 때문이다. 두 가지 상황 모두 `App` 컴포넌트는 `Counter`를 첫 번째 자식으로 가진 div를 반환한다. 따라서 리액트는 두 `Counter`를 같은 것으로 보는 것이다.

<br />

## 같은 위치의 다른 컴포넌트는 state를 초기화한다

```jsx
import { useState } from "react";

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? <p>See you later!</p> : <Counter />}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={(e) => {
            setIsPaused(e.target.checked);
          }}
        />
        Take a break
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = "counter";
  if (hover) {
    className += " hover";
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

여기서는 같은 자리이지만 다른 컴포넌트 타입으로 바꾸고 있다. 처음에는 `<div>`가 `Counter`를 갖고 있지만 `p`로 바꾸면 리액트는 UI 트리에서 `Counter`와 그 state를 제거한다.

![image.png](attachment:f49c3b35-e27a-4b78-ae56-3c9e4d296b23:image.png)

`Counter`가 `p`로 바뀌면 `Counter`는 삭제되고 `p`가 추가된다.

![image.png](attachment:7412feb3-d322-4e46-a27c-c61c82fcf773:image.png)

다시 되돌리면 `p`는 삭제되고 `Counter`가 추가된다.

같은 위치에 다른 컴포넌트를 렌더링하면 state는 초기화된다.

```jsx
import { useState } from "react";

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <div>
          <Counter isFancy={true} />
        </div>
      ) : (
        <section>
          <Counter isFancy={false} />
        </section>
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={(e) => {
            setIsFancy(e.target.checked);
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}
```

위 코드에서 체크 박스를 선택할 때 카운터 state가 초기화된다. `div`의 첫 번째 자식으로 `Counter`를 렌더링하는 것에서 `section`의 첫 번째 자식으로 바뀐다. 자식 `div`가 DOM에서 제거될 때 해당 `div`의 전체 하위 트리 역시 제거된다.

![image.png](attachment:bea58a76-e4ac-4307-9da6-955583352b9f:image.png)

`section`이 `div`로 바뀌면 `section`은 삭제되고 새로운 `div`가 추가된다.

![image.png](attachment:db2a14f1-e649-48d3-9788-4aa45211a3ae:image.png)

다시 되돌리면 `div`는 삭제되고 새로운 `section`이 추가된다.

**리렌더링할 때 state를 유지하고 싶다면 트리 구조가 같아야 한다.** 만약 구조가 다르다면 리액트가 트리에서 컴포넌트를 지울 때 state로 지우기 때문에 state가 유지되지 않는다.

따라서 아래와 같이 컴포넌트 함수를 중첩해서 정의하면 안된다.

```jsx
import { useState } from "react";

export default function MyComponent() {
  const [counter, setCounter] = useState(0);

  function MyTextField() {
    const [text, setText] = useState("");

    return <input value={text} onChange={(e) => setText(e.target.value)} />;
  }

  return (
    <>
      <MyTextField />
      <button
        onClick={() => {
          setCounter(counter + 1);
        }}
      >
        Clicked {counter} times
      </button>
    </>
  );
}
```

위 코드는 `MyComponent` 안에서 `MyTextField` 컴포넌트 함수를 정의하고 있다.

![image.png](attachment:7385ba24-8424-4c28-918e-a23d48c18ab0:image.png)

버튼을 누를 때마다 `input`에 입력했던 state가 사라진다. 이는 `MyComponent`를 렌더링할 때마다 다른 `MyTextField` 함수가 만들어지기 때문이다.

1. `MyTextField` 안에는 `text` state가 있다.
2. 버튼을 누르면 `setCounter`가 실행되면서 `MyComponent`가 다시 렌더링된다.
3. 이때 `MyTextField` 함수도 새로 만들어지고, 리액트는 이를 완전히 새로운 컴포넌트로 인식한다.
4. 그 결과 `text` 상태도 초기값인 빈 문자열로 리셋되는 것이다.

따라서 `MyTextField`를 `MyComponent` 함수 밖에, **최상위 범위에 따로 정의**해야 한다. 그럼 리액트가 `MyTextField`를 같은 컴포넌트로 인식해서 안에 있는 `text` 상태도 유지해준다.

<br />

## 같은 위치에서 state를 초기화하기

아래는 두 선수가 턴마다 자신의 점수를 추적하는 앱이다.

```jsx
import { useState } from "react";

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? <Counter person="Taylor" /> : <Counter person="Sarah" />}
      <button
        onClick={() => {
          setIsPlayerA(!isPlayerA);
        }}
      >
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = "counter";
  if (hover) {
    className += " hover";
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>
        {person}'s score: {score}
      </h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

선수를 바꿔도 점수가 유지된다. 두 `Counter`가 같은 위치에 나타나기 때문에 리액트는 그들을 `person` props가 변경된 같은 `Counter`로 본다.

UI에 같은 위치에 나타나지만 하나는 Taylor의 카운터, 하나는 Sarah의 카운터로 두 개의 분리된 카운터가 있어야 한다.

이 둘을 바꿀 때 state를 초기화하기 위한 두 가지 방법이 있다.

1. **다른 위치에 컴포넌트를 렌더링하기**
2. **각 컴포넌트에 `key`로 명시적인 식별자 제공하기**

<br />

## 방법 1: 다른 위치에 컴포넌트를 렌더링하기

```jsx
import { useState } from "react";

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA && <Counter person="Taylor" />}
      {!isPlayerA && <Counter person="Sarah" />}
      <button
        onClick={() => {
          setIsPlayerA(!isPlayerA);
        }}
      >
        Next player!
      </button>
    </div>
  );
}
```

위와 같은 방법으로 해결할 수 있다. 이 내용을 자세히 살펴보면,

- **이전 코드 (한 줄에 조건부 연산자로 Counter 렌더링)**
  ```jsx
  {
    isPlayerA ? <Counter person="Taylor" /> : <Counter person="Sarah" />;
  }
  ```
  리액트 입장에서는 항상 같은 위치에 있는 하나의 `Counter`로 인식한다. 따라서 `person` prop만 바뀐 걸로 판단하고 내부 state는 유지된다.
- **수정한 코드 (각 플레이어를 분리해서 렌더링)**
  ```jsx
  {
    isPlayerA && <Counter person="Taylor" />;
  }
  {
    !isPlayerA && <Counter person="Sarah" />;
  }
  ```
  리액트는 두 개의 `Counter`를 완전히 별개의 컴포넌트로 인식한다.
  > **같은 컴포넌트로 인식하는 예시(이전 내용)**
  >
  > ```jsx
  > if (isFancy) {
  >   return (
  >     <div>
  >       <Counter isFancy={true} />
  >       ...
  >     </div>
  >   );
  > }
  > return (
  >   <div>
  >     <Counter isFancy={false} />
  >     ...
  >   </div>
  > );
  > ```
  >
  > - `Counter`는 항상 `<div>` 안에서 첫 번째 자식 컴포넌트로 등장한다.
  > - JSX 구조상 동일한 위치에서 렌더링된다.
  > - 리액트는 두 경우의 `Counter`를 같은 컴포넌트로 추적하고 상태를 유지한다.
  >
  > **다른 컴포넌트로 인식하는 예시**
  >
  > ```jsx
  > {
  >   isPlayerA && <Counter person="Taylor" />;
  > }
  > {
  >   !isPlayerA && <Counter person="Sarah" />;
  > }
  > ```
  >
  > - 두 개의 `Counter`가 서로 다른 조건문 안에서 각각 렌더링된다.
  > - 리액트 입장에서 보면
  >   - Taylor용 `Counter`는 1번 위치
  >   - Sarah용 `Counter`는 2번 위치
  > - 따라서 번갈아가며 나타나더라도 리액트는 이들을 서로 다른 컴포넌트로 인식한다.

<br />

## 방법 2: key를 이용해 state 초기화하기

key는 배열을 렌더링하기 위해서만 필요한 것이 아니다. 리액트가 컴포넌트를 구별할 수 있도록 key를 사용할 수도 있다.

기본적으로 리액트는 컴포넌트를 구별하기 위해서 부모 안에서의 순서를 이용한다. **하지만 key를 이용하면 단지 첫 번째 카운터나 두 번째 카운터가 아닌 특정한 카운터라고 알려줄 수 있다.**

```jsx
import { useState } from "react";

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button
        onClick={() => {
          setIsPlayerA(!isPlayerA);
        }}
      >
        Next player!
      </button>
    </div>
  );
}
```

key를 명시하면 리액트는 부모 내에서의 순서 대신에 key 자체를 위치의 일부로 사용한다. 따라서 컴포넌트를 JSX에서 같은 자리에 렌더링하지만 리액트 관점에서는 다른 카운터가 되는 것이다.

> key가 전역적으로 유일하지 않다는 것을 유의해야 한다. key는 오직 부모 안에서만 자리를 명시한다.

<br />

## key를 이용해 폼을 초기화하기

key로 state를 초기화하는 것은 특히 폼을 다룰 때 유용하다.

```jsx
import { useState } from "react";
import Chat from "./Chat.js";
import ContactList from "./ContactList.js";

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={(contact) => setTo(contact)}
      />
      <Chat contact={to} />
    </div>
  );
}

const contacts = [
  { id: 0, name: "Taylor", email: "taylor@mail.com" },
  { id: 1, name: "Alice", email: "alice@mail.com" },
  { id: 2, name: "Bob", email: "bob@mail.com" },
];
```

입력란에 타이핑한 후에 Alice나 Bob을 눌러 다른 수신자를 선택해도 `<Chat />`이 트리의 같은 곳에서 렌더링되기 때문에 입력값이 유지되는 것을 볼 수 있다.

```jsx
<Chat key={to.id} contact={to} />
```

key를 추가함으로써 다른 수신자를 선택할 때 `Chat` 컴포넌트가 다시 생성되도록 보장해준다.

<aside>
💡

- 리액트는 같은 컴포넌트가 같은 자리에 렌더링되는 한 state를 유지한다.
- state는 JSX 태그에 저장되지 않는다. state는 JSX로 만든 트리 위치와 연관된다.
- 컴포넌트에 다른 key를 주어서 하위 트리를 초기화하도록 강제할 수 있다.
</aside>
