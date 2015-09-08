# React와 함께 사용하기

처음 시작할때부터 우리는 Redux가 React와는 관계가 없음을 강조했습니다. 여러분은 Redux 앱을 React, Angular, Ember, jQuery, 순수 JavaScript와 함께 만들 수 있습니다.

그렇긴 하지만 Redux는 액션에 반응하여 상태를 변경하기 때문에, [React](http://facebook.github.io/react/)나 [Deku](https://github.com/dekujs/deku)와 같이 UI를 상태에 대한 함수로 기술하는 프레임워크와 특히 잘 어울립니다.

우리의 간단한 할일 앱을 React로 만들어 보겠습니다.


## React Redux 설치하기

[React bindings](https://github.com/gaearon/react-redux)은 Redux에 기본적으로 포함되어있지는 않습니다. 여러분이 명시적으로 설치해줘야 합니다:

```
npm install --save react-redux
```

## 영민한(Smart) 컴포넌트와 우직한(Dumb) 컴포넌트

Redux용 React 바인딩은 [separating “smart” and “dumb” components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)의 아이디어를 채택했습니다.

앱의 최상위 컴포넌트(라우트 핸들러같은)만이 Redux와 연관되는 것이 좋습니다. 그 아래의 컴포넌트들은 우직해야 하고 모든 데이터를 props를 통해 전달받아야 합니다.

<table>
    <thead>
        <tr>
            <th></th>
            <th scope="col" style="text-align:left">영민한 컴포넌트</th>
            <th scope="col" style="text-align:left">우직한 컴포넌트</th>
        </tr>
    </thead>
    <tbody>
        <tr>
          <th scope="row" style="text-align:right">위치</th>
          <td>최상위, 라우트 핸들러</td>
          <td>중간과 말단 컴포넌트</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Redux와 연관됨</th>
          <td>예</th>
          <td>아니오</th>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">데이터를 읽기 위해</th>
          <td>Redux 상태를 구독</td>
          <td>props에서 데이터를 읽음</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">데이터를 바꾸기 위해</th>
          <td>Redux 액션을 보냄</td>
          <td>props에서 콜백을 부름</td>
        </tr>
    </tbody>
</table>

이 할일 앱에서는 뷰 계층의 최상단에 하나의 영민한 컴포넌트만을 두겠습니다. 더 복잡한 앱에서는 여러개를 둘 수도 있습니다. 영민한 컴포넌트를 중첩시킬수도 있지만, 가능하면 props만 내려주는 것을 권합니다.

## 컴포넌트 계층을 설계하기

우리가 어떻게 [루트 상태 객체의 형태를 설계](Reducers.md)했는지 기억하시나요? 이제 그에 맞게 UI 계층을 설계하겠습니다. 이는 Redux에만 한정된 일은 아닙니다. [Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html)는 이 과정을 설명하는 좋은 튜토리얼입니다.

우리의 설계를 요약하면 간단합니다. 우리는 할일 목록을 보여줄겁니다. 할일을 클릭하면 완료한 것으로 표시됩니다. 사용자가 할일을 추가할 필드도 보여줘야 합니다. 푸터에는 모든 할일을 보여주거나 / 완료된 할일만 보여주거나 / 완료되지 않은 할일만 보여주는 토글을 놓겠습니다.

이 요약에서 아래와 같은 컴포넌트(와 그 props)를 끌어낼 수 있습니다:


* **`AddTodo`**는 버튼이 달린 입력 필드입니다.
  - `onAddClick(text: string)`은 버튼을 누르면 불러올 콜백입니다.
* **`TodoList`**는 표시중인 할일 목록입니다.
  - `todos: Array`는 `{ text, completed }` 형태의 할일 배열입니다.
  - `onTodoClick(index: number)`은 할일을 누르면 호출할 콜백입니다.
* **`Todo`**는 할일 하나입니다.
  - `text: string`은 보여줄 텍스트입니다.
  - `completed: boolean`은 할일을 완료된것으로 표시할지 여부입니다.
  - `onClick()`은 할일을 누르면 호출할 콜백입니다.
* **`Footer`**는 표시할 할일 필터를 사용자가 바꿀 수 있는 컴포넌트입니다.
  - `filter: string`은 현재 필터입니다: `'SHOW_ALL'`, `'SHOW_COMPLETED'`, `'SHOW_ACTIVE'`이 있습니다.
  - `onFilterChange(nextFilter: string)`사용자가 다른 필터를 선택했을 때 호출할 콜백입니다.

이들 모두가 우직한 컴포넌트입니다. 이들은 **어디에서** 데이터가 오는지도 모르고, **어떻게** 바꾸는지도 모릅니다. 이들은 주어진대로 그려낼 뿐입니다.

여러분이 Redux에서 다른 무언가로 옮기더라도 이들 컴포넌트는 그대로 둘 수 있습니다. 이들은 Redux에 의존성이 없습니다.

이제 작성해봅시다! 아직은 Redux에 어떻게 바인드할지 생각할 필요가 없습니다. 이들이 제대로 그려내는지 실험하기 위해 가짜 데이터를 넣어봐도 됩니다.

## 우직한 컴포넌트

이들은 보통의 React 컴포넌트이므로 자세한 설명은 생략하겠습니다. 아래와 같습니다:

#### `components/AddTodo.js`

```js
import React, { findDOMNode, Component, PropTypes } from 'react';

export default class AddTodo extends Component {
  render() {
    return (
      <div>
        <input type='text' ref='input' />
        <button onClick={e => this.handleClick(e)}>
          Add
        </button>
      </div>
    );
  }

  handleClick(e) {
    const node = findDOMNode(this.refs.input);
    const text = node.value.trim();
    this.props.onAddClick(text);
    node.value = '';
  }
}

AddTodo.propTypes = {
  onAddClick: PropTypes.func.isRequired
};
```

#### `components/Todo.js`

```js
import React, { Component, PropTypes } from 'react';

export default class Todo extends Component {
  render() {
    return (
      <li
        onClick={this.props.onClick}
        style={{
          textDecoration: this.props.completed ? 'line-through' : 'none',
          cursor: this.props.completed ? 'default' : 'pointer'
        }}>
        {this.props.text}
      </li>
    );
  }
}

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  text: PropTypes.string.isRequired,
  completed: PropTypes.bool.isRequired
};
```

#### `components/TodoList.js`

```js
import React, { Component, PropTypes } from 'react';
import Todo from './Todo';

export default class TodoList extends Component {
  render() {
    return (
      <ul>
        {this.props.todos.map((todo, index) =>
          <Todo {...todo}
                key={index}
                onClick={() => this.props.onTodoClick(index)} />
        )}
      </ul>
    );
  }
}

TodoList.propTypes = {
  onTodoClick: PropTypes.func.isRequired,
  todos: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  }).isRequired).isRequired
};
```

#### `components/Footer.js`

```js
import React, { Component, PropTypes } from 'react';

export default class Footer extends Component {
  renderFilter(filter, name) {
    if (filter === this.props.filter) {
      return name;
    }

    return (
      <a href='#' onClick={e => {
        e.preventDefault();
        this.props.onFilterChange(filter);
      }}>
        {name}
      </a>
    );
  }

  render() {
    return (
      <p>
        Show:
        {' '}
        {this.renderFilter('SHOW_ALL', 'All')}
        {', '}
        {this.renderFilter('SHOW_COMPLETED', 'Completed')}
        {', '}
        {this.renderFilter('SHOW_ACTIVE', 'Active')}
        .
      </p>
    );
  }
}

Footer.propTypes = {
  onFilterChange: PropTypes.func.isRequired,
  filter: PropTypes.oneOf([
    'SHOW_ALL',
    'SHOW_COMPLETED',
    'SHOW_ACTIVE'
  ]).isRequired
};
```

됐습니다! 이들이 제대로 작동하는지 확인하기 위해 더미 `App`을 작성해 보겠습니다:

#### `containers/App.js`

```js
import React, { Component } from 'react';
import AddTodo from '../components/AddTodo';
import TodoList from '../components/TodoList';
import Footer from '../components/Footer';

export default class App extends Component {
  render() {
    return (
      <div>
        <AddTodo
          onAddClick={text =>
            console.log('add todo', text)
          } />
        <TodoList
          todos={[{
            text: 'Use Redux',
            completed: true
          }, {
            text: 'Learn to connect it to React',
            completed: false
          }]}
          onTodoClick={todo =>
            console.log('todo clicked', todo)
          } />
        <Footer
          filter='SHOW_ALL'
          onFilterChange={filter =>
            console.log('filter change', filter)
          } />
      </div>
    );
  }
}
```

`<App />`은 이렇게 표현됩니다:

<img src='http://i.imgur.com/lj4QTfD.png' width='40%'>

이 자체로는 별로 흥미로울게 없습니다. 이제 Redux와 연결해봅시다!

## Redux와 연결하기

우리의 `App` 컴포넌트를 Redux와 연결해서 액션을 보내고 상태를 읽기 위해 두 가지를 변경해야 합니다.

먼저, 아까 설치한 [`react-redux`](http://github.com/gaearon/react-redux)에서 `Provider`를 불러와서, **`<Provider>`로 루트 컴포넌트를 감싸줍시다**.

#### `index.js`

```js
import React from 'react';
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import App from './containers/App';
import todoApp from './reducers';

let store = createStore(todoApp);

let rootElement = document.getElementById('root');
React.render(
  // React 0.13의 이슈를 회피하기 위해 
  // 반드시 함수로 감싸줍니다.
  <Provider store={store}>
    {() => <App />}
  </Provider>,
  rootElement
);
```

이렇게 하면 안의 컴포넌트가 우리의 스토어 인스턴스를 사용할 수 있게 됩니다.(내부적으로 이는 React의 [문서화되지 않은 “context” 기능](http://www.youtube.com/watch?v=H7vlH-wntD4))에 의해 가능하지만 API에 직접적으로 노출되지 않으니 걱정 않으셔도 됩니다.)

다음으로 **Redux와 연결하고 싶은 컴포넌트를 [`react-redux`](http://github.com/gaearon/react-redux)의 `connect()` 함수로 감싸줍시다**. 가능한 최상위 컴포넌트나 라우트 핸들러만 이렇게 해주세요. 기술적으로는 어떤 컴포넌트든지 Redux 스토어에 `connect()` 할 수 있지만, 너무 깊이 연결하면 데이터 흐름을 추적하기가 어려워집니다.

**`connect()` 호출로 감싸진 컴포넌트는 [`dispatch`](../api/Store.md#dispatch) 함수를 prop으로 받게 되고, 필요한 상태는 전역 상태에서 가져오면 됩니다.** `connect()`의 유일한 인수는 **selector**라고 부를 함수 하나뿐입니다. 이 함수는 전역 Redux 스토어의 상태를 받아서 컴포넌트가 필요로 하는 props를 반환합니다. 가장 간단하게는 받은 `state`를 그대로 반환할수도 있겠지만 아마도 상태를 반환하기 전에 변환하고 싶을겁니다.

조합 가능한 셀렉터를 이용해 변환을 메모이즈하고 싶다면 [reselect](https://github.com/faassen/reselect)를 알아보세요. 이 예제에서는 사용하지 않지만, 더 큰 앱에서는 잘 작동할겁니다.

#### `containers/App.js`

```js
import React, { Component, PropTypes } from 'react';
import { connect } from 'react-redux';
import { addTodo, completeTodo, setVisibilityFilter, VisibilityFilters } from '../actions';
import AddTodo from '../components/AddTodo';
import TodoList from '../components/TodoList';
import Footer from '../components/Footer';

class App extends Component {
  render() {
    // connect() 호출을 통해 주입됨:
    const { dispatch, visibleTodos, visibilityFilter } = this.props;
    return (
      <div>
        <AddTodo
          onAddClick={text =>
            dispatch(addTodo(text))
          } />
        <TodoList
          todos={visibleTodos}
          onTodoClick={index =>
            dispatch(completeTodo(index))
          } />
        <Footer
          filter={visibilityFilter}
          onFilterChange={nextFilter =>
            dispatch(setVisibilityFilter(nextFilter))
          } />
      </div>
    );
  }
}

App.propTypes = {
  visibleTodos: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  })),
  visibilityFilter: PropTypes.oneOf([
    'SHOW_ALL',
    'SHOW_COMPLETED',
    'SHOW_ACTIVE'
  ]).isRequired
};

function selectTodos(todos, filter) {
  switch (filter) {
  case VisibilityFilters.SHOW_ALL:
    return todos;
  case VisibilityFilters.SHOW_COMPLETED:
    return todos.filter(todo => todo.completed);
  case VisibilityFilters.SHOW_ACTIVE:
    return todos.filter(todo => !todo.completed);
  }
}

// 주어진 전역 상태에서 어떤 props를 주입하기를 원하나요?
// 노트: 더 나은 성능을 위해서는 https://github.com/faassen/reselect 를 사용하세요
function select(state) {
  return {
    visibleTodos: selectTodos(state.todos, state.visibilityFilter),
    visibilityFilter: state.visibilityFilter
  };
}

// 디스패치와 상태를 주입하려는 컴포넌트를 감싸줍니다.
export default connect(select)(App);
```

됐습니다! 할일 앱은 이제 제대로 동작합니다.

## 다음 단계

배운 지식을 더 잘 소화하려면 [이 튜토리얼의 전체 소스코드](ExampleTodoList.md)를 읽어보세요. 그런 다음 [심화 튜토리얼](../advanced/README.md)에서 네트워크 요청과 라우팅을 어떻게 처리하는지 배워봅시다!
