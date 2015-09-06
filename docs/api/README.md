# API 레퍼런스

Redux API에서 드러난 부분은 매우 작습니다. Redux는 여러분이 구현해야 하는 몇 가지 계약([리듀서](../Glossary.md#reducer)와 같은)을 정의해두고 계약들을 서로 연결하기 위한 몇 가지 헬퍼 함수들을 제공합니다.

이 섹션은 전체 Redux API 문서입니다. Redux는 상태를 관리하는데에만 관여한다는 사실을 기억해두세요. 실제 앱에서는 [react-redux](https://github.com/gaearon/react-redux)와 같은 UI 바인딩이 필요할겁니다.

### 최상위 익스포트

* [createStore(reducer, [initialState])](createStore.md)
* [combineReducers(reducers)](combineReducers.md)
* [applyMiddleware(...middlewares)](applyMiddleware.md)
* [bindActionCreators(actionCreators, dispatch)](bindActionCreators.md)
* [compose(...functions)](compose.md)

### 스토어 API

* [Store](Store.md)
  * [getState()](Store.md#getState)
  * [dispatch(action)](Store.md#dispatch)
  * [subscribe(listener)](Store.md#subscribe)
  * [replaceReducer(nextReducer)](Store.md#replaceReducer)

### 임포트

위에 기술된 모든 함수들은 최상위 익스포트에 해당합니다. 임포트하려면:

#### ES6

```js
import { createStore } from 'redux';
```

#### ES5 (CommonJS)

```js
var createStore = require('redux').createStore;
```

#### ES5 (UMD build)

```js
var createStore = Redux.createStore;
```
