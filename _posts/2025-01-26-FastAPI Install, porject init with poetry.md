---
title: "[FastAPI] Poetry 를 사용해서 FastAPI 설치 및 프로젝트 구성!"
date: 2025-01-26 18:16:00 +0900
author: tyrosteller
categories: [Dev-FastAPI]
tags: [project-COOTTE, python, poetry, FastAPI]
---

이미 파이썬을 통해 웹 개발을 진행하기 위한 여러가지 프레임워크로 Django, flask 등이 있고 Django는 몇 년전에 아주 잠깐 맛보기로 경험한 적 있지만, 
파이썬만으로 웹을 개발하는게 목적이 아닌, 파이썬의 강력한 머신러닝 도구를 사용하는것이 현재로써 가장 멀리보고 있는 목표이다,

이것이 웹에 적용될 부분이 많을 것이라는 기대감으로 Fast API로 API서버를 구성 해보자!

---

## Uvicorn ?

기존에 flask나 Django 등으로 개발을 해본 사람들은 **WSGI(Web server Gateway Interface)** 중 하나인 `gunicorn` 을 사용해 봤을 것 이다.
WSGI는 말그대로 Web Server ( Nginx ) 와 Web Application ( flask )사이의 인터페이스로 웹서버의 요청을 웹 어플리케이션으로 전달해주는 역활을 한다.

Uvicorn은 lightweigth **ASGI(Asynchronous Server Gateway Interface)** 로 이름에서 알 수 있듯 비동기로 여러 request를 처리 할 수 있게 만든 경량 서버 구현체다!

> Uvicorn[standard]를 설치하면 아래와 같은 의존성이 설치되는데 각각의 역활을 간단히 살펴보면 비동기를 위한 도구들이 보인다!
`uvloop`: 고성능 비동기 이벤트 루프
`httptools`: 고성능 HTTP 파서
`watchgod`: 코드 변경 감지 리로딩 지원
{: .prompt-info }

<br>

아무튼 이 구조에 대해 깊이 설명하면 진행이 불가능할 것 같으니 간단하게 설명하자면 FastAPI 같은 framework 만으로는 웹을 운영할 수 없으니 표준 ASGI uvicorn을 같이 설치하는 것!
장점도 많다, 경량이라 가볍고 빠르며 배포에 별도 준비가 필요 없는 등 의 장점이 있지만 일단 이건 체감을 해보고 생각하자!


---

<br>

## Install 

### Uvicorn 추가 하기

```
poetry add uvicorn[standard]
```

일단 uvicorn을 추가 할 때, zsh을 쓰는 분들은 나와 같이 `poetry add "uvicorn[standard]"` 따옴표로 묶어서 입력해주자


```
Using version ^0.34.0 for uvicorn

Updating dependencies
Resolving dependencies... (2.3s)

Package operations: 9 installs, 0 updates, 0 removals

  - Installing click (8.1.8)
  - Installing h11 (0.14.0)
  - Installing httptools (0.6.4)
  - Installing python-dotenv (1.0.1)
  - Installing pyyaml (6.0.2)
  - Installing uvloop (0.21.0)
  - Installing watchfiles (1.0.4)
  - Installing websockets (14.2)
  - Installing uvicorn (0.34.0)

Writing lock file
```

