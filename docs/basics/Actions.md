# 액션

먼저 액션을 정의해봅시다.

**액션**은 애플리케이션에서 스토어로 보내는 데이터 묶음입니다. 이들이 스토어의 **유일한** 정보원이 됩니다. 여러분은 [`store.dispatch()`](../api/Store.md#dispatch)를 통해 이들을 보낼 수 있습니다.

이것이 새 할일의 추가를 나타내는 액션의 예시입니다.

```js
{
  type: 'ADD_TODO',
  text: 'Build my first Redux app'
}
```

액션은 평범한 자바스크립트 객체입니다. 편의를 위해, 액션은 어떤 형태의 액션이 실행될지 나타내는 문자열형의 `type` 필드를 가져야 합니다. 타입은 일반적으로 문자열 상수로 정의됩니다. 여러분의 앱이 충분히 커지면 타입들을 별도의 모듈로 분리할수도 있습니다.

```js
import { ADD_TODO, REMOVE_TODO } from '../actionTypes';
```

>##### 보일러플레이트에 대한 설명

>액션 타입 상수를 반드시 별도의 파일에 정의할 필요는 없으며, 심지어 정의하지 않아도 됩니다. 작은 프로젝트에서는 액션 타입으로 그냥 문자열을 쓰는게 쉬울겁니다. 하지만 코드베이스가 커지면 상수를 정의해서 얻을 수 있는 장점이 있습니다. 코드베이스를 깨끗하게 유지하기 위한 실용적인 팁들을 [보일러플레이트 줄이기](../recipes/ReducingBoilerplate.md)에서 더 읽을 수 있습니다.

`type`외에 액션 객체의 구조는 여러분 마음대로입니다. 혹시 관심이 있다면 [Flux Standard Action](https://github.com/acdlite/flux-standard-action)에서 액션을 어떻게 구성할지에 대한 권장사항을 알아보세요.

사용자가 할일을 완료했다고 체크하는 액션 하나를 더 추가합시다. 할일은 배열 안에 저장되기 때문에 우리는 특정한 할일을 `index`를 통해 참조할 수 있습니다. 진짜 앱에서는 새 할일이 만들어질때마다 유일한 ID를 부여하는게 더 좋겠죠.

```js
{
  type: COMPLETE_TODO,
  index: 5
}
```

각 액션에는 가능한 적은 데이터를 전달하는 것이 좋습니다. 예를 들어, 할일 객체 전체를 전달하는 것 보다는 `index`를 전달하는 것이 낫습니다.

마지막으로, 지금 보이는 할일들을 바꾸는 액션을 추가하겠습니다.

```js
{
  type: SET_VISIBILITY_FILTER,
  filter: SHOW_COMPLETED
}
```

## 액션 생산자

**액션 생산자**는 액션을 만드는 함수입니다. "액션"과 "액션 생산자"는 혼용하기 쉬운 용어이니 적절하게 사용하도록 신경써야 합니다.

[전통적인 Flux](http://facebook.github.io/flux) 구현에서 액션 생산자는 보통 불러와졌을때 디스패치를 작동시킵니다. 이렇게요:

```js
function addTodoWithDispatch(text) {
  const action = {
    type: ADD_TODO,
    text
  };
  dispatch(action);
}
```

이와는 대비되게 Redux의 액션 생산자는 사이드 이펙트가 전혀 없는 순수 함수입니다. 이들은 단지 액션을 반환합니다:

```js
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  };
}
```

이는 액션 생산자를 더 이식하기 좋고 테스트하기 쉽게 합니다. 실제로 디스패치를 시작하려면 결과값을 `dispatch()` 함수에 넘깁니다:

```js
dispatch(addTodo(text));
dispatch(completeTodo(index));
```

아니면 자동으로 디스패치를 해주는 **바인드된 액션 생산자**를 만듭니다:

```js
const boundAddTodo = (text) => dispatch(addTodo(text));
const boundCompleteTodo = (index) => dispatch(completeTodo(index));
```

이들은 바로 호출할 수 있습니다:

```
boundAddTodo(text);
boundCompleteTodo(index);
```

`dispatch()` 함수를 스토어에서 [`store.dispatch()`](../api/Store.md#dispatch)로 바로 접근할 수 있지만, 여러분은 보통 [react-redux](http://github.com/gaearon/react-redux)의 `connect()`와 같은 헬퍼를 통해 접근할 것입니다. 여러 액션 생산자를 `dispatch()`에 바인드하기 위해 [`bindActionCreators()`](../api/bindActionCreators.md)를 사용할수도 있습니다.

## 소스코드

### `actions.js`

```js
/*
 * 액션 타입
 */

export const ADD_TODO = 'ADD_TODO';
export const COMPLETE_TODO = 'COMPLETE_TODO';
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER';

/*
 * 다른 상수들
 */

export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE'
};

/*
 * 액션 생산자
 */

export function addTodo(text) {
  return { type: ADD_TODO, text };
}

export function completeTodo(index) {
  return { type: COMPLETE_TODO, index };
}

export function setVisibilityFilter(filter) {
  return { type: SET_VISIBILITY_FILTER, filter };
}
```

## 다음 단계

이제 이 액션들을 디스패치했을때 상태가 어떻게 변하는지 명시하기 위해 [리듀서를 정의](Reducers.md) 해봅시다! 

>##### 숙련된 사용자들을 위한 한마디
>여러분이 기본적인 컨셉에 익숙하고 이 튜토리얼을 이미 마치셨다면, [심화 튜토리얼](../advanced/README.md)의 [비동기 액션](../advanced/AsyncActions.md)에서 어떻게 AJAX 응답을 다루고 비동기 흐름에 액션 생산자를 통합하는지 알아보세요.
