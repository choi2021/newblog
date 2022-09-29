(드림코딩 유튜브의 javascript 비동기 시리즈를 공부하고 정리했다.)

앞선 part1에서 다루었던 callback function을 이용하다보면 callback function이 callback function을 호출하고 호출하면서 코드의 가독성이 떨어지고 유지,보수의 어려움이 있을 수 있다.

## Callback hell)

```javascript
class UserStorage {
  loginUser(id, password, onSuccess, onError) {
    setTimeout(() => {
      if (
        (id === "ellie" && password == "dream") ||
        (id === "coder" && password === "academy")
      ) {
        onSuccess(id);
      } else {
        onError(new Error("not found"));
      }
    }, 2000);
  }

  getRoles(user, onSuccess, onError) {
    setTimeout(() => {
      if (user === "ellie") {
        onSuccess({ name: "ellie", role: "admin" });
      } else {
        onError(new Error("no access"));
      }
    }, 1000);
  }
}

const id = prompt("id?");
const password = prompt("password");
const userStorage = new UserStorage();
userStorage.loginUser(
  id,
  password,
  (id) => {
    userStorage.getRoles(
      id,
      (userWithRole) => {
        alert(
          `welcome ${userWithRole.name} you have a ${userWithRole.role} role`
        );
      },
      (error) => console.log(error)
    );
  },
  (error) => console.log(error)
);
```

Userstorage라는 class에 사용자의 정보를 통해 login과 role을 불러주는 method이 담겨 있다. 위 class를 이용해 사용자 정보를 입력하고 입력한 정보에 따라 callback 함수를 불러오는데 한번에 읽기 어려움이 있다. 이를 해결하기 위해 Promise를 알아보고자 한다.

## Promise란?

Promise는 정해진 기능을 수행하고 성공과 실패여부를 전해주는 object이다.

Promise에는 수행상태를 담고있는 state가 담겨있어 아직 실행되지 않고 있다면 **pending**, 성공했다면 **fulfiled** 실패했다면 **rejected**로 표현된다.

Promise는 크게 Producer와 consumer로 나뉘게 된다.

### 1. Producer

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("ellie"); //또는  reject (new Error("no network"))
  }, 2000);
});
```

위 promise에는 executer로 resolve를 통해 성공했을 때 전달할 내용을 reject를 통해 실패했을 때 전달할 내용을 담고있다. Promise는 만들어지면서 바로 실행된다.

### 2. Consumer

```javascript
Promise.then((value) => console.log(value))
  .catch((error) => console.log(error))
  .finally(() => console.log("finally"));
```

Promise가 만들어 진뒤에 성공과 실패여부에 따라 전달받은 값을 어떻게 이용할 지를 담고있다. **then**은 성공했을 때 resolve를 통해 전달받은 값을 value로 받아 console에 출력하고 **catch**는 실패했을 때 reject를 통해 전달받은 error를 받아 출력한다. **finally**는 성공과 실패여부에 상관없이 수행하게 된다.

### 3.Promise chaining

promise는 api들을 .으로 연결해서 사용하듯이 간단하게 연결해서 이용가능하다.

```javascript
const fetchNumber = new Promise((resolve, reject) => {
  setTimeOut(() => resolve(1), 1000);
});

fetchNumber
  .then((num) => num * 2) //fetchNumber로 부터 받은 1이 1초뒤에 num으로
  .then((num) => num * 3) //앞에서 받은 2가 num으로
  .then((num) => {
    return new Promise((resolve, reject) => {
      setTimeOut(() => resolve(num - 1), 1000);
    });
  }) // 앞에서 받은 6이 num으로 들어간뒤에 1초뒤에 5를 전달해줘
  .then((num) => console.log(num)); //5를 받아서 console에 호출
```

### 4. error handling

Network에서 받을 때 오류가 났을 때 대처할 수 있는 방법으로 성공했을 때를 위한 then다음에 바로 catch를 넣어줘서 실패했을 때 계속해서 promise가 수행되게 할 수 있다.

```javascript
const getHen = () =>
  new Promise((resolve, reject) => {
    setTimeOut(() => resolve(`🐓`), 1000);
  });

const getEgg = (hen) =>
  new Promise((resolve, reject) => {
    setTimeOut(() => resolve(`${hen}=>🥚`));
  });

const cook = (egg) =>
  new Promise((resolve, reject) => {
    setTimeOut(() => resolve(`${egg}=>🍳`), 1000);
  });

getHen() //getHen으로 성공시 '🐓'을 전달
  .then((hen) => getEgg(hen)) //성공시 '🐓=>🥚'을 전달
  .then((egg) => cook(egg)).then; //성공시 '🐓=>🥚=>🍳'전달
  .then(meal=>console.log(meal));//성공시 console에 호출

```

만약 여기서 getEgg에서 error가 발생했다면 then으로만 연결되어 있어서 진행되지 않는다. 이를 해결하기 위해서 catch를 error가 발생한 곳에 다음으로 연결해준다.

```javascript
const getHen = () =>
  new Promise((resolve, reject) => {
    setTimeOut(() => resolve(`🐓`), 1000);
  });

const getEgg = (hen) =>
  new Promise((resolve, reject) => {
    setTimeOut(() => reject(new Error(`can't get egg`)));
  });

const cook = (egg) =>
  new Promise((resolve, reject) => {
    setTimeOut(() => resolve(`${egg}=>🍳`), 1000);
  });

getHen() //getHen으로 성공시 '🐓'을 전달
  .then((hen) => getEgg(hen)) //error 발생
  .catch(error=>{
    return '🎂';
  })
  .then((egg) => cook(egg)).then; //받은 🎂으로 '🎂=>🍳'전달
  .then(meal=>console.log(meal));//성공시 console에 호출
```

이제 Promise를 사용해 위 callback hell을 좀 더 이해하기 쉽게 바꿔보자

## Callback hell to Promise

```javascript
class UserStorage {
  loginUser(id, password) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        if (
          (id === "ellie" && password == "dream") ||
          (id === "coder" && password === "academy")
        ) {
          resolve(id);
        } else {
          reject(new Error("not found"));
        }
      }, 2000);
    });
  }

  getRoles(user) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        if (user === "ellie") {
          resolve({ name: "ellie", role: "admin" });
        } else {
          reject(new Error("no access"));
        }
      }, 1000);
    });
  }
}

const id = prompt("id?");
const password = prompt("password");
const userStorage = new UserStorage();
userStorage
  .loginUser(id, password)
  .then((id) => userStorage.getRoles(id))
  .then((userWithRole) =>
    alert(`Welcome ${userWithRole.name} You have a ${userWithRole.role} role`)
  )
  .catch((error) => console.log(error));
```

callback hell이 조금 더 깔끔해진 걸 확인할 수 있고 가독성이 좋아진 걸 확인 할 수있었다. 이후에 error handling 파트는 아직 조금 더 공부가 필요해보이고 제일 마지막에 catch를 넣어줌으로써 어떤 error가 발생해도 console에 출력되는 것을 확인할 수 있었다.