> `uvicorn` 이 아닌 `uvicorn[standard]`를 설치하는 이유는 [공식 홈페이지](https://www.uvicorn.org/#quickstart)에서 설명 하듯 Cython으로 만들어진 `uvloop` `httptools` 등 을 같이 설치하기 위해서다.
> Standard 가 아니면 `uvloop` 대신 [event loop - asyncio](https://github.com/encode/uvicorn/blob/master/uvicorn/loops/auto.py) 를 사용한다고 한다.

<br>

### FastAPI 추가 하기


```shell
poetry add fastapi
```


```shell
Using version ^0.115.7 for fastapi

Updating dependencies
Resolving dependencies... (1.6s)

Package operations: 9 installs, 0 updates, 0 removals

  - Installing idna (3.10)
  - Installing sniffio (1.3.1)
  - Installing typing-extensions (4.12.2)
  - Installing annotated-types (0.7.0)
  - Installing anyio (4.8.0)
  - Installing pydantic-core (2.27.2)
  - Installing pydantic (2.10.6)
  - Installing starlette (0.45.3)
  - Installing fastapi (0.115.7)

Writing lock file
```

>fastAPI를 설치하니 원래 설치 예정이었던 pydantic도 의존성에 추가되어 있고 무엇보다.. poetry가 의존성을 직접 관리해주니 너무 편하다..

<br>

## Hello world

위 처럼 설치가 모두 끝났으면 늘 하는 일 중 하나인 `hello world`를 해봐야지
공식 홈페이지에 있는 가이드를 따라 비어있는 프로젝트에 `main.py`를 작성해주자


```python
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: Union[str, None] = None):
    return {"item_id": item_id, "q": q}
```

그리고 내 zsh에 가상환경을 활성화 해주고! uvicorn 을 실행해보자

```
poetry shell
uvicorn main:app --reload
```

그러면 기본적으로 http://localhost:8000/ 에서 Hello world 를 확인할 수 있다!

![](/assets/img/post/2025-01-26_fastapi-docs.png)

기본적으로 fastAPI는 swagger 도 작성해주기 때문에 아주 편리한 것 같다.

<br>

## 프로젝트 구조 설계!

보통 spring-boot 나 vue-cli, next.js 등 기본적인 프로젝트의 스캐폴딩을 해주는 기능을 가진 framework들이 다양하지만 ( 보통 web 만을 위한 프레임워크라 그런가.. ) FastAPI의 경우 프로젝트 구조를 기본적으로 이렇게 해라~ 하는 가이드도 없고 다르게 말하면 아주 자유도와 활용도가 높은 framework여서 프로젝트 구조를 고민하게 했다.

물론 여러가지 서드파티로 프로젝트 구조를 잡을 수 있는데 아래 몇 가지 예시를 남겨두겠다.


---

<br>

### FastAPI Project Generator (`fastapi-mvc`)

**주요 기능**:
- 기본 디렉토리 구조 생성
- Docker 설정 생성
- 데이터베이스 설정 및 초기화
- 기본적인 API 엔드포인트 생성
```
pip install fastapi-mvc
fastapi-mvc new my-fastapi-project
```

### FastAPI-CLI
**주요 기능**:
- 프로젝트 초기 설정 및 구조 생성
- 기본적인 API, 모델, 스키마 생성
- Docker 설정 및 환경 파일 생성

```
pip install fastapi-cli
fastapi-cli create my-fastapi-project
```


### FastAPI Starter Kit
- **설명**: FastAPI Starter Kit은 기본적인 디렉토리 구조와 설정을 제공하는 템플릿 프로젝트입니다. CLI 도구는 아니지만, 이 레포지토리를 클론하여 프로젝트를 시작할 수 있습니다.
- **주요 기능**:
    - 설정이 완료된 FastAPI 프로젝트 구조 제공
    - 인증, 데이터베이스, 테스트 환경 포함

```
git clone https://github.com/zhanymkanov/fastapi-best-practices.git my-fastapi-project
```


---

하지만 위의 서드파티 기능들을 사용하기 보단 내가 익숙한 구조로 프로젝트 설계를 진행하기로 마음먹었고 아래가 그 뼈대이다!

<br>

### 내 프로젝트 구조 소개!


```
my_fastapi_project/
├── app/
│   ├── __init__.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py
│   │   └── security.py
│   ├── models/
│   │   ├── __init__.py
│   │   └── user.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   └── user.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── deps.py
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   └── user.py
│   ├── services/
│   │   ├── __init__.py
│   │   └── user_service.py
│   ├── db/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   └── session.py
│   ├── main.py
│   └── tests/
│       ├── __init__.py
│       ├── conftest.py
│       └── test_user.py
├── .env
├── .gitignore
├── alembic.ini
├── requirements.txt
└── README.md
```

#### **디렉토리 및 파일 설명**

- **`app/`**: 애플리케이션의 모든 코드가 들어있는 최상위 directory.
    
    - **`__init__.py`**: 이 디렉토리를 패키지로 인식하게 하는 파일.
        
    - **`core/`**: 핵심 설정 및 보안 관련 파일을 포함하는 directory.
        
        - **`config.py`**: 환경 설정 파일로, 환경 변수를 로드하고 설정을 관리.
        - **`security.py`**: 보안 관련 로직(JWT 토큰 생성, 패스워드 해싱 등)을 처리.
    - **`models/`**: 데이터베이스 모델을 정의하는 directory.
        
        - **`user.py`**: 사용자와 관련된 데이터베이스 모델을 정의. (예: SQLAlchemy 모델)
    - **`schemas/`**: 데이터 검증과 직렬화/역직렬화 작업을 위한 Pydantic 스키마가 포함.
        
        - **`user.py`**: 사용자와 관련된 Pydantic 모델을 정의.
    - **`api/`**: API 엔드포인트를 정의하는 directory.
        
        - **`deps.py`**: 의존성 주입을 위한 파일로, 공통적으로 사용되는 종속성들을 정의.
        - **`v1/`**: API 버전 1에 해당하는 엔드포인트들이 들어 있는 directory. (버전 관리 용이)
            - **`user.py`**: 사용자 관련 API 엔드포인트를 정의.
    - **`services/`**: 비즈니스 로직을 처리하는 서비스 계층. 데이터베이스 모델과 분리된 비즈니스 로직을 작성하여 코드의 재사용성을 높일 목적.
        
        - **`user_service.py`**: 사용자와 관련된 비즈니스 로직을 처리.
    - **`db/`**: 데이터베이스와 관련된 설정과 초기화 파일을 포함.
        
        - **`base.py`**: SQLAlchemy 베이스 클래스 정의.
        - **`session.py`**: 데이터베이스 세션 생성 및 관리.
    - **`main.py`**: FastAPI 애플리케이션의 진입점입니다. 여기서 FastAPI 인스턴스를 생성하고, 라우터를 포함한 다른 구성 요소들을 연결.
        
    - **`tests/`**: 테스트 코드가 포함된 directory.
        
        - **`conftest.py`**: 테스트를 위한 공통 설정 및 픽스처 정의.
        - **`test_user.py`**: 사용자 관련 API의 테스트 코드를 작성합니다.
- **`.env`**: 환경 변수를 저장하는 파일입니다. API 키나 데이터베이스 URL 등 민감한 정보를 포함.
    
- **`.gitignore`**
    
- **`alembic.ini`**: 데이터베이스 마이그레이션 도구인 Alembic의 설정 파일.


위와 같은 기본 구조는 기존에 익숙한 사용하던 model, controller 를 기반으로, 스키마와 db 를 위한 디렉토리를 추가한 구조다.
앞으로 얼마나 변화가 있을 지 모르겠지만 위 구조를 기반으로 수정을 해 나갈 목적이다!


### 파일 생성 명령어

이건 언젠가 내가 또 쓸 것 같기도하고 해서 남겨두는 내용이다.

```python
import os

# 프로젝트 이름
project_name = "my_fastapi_project"

# 디렉토리 구조
directories = [
    f"{project_name}/app",
    f"{project_name}/app/core",
    f"{project_name}/app/models",
    f"{project_name}/app/schemas",
    f"{project_name}/app/api",
    f"{project_name}/app/api/v1",
    f"{project_name}/app/services",
    f"{project_name}/app/db",
    f"{project_name}/app/tests",
]

# 파일 구조
files = [
    f"{project_name}/app/__init__.py",
    f"{project_name}/app/core/__init__.py",
    f"{project_name}/app/core/config.py",
    f"{project_name}/app/core/security.py",
    f"{project_name}/app/models/__init__.py",
    f"{project_name}/app/models/user.py",
    f"{project_name}/app/schemas/__init__.py",
    f"{project_name}/app/schemas/user.py",
    f"{project_name}/app/api/__init__.py",
    f"{project_name}/app/api/deps.py",
    f"{project_name}/app/api/v1/__init__.py",
    f"{project_name}/app/api/v1/user.py",
    f"{project_name}/app/services/__init__.py",
    f"{project_name}/app/services/user_service.py",
    f"{project_name}/app/db/__init__.py",
    f"{project_name}/app/db/base.py",
    f"{project_name}/app/db/session.py",
    f"{project_name}/app/main.py",
    f"{project_name}/app/tests/__init__.py",
    f"{project_name}/app/tests/conftest.py",
    f"{project_name}/app/tests/test_user.py",
    f"{project_name}/.env",
    f"{project_name}/.gitignore",
    f"{project_name}/alembic.ini",
]

# 디렉토리 생성
for directory in directories:
    os.makedirs(directory, exist_ok=True)

# 파일 생성
for file in files:
    with open(file, 'w') as f:
        pass

print(f"FastAPI 프로젝트 '{project_name}' 구조가 생성되었습니다!")

```

이렇게 프로젝트 구조만 생성해 두는 것 까지 오늘 진행했으니, 다음엔 lint 와 formmater를 적용해야겠다


---
## Ref

uvicorn 정보 참고 - https://jybaek.tistory.com/890  ( 해당 블로그에서 uvicorn github - loop setup 코드를 링크 걸어둔것 참조 )