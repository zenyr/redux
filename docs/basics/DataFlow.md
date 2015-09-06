# 데이터 흐름

Redux의 아키텍쳐는 **엄격한 일방향 데이터 흐름**을 따라 전개됩니다.

이는 애플리케이션 내의 모든 데이터가 같은 생명주기 패턴을 따르며, 앱의 로직을 좀 더 예측가능하게 하고 이해하기 쉽게 만든다는 뜻입니다. 이는 또한 데이터 정규화를 도와서 같은 데이터의 복제본들이 서로를 모르는 여럿으로 나눠지고 말지 않도록 해줍니다.

여러분이 아직도 이에 익숙치 않으시다면, [동기](../introduction/Motivation.md)와 [The Case for Flux](https://medium.com/@dan_abramov/the-case-for-flux-379b7d1982c6)의 일방향 데이터 흐름에 대한 설득당할수밖에 없는 논거를 읽어보시기 바랍니다. [Redux가 정확히 Flux는 아니지만](../introduction/PriorArt.md) 같은 중요한 잇점들을 함께합니다.

모든 Redux 앱에서의 데이터는 아래와 같이 4단계의 생명주기를 따릅니다:

1. **여러분이 [`store.dispatch(action)`](../api/Store.md#dispatch)를 호출합니다**.

  액션은 **무엇이 일어날지** 기술하는 보통의 오브젝트입니다. 예를 들어:
  
    ```js
    { type: 'LIKE_ARTICLE', articleId: 42 };
    { type: 'FETCH_USER_SUCCESS', response: { id: 3, name: 'Megan' } };
    { type: 'ADD_TODO', text: 'Read the Redux docs.'};
    ```

  액션을 간단한 소식들의 단편이라고 생각하세요. "Mary가 42번 기사를 좋아합니다."나 "'Redux 문서를 읽는다.'가 할 일 목록에 추가되었습니다."
  
  여러분은 [`store.dispatch(action)`](../api/Store.md#dispatch)를 앱 내의 어디서나 호출할 수 있습니다. 컴포넌트나 XHR 콜백, 심지어 일정한 간격으로요.

2. **Redux 스토어가 여러분이 지정한 리듀서 함수들을 호출합니다.**

  스토어는 리듀서에 현재의 상태 트리와 액션의 두 가지 인수를 넘깁니다. 예를 들어 할일 앱에서, 루트 리듀서는 아래와 비슷한 인수들을 받을겁니다:
  
    ```js
    // 애플리케이션의 현재 상태(할일 목록과 선택된 필터)
    let previousState = {
      visibleTodoFilter: 'SHOW_ALL',
      todos: [{
        text: 'Read the docs.',
        complete: false
      }]
    };

    // 실행되는 액션(할일 추가)
    let action = {
      type: 'ADD_TODO',
      text: 'Understand the flow.'
    };

    // 리듀서가 다음 상태를 반환함
    let nextState = todoApp(previousState, action);
    ```

    리듀서는 단지 다음 상태를 **계산**하는 순수 함수라는 점을 기억하세요. 리듀서는 완전히 예측 가능해야 합니다: 같은 입력을 가지고 몇번을 호출하든지 같은 출력이 나와야 합니다. API 호출이나 라우터 전환같은 사이드이펙트를 일으켜서는 안됩니다. 이런 일들은 액션이 전달되기 전에 행해져야 합니다.

3. **루트 리듀서가 각 리듀서의 출력을 합쳐서 하나의 상태 트리로 만듭니다.**

  루트 리듀서를 어떻게 구성하는지는 완전히 여러분에게 달렸습니다. Redux는 루트 리듀서를 각각이 상태 트리의 가지 하나씩을 다루는 함수들로 나눌 수 있도록 [`combineReducers()`](../api/combineReducers.md) 헬퍼 함수를 제공합니다.
  
  [`combineReducers()`](../api/combineReducers.md)의 작동 방식은 아래와 같습니다. 여러분이 두 개의 리듀서를 가지고 있다고 합시다. 하나는 할일 목록을 위한 것이고, 하나는 선택된 필터 설정을 위한 것입니다:
  
    ```js
    function todos(state = [], action) {
      // Somehow calculate it...
      return nextState;
    }

    function visibleTodoFilter(state = 'SHOW_ALL', action) {
      // Somehow calculate it...
      return nextState;
    }

    let todoApp = combineReducers({
      todos,
      visibleTodoFilter
    });
    ```

  여러분이 액션을 보내면, `combineReducers`가 반환한 `todoApp`은 두 리듀서를 모두 호출합니다:
  
    ```js
    let nextTodos = todos(state.todos, action);
    let nextVisibleTodoFilter = visibleTodoFilter(state.visibleTodoFilter, action);
    ```

  그리고 두 결과를 합쳐서 하나의 상태 트리로 만듭니다:

    ```js
    return {
      todos: nextTodos,
      visibleTodoFilter: nextVisibleTodoFilter
    };
    ```
  [`combineReducers()`](../api/combineReducers.md)가 편리한 헬퍼 유틸리티이긴 하지만, 반드시 써야 하는건 아닙니다; 원하신다면 루트 리듀서를 직접 작성하세요!
  
4. **Redux 스토어가 루트 리듀서에 의해 반환된 상태 트리를 저장합니다.**

  이 새 트리가 여러분의 앱의 다음 상태입니다! [`store.subscribe(listener)`](../api/Store.md#subscribe)를 통해 등록된 모든 리스너가 불러내지고 이들은 현재 상태를 얻기 위해 [`store.getState()`](../api/Store.md#getState)를 호출할겁니다.
  
  이제 새로운 상태를 반영하여 UI가 변경될겁니다. 여러분이 [React Redux](https://github.com/gaearon/react-redux)으로 바인딩을 했다면, 이 시점에 component.setState(newState)가 호출됩니다.
  
## 다음 단계

이제 Redux가 어떻게 작동하는지 알았으니, [React 앱에 연결](UsageWithReact.md) 해봅시다.
Now that you know how Redux works, let’s [connect it to a React app](UsageWithReact.md).

>##### 숙련된 사용자들을 위한 한마디
>여러분이 기본적인 컨셉에 익숙하고 이 튜토리얼을 이미 마치셨다면, [심화 튜토리얼](../advanced/README.md)의 [비동기 흐름](../advanced/AsyncFlow.md)에서 미들웨어가 어떻게 [비동기 액션](../advanced/AsyncActions.md)이 리듀서에 도착하기 전에 이를 변환하는지를 알아보세요.
