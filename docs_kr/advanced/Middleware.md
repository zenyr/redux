# 미들웨어

여러분이 [Express](http://expressjs.com/)나 [Koa](http://koajs.com/)같은 서버사이드 라이브러리를 사용하신다면, **미들웨어**라는 컨셉에 익숙하실겁니다. 이들 프레임워크에서 미들웨어는 프레임워크가 요청을 받고 응답을 만드는 사이에 놓을 수 있는 코드입니다. 예를 들어, Express나 Koa 미들웨어는 CORS 헤더를 추가하거나 로깅을 하거나 압축을 하거나 다른 것들도 할 수 있죠. 미들웨어의 가장 좋은 점은 체이닝을 통해 조합 가능하다는 점입니다. 여러분은 하나의 프로젝트에서 여러개의 개별 서드파티 미들웨어들을 사용할 수 있습니다.

Redux의 미들웨어는 Express나 Koa와는 다른 문제를 해결하지만 해결방법의 컨셉은 비슷합니다. **미들웨어는 액션을 보내는 순간부터 스토어에 도착하는 순간까지 사이에 서드파티 확장을 사용할 수 있는 지점을 제공합니다.** 여러분은 미들웨어를 로깅이나, 충돌 보고나, 비동기 API와의 통신이나, 라우팅이나 기타 등등에 사용할 수 있습니다.

이 글은 여러분이 컨셉을 완전히 이해하도록 소개하는 부분과 [몇가지 실용적인 예시](#일곱가지-예시) 부분으로 나뉩니다. 혹시 지루해지거나 와닿는 부분이 있으면 앞뒤로 왔다갔다하면서 보시는 것도 도움이 될겁니다.


>##### 성격 급한 분들을 위한 한마디

>여러분은 [다음 섹션](AsyncActions.md)에서 비동기 액션을 위한 실용적인 미들웨어 사용법에 대해 확인하실 수 있습니다. 하지만 우리는 다음 섹션으로 건너뛰지 않기를 권합니다.

>미들웨어는 Redux에서 여러분이 만날 가장 "마술적인" 부분입니다. 미들웨어가 어떻게 작동하는지 배우고 직접 작성하는 방법을 배우는 것은 Redux의 생산성을 위한 가장 좋은 투자입니다.

>정말 못참겠다면 [일곱가지 예시](#일곱가지-예시)로 넘어갔다가 돌아오세요.

## 미들웨어 이해하기

미들웨어가 비동기 API 호출을 포함해서 여러가지로 쓰일 수 있지만, 이 기능이 어디서 나왔는지 아는 것은 매우 중요합니다. 로깅과 충돌 보고를 예로 들어서 여러분이 미들웨어가 나온 생각의 흐름을 따라갈 수 있도록 이끌어보겠습니다.

### 문제: 로깅

Redux의 장점 중 하나는 상태 변화를 예측가능하고 투명하게 만든다는 점입니다. 액션이 보내질때마다 새 상태가 계산되고 저장됩니다. 상태는 스스로 변할 수 없으며 특정 액션의 결과로만 변경됩니다.

앱에서 일어나는 모든 액션을 이후 계산되는 상태와 묶어서 로깅한다면 멋지지 않을까요? 뭔가 잘못된다면 로그를 들여다보고 어느 액션이 상태를 망쳤는지 확인할 수 있을겁니다.

<img src='http://i.imgur.com/BjGBlES.png' width='70%'>

Redux에서는 어떻게 접근해야 할까요?

### 시도 #1: 직접 로깅하기

가장 세련되지 못한 방법은 [`store.dispatch(action)`](../api/Store.md#dispatch)을 호출할때마다 액션과 다음 상태를 직접 로깅하는겁니다. 이건 방법이라고 하기도 어렵고 문제를 이해하기 위한 첫 단계일 뿐이죠.

>##### 한마디

>여러분이 [react-redux](https://github.com/gaearon/react-redux)나 비슷한 바인딩을 사용한다면 컴포넌트에서 바로 스토어로 접근할 수 없을겁니다. 다음 몇 문단 동안은 여러분이 스토어를 명시적으로 전달했다고 가정하겠습니다.

여러분이 할일을 만들때 이렇게 호출한다고 해봅시다:

```js
store.dispatch(addTodo('Use Redux'));
```

액션과 상태를 로깅하려면, 이렇게 바꾸면 됩니다:

```js
let action = addTodo('Use Redux');

console.log('dispatching', action);
store.dispatch(action);
console.log('next state', store.getState());
```

이렇게 하면 원하던 효과를 얻지만, 이렇게 매번 할 수는 없습니다.

### 시도 #2: 디스패치 감싸기

로깅을 함수로 뽑아낼 수 있습니다:

```js
function dispatchAndLog(store, action) {
  console.log('dispatching', action);
  store.dispatch(action);
  console.log('next state', store.getState());
}
```

이걸 `store.dispatch()` 대신 어디에나 사용할 수 있습니다:

```js
dispatchAndLog(store, addTodo('Use Redux'));
```

이걸로 끝이지만, 매번 특별한 함수를 불러오는건 별로 편리하지 못합니다.

### Attempt #3: 디스패치 몽키패칭하기

우리가 스토어 인스턴스에 있는 `dispatch` 함수를 대체한다면 어떨까요? Redux의 스토어는 [몇개의 메서드](../api/Store.md)를 가진 평범한 오브젝트일 뿐이고, 우리는 자바스크립트로 작성하고 있으니 `dispatch`구현을 몽키패칭할 수 있습니다:

```js
let next = store.dispatch;
store.dispatch = function dispatchAndLog(action) {
  console.log('dispatching', action);
  let result = next(action);
  console.log('next state', store.getState());
  return result;
};
```

벌써 우리가 원하는 수준에 가까워졌습니다! 어디서 액션을 보내건 로깅이 보장됩니다. 몽키패칭이 좋게 느껴지진 않지만, 일단 이렇게 해봅시다.

### 문제: 충돌 보고

우리가 `dispatch`에 이런 변환을 **두 개 이상** 적용하고 싶다면 어떨까요?

저에게 떠오르는 다른 쓸만한 변환은 실제 환경에서 자바스크립트 에러를 보고해주는 것입니다. 전역 `window.onerror` 이벤트는 구형 브라우저에서는 스택 정보를 제공하지 않아서 에러가 왜 일어났는지 알기 어렵기 때문에 믿을만하지 못합니다.

액션을 보내서 에러가 날 때마다 스택 추적과 에러를 일으킨 액션과 현재 상태를 [Sentry](https://getsentry.com/welcome/) 같은 충돌 보고 서비스에 보내준다면 쓸만하지 않을까요? 이를 통해 에러를 개발 환경에서 재현하기 쉬워질겁니다.

하지만 로깅과 충돌 보고를 분리된채로 유지하는 것은 중요합니다. 이상적으로는 이들을 서로 다른 모듈로 하고, 어쩌면 다른 패키지에 두었으면 합니다. 그러지 않으면 이런 유틸리티들로 이루어진 생태계를 가질 수 없겠죠. (힌트: 우리는 점차 미들웨어가 무엇인지에 가까워지고 있습니다!)

만약 로깅과 충돌 보고가 분리된 유틸리티라면, 이런 식으로 보일겁니다:

```js
function patchStoreToAddLogging(store) {
  let next = store.dispatch;
  store.dispatch = function dispatchAndLog(action) {
    console.log('dispatching', action);
    let result = next(action);
    console.log('next state', store.getState());
    return result;
  };
}

function patchStoreToAddCrashReporting(store) {
  let next = store.dispatch;
  store.dispatch = function dispatchAndReportErrors(action) {
    try {
      return next(action);
    } catch (err) {
      console.error('Caught an exception!', err);
      Raven.captureException(err, {
        extra: {
          action,
          state: store.getState()
        }
      });
      throw err;
    }
  };
}
```

이들 함수를 분리된 모듈로 내놓을 수 있다면, 나중에 다시 스토어에 적용할 수 있습니다:

```js
patchStoreToAddLogging(store);
patchStoreToAddCrashReporting(store);
```

아직 별로 좋지 않네요.

### 시도 #4: 몽키패칭 숨기기

몽키패칭은 임시방편입니다. "여러분이 원하는 메서드를 대체합니다", 이런 API가 어딨나요? 대신 핵심이 뭔지 알아봅시다. 앞에서 우리는 `store.dispatch`를 대체했습니다. 만약 새 `dispatch` 함수를 **반환**한다면 어떨까요? 

```js
function logger(store) {
  let next = store.dispatch;

  // Previously:
  // store.dispatch = function dispatchAndLog(action) {

  return function dispatchAndLog(action) {
    console.log('dispatching', action);
    let result = next(action);
    console.log('next state', store.getState());
    return result;
  };
}
```
Redux 안에 실제 몽키패칭을 적용할 수 있게 돕는 핼퍼를 제공할 수 있습니다:

```js
function applyMiddlewareByMonkeypatching(store, middlewares) {
  middlewares = middlewares.slice();
  middlewares.reverse();

  // Transform dispatch function with each middleware.
  middlewares.forEach(middleware =>
    store.dispatch = middleware(store)
  );
}
```

여러 미들웨어를 적용할때는 이렇게 사용하면 됩니다:

```js
applyMiddlewareByMonkeypatching(store, [logger, crashReporter]);
```

하지만 이건 아직 몽키패칭이죠.
라이브러리 안에 숨긴다고 해서 이 사실이 변하지는 않습니다.

### 시도 #5: 몽키패칭 제거하기

왜 우리는 `dispatch`를 덮어씌워야 하죠? 물론, 나중에 호출하기 위해서이지만 다른 이유도 있습니다: 그래야 모든 미들웨어들이 이전에 감싸진 `store.dispatch`에 접근(하고 호출)할 수 있기 때문이죠:

```js
function logger(store) {
  // Must point to the function returned by the previous middleware:
  let next = store.dispatch;

  return function dispatchAndLog(action) {
    console.log('dispatching', action);
    let result = next(action);
    console.log('next state', store.getState());
    return result;
  };
}
```

이게 미들웨어 체이닝의 핵심입니다!

첫 번째 미들웨어 처리가 끝나고 바로 `applyMiddlewareByMonkeypatching`를 `store.dispatch`에 할당하지 않는다면, `store.dispatch`는 여전히 원래의 `dispatch`함수를 가리키고 있을겁니다. 그러면 두번째 미들웨어 또한 원래의 `dispatch` 함수에 바인딩될 수 있겠죠.

하지만 체이닝을 가능하게 하는 다른 방법이 있습니다. 미들웨어가 `next()` 디스패치 함수를 `store` 인스턴스에서 읽어오는 대신 인자로 받을 수 있습니다.

```js
function logger(store) {
  return function wrapDispatchToAddLogging(next) {
    return function dispatchAndLog(action) {
      console.log('dispatching', action);
      let result = next(action);
      console.log('next state', store.getState());
      return result;
    };
  }
}
```

이건 일종의 [“we need to go deeper”](http://knowyourmeme.com/memes/we-need-to-go-deeper)여서 이해하는데 좀 걸립니다. 이렇게 함수가 늘어서있으면 겁먹게 되죠. ES6의 화살표 함수가 이 [커링](https://en.wikipedia.org/wiki/Currying)을 더 알아보기 쉽게 만듭니다:

```js
const logger = store => next => action => {
  console.log('dispatching', action);
  let result = next(action);
  console.log('next state', store.getState());
  return result;
};

const crashReporter = store => next => action => {
  try {
    return next(action);
  } catch (err) {
    console.error('Caught an exception!', err);
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    });
    throw err;
  }
}
```

**이게 바로 Redux의 미들웨어가 생긴 모양입니다.**

이제 미들웨어는 `next()` 디스패치 함수를 받아서, 디스패치 함수를 반환하고, 이는 다시 왼쪽의 미들웨어에 `next()`로 전달되고, 이런 식으로 계속됩니다. 스토어의 `getState()` 같은 메서드에 접근할 수 있으면 유용하기 때문에 `store`는 계속 최상위 인수로 남아있습니다.

### 시도 #6: 적당히 미들웨어 적용하기 

`applyMiddlewareByMonkeypatching()` 대신, 완전히 감싸여진 `dispatch()` 함수를 가지고 스토어의 복사본을 반환하는 `applyMiddleware()`를 작성할 수 있습니다:

```js
// 주의: 적당히 구현함!
// Redux API가 **아님**.

function applyMiddleware(store, middlewares) {
  middlewares = middlewares.slice();
  middlewares.reverse();

  let dispatch = store.dispatch;
  middlewares.forEach(middleware =>
    dispatch = middleware(store)(dispatch)
  );

  return Object.assign({}, store, { dispatch });
}
```

Redux에 포함되어 나오는 [`applyMiddleware()`](../api/applyMiddleware.md) 구현은 비슷하지만 **세 가지 중요한 면에서 다릅니다**:

* [store API](../api/Store.md)의 일부만을 미들웨어에 노출합니다: [`dispatch(action)`](../api/Store.md#dispatch) 와 [`getState()`](../api/Store.md#getState).

* 여러분이 미들웨어 안에서 `next(action)`대신 `store.dispatch(action)`를 호출할 경우 액션이 현재 미들웨어를 포함한 전체 미들웨어 체인을 다시 따라가도록 꼼수를 써뒀습니다. 이건 [나중에](AsyncActions.md) 볼 비동기 미들웨어에서 유용합니다.

* 여러분이 미들웨어를 한번만 적용했는지 확인하기 위해, `store` 자체보다는 `createStore()`상에서 작동합니다. 그래서 용법은 `(store, middlewares) => store` 대신 `(...middlewares) => (createStore) => createStore`입니다.

### 최종적인 접근

우리가 방금 작성한 미들웨어는 아래와 같습니다:

```js
const logger = store => next => action => {
  console.log('dispatching', action);
  let result = next(action);
  console.log('next state', store.getState());
  return result;
};

const crashReporter = store => next => action => {
  try {
    return next(action);
  } catch (err) {
    console.error('Caught an exception!', err);
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    });
    throw err;
  }
}
```

이것을 Redux 스토어에 이렇게 적용합니다:

```js
import { createStore, combineReducers, applyMiddleware } from 'redux';

// applyMiddleware takes createStore() and returns
// a function with a compatible API.
let createStoreWithMiddleware = applyMiddleware(logger, crashReporter)(createStore);

// Use it like you would use createStore()
let todoApp = combineReducers(reducers);
let store = createStoreWithMiddleware(todoApp);
```

됐습니다! 이제 스토어 인스턴스로 전달되는 모든 액션은 `logger`와 `crashReporter`를 지납니다:

```js
// Will flow through both logger and crashReporter middleware!
store.dispatch(addTodo('Use Redux'));
```

## 일곱가지 예시

여러분이 위의 섹션을 읽으면서 머리가 터질 것 같았다면, 우리가 작성하려 했던 것이 무엇인지 떠올려보세요. 이 섹션이 여러분과 저를 쉬게 하는 동시에 여러분이 더 잘 이해하게 도울겁니다.

아래의 각각의 함수는 유효한 Redux 미들웨어입니다. 전부 똑같이 유용하진 않지만, 다들 재미있을겁니다.

```js
/**
 * 모든 액션과 전달된 후의 상태를 로깅합니다.
 */
const logger = store => next => action => {
  console.group(action.type);
  console.info('dispatching', action);
  let result = next(action);
  console.log('next state', store.getState());
  console.groupEnd(action.type);
  return result;
};

/**
 * 상태가 변경되고 리스너가 알림을 받을때마다 충돌 보고를 보냅니다.
 */
const crashReporter = store => next => action => {
  try {
    return next(action);
  } catch (err) {
    console.error('Caught an exception!', err);
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    });
    throw err;
  }
}

/**
 * 액션을 { meta: { delay: N } }에 따라 N 밀리초만큼 지연시킵니다.
 * 이 경우 `dispatch`가 함수를 반환해서 취소할 수 있게 합니다.
 */
const timeoutScheduler = store => next => action => {
  if (!action.meta || !action.meta.delay) {
    return next(action);
  }

  let intervalId = setTimeout(
    () => next(action),
    action.meta.delay
  );

  return function cancel() {
    clearInterval(intervalId);
  };
};

/**
 * 액션을 { meta: { raf: true } }일 경우 한 rAF 프레임만큼 지연시킵니다.
 * 이 경우 `dispatch`가 함수를 반환해서 취소할 수 있게 합니다.
 */
const rafScheduler = store => next => {
  let queuedActions = [];
  let frame = null;

  function loop() {
    frame = null;
    try {
      if (queuedActions.length) {
        next(queuedActions.shift());
      }
    } finally {
      maybeRaf();
    }
  }

  function maybeRaf() {
    if (queuedActions.length && !frame) {
      frame = requestAnimationFrame(loop);
    }
  }

  return action => {
    if (!action.meta || !action.meta.raf) {
      return next(action);
    }

    queuedActions.push(action);
    maybeRaf();

    return function cancel() {
      queuedActions = queuedActions.filter(a => a !== action)
    };
  };
};

/**
 * 액션에 더해 약속(promise)를 보낼 수 있게 합니다.
 * 약속이 해결되면, 그 결과가 액션으로써 보내집니다.
 * 약속은 `dispatch`에서 반환되므로 호출자가 거부를 처리할 수 있습니다.
 */
const vanillaPromise = store => next => action => {
  if (typeof action.then !== 'function') {
    return next(action);
  }

  return Promise.resolve(action).then(store.dispatch);
};

/**
 * { promise } 필드를 통해 특별한 액션들을 보낼 수 있게 합니다.
 *
 * 이 미들웨어는 처음에 액션들을 하나의 액션으로 바꾸고,
 * `promise`가 해결되면 하나의 성공(또는 실패) 액션을 보냅니다.
 *
 * 편의를 위해, `dispatch`는 호출자가 기다릴 수 있게 약속을 반환합니다.
 */
const readyStatePromise = store => next => action => {
  if (!action.promise) {
    return next(action)
  }

  function makeAction(ready, data) {
    let newAction = Object.assign({}, action, { ready }, data);
    delete newAction.promise;
    return newAction;
  }

  next(makeAction(false));
  return action.promise.then(
    result => next(makeAction(true, { result })),
    error => next(makeAction(true, { error }))
  );
};

/**
 * 액션 대신 함수를 보낼 수 있게 합니다.
 * 이 함수는 `dispatch`와 `getState`를 인수로 받습니다.
 *
 * (`getState()`의 조건에 따른) 이른 종료나 
 * 비동기 흐름 제어에 유용합니다(다른것들을 `dispatch()`할 수 있습니다).
 *
 * `dispatch`는 보내진 함수의 반환값을 반환합니다.
 */
const thunk = store => next => action =>
  typeof action === 'function' ?
    action(store.dispatch, store.getState) :
    next(action);


// 이들 전부를 함께 사용할 수 있습니다!(그래야 한다는 뜻은 아닙니다.)
let createStoreWithMiddleware = applyMiddleware(
  rafScheduler,
  timeoutScheduler,
  thunk,
  vanillaPromise,
  readyStatePromise,
  logger,
  crashReporter
)(createStore);
let todoApp = combineReducers(reducers);
let store = createStoreWithMiddleware(todoApp);
```
