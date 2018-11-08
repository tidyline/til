# AMP

> 공부 겸 요약한 것이라 오역이 있을 수도 있습니다.

## 왜 AMP 를 살펴보기 시작하였나?
파트 내에서 SEO와 관련해서 AMP도 살펴보기 시작하였는데,
이 때 처음 AMP Story를 발견하게 되었다.

AMP Story 사이트를 방문해보니 가이드도 잘 되어 있고,
굉장히 쉽게 컨텐츠들을 생산해 낼 수 있을 것 같다는 생각이 들었기 때문에
AMP Story로 뭔가 만들어 보고 싶은 욕망이 생겼따...

또한 Google이 만든 것이니.. 검색엔진에 최적화 되어있지 않을까...
렌더링 성능도 최적화 되어 있지 않을까 하는 믿음에...ㅎㅎ

그래서 결론적으로는 GUI 화면을 만들어서 쉽게 AMP Story 컨텐츠를 html로 뽑아낼 수 있도록 하면 어떨까??
하는 생각을 하게 되어서 살펴보게 되었다.

## [AMP Overview](https://www.ampproject.org/learn/overview/)
- AMP는 `AMP html`, `AMP script`, `AMP cache` 세 가지 구성요소로 이루어져 있다.

### AMP HTML
- AMP HTML은 믿을만한 성능을 내기 위해서 어느정도 제약을 둔 HTML 이다.

```html
<!doctype html>
<html ⚡>
 <head>
   <meta charset="utf-8">
   <link rel="canonical" href="hello-world.html">
   <meta name="viewport" content="width=device-width,minimum-scale=1,initial-scale=1">
   <style amp-boilerplate>body{-webkit-animation:-amp-start 8s steps(1,end) 0s 1 normal both;-moz-animation:-amp-start 8s steps(1,end) 0s 1 normal both;-ms-animation:-amp-start 8s steps(1,end) 0s 1 normal both;animation:-amp-start 8s steps(1,end) 0s 1 normal both}@-webkit-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-moz-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-ms-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-o-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}</style><noscript><style amp-boilerplate>body{-webkit-animation:none;-moz-animation:none;-ms-animation:none;animation:none}</style></noscript>
   <script async src="https://cdn.ampproject.org/v0.js"></script>
 </head>
 <body>Hello World!</body>
</html>
```
- 대부분 AMP HTML 페이지의 HTML tag는 일반적인 것들이지만,
  어떤 것들은 특정 AMP tag들로 대체되기도 한다.

- 이 특정 엘리먼트들을 **AMP HTML 컴포넌트**라고 하고, 성능이 좋은 엘리먼트로 만들어 준다.
- 예를 들어, `amp-img` 태그는 브라우저에서 아직 지원하지 않는 `srcset`을 지원한다.
- 또한 AMP 페이지는 검색엔진에서도 검색되어 질 때 `<link rel="">` 태그로
   non-AMP 버전을 보여줄지, AMP 버전을 보여줄지?? 그냥 AMP 버전을 보여줄지... 정할 수도 있다.

### AMP Script
- AMP JS 라이브러리는 AML Html 페이지를 빠르게 렌더링 할 수 있도록 보장해준다.

- AMP Script는 모두 비동기로 처리되기 때문에 JS로 인해서 페이지 로드가 블로킹 되지 않도록 한다.
- third-party JS는 모두 iframe으로 sandbox 처리되고,
  리소스 로드와 상관없이 미리 화면의 엘리먼트 사이즈와 위치를 계산해 놓기 때문에 성능이 좋다.
- 그리고 성능이 좋지 않는 CSS 셀렉터를 사용하지 못하게 한다.

### AMP Cache
- Google AMP Cache가 캐싱된 AMP HTML Page를 서빙하기 위해서 사용된다.

- Google AMP Cache는 proxy 기반의 컨텐츠 전송 네트워크이다.
- Google AMP Cache를 사용하는 도큐먼트는 동일 origin에서 최고 효율의 HTTP/2.0으로 JS파일과 모든 이미지들을 다운 받게 될 것이다.

## [AMP How It Works?](https://www.ampproject.org/learn/about-how/)
### 1) 모든 AMP JavaScript 를 비동기로 호출
- `<script>` tag를 만나게 되면 DOM을 구축하는 행위를 잠시 멈추고
   스크립트를 로드하기 때문에 페이지 렌더 속도를 지연시킨다.
- AMP script를 제외한 사용자가 만든 JavaScript는 AMP 페이지에 포함되지 못하고,
  이런 third-party JS는 iframe 내에서만 허용된다.

