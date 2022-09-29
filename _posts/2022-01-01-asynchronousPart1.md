Javascript는 동기적으로 작동하게 되어있는 언어이다. 동기적이란 뜻은 hoisting 이후에 Code를 작성한 한줄 한줄을 읽어나간다. Hoisting이란 지금은 잘 안쓰이는 **Var**나 **function declaration** 와 같은 code를 Javascript가 code의 가장 위로 올려서 먼저 읽는 것을 의미한다. 그래서 fuction declaration이라 var로 변수를 지정하기 전에 변수를 사용하는 것 등이 가능하지만 이 점이 많은 혼란을 주기 때문에 요즘에 var가 잘 안쓰이는 것으로 알고 있다.

이러한 동기적인 일은 일을 순서대로 처리하는데 도움이 되지만 code의 양이 엄청 많고 처리할 일이 많아서 그 일을 끝내고 다음 일을 하기에 사용자가 너무 오랜 시간을 기다려야하는 일이 생길 수 있다. 이를 위해서 **비동기적인 일처리**가 필요하다.

그중에서 callback 함수를 먼저 알아보려 한다.

## Callback function

---

Javascript에서 함수자체가 변수로 설정될 수 있기 때문에 call back function에 대해 쉽게 접할 수 있었다. 하지만 이름이 왜 call back function인지에 대해서는 이번 기회에 공부하면서 알게 되었는데 function이란 기능적인 부분을 하는데 **내가 필요할 때, 필요한 타이밍에 맞게 이 함수를 불러줘** 라는 의미를 갖는다.

## 비동기의 예시

---

예시 1)

비동기 처리의 대표적인 API로 browser의 setTimeOut이 있다.
setTimeOut은 변수로 callback function과 어느정도의 시간 뒤에 전달받은 callback function을 실행할지를 전달 해주어야 한다. 시간은 ms단위로 되어있기에 예시로 1초는 1000으로 전달해준다.

```javascript
console.log(`1`);
setTimeOut(function () {
  console.log("2");
}, 1000);
console.log("3");

//  result
1;
3;
2;
```

동기적으로 실행했다면 콘솔창에 1,2,3 순서대로 나왔겠지만 setTimeOut을 통해서 browser에서 console에 2를 표현해달라는 함수를 1초뒤에 호출해달라 전달함으로써 다음 code였던 3이 먼저 콘솔에 나오게 되고 1초 뒤에 2가 콘솔에 표현된다.
