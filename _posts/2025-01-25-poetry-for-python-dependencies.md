---
title: "[python] Poetry 의존성 관리하기 ( virtual env )"
date: 2025-01-25 23:42:00 +0900
author: tyrosteller
categories: [Dev-Python]
tags: [project-COOTTE, python, poetry, virtualenv, dependencies]
---

**나는! 이 게임을 해봤어요!**

예 무슨 소리냐.. 사실 지금 하려는 `poetry`를 사용하기 전에 pip, venv를 사용한 가상 환경을 이용하고 requirements를 직접 관리하는 식으로 프로젝트를 한번 만들어 봤습니다..

물론 제가 requirements 를 `pip freeze_ > requirements.txt` 만 사용해서 무지성으로 관리한 탓 도 있겠지만, 다른 팀원이 있는 것도 아닌 단독 프로젝트에서 의존성 관리 문제를 초기에 겪고나서 poetry를 도입하기로 마음 먹고 작성하는 글 입니다.


## Poetry?

`poetry`는 파이썬 생태계에서 의존성 관리와 패키징을 효율적으로 처리할 수 있도록 설계된 도구로, 기존의 `pip`와 `virtualenv`를 대체하거나 보완하는 방식으로 사용
프로젝트마다 하나의 가상환경을 자동으로 생성해줘서 관리가 편함!

- **의존성 관리 통합**
    - `requirements.txt`와 같은 파일 없이도 의존성 관리가 가능하며, `pyproject.toml` 파일을 사용하여 의존성을 명확하게 정의.
    - 의존성 충돌을 자동으로 해결!
    - 물론 후에 pip 환경에서 배포를 해야한다면 poetry에서 requirements 를 만들어서 배포도 가능
    - `poetry.lock` 파일을 통해 의존성 버전을 고정하여 모든 환경에서 일관된 결과를 보장합니다.
- **가상환경 관리**
    - 프로젝트별로 독립된 가상환경을 자동으로 생성 및 관리합니다.
    - 기존의 가상환경을 사용하도록 설정할 수도 있습니다.
- **빌드 및 배포 도구**
    - 패키지 빌드와 배포를 쉽게 처리할 수 있습니다. 패키징 설정도 `pyproject.toml`에 포함됩니다.
- **직관적인 CLI**
    - 간단한 명령어로 의존성 추가, 제거, 업데이트, 패키지 빌드 등을 처리 가능

위와같은 장점에 더해 기본적으로 pip로 설치할 수 있는 패키지와 Poetry로 설치할 수 있는 패키지, 두 도구 모두 PyPI (Python Package Index)에서 패키지를 가져오기 때문에 차이는 없다는 점이 안심되는 부분이고

npm에 익숙해져 있다보니 lock 파일도 좋았고, dev-def와 product-def 모두 관리하기엔 poetry가 더 편해 보였다

그래서 일단 Poetry로 가상환경을 먼저 셋팅하고, fastAPI 와 uvicorn을 설치해서 프로젝트를 진행해 보자

---

## **Poetry install**

