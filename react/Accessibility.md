# [Accessibility](https://reactjs.org/docs/accessibility.html) (WIP)

## 왜 Accessibility 인가?
- [`ally`](https://en.wiktionary.org/wiki/a11y) 라고도 불리는 웹 접근성(web accessibility)은 모두가 사용할 수 있는 웹을 위한 웹 디자인이다.

- 접근성은 웹 페이지를 해석하는데 필요한 도움이 되는 기술이다.
- React는 접근성이 높은 페이지를 만들기 위해서 전적으로 지원하고, 종종 기본 HTML 테크닉을 사용하기도 한다.

## 기준과 가이드라인

### WCAG
- [Web Content Accessibility Guideline](https://www.w3.org/WAI/standards-guidelines/wcag/)은 접근성 높은 웬 사이트를 만들기 위한 가이드 라인을 제공한다.

- 아래는 WCAG checklist 개요이다.
  - [WCAG checklist from Wuhcag](https://www.wuhcag.com/wcag-checklist/)
  - [WCAG checklist from WebAIM](http://webaim.org/standards/wcag/checklist)
  - [Checklist from The A11Y Project](https://a11yproject.com/checklist.html)

### WAI-ARIA
- [웹 접근성 초기화 계획인 - Accessible Rich Internet Applications](https://www.w3.org/WAI/intro/aria) 도큐먼트는
  웹 접근성이 높은 JavaScript 위젯을 만들기 위한 테크닉들을 가지고 있다.

- JSX에서 모든 `aria-*` HTML attribute들이 지원된다.
- React의 대부분 DOM 속성과 attribute들은 camelCase인데,
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

## Semantic HTML
- Semantic HTML은 웹 어플리케이션 접근성의 토대이다.

- 다양한 HTML element를 사용해서 웹 사이트의 정보를 강화하면, 가끔 접근성을 무료로 얻을 수 있다.

  - [MDN HTML elements reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)

- 우리는 `<div>` 엘리먼트를 추가해서 HTML semantic을 분리하기도 하고,
  리스트를 위해서는 `<ol>`, `<ul>`, `<dl>` 또는 `<table>`을 사용한다.

- 이런 경우, 많은 엘리먼트를 묶기 위해서 [React Fragment](https://reactjs.org/docs/fragments.html)를 사용하자.

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

- 만약 마땅히 쓸 다른 element가 없다면, 아이템 collection을 fragment의 array로 구성해도 된다.
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
- 만약 Fragment가 아무런 props를 받지 않는다면, [short syntax](https://reactjs.org/docs/fragments.html#short-syntax)를 사용할 수도 있다.

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

- 더 알고 싶다면 [Fragment 도큐멘트](https://reactjs.org/docs/fragments.html)를 보자!

## 접근 가능한 form
### 레이블링
- `<input>` 또는 `<textarea>` HTML form을 컨트롤 하기 위해선, 레이블이 필요하다.
- 스크린 리더에게 노출 될 수 있는 묘사가 가능한 레이블을 제공해줘야한다.

- 아래의 리소스가 이걸 가능하게 해준다:
  - [W3C의 엘리먼트 라벨링 방법](https://www.w3.org/WAI/tutorials/forms/labels/)
  - [WebAIM의 엘리먼트 라벨링 방법](https://webaim.org/techniques/forms/controls)
  - [Paciello Group의 접근성이 높은 이름들에 대한 설명](https://www.paciellogroup.com/blog/2017/04/what-is-an-accessible-name/)

- 비록 이 표준 HTML 관행들이 React에서 대부분 바로 사용될 수는 있지만,
  `for` attribute는 JSX에서 `htmlFor`로 사용된다는 것은 알아두자.
```jsx
<label htmlFor="namedInput">Name:</label>
<input id="namedInput" type="text" name="name"/>
```

## Notifying the user of errors
- 모든 유저들이 에러 발생 시 이 에러를 이해해야한다.

- 아래의 링크가 어떻게 에러 텍스트를 노출 시키고, 스크린 리더가 읽을 수 있는지를 가이드 해준다.
  - [W3C 사용자 알람 설명](https://www.w3.org/WAI/tutorials/forms/notifications/)
  - [WebAIM의 form validation](https://webaim.org/techniques/formvalidation/)

## Focus Control
- 웹 어플리케이션은 키보드로만으로도 동작해야 한다는 것을 명심해야한다.

  - [WebAIM 이 이야기하는 keyboard accessibility](https://webaim.org/techniques/keyboard/)
### 키보드 focus와 focus 아웃라인
- 키보드 포커스는 keyboard 입력을 받아드리기 위해서 선택된 element를 나타낸다.
- 파랑색 키보드 포커스 아웃라인이 선택된 링크 주변에서 나타난다.
- CSS를 사용해서만 아웃라인을 없앨 수 있다.

- 만약 `outline: 0`을 세팅하면, 다른 아웃라인으로 구현할 수 있다.


