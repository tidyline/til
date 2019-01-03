# amp-story

## [AMP CORS](https://www.ampproject.org/docs/fundamentals/amp-cors-requests) (WIP)

> AMP 공식 문서 번역 및 간추려서.. 정리하면서 알아가보기

### 왜 동일 origin에서 요청하는게 CORS가 필요할까?
- 만약 어떤 AMP Page가 프로덕트의 리스트로 구성되어 있고, 각 아이템은 가격 정보를 갖고 있다고 해보자.
- 이 때 사용자가 버튼을 클릭하면 동일 도메인의 endpoint에서 최신 가격정보를 받아온다고 해보자.
   (`amp-list`, `amp-form`와 같이 데이터를 fetch하는 AMP 컴포넌트들은 기본으로 CORS 요청을 한다.)
- 이런 경우에는 CORS 문제없이 잘 동작한다.
  (origin domain → origin domain)
<br />

- 하지만, 구글 검색에서 유입된 경우에는 문제가 발생한다.
- Google Search가 AMP Page를 빠르게 로드하기 위해 Google AMP Cache를 사용하기도 하기 때문이다.
- 이와 같이 캐싱 된 페이지는 Google AMP Cache 서버에서 서빙되고,
   그렇기 때문에 서로 도메인이 불일치하게되어 CORS 이슈가 발생한다.
   (cache → origin domain)
    ![CORS_with_Cache](https://www.ampproject.org/static/img/docs/CORS_with_Cache.png)

### CORS 요청에서 쿠키 사용하기
- CORS 요청을 사용하는 대부분의 AMP 컴포넌트들은 자동으로 [credentials mode](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/withCredentials)를 세팅하거나,
  개발자로 하여금 옵션을 제어할 수 있도록 허용해놓았다.

```html
<amp-list credentials="include"
    src="<%host%>/json/product.json?clientId=CLIENT_ID(myCookieId)">
  <template type="amp-mustache">
    Your personal offer: ${{price}}
  </template>
</amp-list>
```
-  이 origin 에서 credential mode를 직접 설정함으로써 CORS 요청 / 응답 시 쿠키를 포함할 수 있게 되었다.


#### Third-party cookie restrictions
- 브라우저에 정의되어 있는 [third-party cookie](https://en.wikipedia.org/wiki/HTTP_cookie#Third-party_cookie) 제약이 AMP CORS credential 요청에도 동일하게 적용된다.
- 이 제약들은 브라우저와 플랫폼에 따라서 다르다.

### AMP에서 CORS 보안
- AMP 페이지에서 유효하고 안전한 요청과 응답을 위해서는 아래 조건을 꼭 지켜줘야 한다:
  - 요청 검증
  - 적합한 Response Header 보내기

#### 요청 검증
- endpoint에서 AMP 의 CORS 요청을 받았을 때:
1. CORS Origin Header가 허용된 origin 인지 확인 (본인 domain과 AMP cache의 origin)
2. 만약 Origin 헤더가 없다면, 요청이 동일한 origin 인지 `AMP-Same-Origin`을 통해서 확인해라
3. 만약 요청이 POST와 같이 상태 변경에 대한 것이라면, origin이 source origin으로 부터 온 것인지 확인해야한다. (__amp_source_origin)
