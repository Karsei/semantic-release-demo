# semantic-release 을 이용하기 위한 예제

GitHub Actions 기준으로 작성되었으며, `semantic-release` 에서 지원하는 npm 배포 기능은 생략되었음

## 사용법

### 기본

1. 해당 예제에서는 npm 배포를 이용하지 않을 것이므로 `package.json` 에서 아래와 같이 `private` 속성의 값을 `true` 로 설정한다.
    ```json
    {
      "private": true
    }
    ```
2. 개인 Token 을 발급받아야 한다. 자동화 과정에서 파일을 커밋하거나 release 배포를 하기 때문에 반드시 필요하다. `push` 권한이 필요하므로 scope 는 `repo` 에 체크해주자. 
    > **Note**
    > * GitHub - https://github.com/settings/tokens 
3. 저장소 CI Secret 에 해당 토큰을 `GH_TOKEN` 이라는 이름으로 저장한다. Git 서비스마다 다른데 [여기](https://github.com/semantic-release/semantic-release/blob/master/docs/usage/ci-configuration.md#push-access-to-the-remote-repository)를 참고하자. 참고로 GitHub 는 Secret 이름 앞에 `GITHUB_` 가 들어가는 것을 금지하고 있다.
4. CI 스크립트를 작성한다. `semantic-release` 에서는 가능하면 `npm install` 로 설치하여 저장소에서 실행하는 방법을 권장하는데, `npx` 와 같이 전역으로 실행하는 방법으로 진행할 경우 버전명을 같이 기입하도록 하는 것을 권고하고 있다.
    > **Note**
    > GitHub Actions 의 경우 CI 스크립트를 작성할 때 step 의 `permissions` 에 `content` 를 `write` 로 해주어야 토큰 접근이 가능하다.
    > ```yml
    > jobs:
    >   build:
    >     runs-on: ubuntu-latest
    >     permissions:
    >       contents: write
    > ...
    >     steps:
    >     - name: Release
    >       env:
    >         GITHUB_TOKEN: ${{ secrets.GH_TOKEN }}
    >       run: npx semantic-release@18
    > ```

### 선택

#### `.releaserc` 파일 작성

CHANGELOG 파일을 작성하게 한다거나 특정 커밋 메시지일 경우 어떻게 작동되게 할 것인지 여러가지로 지정해줄 수 있다.

##### 템플릿1.

* 커밋 메시지에 따라 버전 관리 + package.json 버전 변경 + release 배포

```json
{
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    [
      "@semantic-release/npm",
      {
        "npmPublish": false
      }
    ],
    "@semantic-release/github",
    [
      "@semantic-release/git",
      {
        "assets": "package.json",
        "message": "chore(release): ${nextRelease.version} [skip ci]\\n\\n${nextRelease.notes}"
      }
    ]
  ]
}
```



# References

* https://github.com/semantic-release/semantic-release/blob/master/docs/usage/getting-started.md
* https://semantic-release.gitbook.io/semantic-release/usage/installation
* https://semantic-release.gitbook.io/semantic-release/usage/plugins
* https://velog.io/@young_pallete/semantic-release