### 2) 모든 리소스를 정적으로 계산
- 이미지, 광고 또는 iframe과 같은 외부 리소스는 HTML에 반드시 사이즈를 명시해주어야 한다.
- AMP가 리소스가 다운로드 되기 전에 각 엘리먼트의 사이즈와 위치를 계산할 수 있기 때문이다.
- AMP는 리소스의 다운로드 여부와 상관없이 페이지 레이아웃팅을 먼저한다.
- AMP는 고비용의 Style Recacluation과 Layout 과정을 피하도록 최적화 되어있기 때문에,
   리소스가 로드 된 다음에 다시 레이아웃하는 일은 없을 것이다.

### 3) Extension mechanism이 렌더링을 블로킹하게 두지 않음
- AMP는 `lightbox`, `instagram embed`, `tweets`와 같은 extension을 지원한다.
- 이 extention 들이 HTTP 요청을 필요로 하지만, 이 요청이 페이지 렌더링을 블로킹하지 않는다.
- custom 스크립트를 사용하는 페이지가 있다면,
  AMP 에게 이 스크립트는 결국 DOM의 `custom-tag`를 필요로 할 것이라고 말해주어야 한다.
- 예를 들어 DOM의 `amp-iframe` 이라는 태그가 있어야 한다는 것을 미리 말해주기 위해서
  스크립트 로드 시 `custom-element="amp-iframe"`을 선언해줘야한다.
  그럼 미리 이 iframe box를 DOM에 선언해놓게 된다.
```html
<script async custom-element="amp-iframe" src="https://cdn.ampproject.org/v0/amp-iframe-0.1.js"></script>
```

### 4) third-party JavaScript는 Critical Path에서 제외하도록 해라
- third-party JS는 동기로 로드되는 JS를 말한다.
- AMP는 third-party JS를 iframe 내에서만 사용할 수 있게 한다.
- iframe 내에서 많은 Style Recacluation 이나 layout이 발생할 수는 있지만,
  초기 iframe 세팅 시 width, height를 지정해놓기 때문에
   third-party script로 인해 블로킹되는 것 보다는 좋은 성능을 가질 것이다.

### 5) 모든 CSS는 inline이여야 하고, size 제한이 있다.
- CSS는 모든 렌더링을 막고, 페이지 로드를 막기 때문에 AMP HTML 페이지에서는 inline style만 허용된다.
- 또한 inline style sheet의 최대 사이즈는 50kb 이다.

### 6) Font는 효율적으로 로드된다.
- 웹 폰트의 용량이 정말 크기 때문에, 폰트 최적화는 퍼포먼스에 큰 영향을 미친다.
- 일반적인 웹 브라우저에서는 동기 script와 style sheet 로드가 끝난 다음에서야 웹 폰트를 로드하게 된다.
- 하지만 AMP Stytem에서는 모든 script는 비동기로 로드되고,
  style sheet 는 inline으로 선언되어 있기 때문에 폰트 로드를 막는 HTTP 요청은 없다.

### 7) Style Recaclutation을 최소화 한다.
- AMP 페이지에서는 화면에 DOM 엘리먼트를 그리기 전에 모든 DOM을 읽어놓도록 한다.
- 그렇기 때문에 AMP 페이지에서는 한 프레임 당 한 번 이상의 recalculate 과정이 없는 것을 보장한다.

### 8) 애니메이션은 GPU 환경에서만 실행된다.
- 애니메이션에 관련된 CSS들은 GPU가속이 되도록 보장한다.
- AMP는 `transform`과 `opacity`를 통해서만 애니메이션을 허용하므로, 페이지 레이아웃이 필요하지 않다.

### 9) 페이지를 즉각 로드한다.
- 새로 나온 [preconnect API](https://www.w3.org/TR/resource-hints/#dfn-preconnect)는 HTTP 요청을 최대한 빠르게 되도록 보장이 필요할 때 사용된다.
- 이 API를 적용함으로써 사용자가 실제로 화면을 클릭하기 전에 미리 로드해 놓을 수 있게되고
    instant loading이 가능해지게 된다.
- 이와 같은 사전 렌더링은 모든 웹 컨텐츠에서 사용될 수는 있지만, 많은 bandwith와 CPU를 사용하게 된다.
- AMP 도큐먼트가 instant loading을 위해서 사전 렌더링을 할 때에는 스크롤 하단의 리소스만 다운해놓고,
   third-party iframe과 같이 고비용의 CPU를 사용하는 것은 다운로드 하지 않는다.