## 🔎  Chrome Extension을 개발하기 전..

크롬 익스텐션!! 요것만 알면 시작할 수 있따! 하는 것들을 간략하게 정리해보도록 하겠습니다. 🙂

- 크롬 익스텐션에서 사용되는 스크립트와 이벤트
- manifest.json 파일 살펴보기
- 디버깅은 어떻게 하는가?


### 1) 크롬 익스텐션에서 사용되는 스크립트와 이벤트

크롬 익스텐션을 개발하는데 사용되는 스크립트는 크게 `background`, `contentscript`, `popup` 이 있습니다.
![image](https://user-images.githubusercontent.com/25981942/51577997-2fdcbd00-1eff-11e9-8700-40e487ea9307.png)

`background`는 크롬 익스텐션이 실제로 동작하는데 필요한 스크립트이고,
`contentscript`는 사용자 화면의 DOM, 스크립트들을 제어하는 데 필요한 스크립트 입니다.
`popup`은 크롬 익스텐션 버튼을 눌렀을 때 아래에 뜨는 팝업창을 위한 스크립트 입니다.
팝업창의 모양을 그리기 위해서 html 파일도 있어야겠져?!! 그래서 `popup.html`도 존재합니다..

크롬에서는 크롬 익스텐션의 `background` 스크립트를 디버깅 하기 위해서 [별도의 개발자 도구 창](https://developer.chrome.com/extensions/getstarted#background)을 제공하는데요,
`background` 스크립트의 console.log는 이 개발자 도구에 기록되고,
`contentscript` 스크립트는 크롬 익스텐션을 실행 중인 사용자의 화면의 개발자 도구 창에 기록이 남습니다.
~~나의 로그가.. 왜 안나타나는가 헤맬까봐...*~~

그리고 storage에 저장하거나.. 하는 등의 액션은 모두 `background` 스크립트에서만 동작 가능하기 때문에,
`contentscript`에서는 이벤트를 발생시켜서 이런 액션들을 처리해주어야 합니다.

- `contentscript`에서 이벤트 발생 후 `background` 에서 listening 하기

```js
// contentscript.js
chrome.runtime.sendMessage({action: "FINISH"}, function(response) {
    alert(response);
});
```
```js
// background.js
chrome.runtime.onMessage.addListener(function (request, sender, sendResponse) {
	console.log(sender.tab ?
		"from a content script:" + sender.tab.url :
		"from the extension");

	if (request.action === "FINISH")
		sendResponse({farewell: "goodbye"});
});
```
> `chrome.runtime` 의 내부 메서드인 `sendMessage`를 사용해서 이벤트를 보내고,
>`addListener`를 통해서 이벤트를 듣고 있도록 되어 있습니다.

- `popup` 에서 클릭 했을 때 사용자 화면을 바꾸도록 하려면?

```html
<!--popup.html-->
<button id="start">분석 시작</button>
<script type="text/javascript" src="scripts/popup.js"></script>
```
```js
// popup.js
document.getElementById("start").onclick = function () {
	chrome.tabs.query({active: true, currentWindow: true}, function (tabs) {
		chrome.tabs.sendMessage(tabs[0].id, {action: "START"}, /* callback */);
	});
};
```
```js
// contentscript.js
chrome.extension.onMessage.addListener(function (msg, sender, sendResponse) {
	if (msg.action === "START") { /* do something */ }
});
```
> 런타임 환경이 아닌 익스텐션에서 메시지를 주고 받을 때는
> `chrome.extension.onMessage.addListener` 를 사용합니다.
> ⚠️ 그리고, `popup.js`에서 특정 탭에 이벤트를 보내기 위해서 tab의 id를 꼭 포함해서 보내주어야 합니다.

### 2) [manifest.json 파일](https://developer.chrome.com/extensions/extensions/manifest)
크롬 익스텐션의 설정파일들을 담아주는 곳이고, 아래와 같이 간-단한 정보들을 담고 있습니다.
```json
{
    "name": "Getting Started Example",
    "version": "1.0",
    "description": "Build an Extension!",
    "manifest_version": 2
}
```
그리고 크롬익스텐션에서 사용할 스크립트도 등록해주어야 합니다.

- `background` 스크립트 등록

```diff
{
    "name": "Getting Started Example",
    "version": "1.0",
    "description": "Build an Extension!",
+   "background": {
+     "scripts": ["background.js"],
+     "persistent": false
+   },
    "manifest_version": 2
}
```

> ⚠️ `persistent` 옵션은 `false`가 기본이지만,
> 사용자의 화면에서 발생하는 request들을 핸들링 하기위해
>`webRequest`를 사용할 때는 `true`로 해주어야 정상 동작합니다.

- `contentscript` 등록

```diff
{
    "name": "Getting Started Example",
    "version": "1.0",
    "description": "Build an Extension!",
    "background": {
      "scripts": ["background.js"],
      "persistent": false
    },
+   "content_scripts": [
+     {
+       "matches": ["http://*.nytimes.com/*"],
+       "css": ["myStyles.css"],
+       "js": ["contentScript.js"]
+     }
+   ],
    "manifest_version": 2
}
```
- `popup` 등록

```diff
{
    "name": "Getting Started Example",
    "version": "1.0",
    "description": "Build an Extension!",
+   "page_action": {
+     "default_icon": "icons/32.png",
+     "default_title": "Extension",
+     "default_popup": "popup.html"
+   },
    "background": {
      "scripts": ["background.js"],
      "persistent": false
    },
   "content_scripts": [
     {
       "matches": ["http://*.nytimes.com/*"],
       "css": ["myStyles.css"],
       "js": ["contentScript.js"]
     }
   ],
    "manifest_version": 2
}
```

```html
<!--popup.html-->
<button id="start">분석 시작</button>
<script type="text/javascript" src="scripts/popup.js"></script>
```
> ⚠️ `popup` 스크립트는 manifest에 등록하는 것이 아니라,
> `popup.html`에 script tag로 삽입해주면 됩니다.

- Permission 등록하기

```diff
{
    "name": "Getting Started Example",
    "version": "1.0",
    "description": "Build an Extension!",
    "manifest_version": 2,
+   "permissions": [
+     "tabs",
+     "declarativeContent"
+   ]
}
```
> 권한 종류는 [요기](https://developer.chrome.com/apps/declare_permissions) 가면 더 볼 수 있습니다.

------

### 3) 디버깅은 어떻게 하는가?

디버깅은 짱! 간단합니다..
주소창에 chrome://extensions/ 를 입력하시고..

아래와 같이 개발중인 폴더째로 로드하고..
성공하면 백그라운드 페이지를 누르면 디버깅 할 수 있는 별도의 개발자도구 창이 뜹니다.. 끗...

![image](https://user-images.githubusercontent.com/25981942/51578211-1b4cf480-1f00-11e9-978d-ff51fdea7919.png)

------

### 🐝  특정 도메인에서만 툴팁이 노출되었으면 좋겠어요!
> manifest permission : `"declarativeContent"`
> 구현 스크립트 : background.js

```js
chrome.runtime.onInstalled.addListener(function () {
	chrome.declarativeContent.onPageChanged.removeRules(undefined, function () {
		chrome.declarativeContent.onPageChanged.addRules([{
			conditions: [new chrome.declarativeContent.PageStateMatcher({
				pageUrl: {hostSuffix: 'abc.com'},
			})],
			actions: [new chrome.declarativeContent.ShowPageAction()]
		}]);
	});
});
```

- 런타임 `onInstalled` 이벤트가 발생하였을 때,
  이런 조건(`conditions`)일 때 이런 액션(`actions`)를 해라! 라는 코드인데요!
   - 조건 종류는 [여기](https://developer.chrome.com/extensions/declarativeContent)에서 더 확인할 수 있습니다.

- `hostSuffix` 즉, host가 'abc.com' 으로 끝나는 경우 `ShowPageAction()` 을 하라는 것입니다.
   manifest 에.. popup.html을 `page_action` 에 지정했던 것 기억하시나요?!
   그래서 `ShowPageAction()`을 호출하면 popup을 노출 시킬 수 있습니다.
   ~왠지.. `ShowPopup()` 이여야할 것 같은데...*~

- 그렇다면, `abc.com`이 아닌 도메인에서는 어떻게 노출되는지 궁금하시죠?
   ![image](https://user-images.githubusercontent.com/25981942/51578302-6830cb00-1f00-11e9-923c-3becdd2ed931.png)

- 조건이 일치하는 경우에는 팝업이 뜹니다..

---

## 기타

### 🐝 사용자 화면의 Http 요청을 제어할 수 있을까요?
> manifest permission : `"webRequest"`
> "persistent" : true
>```diff
> // manifest.json
> {
>   "background" : {
>       "scripts" : ["background.js"],
>+     "persistent" : true
>   }
>}
>```
> 구현 스크립트 : background.js

- 네!! 제어할 수 있습니다.
  자세한 설정은 [chrome.webRequest](https://developer.chrome.com/extensions/webRequest) 에 있습니다.

#### `onBeforeRequest` 예시

```js
chrome.webRequest.onBeforeRequest.addListener(
	function(details) {
		console.log(details.url.match(/a=(.*?)&/i)[1]);
                // return ~~ ;
	},
	{
		urls: [
			"*://abc.com/*",
			"*://beta-abc.com/*",
			"*://alpha-abc.com/*"
		]
	}
);
```
- 위 코드는 http 요청 전의 `onBeforeRequest` hook을 사용한 것입니다.

- http 요청 전 urls에 해당하는 것들만 걸러서,
  요청에 대한 정보를 콜백의 인자(`details`)로 넘겨줍니다.

#### 🚫 요청 cancel 예시

```js
chrome.webRequest.onBeforeRequest.addListener(
	function(details) {
		return {cancel: details.url.indexOf("://www.evil.com/") != -1};
	},
	{urls: ["<all_urls>"]},
	["blocking"]
);
```

- 뿐만 아니라 이 콜백에서 `return {}` 시 여러 옵션을 넘겨서
  요청을 수정하거나 취소하거나 하는 등의 제어도 가능합니다.

- 콜백의 리턴 값으로 `cancel : boolean` 프로퍼티를 이용해서 request를 취소 할 수도 있습니다.

-   ⚠️이렇게 request를 취소하는 경우에는 `"webRequest"` Permission 뿐만아니라
    `"webRequestBlocking"`도 추가해주어야 합니다.

- 그러나 아래와 같이 custom 한 스크립트를 동작시키고 싶을 때
   원하는 동작이 실행되지 않을 수도 있습니다.

```js
window.__myString = "퇴근하자";
```
이렇게 대입을 해도 클라이언트 사이드에서는 적용이 안됩니다. 😓

- 이런 경우에는 `script` 태그를 만들어서 DOM에 append 해주는 방식으로 처리해야 합니다.

```js
const clickcrScriptEl = document.createElement("script");
clickcrScriptEl.innerHTML = `window.__myString = "퇴근하자";`;
document.body.appendChild(clickcrScriptEl);
```

- 하지만!! 이렇게 모두 script를 string으로 대입하는 것은.. 굉장히 번거로운 작업이져!!

```js
var fullPath = chrome.extension.getURL('scripts/abc.js');

const script = document.createElement('script');
script.type = 'text/javascript';
script.src = fullPath;
document.body.appendChild(script);
```

> `chrome.extension.getURL`을 사용하면 chrome extension 내 스크립트 환경에 있는
> script 를 fullpath로 가져올 수 있습니다.
```