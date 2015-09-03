# 리듀서

[액션](./Actions.md)은 **무언가 일어난다**는 사실을 기술하지만, 그 결과 애플리케이션의 상태가 어떻게 바뀌는지는 특정하지 않습니다. 이것은 리듀서가 할 일이죠.

## 상태 디자인하기

Redux에서 애플리케이션의 모든 상태는 하나의 객체에 저장됩니다. 어떤 코드건 작성하기 전에 형태를 생각해보는건 좋은 일이죠. 여러분의 앱의 상태를 객체로 만든다면 어떤 표현이 가장 단순할까요?

우리의 할일 앱을 위해 두 가지를 저장하고 싶습니다:

* 현재 선택된 필터;
* 할일의 실제 목록.

여러분은 종종 데이터 뿐만 아니라 UI 상태도 상태 트리에 저장해야 한다는걸 발견하실겁니다. 그래도 좋지만, 데이터를 UI 상태와 분리하도록 하세요.

```js
{
  visibilityFilter: 'SHOW_ALL',
  todos: [{
    text: 'Consider using Redux',
    completed: true,
  }, {
    text: 'Keep all state in a single tree',
    completed: false
  }]
}
```

>##### 관계에 대한 한마디

>더 복잡한 앱에서는 각기 다른 개체들이 서로를 참조하게 만들고 싶으실겁니다. 우리는 여러분이 앱의 상태를 가능한한 중첩되지 않도록 정규화할것을 권합니다. 모든 개체가 ID를 키로 가지고, ID를 통해 다른 개체나 목록을 참조하도록 하세요. 앱의 상태를 데이터베이스라고 생각하시면 됩니다. 이런 접근은 [normalizr's](https://github.com/gaearon/normalizr)의 문서에 자세히 나와있습니다. 예를 들어 상태 안에 `todosById: { id -> todo }`와 `todos: array<id>`처럼 구현하는 것이 실제 앱에서는 더 적절합니다. 하지만 예시에서는 단순하게 하겠습니다.

## 액션 다루기

상태 객체가 어떻게 생겼는지 정했으니 리듀서를 작성해봅시다. 리듀서는 이전 상태와 액션을 받아서 다음 상태를 반환하는 순수 함수입니다. 

```js
(previousState, action) => newState
```

여러분이 이 형태의 함수를 [`Array.prototype.reduce(reducer, ?initialValue)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)로 넘길 것이기 때문에 리듀서라고 부릅니다. 리듀서를 순수하게 유지하는것은 매우 중요합니다. 여러분이 **절대로** 리듀서 내에서 하지 말아야 할 것들은:

* 인수들을 변경하거나;
* API 호출이나 라우팅 전환같은 사이드이펙트를 일으키는 일들.

사이드이펙트를 어떻게 일으키는지는 [심화과정](../advanced/README.md)에서 확인하게 될겁니다. 지금은 리듀서가 반드시 순수해야 한다는 점만 기억해두세요. **인수가 주어지면, 다음 상태를 계산해서 반환하면 됩니다. 예기치 못한 일은 없어야 합니다. 사이드 이펙트도 없어야 합니다. API 호출도 안됩니다. 변경도 안됩니다. 계산만 가능합니다.**

이 정도로 해두고, 우리가 전에 정의했던 [actions](Actions.md)을 이해하도록 천천히 리듀서를 작성해봅시다.

초기 상태를 정하는데서 시작하겠습니다. Redux는 처음에 리듀서를 `undefined` 상태로 호출합니다. 그때가 초기 상태를 반환할 기회입니다:

```js
import { VisibilityFilters } from './actions';

const initialState = {
  visibilityFilter: VisibilityFilters.SHOW_ALL,
  todos: []
};

function todoApp(state, action) {
  if (typeof state === 'undefined') {
    return initialState;
  }

  // For now, don’t handle any actions
  // and just return the state given to us.
  return state;
}
```

더 간단하게 작성하는 방법은 [ES6 default arguments syntax](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/default_parameters)를 사용하는 것입니다:

```js
function todoApp(state = initialState, action) {
  // For now, don’t handle any actions
  // and just return the state given to us.
  return state;
}
```

이제 `SET_VISIBILITY_FILTER`를 처리합시다. 우리가 할 일은 상태에서 `visibilityFilter`를 바꾸는 것 뿐입니다. 간단히:

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return Object.assign({}, state, {
      visibilityFilter: action.filter
    });
  default:
    return state;
  }
}
```

짚고 넘어갈 점은:

1. **우리는 `state`를 변경하지 않았습니다.** [`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)을 통해 복사본을 만들었죠. `Object.assign(state, { visibilityFilter: action.filter })`이라고 써도 여전히 틀립니다: 첫번째 인수를 변경하게 되니까요. 여러분은 **반드시** 첫번째 인수로 빈 객체를 전달해야 합니다. ES7로 제안된 [object spread syntax](https://github.com/sebmarkbage/ecmascript-rest-spread)을 써서 `{ ...state, ...newState }`로 작성할수도 있습니다.

2. **`default` 케이스에 대해 이전의 `state`를 반환했습니다.** 알 수 없는 액션에 대해서는 이전의 `state`를 반환하는것이 중요합니다.

>##### `Object.assign`에 관하여

>[`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)은 ES6의 일부이지만, 대부분의 브라우저에서 구현되지 않았습니다. 폴리필을 사용하거나 [Babel 플러그인](https://github.com/babel-plugins/babel-plugin-object-assign)이나 [`_.assign()`](https://lodash.com/docs#assign)같이 다른 라이브러리의 헬퍼를 사용해야 합니다.

>##### `switch`와 보일러플레이트에 관하여

>`switch`문은 진짜 보일러플레이트가 **아닙니다**. Flux의 진짜 보일러플레이트는 개념적인 부분입니다: 변경사항을 보내야 하고, 디스패쳐에 스토어를 등록해야 하고, 스토어가 객체가 되어야 합니다(그리고 유니버셜 앱을 만들때 그 복잡성이 드러나죠). Redux는 이러한 문제들을 이벤트 이미터 대신 순수 리듀서를 사용함으로써 해결했습니다.

>많은 이들이 아직도 문서에 `switch`문을 사용하는가를 보고 프레임워크를 선택한다는건 불행한 일입니다. 만약 여러분이 `switch`를 싫어한다면 [“보일러플레이트 줄이기”](../recipes/ReducingBoilerplate.md#reducers)에 나온 것처럼 핸들러 맵을 받도록 직접 만든 `createReducer` 함수를 사용할 수 있습니다.

## 더 많은 액션 다루기

다룰 액션이 2개 더 있습니다! 우리의 리듀서가 `ADD_TODO`를 다룰 수 있도록 확장합시다.

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return Object.assign({}, state, {
      visibilityFilter: action.filter
    });
  case ADD_TODO:
    return Object.assign({}, state, {
      todos: [...state.todos, {
        text: action.text,
        completed: false
      }]
    });    
  default:
    return state;
  }
}
```

앞에서와 마찬가지로 `state`나 그 필드들을 직접 쓰는 대신 새 객체를 반환했습니다. 새 `todos`는 이전의 `todos`의 끝에 새로운 할일 하나를 붙인 것과 동일합니다. 새로운 할일은 액션의 데이터를 이용해서 만들어졌습니다.

마지막으로, `COMPLETE_TODO` 핸들러를 구현합시다.

```js
case COMPLETE_TODO:
  return Object.assign({}, state, {
    todos: [
      ...state.todos.slice(0, action.index),
      Object.assign({}, state.todos[action.index], {
        completed: true
      }),
      ...state.todos.slice(action.index + 1)
    ]
  });
```

변경 없이 배열의 특정 할일만 수정하고 싶기 때문에, 그 할일의 앞과 뒤를 잘라냈습니다. 만약 이런 코드를 자주 작성해야 한다면 [React.addons.update](https://facebook.github.io/react/docs/update.html), [updeep](https://github.com/substantial/updeep)같은 헬퍼나 [Immutable](http://facebook.github.io/immutable-js/) 같이 깊은 수정을 지원하는 라이브러리를 사용하는것이 좋습니다. `state`를 복사하기 전엔 그 안의 무엇에도 할당하지 말아야 한다는걸 기억하세요.

## 리듀서 쪼개기

우리의 코드가 여기까지 왔습니다. 좀 번잡스럽네요:

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return Object.assign({}, state, {
      visibilityFilter: action.filter
    });
  case ADD_TODO:
    return Object.assign({}, state, {
      todos: [...state.todos, {
        text: action.text,
        completed: false
      }]
    });
  case COMPLETE_TODO:
    return Object.assign({}, state, {
      todos: [
        ...state.todos.slice(0, action.index),
        Object.assign({}, state.todos[action.index], {
          completed: true
        }),
        ...state.todos.slice(action.index + 1)
      ]
    });
  default:
    return state;
  }
}
```

좀 더 이해하기 쉽게 만들 방법이 있을까요? `todos`와 `visibilityFilter`는 서로 완전히 독립적으로 수정되는것 같습니다. 간혹 상태의 필드들이 서로에게 의존하고 있어 더 고려할 사항이 있는 경우도 있지만, 이번엔 쉽게 `todos`의 수정을 별도의 함수로 분리할 수 있습니다:

```js
function todos(state = [], action) {
  switch (action.type) {
  case ADD_TODO:
    return [...state, {
      text: action.text,
      completed: false
    }];
  case COMPLETE_TODO:
    return [
      ...state.slice(0, action.index),
      Object.assign({}, state[action.index], {
        completed: true
      }),
      ...state.slice(action.index + 1)
    ];
  default:
    return state;
  }
}

function todoApp(state = initialState, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return Object.assign({}, state, {
      visibilityFilter: action.filter
    });
  case ADD_TODO:
  case COMPLETE_TODO:
    return Object.assign({}, state, {
      todos: todos(state.todos, action)
    });
  default:
    return state;
  }
}
```

`todos`가 `state`도 받지만 이건 그냥 배열이라는걸 잘 봐두세요! 이제 `todoApp`은 관리할 상태의 조각만을 넘기고, `todos`는 그 조각을 어떻게 수정할지 알고 있습니다. **이것을 *리듀서 조합*이라고 부르고, 이것이 Redux 앱을 만드는 기본 패턴이 됩니다.**

리듀서 조합에 대해 더 알아봅시다. `visibilityFilter`만을 관리하는 리듀서도 뽑아낼 수 있을까요? 이렇게 할 수 있습니다:

```js
function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return action.filter;
  default:
    return state;
  }
}
```

우리는 이제 메인 리듀서를 상태의 부분들을 관리하는 리듀서를 부르고 하나의 객체로 조합하는 함수로 재작성할 수 있습니다. 또한 이제 완전한 초기 상태도 필요 없습니다. 처음에 `undefined`가 주어지면 자식 리듀서들이 각각의 초기 상태를 반환하면 됩니다.

```js
function todos(state = [], action) {
  switch (action.type) {
  case ADD_TODO:
    return [...state, {
      text: action.text,
      completed: false
    }];
  case COMPLETE_TODO:
    return [
      ...state.slice(0, action.index),
      Object.assign({}, state[action.index], {
        completed: true
      }),
      ...state.slice(action.index + 1)
    ];
  default:
    return state;
  }
}

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return action.filter;
  default:
    return state;
  }
}

function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  };
}
```

**각각의 리듀서는 전체 상태에서 자신의 부분만을 관리합니다. 모든 리듀서의 `state` 인자는  서로 다르고, 자신이 관리하는 부분에 해당합니다.

벌써 그럴듯해 보이네요! 앱이 커지면 리듀서를 별도의 파일로 분리해서 완전히 독립적이고 다른 데이터 도메인을 관리하도록 할 수 있습니다.

마지막으로, Redux는 `todoApp`이 위에서 했던것과 동일한 보일러플레이트 로직을 지원하는 [`combineReducers()`](../api/combineReducers.md)라는 유틸리티를 제공합니다. 이를 이용하면 `todoApp`을 이렇게 재작성할 수 있습니다:

```js
import { combineReducers } from 'redux';

const todoApp = combineReducers({
  visibilityFilter,
  todos
});

export default todoApp;
```

이는 아래와 완전히 의미가 같은 코드입니다:

```js
export default function todoApp(state, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  };
}
```

이들에게 서로 다른 키를 주거나, 다른 함수를 호출할 수도 있습니다. 결합된 리듀서를 작성하는 이 두 방법은 완전히 의미가 같습니다:

```js
const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
});
```

```js
function reducer(state, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  };
}
```

[`combineReducers()`](../api/combineReducers.md)가 하는 일은 여러분의 리듀서들을 **키에 따라 선택해서 잘라낸 상태들**로 호출하고 그 결과를 다시 하나의 객체로 합쳐주는 함수를 만드는 것 뿐입니다. [딱히 마법같은건 아닙니다.](https://github.com/rackt/redux/issues/428#issuecomment-129223274)

>##### ES6을 이해하는 사용자를 위한 한마디

>`combineReducers`는 객체를 기대하기 때문에, 모든 최상위 리듀서들을 각기 다른 파일에 놓고 `export`한 다음 `import * as reducers`를 이용해 각각의 이름을 키로 가지는 객체를 얻을 수 있습니다:

>```js
>import { combineReducers } from 'redux';
>import * as reducers from './reducers';
>
>const todoApp = combineReducers(reducers);
>```
>
>`import *`은 아직은 새로운 문법이기 때문에 이 문서에서는 [혼동](https://github.com/rackt/redux/issues/428#issuecomment-129223274)을 막기 위해 더 이상 사용하지 않겠지만, 커뮤니티의 예시들에서 만날수도 있습니다.

## Source Code

#### `reducers.js`

```js
import { combineReducers } from 'redux';
import { ADD_TODO, COMPLETE_TODO, SET_VISIBILITY_FILTER, VisibilityFilters } from './actions';
const { SHOW_ALL } = VisibilityFilters;

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return action.filter;
  default:
    return state;
  }
}

function todos(state = [], action) {
  switch (action.type) {
  case ADD_TODO:
    return [...state, {
      text: action.text,
      completed: false
    }];
  case COMPLETE_TODO:
    return [
      ...state.slice(0, action.index),
      Object.assign({}, state[action.index], {
        completed: true
      }),
      ...state.slice(action.index + 1)
    ];
  default:
    return state;
  }
}

const todoApp = combineReducers({
  visibilityFilter,
  todos
});

export default todoApp;
```

## 다음 단계

다음으로는 상태를 보관하고 액션을 디스패치할때 리듀서를 호출해주는 [Redux 스토어를 만드는 법](Store.md)에 대해 알아보겠습니다.
