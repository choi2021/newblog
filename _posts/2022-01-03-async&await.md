(드림코딩 유튜브의 javascript 비동기 시리즈를 공부하고 정리했다.)

## Promise의 문제점

callback function을 연달아 chaining을 하다보면 가독성을 떨어뜨려 유지 보수와 코드를 이해하는데 어려움을 주기 때문에 promise를 사용하듯이 promise를 연속해서 사용하다보면 promise chaining으로 동일한 어려움이 생길 수 있다.

이점을 해결하기 위해 좀 더 간편하게 promise를 만들고 동기적으로 코드를 쓰는 것처럼 쓸수 있게 도와주는 async와 await을 알아보려 한다.

# async

async는 promise를 만드는 함수에서 사용할 수 있다. 다음 예시 코드를 보자

```javascript
function fetchUser (){
  return new Promise((resolve,reject)=>{
    //do network requests in 10secs
    return 'youngjun'
  })
}

// Async로

async function fetchUser(){
  return 'ellie'
}

간단하게 async을 함수 선언의 앞에 붙여주면 promise를 만들어주고 return 으로 값을 전달해준다.
```

# await

await은 말그대로 기다려주는 것으로 async가 쓰이는 함수에서 쓰이고 promise가 fulfiled상태가 될때까지 기다렸다가 결과를 반환하고 외부의 다른 동기 코드는 실행될 수 있게 한다.

```javascript
function delay(ms) {
  return new Promise((resolve) => setTimeOut(resolve, ms));
}

//await을 이용
async function getBanana() {
  await delay(3000);
  return "banana";
}

// await 없이
function getBanana() {
  return delay(3000) //promise 반환
    .then(() => "banana");
}
```

위 코드는 동일한 기능을 하는 code지만 await을 사용했을 때와 하지 않았을 때를 담고 있다. delay함수에 3초의 시간을 전해주어 새로운 promise를 만들고 성공했을 때 banana를 전해주는 함수이다. 위 코드를 이용해 좀 더 복잡하게 만들면 다음과 같다.

```javascript
async function getApple() {
  await delay(3000);
  return "apple";
}

async function getBanana() {
  await delay(3000);
  return "banana";
}

//await 없이 promise로만
function pickFruits() {
  return getApple().then((apple) => {
    return getBanana.then((banana) => `${banana}+${apple}`);
  });
}

//await을 이용해서
async function pickFruits() {
  const apple = await getApple();
  const banana = await getBanana();
  return `${apple}+${banana}`;
}
```

pickFruits함수를 await을 사용해서 나타내면 동기적인 code처럼 쓸 수 있어서 훨씬 편하게 이해할 수 있고 간단하게 표현가능하다는 장점을 잘 보여준다.

## 서로 독립된 Promise 처리

promise들을 받아서 로직을 처리할 때 서로 독립되어있다면 순서대로 동기적으로 처리되게 기다리는 게 아니라 거의 같이 처리할 수있게 하는 것이 더 효율적이다.

```javascript
async function pickFruits() {
  const apple = await getApple(); //1초 필요
  const banana = await getBanana(); //1초 필요
  return `${apple}+${banana}`; //총 2초 필요
}

async function pickFruits() {
  const applePromise = getApple(); //선언하자마자 promise만들어
  const bananaPromsie = getBanana(); //선언하자마자 promise 만들어
  const apple = await getApple();
  const banana = await getBanana();
  return `${apple}+${banana}`; //총 1초 필요
}
```

위의 예는 독립적인 apple과 banana promise를 promise를 선언하면 바로 만들어지는 것을 이용해 거의 동시에 만든 후에 await을 사용해 더 효율적으로 처리했다.

# Promise의 API

Promise 자체에도 api를 가지고 있어 다양한 목적에 맞게 사용할 수 있다.

```javascript
// Promise.all
function pickAllFruits() {
  return Promise.all([getApple(), getBanana()]).then((fruits) =>
    fruits.join("+")
  );
}

function pickOnlyOne() {
  return Promise.race([getApple(), getBanana()]);
}
```

Promise.all은 배열로 받은 각각의 promise 모두를 수행하고 반환받은 값들을 배열로 각각 전달하는 api이고 Promise.race는 배열로 전달받은 promise중 먼저 수행한 promise의 반환값을 전달하는 api이다.

# async await으로 callback hell 문제 해결

이전 callback hell 예제를 promise로 바꾸었다면 이번에는 promise를 async와 await을 이용해 수정해보려 한다.

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

  async getUserWithRoles(id, password) {
    const user = await this.loginUser(id, password); //
    const userInfo = await this.getRoles(user);
    return userInfo;
  }
}

const id = prompt("id?");
const password = prompt("password");
const userStorage = new UserStorage();
userStorage
  .getUserWithRoles(id, password)
  .then((userInfo) =>
    alert(`Welcome ${userInfo.name} you have a ${userInfo.role} role`)
  )
  .catch((error) => console.log(error));
```
