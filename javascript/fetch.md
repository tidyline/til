# `fetch()`

>`$.ajax`, `axios` 같은 XMLHttpRequest를 래핑한 라이브러리만 쓰다가..
> 바닐라스크립트로 개발하면서 XMLHttpRequest 쓰려고 여러 줄 쓰다보니
> 아!! fetch가 있었지!! 라는 생각이나서 한 번 보게 되었다!
> 지나가면서 본 정도라 이번 기회에 뭐가 다

## XMLHttpRequest
만약 웹 페이지에서 제목 하나를 바꾸어야 한다고 했을 때,
모든 페이지를 다시 불러와서 그려야 한다면 그건 얼마나 비효율 적일까?

그래서 `ajax`(Asynchronous Javascript And XML) 가 등장하게 되었다.
브라우저는 화면의 일부를 업데이트 하기 위해, 서버로 요청을 하고 응답이 올 때까지 다른 작업들을 수행한다.
그리고 서버에서 응답이 오면 브라우저가 화면을 업데이트하라는 콜백이 호출된다.
이 것이 비동기로 동작하는 ajax의 기능을 설명한 것이다.

> ajax의 x는 XML을 말하는데, 요즘에는 클라이언트와 서버가 통신할 때 XML 형식을 쓰기보다 JSON 형식을 많이 쓰기 때문에 이름이 혼란스럽긴 하다 ㅜㅜ

여튼, 이 ajax 기능을 구현하기 위한 실제 구현체는 브라우저 내장 객체인 [`XMLHttpRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)이다.

-----
> 기록용으로 먼저 작성

- 간단한 get 요청

```js
const result = await fetch(url).then(res => res.json());
```

- 요청 시 cookie 포함

> https://developer.mozilla.org/en-US/docs/Web/API/Request/credentials

```js
fetch(url, {
    credentials: 'include'
}).then(res => res.json());
```