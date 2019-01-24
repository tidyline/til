## rebase 시 pick을 모두 다른 단어로 바꾸고 싶을때!

- 한땀한땀 치고 있었다니..

```bash
:%s/pick/edit/g
```

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

- 해결
    - IDE 터미널에서 에러가 발생하였는데, `cd ..` 갔다가 다시 `cd folder` 돌아오면 해결...*

- 관련 링크
    - https://github.com/npm/npm/issues/10983

-----

## 어느날 gitbook이 동기화가 안됨 ㅠㅠ

- 현상
    - github hook으로 gitbook 동기화를 사용하고 있었음
        - https://hooks.gitbook.com/hooks/github?spaceID={spaceId}
    - 그런데 이 방법이 바뀌었던 것 같음!!

- 해결
    - https://docs.gitbook.com/integrations/github/content-configuration
    - hook을 지우고 `.gitbook.yaml` 만들어서 넣어줌..

----

## 띠로리.. 깃헙 커밋을 잘못된 user.name과 user.email로 하였을 때..

```bash
git rebase -i HEAD~1
```

```bash
// interactive mode
// pick을 edit(e)로 변경하고 :wq!

e 4bfa08c delete file

# Rebase 422cbed..4bfa08c onto 422cbed (1 command)
```

```bash
git commit --amend --author="tidyline <tidyaline@gmail.com>"
```

```bash
git rebase --continue

// tada!
```