python3 부터는 자동으로 설치되어있는 venv와 다르게 설치가 필요하다
일단 먼저 [python poetry docs](https://python-poetry.org/docs/#installing-with-the-official-installer)에서 가이드 해주는대로 poetry 를 설치해보자

```shell
curl -sSL https://install.python-poetry.org | python3 -
```

음 시작부터 순탄치 않은건 늘 여전하다

<br>

### Issue! - python ssl 에러
위 명령어로 poetry를 설치하려 했더니 에러가 발생했다.

```
urllib.error.URLError: <urlopen error [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1000)>
```
> 이 오류는 Python이 SSL 인증서를 검증할 때, **로컬 인증 기관(CA)의 인증서를 찾을 수 없다**는 문제로 발생한다고 한다. macOS 환경에서 종종 나타나는 문제인데 Python의 SSL 라이브러리가 macOS의 기본 인증서 저장소를 사용하지 못하거나 최신 인증서가 설치 되지 않았을 때 발생한다고 한다!
{ :.prompt-error }

macOS에서 기본적으로 인증서를 관리하므로, 최신 인증서를 설치하는 것이 첫 번째 해결책이라고 한다! 
터미널에서 아래 명령어를 실행하여 Python 인증서를 업데이트를 진행하자

<br>

```
/Applications/Python\ 3.x/Install\ Certificates.command
```
>위 명령어는 Python 설치 시 포함된 인증서 설치 스크립트를 실행하여, SSL 관련 문제를 해결 해준다!
{: .prompt-tip }
   
<br>

```
/Applications/Python\ 3.12/Install\ Certificates.command
 -- pip install --upgrade certifi
Collecting certifi
  Obtaining dependency information for certifi from https://files.pythonhosted.org/packages/a5/32/8f6669fc4798494966bf446c8c4a162e0b5d893dff088afddf76414f70e1/certifi-2024.12.14-py3-none-any.whl.metadata
  Downloading certifi-2024.12.14-py3-none-any.whl.metadata (2.3 kB)
Downloading certifi-2024.12.14-py3-none-any.whl (164 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 164.9/164.9 kB 3.2 MB/s eta 0:00:00
Installing collected packages: certifi
Successfully installed certifi-2024.12.14

[notice] A new release of pip is available: 23.2.1 -> 25.0
[notice] To update, run: pip3 install --upgrade pip
 -- removing any existing file or link
 -- creating symlink to certifi certificate bundle
 -- setting permissions
 -- update complete
```

위와 같이 pip로 certifi install을 하며 자동으로 무언가 진행을 해준다.


이후 다시 처음 curl 명령어로 poetry를 설치해주면!

```
Retrieving Poetry metadata

# Welcome to Poetry!

This will download and install the latest version of Poetry,
a dependency and package manager for Python.

It will add the `poetry` command to Poetry's bin directory, located at:

/Users/name/.local/bin

You can uninstall at any time by executing this script with the --uninstall option,
and these changes will be reverted.

Installing Poetry (2.0.1): Done

Poetry (2.0.1) is installed now. Great!

To get started you need Poetry's bin directory (/Users/name/.local/bin) in your `PATH`
environment variable.

Add `export PATH="/Users/name/.local/bin:$PATH"` to your shell configuration file.

Alternatively, you can call Poetry explicitly with `/Users/wooseok/.local/bin/poetry`.

You can test that everything is set up by executing:

`poetry --version`
```

<br>

멀쩡하게 설치가 완료된다!
물론 이번에는 설치 후 나오는 메시지를 꼼꼼히 읽었으니, 내 zsh에 Path를 추가해주자


```shell
export PATH="/Users/username/.local/bin:$PATH"
```

이렇게하면 `Poetry (version 2.0.1)` 설치 완료!

<br>

---

<br>

## 프로젝트 Poetry 로 의존성 관리하기

poetry 만 설치하고 프로젝트를 init 하는 경우

```python
poetry new my-project
```

new 명령어를 사용해서 기본 프로젝트 구조를 가져갈 수 있지만, 프로젝트 directory가 이미 있는 나는 init 명령어를 사용해서 인터렉티브하게 `pyproject.toml` 파일을 만들어봤다

### Initialize

```
cd my-project
poetry init
```


```shell
This command will guide you through creating your pyproject.toml config.

Package name [cootte-api]:
Version [0.1.0]:
Description []:
Author [tyrosteller <tyros.dev@gmail.com>, n to skip]:
License []:
Compatible Python versions [>=3.12]:

Would you like to define your main dependencies interactively? (yes/no) [yes]
        You can specify a package in the following forms:
          - A single name (requests): this will search for matches on PyPI
          - A name and a constraint (requests@^2.23.0)
          - A git url (git+https://github.com/python-poetry/poetry.git)
          - A git url with a revision         (git+https://github.com/python-poetry/poetry.git#develop)
          - A file path (../my-package/my-package.whl)
          - A directory (../my-package/)
          - A url (https://example.com/packages/my-package-0.1.0.tar.gz)

Package to add or search for (leave blank to skip):

Would you like to define your development dependencies interactively? (yes/no) [yes]
Package to add or search for (leave blank to skip):

Generated file

[project]
name = "cootte-api"
version = "0.1.0"
description = ""
authors = [
    {name = "tyrosteller",email = "tyros.dev@gmail.com"}
]
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
]


[build-system]
requires = ["poetry-core>=2.0.0,<3.0.0"]
build-backend = "poetry.core.masonry.api"
```

자 이제 위 선택한 내용대로 `pyproject.toml` 파일이 생성되었다!

<br>   

### **가상환경 생성 및 활성화**

일단 init 과정에서 따로 dependencies를 지정하지 않아서 비어있는 상태인데, 이상태 그대로 `poetry install` 을 진행하면, 가상환경만 설치를 하게 된다!

```
poetry install
Creating virtualenv cootte-api-3vJMG5Bj-py3.12 in /Users/name/Library/Caches/pypoetry/virtualenvs
Updating dependencies
Resolving dependencies... (0.1s)

Writing lock file

Installing the current project: cootte-api (0.1.0)
Error: The current project could not be installed: Readme path `/Users/name/Applications/cootte-api/README.md` does not exist.
If you do not want to install the current project use --no-root.
If you want to use Poetry only for dependency management but not for packaging, you can disable package mode by setting package-mode = false in your pyproject.toml file.
If you did intend to install the current project, you may need to set `packages` in your pyproject.toml file.
```
이렇게 가상환경과 lock 파일이 생성된다!
* 살짝 에러가 발생했는데, 이건 Poetry가 현재 프로젝트를 "패키지로 설치" 하려고 하는데, README.md 가 없어서 발생한 에러다


```toml
[tool.poetry]
package-mode = false
```


> 기본적으로 `pyproject.toml` 파일이 있는 프로젝트는 Poetry가 "패키지 관리 + 패키징"을 위한 것으로 간주한다고 하는데, 나는 API 서버로 사용할 프로젝트기 때문에
따로 PyPI 같은 곳에 배포할 이유가 없어서 `package-mode` 를 false로 설정해줬다
{: .prompt-tip }

<br>

생성된 가상환경 목록을 확인해보면

```shell
poetry env list

cootte-api-3vJMG5Bj-py3.12 (Activated)
```

이렇게 생성된 가상환경이 Activated 된 상태로 보여진다

좀더 자세한 정보를 보려면 `poetry env info` 를 사용해보자

```
poetry env info

Virtualenv
Python:         3.12.1
Implementation: CPython
Path:           /Users/name/Library/Caches/pypoetry/virtualenvs/cootte-api-3vJMG5Bj-py3.12
Executable:     /Users/name/Library/Caches/pypoetry/virtualenvs/cootte-api-3vJMG5Bj-py3.12/bin/python
Valid:          True

Base
Platform:   darwin
OS:         posix
Python:     3.12.1
Path:       /Library/Frameworks/Python.framework/Versions/3.12
Executable: /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12
```


--- 

<br>

### 번외 poetry shell 

물론 install로 가상환경을 설정하는 것 말고도 `poetry shell`을 사용할 수 있다고 한다
기존에는 별다른 설치 없이 사용 가능한 명령어 였던 듯 한데, 지금은 [poetry shell 공식 문서](https://python-poetry.org/docs/cli/#shell) 를 보니 

>The `shell` command was moved to a plugin: [`poetry-plugin-shell`](https://github.com/python-poetry/poetry-plugin-shell)

이런 문구를 확인할 수 있었다, 위 git repo readme 에서 설치법과 사용법을 간단하게 확인할 수 있었고, 
[Issue!](https://github.com/python-poetry/poetry-plugin-shell/issues/3)를 보니 플러그인 유지보수에 관심있는 사람도 물색 중 인듯 하다, 나도 실력이 되면 저런것도 해보고 싶다..

---

<br>

### 의존성 추가하기

이제 가상환경까지 생성되었고 원하는 내 프로젝트를 만들기 위해 `pip install` 하듯 사용해주면 된다!
자세한 명령어는 [poetry-cli](https://python-poetry.org/docs/cli/)에서 확인해보자!

```shell
poetry add <package_name>

poetry add <package_name> --dev | poetry add <package_name> --D
poetry add <package_name> --group test | poetry add <package_name> -G test
```

개발 의존성 뿐만아니라 특정 그룹의 의존성을 관리 할 수 있도록 옵션을 사용할 수 있으며 설치 할 때 특정 그룹 의존성들만 설치도 가능하다!


```shell
poetry install
poetry install --with group 
poetry install --without group 
poetry install --only main,group 
```

>`--only`는 입력한 모든 그룹의 의존성을 설치하지만, 명시하지 않는다면 main도 제외하고 설치를 한다.
>또한 only 옵션을 사용하면 with, without은 무시된다!

<br>

<br>

---
## 2025-01-26 추가된 글

원래는 poetry shell을 사용할 생각을 하지 않았는데..
uvicorn을 설치하고 서버를 열기 위해 `uvicorn main:app --reload` 명령어를 사용했더니 `zsh: command not found: uvicorn` 에러가 발생했다..
분명히 위에서 확인 했 듯 내 가상환경은 `cootte-api-3vJMG5Bj-py3.12 (Activated)` Activated 된 상태인데!


```
poetry run uvicorn main:app --reload
```
위 처럼 사용하면 동작은 한다.. 하지만 너무 귀찮지 않은가!!


### 가상환경 활성화와 관련된 오해

`poetry env list` 이 명령은 현재 프로젝트에 연결된 가상환경을 보여주지만, 이는 가상환경의 경로와 상태를 알려줄 뿐, 가상환경이 **현재 쉘에서 활성화되었는지**는 보장하지 않는다고 한다!

즉 프로젝트에 가상환경은 존재하고 프로젝트와 연결되어 있지만! 내가 지금 사용하고 있는 shell에서 연결된건 아니라는 말로 보인다.. 
결국 나는 이 shell 에서 가상환경을 활성화하고 path를 인식 시키기 위해 `poetry shell`도 설치하기로 한다..

>남들이 하는덴 다 이유가 있는 법 이더라..


```
poetry self add poetry-plugin-shell
```

먼저 위 명령어로 poetry-plugin-shell 을 설치해주자


```
Using version ^1.0.1 for poetry-plugin-shell

Updating dependencies
Resolving dependencies... (10.0s)

Package operations: 3 installs, 0 updates, 0 removals

  - Installing ptyprocess (0.7.0)
  - Installing pexpect (4.9.0)
  - Installing poetry-plugin-shell (1.0.1)

Writing lock file
```

dependencies resolve 하는데 10초나 걸려서 살짝 걱정했지만.. 문제없이 설치 된 듯 하다

이제 내가 쓰는 `zsh` 에서 `poetry shell` 명령어로 가상환경을 활성화 시켜주자
> venv 에서도 가상환경을 활성화하면 shell의 username 앞에 Venv 이름이 붙었는데 Poetry도 동일하다는걸 이제 알았다!
{: .prompt-tip }

---



자 이제 다음시간엔 fast API 프로젝트를 만들고, lint와 formatter도 구성해보자.