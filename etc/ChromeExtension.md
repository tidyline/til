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