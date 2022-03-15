# Event Loop와 비동기 처리

> 싱글스레드를 기반으로 동작하는 자바스크립트
이벤트 루프를 기반으로 하는 싱글 스레드 Node.js
> 

# 자바스크립트 엔진

### 자바스크립트 엔진이란?

<aside>
👉🏻 자바스크립트로 작성한 코드를 해석하고 실행하는 **인터프리터**

</aside>

주로 웹 브라우저에서 이용되지만, 최근 node.js가 등장하면서 server side에선 V8과 같은 Engine이 사용됨

- **기본적으로 하나의 스레드에서 동작**
하나의 스레드 = 하나의 stack을 가지고 있다 = **`동시에 단 하나의 작업만 할 수 있다`**
- 자바스크립트 엔진은 하나의 코드 조각을 실행하는 일을 하고, 비동기적 이벤트를 처리하거나 Ajax 통신하는 작업은 모두 **Web API에서 처리** 됨

자바스크립트는 동시에 단 하나의 작업만 함, 그렇다면 어떻게 여러가지 작업을 비동기로 작업할 수 있을까?

**→ Event Loop와 Queue**

### 자바스크립트 엔진 영역

![Untitled](Event%20Loop%2016e05/Untitled.png)

- Call Stack
- Task Queue (Event Queue)
- Heap
    
    추가적으로 Event Loop가 존재 (계속해서 Call Stack과 Queue 사이의 작업을 확인하고, Call Stack이 비워져있는 경우 작업을 dequeue해 Call Stack에 넣음)
    

# Event Loop와 Queue

**Event Loop에서 Loop의 사전적인 의미는 '반복. 순환'**

- Event Loop는 계속 반복해서 Call stack과 queue 사이의 작업들을 확인
- Call stack이 비워있는 경우 queue에서 작업을 꺼내어 Call stack에 넣음

자바스크립트는 **Event Loop와 Queue**들을 이용하여 비동기 작업을 수행함
직접적인 작업은 Web API에서 처리되고, 작업들이 완료되면 요청시 등록했던 callback이 queue에 등록됨

**`Event Loop는 Queue에서 작업들을 꺼내어 처리한다.`**

- Event Loop는 stack에 처리할 작업이 없을 경우 우선적으로 microtask queue를 확인
- 만약 microtask의 queue가 비어서 더 이상 처리할 작업이 없으면, 이때 task queue를 확인

이렇게 **Event Loop와 Queue는 자바스크립트 엔진이 하나의 코드 조각을 하나씩 처리할 수 있도록 작업을 스케줄링**하는 동시에 이러한 이유로 우리는 자바스크립트에서 비동기 작업을 할수 있도록 함

# Call Stack

자바스크립트는 단 하나의 호출 스택을 사용

- 자바스크립트 함수가 실행되는 방식을 `Run to Completion`이라고 함
- 하나의 함수가 실행되면 함수의 실행이 끝날 때까지 어떤 task도 수행 불가능
- 요청이 들어올 때마다 해당 요청을 순차적으로 호출 스택에 담아 처리

📌 해당 코드는 어떻게 동작할까?

```jsx
function foo(b) {
	let a = 10;
	return a + b;
}

function bar(x) {
	let y = 2;
	return foo(x + y);
}

console.log(bar(1));
```

1. bar 함수를 호출하면 bar에 해당하는 스택 프레임 생성, y와 같은 local variable과 arguments가 함께 생성
2. bar 함수는 foo 함수를 호출 (bar는 종료되지 않았으므로 stack에서 pop되지 않음)
3. foo 함수가 Call stack에 push
4. foo 함수에서 a+b를 return하면서 종료되므로 stack에서 pop
5. bar함수도 foo에서 받은 값을 return하면서 stack에서 pop

# Task Queue

처리해야하는 Task들을 임시저장하는 대기 큐가 존재 → Task Queue (Event Queue)

**Call stack이 비었을 때, 먼저 대기열에 들어온 순서대로 진행**

```jsx
setTimeout(function() {
	console.log("first");
}, 0);

console.log("second");

// Console >>
// second
// first
```

- 자바스크립트에서 비동기로 호출되는 함수들은 Call stack에 쌓이지 않고, **Task Queue에 enqueue 됨**
- **이벤트에 의해 실행되는 함수**(핸들러)들이 비동기로 실행
- 자바스크립트 엔진이 아닌 **Web API** 영역에 따로 정의되어있는 함수도 비동기로 실행

📌 해당코드는 어떤 순으로 출력될까?

```jsx
function test1() {
	console.log("test1");
	test2();
}

function test2() {
	let timer = setTimeout(function() {
		console.log("test2");
  }, 0);	
	test3();
}

function test3() {
	console.log("test3");
}

test1();
```

1. test1 출력
2. test2 함수가 호출되면서 setTimeout 함수 실행되고 Call stack에 들어간 다음 바로 빠져나옴 (0초)
3. 내부에 있는 익명함수(핸들러)는 콜스택에 들어가서 바로 실행되는 것이 아니라 event queue 영역으로 들어감
4. test3 함수가 호출되고 test3이 출력되면서 stack에서 pop 됨
5. 이후 test2와 test1이 pop되고 Call stack이 비게 됨
6. event queue에서 이벤트를 가져와서 Call stack으로 넣음 (익명함수 실행)

# 자바스크립트 처리 과정

