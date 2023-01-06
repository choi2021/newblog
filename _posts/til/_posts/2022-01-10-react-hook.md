---
title: '2022-01-10-TIL'
layout: single
categories:
  - til
---

![react](https://media.vlpt.us/images/ajt1097/post/870bfbf7-2d22-49ea-9ae6-10033ae353c6/React.jpeg)

# React Hooks

react hook은 react의 component를 만드는 2가지 방법 (class와 함수) 중에서 **function**을 이용할 때 쓰인다. class와 달리 state와 life cycle 메소드를 가질 수 없는 함수에 **hook**을 연결함으로써 this binding 이슈를 해결하고 좀 더 이해가 쉽게 되는 장점을 갖게 되었다. 이러한 유용한 **Hooks**의 대표적인 예: **useState, useCallback, useEffect** 3가지에 대해 정리하고자 한다.

# 1.useState

class와 달리 state를 가질 수 없는 function component에 state를 가질 수 있게 해주고 다음과 같이 쓰인다.

```javascript
//class componenet

class Button extends Component {
  state = {
    count: 0,
  };

  onIncrement() {
    this.setState({ count: this.state.count++ });
  }

  render() {
    return <button onClick={this.onIncrement}>{this.state.count}</button>;
  }
}

// Function component

const Button = () => {
  const [count, setCount] = useState(0);
  const onIncrement = () => {
    setCount((count) => count + 1);
  };
  return <button onClick={onIncrement}>{count}</button>;
};
```

class component와 달리 바로 변수를 선언하고 사용할 수 있다. useState를 사용할 떄는 배열로 state와 state를 변경할 때 사용할 함수를 정한다. 그리고 useState에 초기값을 전해주는데 예시에는 0을 count에 주었다. class를 사용해 state를 업데이트를 할 떄는 **setState()**를 이용하지만 함수에서는 useState를 통해 정한 setCount를 이용해 업데이트한다.

# 2.useCallback

함수형 component가 가지는 큰 특징중 하나는 class와 달리 페이지가 rendering될 때마다 계속해서 새롭게 함수들이 만들어지는 것이다. 불필요한 re-rendering을 막아 성능을 높이기 위해 useCallback을 이용한다.

```javascript
const Button = () => {
  const [count, setCount] = useState(0);
  const onIncrement = useCallback(() => {
    setCount((count) => count + 1);
  }, [count]);
  return <button onClick={onIncrement}>{count}</button>;
};
```

위 예시에서 useCallback을 이용했는데 변수로 불필요한 re-rendering을 막고싶은 함수와 배열을 전달하는데 배열안 값이 변할 때 함수를 다시 만들 수 있게한다. 함수도 object와 같이 reference값을 가지고 있기 때문에 새롭게 만들어 새로운 reference값을 할당하게 된다.

# 3.useEffect

useEffect는 아직 사용할 기회가 없어서 구체적인 예시는 들지 못하지만 class component에서 사용되는 **componentDidMount()** 와 **componentDidUpdate()** 2가지 함수를 함께 담고 있어 인자로 useCallback과 같이 함수와 배열을 전달하는데 배열에 전달된 값이 변화가 있을 때 update해주고 빈 배열을 전달했을 때는 mount됬을 때, 처음 만들어졌을 때만 전달한 함수를 수행한다.
