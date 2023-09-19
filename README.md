# semantic-release 을 이용하기 위한 예제

* GitHub Actions 기준으로 작성
* `semantic-release` 에서 지원하는 npm 배포 기능은 생략
* NodeJS 어플리케이션 기준으로 작성되었으나 사실상 상관없음. 주로 형상 관리를 자동화하는 내용 위주이기 때문에 Java, Go, Python 등 아무거나 가능

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

Git 서비스마다 다른데 [여기](https://github.com/semantic-release/semantic-release/blob/master/docs/usage/ci-configuration.md#push-access-to-the-remote-repository)를 참고하자. 참고로 GitHub 는 Secret 이름 앞에 `GITHUB_` 가 들어가도록 하는 것을 금지하고 있다.

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
> GitHub Actions 의 경우 CI 스크립트를 작성할 때 step 의 `permissions` 에 `contents` 를 `write` 로 해주어야 토큰 접근이 가능하다.

## `.releaserc` 파일 작성

`semantic-release` 설정을 담당하는데 아래와 같은 요소들을 설정해줄 수 있다.

* Git 저장소(URL, 배포할 브랜치, 태그 포맷)
* 사용할 플러그인 및 옵션 설정
* 실행 모드(디버그, dry run, local)

[문서](https://semantic-release.gitbook.io/semantic-release/usage/configuration)에 따르면 `.releaserc`, `.releaserc.yaml` 등의 파일로 관리하는 방법, `package.json` 내의 `release` 키로 관리하는 방법, 명령어 인자로 넘겨주는 방법으로 3가지 방식을 지원하고 있다.

### 플러그인

제공하는 플러그인 목록은 [여기](https://github.com/semantic-release/semantic-release/blob/master/docs/extending/plugins-list.md)에서 확인할 수 있다. 

중요한 점은 `semantic-release` 는 아래와 같은 순서로 진행되는데 만약, `.releaserc` 파일에서 플러그인 작성 시 순서가 맞지 않거나 서로 호환하는 플러그인이 없다면 제대로 동작되지 않을 수 있으므로 주의해야 한다.

따라서 플러그인을 사용할 때 아래 순서를 참고하여 작성하여야 한다. 위의 플러그인 목록 링크에도 간단히 명시되어 있고, 각 플러그인의 GitHub 저장소에도 `README.md` 파일 상단에도 명시되어 있다. 

* verifyConditions
* analyzeCommits
* verifyRelease
* generateNotes
* prepare
* publish
* addChannel
* success
* fail

자세한 내용을 알고 싶다면 [이곳](https://semantic-release.gitbook.io/semantic-release/usage/plugins)을 참고한다.

### 템플릿 모음

#### 템플릿 1

* 커밋 메시지에 따라 버전 관리
* 태그 배포

> **의존성**
> ```shell
> $ npm install -D semantic-release @semantic-release/git
> ```

```json
{
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator"
  ]
}
```

#### 템플릿 2

* 커밋 메시지에 따라 버전 관리
* 태그 배포
* package.json 버전 동기화

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

#### 템플릿 3

* 커밋 메시지에 따라 버전 관리
* 태그 배포
* package.json 버전 동기화
* release 배포

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

#### 템플릿 4

* 커밋 메시지에 따라 버전 관리
* 태그 배포
* package.json 버전 동기화
* release 배포
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

#### 템플릿 5

* 커밋 메시지에 따라 버전 관리
* 태그 배포
* package.json 버전 동기화
* release 배포
* CHANGELOG.md 파일 작성 & 템플릿 수정

> **Note**
> `@semantic-release/release-notes-generator` 에서 `conventionalcommits` 을 이용할 경우 반드시 `presetConfig` 가 설정되어 있어야 한다.

> **의존성**
> ```shell
> $ npm install -D semantic-release @semantic-release/git @semantic-release/changelog conventional-changelog-conventionalcommits
> ```

```json
{
  "plugins": [
    "@semantic-release/commit-analyzer",
    [
      "@semantic-release/release-notes-generator",
      {
        "preset": "conventionalcommits",
        "presetConfig": {
          "types": [
            { "type": "feat", "section": "✨ Features", "hidden": false },
            { "type": "fix", "section": "🐛 Bug Fixes", "hidden": false },
            { "type": "perf", "section": "🌈 Performance", "hidden": false },
            { "type": "refactor", "section": "♻️ Refactor", "hidden": false },
            { "type": "docs", "section": "📝 Docs", "hidden": false },
            { "type": "style", "section": "💄 Styles", "hidden": false },
            { "type": "revert", "section": "🕐 Reverts", "hidden": false },
            { "type": "ci", "section": "💫 CI/CD", "hidden": false },

            { "type": "test", "section": "✅ Tests", "hidden": true },
            { "type": "chore", "section": "📦 Chores", "hidden": true },
            { "type": "move", "section": "🚚 Move Files", "hidden": true },
            { "type": "remove", "section": "🔥 Remove Files", "hidden": true }
          ]
        }
      }
    ],
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

## commitlint (feat. Husky), commitizen (cz-cli)

* [commitlint](https://github.com/conventional-changelog/commitlint) - 커밋 메세지 작성 시 [Conventional Commit Format](https://conventionalcommits.org/) 를 준수하도록 메세지를 체크해주는 도구
  * **`git commit` 시 위 규칙의 준수를 강제하도록 Husky 가 주로 이용된다.** GitHub 저장소로 가보면 Husky 설치부터 commitlint 설치까지 전부 자세하게 설명되어 있다.
  * Windows, Linux 둘 다 상관없이 잘 작동되므로 걱정하지 말자
* [commitizen](https://github.com/commitizen/cz-cli) - 커밋 메세지 작성 시 [Conventional Commit Format](https://conventionalcommits.org/) 를 준수할 수 있도록 작성을 도와주는 도구

### commitlint 설치

1. commitlint cli 및 conventional 설정 설치

```shell
$ npm install --save-dev @commitlint/config-conventional @commitlint/cli
```

2. conventional 설정을 이용하기 위해 commitlint 설정

```shell
# conventional 설정을 이용하기 위해 commitlint 설정
# 운영체제 환경에 따라 쌍따옴표까지 작성되는 경우가 있는데 쌍따옴표가 들어가지 않도록 다시 확인할 것 
$ echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```

또는 저장소 루트 디렉토리에 `commitlint.config.js` 파일로 아래 내용을 저장한다. 

```javascript
module.exports = {
    extends: ['@commitlint/config-conventional'],
};
```

3. husky 설치

```shell
# husky 의존성 추가
$ npm install husky --save-dev

# husky 설치
$ npx husky install
```

최종적으로 저장소 루트 디렉토리에 `.husky` 폴더가 만들어졌는지 확인한다.

4. git hook 추가

```shell
$ npm pkg set scripts.commitlint="commitlint --edit"
$ npx husky add .husky/commit-msg 'npm run commitlint ${1}'
```

또는 `.husky/commit-msg` 파일로 아래 내용을 저장한다.

```shell
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

node_modules/.bin/commitlint --edit $1
```

5. 작동되는지 확인

```shell
$ git commit -m "test"
⧗   input: test
✖   subject may not be empty [subject-empty]
✖   type may not be empty [type-empty]

✖   found 2 problems, 0 warnings
ⓘ   Get help: https://github.com/conventional-changelog/commitlint/#what-is-commitlint

husky - commit-msg hook exited with code 1 (error)
```

### commitizen 설치

1. commitizen 설치

```shell
$ npm install commitizen cz-conventional-changelog -D
```

2. git hook 추가

```shell
$ npx husky add .husky/prepare-commit-msg '[ -z "${2-}" ] && exec < /dev/tty && node_modules/.bin/cz --hook || true'
```

또는 `.husky/prepare-commit-msg` 파일로 아래 내용을 저장한다.

```shell
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

[ -z "${2-}" ] && exec < /dev/tty && node_modules/.bin/cz --hook || true
```

3. `package.json` 에 아래 내용 추가

```json
{
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
}
```

# References

* https://github.com/semantic-release/semantic-release/blob/master/docs/usage/getting-started.md
* https://semantic-release.gitbook.io/semantic-release/usage/installation
* https://semantic-release.gitbook.io/semantic-release/usage/plugins
* https://github.com/semantic-release/release-notes-generator
* https://velog.io/@young_pallete/semantic-release