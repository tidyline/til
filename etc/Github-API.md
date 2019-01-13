## Github API

### 이슈 생성 하는 법

- url 생성
  - https://github.com/api/v3/repos/{organization}/{project}/issues
  - ex) https://github.com/api/v3/repos/tidyline/til/issues

1) `curl`로 이슈 생성하기

```
curl -X POST -i -u {id}:{token} -d \
'{"title" : "제목이요!", "body" : "내용이요!"}'
https://github.com/api/v3/repos/{organization}/{project}/issues
```

2) `node` https 모듈을 이용해서 이슈 생성하기

```js
const https = require('https');
const issue = {
    title : '제목이요!',
    body : '내용이요'
};

const req = https.request({
  host: 'github.com',
  method: 'POST',
  port: 443,
  path: '/api/v3/repos/{organization}/{project}/issues',
  headers: {
    'Accept': 'application/vnd.github.symmetra-preview+json',
    'Authorization': 'token {token}'
  }
}, res => {
  let data = '';
  res.on('data', chunk => data += chunk);
  res.on('end', () => console.log(JSON.parse(data)));
});

req.write(JSON.stringify(issue));
req.on('error', e => console.log('error', e));
req.end();
```
