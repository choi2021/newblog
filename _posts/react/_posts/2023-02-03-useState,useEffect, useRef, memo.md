---
title: 'React study: useState, useEffect, useRef, memo'
categories:
  - react
---



## ❤ React hooks

**React hooks**는 리액트가 사랑 받게 된 큰 이유 중 하나로 생각된다. 기존의 주로 사용되던 `class component`에서 `funtional Component`로 흐름을 바꿀 수 있었던 가장 큰 계기가 hooks의 도입이었다. 베타버전으로 나온 공식 문서를 읽고 그중 가장 자주 사용하는 hooks들의 역할과 내부 동작에 대해 정리해보려 한다.



### 😁 Rule of Hooks

각각의 Hooks에 대해 정리하기 전에 Hooks들이 지켜야 하는 규칙에 대해 먼저 정리하려 한다. 이 부분도 면접에서 질문받았던 부분으로 스코프에 대해서 쉐도잉과 오버 라이팅으로 이야기 했었는데 틀렸던 부분이라 새롭게 정리했다.



#### Only Call Hooks at the Top Level

hooks들은 항상 React fuction의 상위 level에서 사용되어야 한다. 규칙의 장점은 React가 hooks의 작동 순서에 대해서 보장해 줄 수 있다는 점이다.

```jsx
function Form() {
  // 1. Use the name state variable
  const [name, setName] = useState('Mary');

  // 2. Use an effect for persisting the form
  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });

  // 3. Use the surname state variable
  const [surname, setSurname] = useState('Poppins');

  // 4. Use an effect for updating the title
  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

  // ...
}
```

위 예제에서 hooks들이 총 네 개의 hooks가 매 렌더링 때마다 순서대로 진행되게 된다. 여기에 만약 조건문으로 `useEffect(persistForm)`으로 감싼다면 어떻게 될까?

```jsx
function Form() {
  // 1. Use the name state variable
  const [name, setName] = useState('Mary');

  // 2. Use an effect for persisting the form
 if (name !== '') {
    useEffect(function persistForm() {
      localStorage.setItem('formData', name);
    });
  }
  // 3. Use the surname state variable
  const [surname, setSurname] = useState('Poppins');

  // 4. Use an effect for updating the title
  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

  // ...
}
```

`name`이 처음에 'Mary'기 때문에 `useEffect(persistForm)`은 무시된다. 그로 인해 hooks들의 순서가 바뀌게 되고 이전 렌더링과 다른 결과를 만들어 예상치 못하는 버그가 만들어 질 수 있다. 이렇게 순서가 바뀌면 에러가 날 수 있는 부분은 useState와 useEffect



## useState

useState는 **컴포넌트 내의 상태관리**를 위한 hook이다. 리액트 컴포넌트가 기본적으로 리렌더링되는 기준은 `state와 props`가 바뀌었을 때이다. props는 부모 컴포넌트로 부터 전달 받은 변하지 않는 데이터, state는 컴포넌트 내부에서 **변하는 데이터**로 설명할 수 있다. 단순히 변하는 값을 다룬다면 `let`으로 변수에 할당하면 되지 않을까 생각할 수 있지만, `useState`는 (1)데이터를 바꾸고 (2)컴포넌트를 리렌더링하는 2가지 step으로 이루어져 있다. 



### useEffect

useEffect는 리액트 공식문서에서 `외부 system에 따라 컴포넌트를 동기화 시키는 hook`이라고 표현하고 있다. **외부 system**의 예로는 `setInterval`과 같은 timer, `window.addEventListener`와 같은 event subscription이 있다.

useEffect hook은 크게 세가지 파트로 구분된다. (1) hook을 이용해 수행할 동작을 담은 `Setup 함수`, (2) 언제 수행할 지를 담는 `Dependencies`, (3) 수행하던 함수를 멈추기 위한 `cleanup`함수다.  

```jsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
  	const connection = createConnection(serverUrl, roomId); // (1)
    connection.connect(); // (1)
  	return () => connection.disconnect(); // (2) 
  }, [serverUrl, roomId]);
  // ...
}
```

위 예제를 보면 props로 `roomId`를, state로 `serverUrl`를 참조하고 있어 랜더링 이후 connection이 일어난다. dependency로 두 가지 데이터를 참조하고 있기 때문에의 변화가 있을 때, 리랜더링이 되는데 이때 기존에 connection을 하던 것을 끊고 새로운 connection과 연결해 외부와의 연결을 동기화 한다. 



면접에서 질문 받았던 질문 중 기억 남는 문제가 바로 `useEffect`와 관련되어 있었다. **컴포넌트 무한 랜더링** 문제를 어떻게 해결할 지에 대한 질문이었고 처음에 이해하지 못해 브라우저에서 무한 랜더링이 일어날 수 있는 상황에 대해 얘기했던 기억이 있다. 면접관님께서 친절하게 컴포넌트 **무한 랜더링** 상황을 설명해 주셨는데 다음 예제와 유사했다.



