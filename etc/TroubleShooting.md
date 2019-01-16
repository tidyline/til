## 잘되던 npm install 갑자기 왜 안되지??

- 현상
    - `npm -v` 만 해도 에러를 뿜뿜..
    - 특정 폴더에서만 발생

```bash
path.js:424
    var path = (i >= 0) ? arguments[i] : process.cwd();
                                                 ^

Error: ENOENT: no such file or directory, uv_cwd
    at Error (native)
    at Object.posix.resolve (path.js:424:50)
    at Function.Module._resolveLookupPaths (module.js:250:17)
    at Function.Module._resolveFilename (module.js:317:31)
    at Function.Module._load (module.js:277:25)
    at Module.require (module.js:354:17)
    at require (internal/module.js:12:17)
    at /usr/local/lib/node_modules/npm/bin/npm-cli.js:26:13
    at Object.<anonymous> (/usr/local/lib/node_modules/npm/bin/npm-cli.js:76:3)
    at Module._compile (module.js:398:26)
```

- 원인
    - 난 바보..
    - IDE를 열어놓은 상태에서 해당 폴더를 지웠다가, 다시 만들어서 새롭게 개발하려고 했음..
    - IDE 터미널에서 에러가 발생하였는데, `cd ..` 갔다가 다시 `cd folder` 돌아오면 해결...*

- 관련 링크
    - https://github.com/npm/npm/issues/10983