📌 해당 코드는 어떤 순으로 출력될까?

```jsx
console.log("script start");

setTimeout(function() {
  console.log("setTimeout");
}, 0);

Promise.resolve().then(function() {
  console.log("promise1");
}).then(function() {
  console.log("promise2");
});

requestAnimationFrame(function {
    console.log("requestAnimationFrame");
});

console.log("script end");
```

```jsx
// 실행 결과
script start
script end
promise1
promise2
requestAnimationFrame
setTimeout
```

이 코드는 다음과 같이 처리된다.

![Untitled](Event%20Loop%2016e05/Untitled%201.png)

1. Script 실행 작업이 stack에 등록
2. console.log(’script start’)가 처리
3. setTimeout 작업이 stack에 등록되고, Web API에게 setTimeout을 요청
이때 setTimeout의 callback 함수를 함께 전달, 요청 이후 stack에 있는 setTimeout 작업은 제거

![Untitled](Event%20Loop%2016e05/Untitled%202.png)

1. Web API는 setTimeout 작업(0초 후)이 완료되면 setTimeout callback 함수를 task queue에 등록
2. Promise 작업이 stack에 등록되고, Web API에게 Promise 작업을 요청
이때 Promise.then의 callback 함수를 함께 전달, 요청 이후 stack에 있는 Promise 작업은 제거
Web API는 Promise 작업이 완료되면 Promise.then의 callback 함수를 microtask queue에 등록

![Untitled](Event%20Loop%2016e05/Untitled%203.png)

1. requestAnimation 작업이 stack에 등록되고, Web API에게 requestAnimation을 요청
이때 requestAnimation의 callback 함수를 함께 전달, 요청 이후 stack에 있는 requestAnimation 작업은 제거
2. Web API는 requestAnimation의 callback 함수를 animation frame에 등록

![Untitled](Event%20Loop%2016e05/Untitled%204.png)

1. console.log('script end')가 처리
2. 'script 실행 작업'이 완료되어 stack에서 제거
3. stack이 비워있어서 microtask queue에 등록된 Promise.then 의 callback 함수를 stack에 등록

![Untitled](Event%20Loop%2016e05/Untitled%205.png)

![Untitled](Event%20Loop%2016e05/Untitled%206.png)

1. 첫번째 Promise.then의 callback 함수가 실행되어 내부의 console.log('promise1')가 처리
2. 첫번째 Promise.then 다음에 Promise.then이 있다면, 다음 Promise.then의 callback 함수를 microtask queue에 등록
3. stack 에서 첫번째 Promise.then의 callback 함수를 제거하고 microtask queue에서 첫번째 Promise.then의 callback 함수를 제거
4. 두번째 Promise.then의 callback 함수를 stack에 등록

![Untitled](Event%20Loop%2016e05/Untitled%207.png)

![https://sculove.github.io/static/5f5b3b75ac7856148c66e1113321ad66/a6d36/promise-step3.png](https://sculove.github.io/static/5f5b3b75ac7856148c66e1113321ad66/a6d36/promise-step3.png)

1. 두번째 Promise.then의 callback 함수가 실행되어 내부의 console.log('promise2')가 처리
2. stack 에서 두번째 Promise.then의 callback 함수를 제거
3. microtask 작업이 완료되면 animation frame에 등록된 callback 함수를 꺼내 실행

![Untitled](Event%20Loop%2016e05/Untitled%208.png)

![Untitled](Event%20Loop%2016e05/Untitled%209.png)

1. 이후 브라우저는 랜더링 작업을 하여 UI를 업데이트
2. stack과 microtask queue가 비워있어서 task queue에 등록된 callback 함수를 꺼내 stack에 등록
3. setTimeout의 callback가 실행되어 내부의 console.log('setTimeout')이 처리
4. setTimeout의 callback 함수 실행이 완료되면 stack에서 제거

- 비동기 작업으로 등록되는 작업은 task와 microtask. 그리고 animationFrame 작업으로 구분 됨
- microtask는 task보다 먼저 작업이 처리
- microtask가 처리된 이후 requestAnimationFrame이 호출되고 이후 브라우저 랜더링이 발생

> 참조
[https://sculove.github.io/post/javascriptflow/](https://sculove.github.io/post/javascriptflow/)
> 

## Event Loop 작동

<aside>
👉🏻 Q. Event Loop는 Call Stack을 어떻게 비어있는지 확인하는 것인가?
Q. Event Queue에도 event가 있는지 어떻게 확인하는 것인가?
Q. Event Loop에 의해서 Event Queue에 있던 하나의 이벤트가 Call Stack에 들어간 다음에는
     그 이벤트가 끝나기 전까지 이벤트 루프는 이벤트 큐에서 이벤트를 dequeue하지 않나?

</aside>

```jsx
while (queue.waitForMessage()) {
	queue.processNextMessage();
}
```

- 이벤트 루프는 위 코드와 같은 방식으로 실행 중인 테스크의 여부를 반복적으로 확인
- Queue에 처리해야하는 이벤트가 존재하면 while-loop 안으로 들어가서 해당 이벤트를 처리하는 작업을 수행
- 그러고 다시 Queue로 돌아와 새로운 이벤트가 존재하는지 파악, Event Queue에서 대기하고 있는 이벤트들은 한 번에 하나씩 Call Stack으로 호출되어 처리 됨
