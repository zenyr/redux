# [Redux](http://dobbit.github.io/redux)

Redux는 자바스크립트 앱을 위한 예측 가능한 상태 컨테이너입니다.

Redux는 여러분이 일관적으로 동작하고, 서로 다른 환경(서버, 클라이언트, 네이티브)에서 작동하고, 테스트하기 쉬운 앱을 작성하도록 도와줍니다. 여기에 더해서 [시간여행형 디버거와 결합된 실시간 코드 수정](https://github.com/gaearon/redux-devtools)과 같은 훌륭한 개발자 경험을 제공합니다.

여러분은 Redux를 [React](https://facebook.github.io/react/)나 다른 뷰 라이브러리와 함께 사용할 수 있습니다.
Redux는 매우 작고(2kB) 의존성도 없습니다.

[![build status](https://img.shields.io/travis/rackt/redux/master.svg?style=flat-square)](https://travis-ci.org/rackt/redux) 
[![npm version](https://img.shields.io/npm/v/redux.svg?style=flat-square)](https://www.npmjs.com/package/redux) 
[![npm downloads](https://img.shields.io/npm/dm/redux.svg?style=flat-square)](https://www.npmjs.com/package/redux)
[![redux channel on slack](https://img.shields.io/badge/slack-redux@reactiflux-61DAFB.svg?style=flat-square)](http://www.reactiflux.com)


### 추천사

>[“Love what you’re doing with Redux”](https://twitter.com/jingc/status/616608251463909376)  
>Jing Chen, creator of Flux

>[“I asked for comments on Redux in FB's internal JS discussion group, and it was universally praised. Really awesome work.”](https://twitter.com/fisherwebdev/status/616286955693682688)  
>Bill Fisher, creator of Flux

>[“It's cool that you are inventing a better Flux by not doing Flux at all.”](https://twitter.com/andrestaltz/status/616271392930201604)  
>André Staltz, creator of Cycle

### 개발자 경험

저는 Redux를 ["Hot Reloading with Time Travel"](https://www.youtube.com/watch?v=xsSnOQynTHs)이라는 React Europe 발표를 위해 작업하면서 만들었습니다. 제 목표는 최소한의 API를 가지면서도 완전히 예측 가능한 행동을 하는 상태 관리 라이브러리를 만들어서, 이를 통해 로깅, 핫 리로딩, 시간여행, 유니버셜 앱, 기록과 재생 등을 개발자의 노력 없이도 구현하는 것이었습니다.

### 영향을 받은 것들

Redux는 [Flux](https://facebook.github.io/flux)의 아이디어를 발전시키되, [Elm](https://github.com/evancz/elm-architecture-tutorial/)의 큐들을 가져옴으로써 복잡성은 줄였습니다.
이들을 써보셨는지와 상관 없이, Redux를 시작하는데는 몇 분이면 충분합니다.

### 설치

안정된 버전을 설치하시려면:

```
npm install --save redux
```

일반적으로 여러분은 [React 바인딩](http://github.com/gaearon/react-redux)과 [개발자 도구](http://github.com/gaearon/redux-devtools)도 필요하실겁니다.

```
npm install --save react-redux
npm install --save-dev redux-devtools
```

이는 여러분이 [CommonJS 모듈](http://webpack.github.io/docs/commonjs.html)을 다루기 위해 [Webpack](http://webpack.github.io)이나 [Browserify](http://browserify.org/)와 같은 모듈 번들러를 [npm](http://npmjs.com/) 패키지 매니저와 함께 사용하고 있다고 가정합니다.

만약 여러분이 아직 [npm](http://npmjs.com/)이나 최신 모듈 번들러를 사용하고 있지 않다면, 하나의 파일로 된 [UMD](https://github.com/umdjs/umd) 빌드를 통해 `Redux` 를 글로벌 객체로 사용하는 것을 선호하실수도 있습니다. 이 때는 [cdnjs](https://cdnjs.com/libraries/redux)에서 이미 빌드된 버전을 가져오시면 됩니다. Redux를 보완하는 라이브러리 대부분은 [npm](http://npmjs.com/)에서만 사용 가능하기 때문에, 우리는 제대로 만드는 애플리케이션에 대해서 이러한 접근을 **권장하지 않습니다**.

### The Gist

여러분의 앱의 상태 전부는 하나의 스토어(**store**)안에 있는 객체 트리에 저장됩니다. 
상태 트리를 변경하는 유일한 방법은 무엇이 일어날지 서술하는 객체인 액션(**action**)을 보내는 것 뿐입니다. 
액션이 상태 트리를 어떻게 변경할지 명시하기 위해 여러분은 리듀서(**reducers**)를 작성해야 합니다.

이게 다입니다!

```js
import { createStore } from 'redux';

/**
 * 이것이 (state, action) => state 형태의 순수 함수인 리듀서입니다.
 * 리듀서는 액션이 어떻게 상태를 다음 상태로 변경하는지 서술합니다.
 *
 * 상태의 모양은 당신 마음대로입니다: 기본형(primitive)일수도, 배열일수도, 객체일수도, 
 * 심지어 Immutable.js 자료구조일수도 있습니다. 오직 중요한 점은 상태 객체를 변경해서는 안되며,
 * 상태가 바뀐다면 새로운 객체를 반환해야 한다는 것입니다.
 *
 * 이 예시에서 우리는 `switch` 구문과 문자열을 썼지만, 여러분의 프로젝트에 맞게 
 * 다른 컨벤션을 따르셔도 좋습니다.
 */
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1;
  case 'DECREMENT':
    return state - 1;
  default:
    return state;
  }
}

// 앱의 상태를 보관하는 Redux 스토어를 만듭니다.
// API로는 { subscribe, dispatch, getState }가 있습니다.
let store = createStore(counter);

// 업데이트를 직접 구독하거나 뷰 레이어에 바인딩할수 있습니다.
store.subscribe(() =>
  console.log(store.getState())
);

// 내부 상태를 변경하는 유일한 방법은 액션을 보내는 것뿐입니다.
// 액션은 직렬화될수도, 로깅할수도, 저장할수도 있으며 나중에 재생할수도 있습니다.
store.dispatch({ type: 'INCREMENT' });
// 1
store.dispatch({ type: 'INCREMENT' });
// 2
store.dispatch({ type: 'DECREMENT' });
// 1
```

상태를 바로 변경하는 대신, **액션**이라 불리는 평범한 객체를 통해 일어날 변경을 명시합니다. 그리고 각각의 액션이 전체 애플리케이션의 상태를 어떻게 변경할지 결정하는 특별한 함수인 **리듀서**를 작성합니다.

만약 여러분이 Flux를 개발하다가 왔다면, 알아둬야 할 중요한 차이점이 있습니다. Redux는 Dispatcher가 없고 스토어 여러개를 지원하지도 않습니다. 대신 루트 리듀싱 함수 하나를 가지는 단 하나의 스토어가 있습니다. 당신의 앱이 커지면 스토어를 추가하는 대신 루트 리듀서를 쪼개서 상태 트리의 각기 다른 부분을 독립적으로 다루는 리듀서들을 만들면 됩니다. 마치 React 앱에는 하나의 루트 컴포넌트가 있고 이 루트 컴포넌트가 여러개의 작은 컴포넌트로 이루어진 것처럼요. 

이 아키텍쳐는 숫자 세는 앱 하나 만드는데에는 과도해 보일 수 있지만 이 패턴의 아름다움은 크고 복잡한 앱으로 확장하기 좋다는 점입니다. 이는 또한 액션이 일으키는 모든 변경을 추적함으로써 강력한 개발자 도구를 가능하게 합니다. 여러분은 액션을 재생하는 것만으로 사용자 세션을 기록하고 재생산할 수 있습니다.

### 문서

* [소개](http://dobbit.github.io/redux/docs/introduction/index.html)
* [기초](http://dobbit.github.io/redux/docs/basics/index.html)
* [심화](http://dobbit.github.io/redux/docs/advanced/index.html)
* [레시피](http://dobbit.github.io/redux/docs/recipes/index.html)
* [문제해결](http://dobbit.github.io/redux/docs/Troubleshooting.html)
* [용어사전](http://dobbit.github.io/redux/docs/Glossary.html)
* [API 레퍼런스](http://dobbit.github.io/redux/docs/api/index.html)

### 예시

* [Counter](http://dobbit.github.io/redux/docs/introduction/Examples.html#counter) ([source](https://github.com/rackt/redux/tree/master/examples/counter))
* [TodoMVC](http://dobbit.github.io/redux/docs/introduction/Examples.html#todomvc) ([source](https://github.com/rackt/redux/tree/master/examples/todomvc))
* [Async](http://dobbit.github.io/redux/docs/introduction/Examples.html#async) ([source](https://github.com/rackt/redux/tree/master/examples/async))
* [Real World](http://dobbit.github.io/redux/docs/introduction/Examples.html#real-world) ([source](https://github.com/rackt/redux/tree/master/examples/real-world))

만약 여러분이 NPM 생태계가 생소하고 프로젝트를 시작하는데 문제가 있거나 위의 코드를 어디에 붙여넣어야 할지 모르겠다면, Redux를 React와 Browserify와 함께 사용하는 [simplest-redux-example](https://github.com/jackielii/simplest-redux-example)을 참고하세요.

### 논의

[Reactiflux](http://reactiflux.com) Slack 커뮤니티의 **#redux** 채널에 참여하세요.

### 감사의 말

* [The Elm Architecture](https://github.com/evancz/elm-architecture-tutorial) for a great intro to modeling state updates with reducers;
* [Turning the database inside-out](http://blog.confluent.io/2015/03/04/turning-the-database-inside-out-with-apache-samza/) for blowing my mind;
* [Developing ClojureScript with Figwheel](http://www.youtube.com/watch?v=j-kj2qwJa_E) for convincing me that re-evaluation should “just work”;
* [Webpack](https://github.com/webpack/docs/wiki/hot-module-replacement-with-webpack) for Hot Module Replacement;
* [Flummox](https://github.com/acdlite/flummox) for teaching me to approach Flux without boilerplate or singletons;
* [disto](https://github.com/threepointone/disto) for a proof of concept of hot reloadable Stores;
* [NuclearJS](https://github.com/optimizely/nuclear-js) for proving this architecture can be performant;
* [Om](https://github.com/omcljs/om) for popularizing the idea of a single state atom;
* [Cycle](https://github.com/staltz/cycle) for showing how often a function is the best tool;
* [React](https://github.com/facebook/react) for the pragmatic innovation.

NPM 패키지명인 `redux`를 넘겨주신 [Jamie Paton](http://jdpaton.github.io)에게 특별한 감사의 말을 전합니다.

### 후원자

Redux 작업은 [커뮤니티에 의해 펀딩되었습니다](https://www.patreon.com/reactdx).  
이를 가능하게 했던 주요한 회사들을 소개합니다

* [Webflow](http://webflow.com/)
* [Chess iX](http://www.chess-ix.com/)

[전체 후원자 명단 보기.](PATRONS.md)

### 라이선스

MIT

### 번역

유상엽([@Dev_Bono](https://twitter.com/Dev_Bono))