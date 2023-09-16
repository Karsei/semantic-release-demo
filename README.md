# semantic-release 을 이용하기 위한 예제

* GitHub Actions 기준으로 작성
* `semantic-release` 에서 지원하는 npm 배포 기능은 생략

# 사용법

## 기본

### 1. 의존성 설치

```shell
# `package.json` 에서 의존성을 관리하며 이용할 경우
$ npm install -D semantic-release @semantic-release/git

# 전역 명령어로 사용할 경우 (이때는 가급적 버전을 명시하는 것을 권고하고 있다)
$ npx semantic-release@18
```

> **Note**
> 전역 명령어를 사용할 경우 이후 플러그인 이용 시 `@semantic-release/` 로 시작하는 것들은 그럼에도 의존성이 필요하거나 필요하지 않을 수 있다.

### 2. `package.json` 설정

해당 예제에서는 npm 배포를 이용하지 않을 것이므로 `package.json` 에서 아래와 같이 설정한다.

```json
{
  "private": true
}
```

### 3. 개인 Token 발급

자동화 과정에서 파일을 커밋하거나 release 배포를 하기 때문에 반드시 필요하다. `push` 권한이 필요하므로 scope 는 `repo` 에 체크해서 만들어주자.

> **Note**
> * GitHub - https://github.com/settings/tokens

### 4. CI Secret 설정

저장소 CI Secret 에 해당 토큰을 `GH_TOKEN` 이라는 이름으로 저장한다.

Git 서비스마다 다른데 [여기](https://github.com/semantic-release/semantic-release/blob/master/docs/usage/ci-configuration.md#push-access-to-the-remote-repository)를 참고하자. 참고로 GitHub 는 Secret 이름 앞에 `GITHUB_` 가 들어가는 것을 금지하고 있다.

### 5. CI 스크립트 작성

아래는 예시

```yaml
name: Node.js CI

on:
  push:
    branches: [ "master" ]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    strategy:
      matrix:
        node-version: [18.x]

    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm ci
    - run: npm run build --if-present
    # - run: npm test
    - name: Release
      env:
        GITHUB_TOKEN: ${{ secrets.GH_TOKEN }}
      run: npx semantic-release@18
```

> **Note**
> GitHub Actions 의 경우 CI 스크립트를 작성할 때 step 의 `permissions` 에 `content` 를 `write` 로 해주어야 토큰 접근이 가능하다.

## 선택

### `.releaserc` 파일 작성

CHANGELOG 파일을 작성하게 한다거나 특정 커밋 메시지일 경우 어떻게 작동되게 할 것인지 여러가지로 지정해줄 수 있다.

#### 템플릿1

* 커밋 메시지에 따라 버전 관리
* package.json 버전 동기화
* release 상시 배포

> **의존성**
> ```shell
> $ npm install -D semantic-release @semantic-release/git
> ```

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
        "assets": ["package.json"],
        "message": "chore(release): ${nextRelease.version}\n\n${nextRelease.notes}"
      }
    ]
  ]
}
```

#### 템플릿2

* 커밋 메시지에 따라 버전 관리
* package.json 버전 동기화
* release 상시 배포
* CHANGELOG.md 파일 작성

> **의존성**
> ```shell
> $ npm install -D semantic-release @semantic-release/git @semantic-release/changelog
> ```

```json
{
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    [
      "@semantic-release/changelog",
      {
        "changelogFile": "CHANGELOG.md",
        "changelogTitle": "# 🚦 CHANGELOG"
      }
    ],
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
        "assets": ["package.json", "CHANGELOG.md"],
        "message": "chore(release): ${nextRelease.version}\n\n${nextRelease.notes}"
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