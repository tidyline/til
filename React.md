# react

- Higher Order Components


## Higher-Order Components \(WIP\)

* [higher order components](https://reactjs.org/docs/higher-order-components.html)

> HOC는 컴포넌트 로직을 재 사용할 수 있는 개선된 기술이다.. React API는 아니지만, React의 본연 모습에서 나온 패턴이라 할 수 있다.

* 간단하게 higher order component는 컴포넌트를 받아서

  새로운 컴포넌트로 돌려주는 것이라 볼 수 있다.

  ```javascript
  const EnhancedComponent = higherOrderComponent(WrappedComponent);
  ```

* 보통 컴포넌트는 UI에 props을 전달하는 반면,

  hoc는 하나의 컴포넌트를 다른 컴포넌트로 변형시켜준다.

* HOC는 third-party 리액트 라이브러리에서 자주 사용된다.

  예를들어 Redux의 `connect`, Relay의 `createFragmentContainer` 가 있따리..

### [cross-cutting concern](https://en.wikipedia.org/wiki/Cross-cutting_concern) 을 없애기 위해... 우리 HOC를 써보자!

* 리액트에서 컴포넌트는 코드 재사용을 위한 가장 작은 단위이다.
* 하지만, 전통 컴포넌트와는 꼭 맞지않는 좀 다른 패턴을 발견 할 수도 있다.
* 예를 들어 외부 데이터를 구독하면서 `CommentList` 컴포넌트를 그린다고 해보자:

```jsx
class CommentList extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      // "DataSource" is some global data source
      comments: DataSource.getComments()
    };
  }

  componentDidMount() {
    // Subscribe to changes
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    // Clean up listener
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    // Update component state whenever the data source changes
    this.setState({
      comments: DataSource.getComments()
    });
  }

  render() {
    return (
      <div>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id} />
        ))}
      </div>
    );
  }
}
```

* 후에 하나의 블로그 포스트를 보여주는 컴포넌트를 작성한다고 해보자

```jsx
class BlogPost extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      blogPost: DataSource.getBlogPost(props.id)
    };
  }

  componentDidMount() {
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    this.setState({
      blogPost: DataSource.getBlogPost(this.props.id)
    });
  }

  render() {
    return <TextBlock text={this.state.blogPost} />;
  }
  }
```

* `CommentList`와 `BlogPost`는 같진않다.
  * `DataSource` 에서도 다른 메서드를 부르고
  * 다른 output을 기준으로 렌더한다.
* 하지만 구현 방식은 같다:
  * mount 할 때, `DataSource`의 change listener를 달고
  * listener 내에서, data가 변경될 때 마다 `setState`를 부르게 된다.
  * unmount 땐 change listener를 제거한다.
* 대형 앱을 상상해보면, `DataSource`를 구독하고, `setState`를 하는 구문이 계속 계속 반복될 것이다.
* 우리는 로직을 추상화해서 한 곳에 넣어놓고,

  많은 컴포넌트들이 공유하길 원한다.

* 이 것이 바로 HOC가 탁월한 부분이다.
* 우리는 컴포넌트를 생성하는 함수를 작성할 수 있고, 이 컴포넌트가 `DataSource`를 구독한다.
* 이 함수는 하나의 인자로 자식 컴포넌트를 받는데,

  이 컴포넌트에서 data를 prop으로 받게 된다.

* `withSubscription`을 불러보자:

```jsx
const CommentListWithSubscription = withSubscription( CommentList,  DataSource ) = DataSource.getComments\(\)
const BlogPostWithSubscription = withSubscription( BlogPost, (DataSource, props\) =&gt; DataSource.getBlogPost\(props.id\) \);
```


* 첫 번째 인자로 컴포넌트가 넘겨지고,
  두 번째 인자로 `DataSource`에서 받는 데이터와 현제 props를 넘겨준다.
* `CommentListWithSubscript`과 `BlogPostWithSubscription`이 그려지면,
  `CommentList`와 `BlogPost`는 `DataSource`로 부터 넘겨주는 `data`를 받을 것이다:

```jsx
// This function takes a component...
function withSubscription(WrappedComponent, selectData) {
  // ...and returns another component...
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      // ... that takes care of the subscription...
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // ... and renders the wrapped component with the fresh data!
      // Notice that we pass through any additional props
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

* HOC는 input 컴포넌트를 수정하지 않는다는 것을 알아두고,

  상속을 통해서 동작을 복사하지도 않는다.

* 대신 HOC는 기존 컴포넌트를 다른 컴포넌트로 wrapping 하는 일 만 한다.
* HOC는 pure function이기 때문에 사이드 이펙트가 없다.
* 이게 다이다!!
* HOC는 왜? 어떻게? 그 데이터가 사용되는지 관심없고, wrapped 컴포넌트는 어디서 데이터가 들어오는지 상관하지 않아도 된다.
* 왜냐하면 `withSubscription`이 일반 함수이기 때문에, 원하는 만큼의 인자를 설정 할 수 있다.
* 컴포넌트와 유사하게, `withSubscript`과 wrapped 컴포넌트는 전적으로 prop기반이다.
* 이것은 prop만 전달 된다면,

  HOC를 다른 것으로 바꾸기 매우 쉽게 한다.

* fetching 라이브러리를 바꾸거나.... 할 때 매우 유용할 것이다..

### 기존 컴포넌트를 변형하지마! Composition을 사용해

* HOC 내에서 component의 prototype을 수정하고 싶은 유혹이 있을 수도 있다..

```jsx

  function logProps\(InputComponent\) {

  InputComponent.prototype.componentWillReceiveProps = function\(nextProps\) {

    console.log\('Current props: ', this.props\);

    console.log\('Next props: ', nextProps\);

  };

  // The fact that we're returning the original input is a hint that it has

  // been mutated.

  return InputComponent;

  }

// EnhancedComponent will log whenever props are received const EnhancedComponent = logProps\(InputComponent\);

```


* 여기엔 몇가지 문제가 있다.
* 하나는 InputComponent는 EnhancedComponent와 독립적으로 재사용되지 않는다.
* 더 안좋은 것은, 다른 HOC를 `EnhancedComponent`에 적용하면,
  똑같이 `componentWillReceiveProps`를 변형하려고 할 것이다.
* 첫 번쨰 HOC의 기능이 override 될 것이다!
* 이 것은 HOC가 함수형 컴포넌트로 동작하지 않는다는 것이고,
  lifecyle 메서드가 없는 것과 마찬가지다...
<br>

* 수정 하는 것 대신, HOC에서는 composition을 사용해라..
* InputComponent를 container component로 감싸라..

```jsx
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log('Current props: ', this.props);
      console.log('Next props: ', nextProps);
    }
    render() {
      // Wraps the input component in a container, without mutating it. Good!
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

* 이 HOC는 변형하는 버전이랑 동일한 기능을 수행한다.
* 하지만.. 잠재적인 오류들을 피할 수 있겠지...
* 이건 pure function 이기 때문에, 다른 HOC와도 잘 연동 될 수 있따리..
* HOC와 **container components** 와 유사성을 알고 있을 수도 있다.
* 컨테이터 컴포넌트는 high-level과 low-level의 고려사항의 책임을

  분리할 수 있는 전략이다.

* 컨테이너 컴포넌트는 state 구독하거나,

   UI를 렌더링 하기 위한 prop이 통과되는 컴포넌트 같은 친구들을 관리한다.

----

## Accessibility \(WIP\)

### 왜 Accessibility 인가?

* [`ally`](https://en.wiktionary.org/wiki/a11y) 라고도 불리는 웹 접근성\(web accessibility\)은 모두가 사용할 수 있는 웹을 위한 웹 디자인이다.
* 접근성은 웹 페이지를 해석하는데 필요한 도움이 되는 기술이다.
* React는 접근성이 높은 페이지를 만들기 위해서 전적으로 지원하고, 종종 기본 HTML 테크닉을 사용하기도 한다.

### 기준과 가이드라인

#### WCAG

* [Web Content Accessibility Guideline](https://www.w3.org/WAI/standards-guidelines/wcag/)은 접근성 높은 웬 사이트를 만들기 위한 가이드 라인을 제공한다.
* 아래는 WCAG checklist 개요이다.
  * [WCAG checklist from Wuhcag](https://www.wuhcag.com/wcag-checklist/)
  * [WCAG checklist from WebAIM](http://webaim.org/standards/wcag/checklist)
  * [Checklist from The A11Y Project](https://a11yproject.com/checklist.html)

#### WAI-ARIA

* [웹 접근성 초기화 계획인 - Accessible Rich Internet Applications](https://www.w3.org/WAI/intro/aria) 도큐먼트는 웹 접근성이 높은 JavaScript 위젯을 만들기 위한 테크닉들을 가지고 있다.
* JSX에서 모든 `aria-*` HTML attribute들이 지원된다.
* React의 대부분 DOM 속성과 attribute들은 camelCase인데,

  이 attribute들은 반드시 hyphen-case여야 한다.

```jsx
<input
  type="text"
  aria-label={labelText}
  aria-required="true"
  onChange={onchangeHandler}
  value={inputValue}
  name="name"
/>
```

### Semantic HTML

* Semantic HTML은 웹 어플리케이션 접근성의 토대이다.
* 다양한 HTML element를 사용해서 웹 사이트의 정보를 강화하면, 가끔 접근성을 무료로 얻을 수 있다.
  * [MDN HTML elements reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)
* 우리는 `<div>` 엘리먼트를 추가해서 HTML semantic을 분리하기도 하고, 리스트를 위해서는 `<ol>`, `<ul>`, `<dl>` 또는 `<table>`을 사용한다.
* 이런 경우, 많은 엘리먼트를 묶기 위해서 [React Fragment](https://reactjs.org/docs/fragments.html)를 사용하자.

```jsx
import React, { Fragment } from 'react';

function ListItem({ item }) {
  return (
    <Fragment>
      <dt>{item.term}</dt>
      <dd>{item.description}</dd>
    </Fragment>
  );
}

function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        <ListItem item={item} key={item.id} />
      ))}
    </dl>
  );
}
```

* 만약 마땅히 쓸 다른 element가 없다면, 아이템 collection을 fragment의 array로 구성해도 된다.

  ```jsx
  function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // Fragments should also have a `key` prop when mapping collections
        <Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </Fragment>
      ))}
    </dl>
  );
  }
  ```

* 만약 Fragment가 아무런 props를 받지 않는다면, [short syntax](https://reactjs.org/docs/fragments.html#short-syntax)를 사용할 수도 있다.

```jsx
function ListItem({ item }) {
  return (
    <>
      <dt>{item.term}</dt>
      <dd>{item.description}</dd>
    </>
  );
}
```

* 더 알고 싶다면 [Fragment 도큐멘트](https://reactjs.org/docs/fragments.html)를 보자!

### 접근 가능한 form

#### 레이블링

* `<input>` 또는 `<textarea>` HTML form을 컨트롤 하기 위해선, 레이블이 필요하다.
* 스크린 리더에게 노출 될 수 있는 묘사가 가능한 레이블을 제공해줘야한다.
* 아래의 리소스가 이걸 가능하게 해준다:
  * [W3C의 엘리먼트 라벨링 방법](https://www.w3.org/WAI/tutorials/forms/labels/)
  * [WebAIM의 엘리먼트 라벨링 방법](https://webaim.org/techniques/forms/controls)
  * [Paciello Group의 접근성이 높은 이름들에 대한 설명](https://www.paciellogroup.com/blog/2017/04/what-is-an-accessible-name/)
* 비록 이 표준 HTML 관행들이 React에서 대부분 바로 사용될 수는 있지만, `for` attribute는 JSX에서 `htmlFor`로 사용된다는 것은 알아두자.

  ```jsx
  <label htmlFor="namedInput">Name:</label>
  <input id="namedInput" type="text" name="name"/>
  ```

### Notifying the user of errors

* 모든 유저들이 에러 발생 시 이 에러를 이해해야한다.
* 아래의 링크가 어떻게 에러 텍스트를 노출 시키고, 스크린 리더가 읽을 수 있는지를 가이드 해준다.
  * [W3C 사용자 알람 설명](https://www.w3.org/WAI/tutorials/forms/notifications/)
  * [WebAIM의 form validation](https://webaim.org/techniques/formvalidation/)

### Focus Control

* 웹 어플리케이션은 키보드로만으로도 동작해야 한다는 것을 명심해야한다.
  * [WebAIM 이 이야기하는 keyboard accessibility](https://webaim.org/techniques/keyboard/)

    **키보드 focus와 focus 아웃라인**
* 키보드 포커스는 keyboard 입력을 받아드리기 위해서 선택된 element를 나타낸다.
* 파랑색 키보드 포커스 아웃라인이 선택된 링크 주변에서 나타난다.
* CSS를 사용해서만 아웃라인을 없앨 수 있다.
* 만약 `outline: 0`을 세팅하면, 다른 아웃라인으로 구현할 수 있다.