```jsx
function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setCount(count + 1) // infinite loop
  }, [count])

  return ...
}
```

위 코드에서 무한 렌더링이 일어나는 이유는 랜더링 이후 useEffect로 `count` 업데이트 *→* dependency의 `count`가 업데이트를 감지*→*  useEffect로 `count`업데이트 ... 가 반복되기 때문이다. 

[codesandbox로 실행했을 때 나온 에러]

![image-20230207193611288](../../../assets/img/2023-02-03-useState,useEffect, useRef, memo/image-20230207193611288.png)



이러한 문제를 해결하기 위해서는 `count`를 useEffect의 dependency로 전달해 업데이트 하는 것이 아니라 useState의 functional Update를 이용해 해결할 수 있다.

```jsx
function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setCount(previousCount => previousCount + 1)
  }, [])

  return ...
}
```





### useRef

`useRef`는 리액트 공식문서에서 `랜더링이 되지 않을 값을 참조하는 hook`으로 소개하고 있다. 다양한 타입의 값을 할당할 수 있고 변경 가능한 값이다. 컴포넌트 내부에서 변경이 가능한 데이터로 state가 있는데 state와 구분되는 점은 값의 변경으로 인해 리랜더링이 되지 않는다는 점이다. 또한 일반 변수와 다른 점은 useRef로 저장된 값을 리랜더링 이후에도 같은 값을 가질 수 있다.

`useRef`로 저장한 값을 변경할 때는 객체의 `current`속성을 직접 변경해야 한다.

```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}

```



`useRef`를 사용하면서 주의해야 할 점은 **rendering과정에 ref 값을 쓰고 읽어서는 안된다**는 점이다. 

```jsx
function MyComponent() {
  // ...
  // 🚩 Don't write a ref during rendering
  myRef.current = 123;
  // ...
  // 🚩 Don't read a ref during rendering
  return <h1>{myOtherRef.current}</h1>;
}
```



`useRef`가 가장 많이 사용되는 경우는 react에서 **DOM요소를 참조할 때**다. DOM Node의 `ref` 속성으로 `useRef`로 선언한 변수를 연결하면 `useRef`객체의 `current`속성에 해당 DOM node를 참조할 수 있고, Node가 가지는 속성과 method를 사용할 수 있다.

```jsx
import { useRef } from 'react';

export default function CatFriends() {
  const listRef = useRef(null);

  function scrollToIndex(index) {
    const listNode = listRef.current;
    // This line assumes a particular DOM structure:
    const imgNode = listNode.querySelectorAll('li > img')[index];
    imgNode.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToIndex(0)}>
          Tom
        </button>
        <button onClick={() => scrollToIndex(1)}>
          Maru
        </button>
        <button onClick={() => scrollToIndex(2)}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul ref={listRef}>
          <li>
            <img
              src="https://placekitten.com/g/200/200"
              alt="Tom"
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/300/200"
              alt="Maru"
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/250/200"
              alt="Jellylorum"
            />
          </li>
        </ul>
      </div>
    </>
  );
}


```



### Memo

memo는 부모로 전달받은 props가 변하지 않으면 컴포넌트의 리랜더링을 막는 방법이다. props만을 비교하기 때문에  컴포넌트 내부의 state 변화나 context API로 감싸져 있을 때 전달되는 값이 변할 때는 리랜더링이 된다. 

props를 비교할 때 사용하는 방법이 `Object.is(a,b)`로 **얕은 비교**를 하기 때문에 object를 전달할 때는 useMemo나 useCallback와 같은 hook을 통해 memoization을 통해 저장해 같은 값을 전달하거나 필요한 props를 원시값 속성으로 전달할 수 있다.

```jsx
function Page() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);

  const person = useMemo(
    () => ({ name, age }),
    [name, age]
  );

  return <Profile person={person} />;
}

const Profile = memo(function Profile({ person }) {
  // ...
});
```



면접을 통해 또 배웠던 부분은 memo를 사용 시에 두 번째 인자로 **custom comparison function**을 전달해 사용할 수 있다는 점이다. 이때 주의할 점은 모든 props들에 대해서 알고 있어야 하고 기존 얕은 복사보다 성능이 더 개선될 수 있을 때 사용해야 한다. 

```jsx
const Chart = memo(function Chart({ dataPoints }) {
  // ...
}, arePropsEqual);

function arePropsEqual(oldProps, newProps) {
  return (
    oldProps.dataPoints.length === newProps.dataPoints.length &&
    oldProps.dataPoints.every((oldPoint, index) => {
      const newPoint = newProps.dataPoints[index];
      return oldPoint.x === newPoint.x && oldPoint.y === newPoint.y;
    })
  );
}
```

