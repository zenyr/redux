# 스토어

앞의 부분에서 우리는 "무엇이 일어날지"를 나타내는 [액션](Action.md)과 이 액션에 따라 상태를 수정하는 [리듀서](Reducers.md)를 정의했습니다.

**스토어**는 이들을 함께 가져오는 객체입니다. 스토어는 아래와 같은 일들을 해야 합니다:

* 애플리케이션의 상태를 저장하고;
* [`getState()`](../api/Store.md#getState)를 통해 상태에 접근하게 하고;
* [`dispatch(action)`](../api/Store.md#dispatch)를 통해 상태를 수정할 수 있게 하고;
* [`subscribe(listener)`](../api/Store.md#subscribe)를 통해 리스너를 등록합니다.

Redux 애플리케이션에서 단 하나의 스토어만 가질 수 있음을 알아두는것이 중요합니다. 만약 데이터를 다루는 로직을 쪼개고 싶다면, 여러개의 스토어 대신 [리듀서 조합](Reducers.md#splitting-reducers)을 사용할 수 있습니다.

리듀서를 만들었다면 스토어를 만드는건 쉽습니다. [앞부분](Reducers.md)에서 우리는 [`combineReducers()`](../api/combineReducers.md)를 통해 여러 리듀서를 하나로 합쳤습니다. 우리는 이것을 가져와서 [`createStore()`](../api/createStore.md)에 넘길겁니다.

```js
import { createStore } from 'redux';
import todoApp from './reducers';

let store = createStore(todoApp);
```

[`createStore()`](../api/createStore.md)의 두번째 인수로 초기 상태를 지정해줄수도 있습니다. 이는 서버에서 실행중인 Redux 애플리케이션의 상태와 일치하도록 클라이언트의 상태를 채워줄때 유용합니다.

```js
let store = createStore(todoApp, window.STATE_FROM_SERVER);
```

## 액션을 보내기

스토어를 만들었으니, 우리 프로그램이 작동하는지 검증해봅시다! 아무 UI도 없지만 이미 우리는 수정하는 로직을 테스트할 수 있습니다.

```js
import { addTodo, completeTodo, setVisibilityFilter, VisibilityFilters } from './actions';

// 초기 상태를 기록합니다.
console.log(store.getState());

// 상태가 바뀔때마다 기록합니다.
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
);

// 액션들을 보냅니다.
store.dispatch(addTodo('Learn about actions'));
store.dispatch(addTodo('Learn about reducers'));
store.dispatch(addTodo('Learn about store'));
store.dispatch(completeTodo(0));
store.dispatch(completeTodo(1));
store.dispatch(setVisibilityFilter(VisibilityFilters.SHOW_COMPLETED));

// 상태 변경을 더 이상 받아보지 않습니다.
unsubscribe();
```

여러분은 스토어에 보관된 상태가 어떻게 바뀌는지 볼 수 있습니다:

<img src='http://i.imgur.com/zMMtoMz.png' width='70%'>

우리는 UI를 작성하기도 전에 앱이 어떻게 행동할지 정했습니다. 이 튜토리얼에서는 다루지 않겠지만, 이 시점에서 여러분은 리듀서와 액션 생산자들을 위한 테스트를 작성할 수 있습니다. 이들은 단지 함수이기 때문에 아무것도 모조(mock)할 필요가 없습니다. 이들을 호출하고, 반환하는 것들을 검증하세요.

## Source Code

#### `index.js`

```js
import { createStore } from 'redux';
import todoApp from './reducers';

let store = createStore(todoApp);
```

## 다음 단계

우리의 할일 앱을 위한 UI를 만들기 전에, [Redux 애플리케이션에서 데이터 흐름](DataFlow.md)을 보기 위해 잠시 옆길로 빠지겠습니다.
