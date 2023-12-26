---
title: 'nvm으로 노드 버전관리 자동화하기'
date: 2023-10-26 19:50:00 +0800
category: 'Tip'
draft: false
---

nvm으로 프로젝트 노드 버전 관리를 하고 있다.
하지만 프로젝트를 새롭게 낄 때마다 node를 재설정해줘야 되서 너무 불편했다.
그래서 프로젝트로 진입했을 때 지정한 노드버전으로 해줄 방법을 찾다가 알게된 방법을 공유하고자 한다.

## nvm 설치

우선 nvm을 설치해준다. nvm은 (Node Version Manager)의 약자로 Node.js의 버전을 관리할 수 있는 도구이다. 아래 명령어를 사용하여 nvm을 설치하자.

```shell
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```

## .nvmrc 파일 생성

프로젝트 루트 디렉토리에 `.nvmrc` 파일을 생성하고 원하는 노드 버전을 명시한다. 아래 명령어를 입력하면 노드 버전을 명시한 파일이 생성된다.

```shell
$ echo "14.15.0" > .nvmrc
```
![](/assets/img/contents/node-version/node-version.png){: w="830"}

## 자동화 스크립트 작성

`.zshrc` 파일에 자동화 스크립트를 추가하여 프로젝트로 이동할 때 자동으로 .nvmrc 파일에 지정된 노드 버전을 활성화하고 필요한 경우 해당 버전을 자동으로 설치한다. `.zshrc` 파일을 열어 다음 스크립트를 추가한다.

```
# vim
vim ~/.zshrc

# vscode
code ~/.zshrc
```

```shell
# place this after nvm initialization!
autoload -U add-zsh-hook
load-nvmrc() {
  local node_version="$(nvm version)"
  local nvmrc_path="$(nvm_find_nvmrc)"

  if [ -n "$nvmrc_path" ]; then
    local nvmrc_node_version=$(nvm version "$(cat "${nvmrc_path}")")

    if [ "$nvmrc_node_version" = "N/A" ]; then
      nvm install
    elif [ "$nvmrc_node_version" != "$node_version" ]; then
      nvm use
    fi
  elif [ "$node_version" != "$(nvm version default)" ]; then
    echo "Reverting to nvm default version"
    nvm use default
  fi
}
add-zsh-hook chpwd load-nvmrc
load-nvmrc
```

스크립트를 붙여넣었다면 `:wq` 로 저장해서 나오자.

프로젝트 디렉토리로 이동하면 다음과 같은 메세지가 출력된다.

![](https://github.com/saseungg/saseungg/assets/115215178/19ef8032-9350-4655-ade0-c75cec307316)

.nvmrc 파일을 찾았고 파일에 정의한대로 node.js 버전을 사용한다는 의미이다.
프로젝트에 이동 시 해당 명령어가 뜨면서 버전이 변경된다.

이렇게 간편하게 노드 버전을 관리할 수 있다.

## 기타 nvm 관련 명령어

node 버전 설치

```
$ nvm install {version}
```

버전 적용

```shell
$ nvm use {version}
```

현재 디렉토리 노드 버전 확인

```shell
$ nvm current
```

컴퓨터 설치된 노드 버전 확인

```shell
$ nvm ls
```

default 버전 등록

```shell
$ nvm alias default {version}
```

