---
title: "[Issue] vscode fastapi could not be resolved ( poetry )"
date: 2025-01-26 20:32:00 +0900
author: tyrosteller
categories: [Dev-Python]
tags: [project-COOTTE, vscode, python, poetry, virtualenv]
---

## Issue!

poetry를 사용해 virtual env 를 설정하고 vscode에서 코드를 작성하려고 하는데, pylence가 갑자기 import 에러를 반환한다

![](/assets/img/post/2025-01-26_vscode-import-error.png)

아주 간단한 hello world 코드인데, fastapi를 import 하려고하니 error가 발생한다.
일단 문제는 vscode가 인식하고 있는 interpreter가 poetry의 가상환경을 바라보고 있지 않아서, poetry 가상 환경에서 설치한 fastapi를 찾지 못한다는 것 
> 혹시 저 pylence problem이 한글로 나오는데, 이유를 아시거나 영어로 변경하는 방법을 아신다면 누가 알려주면 좋겠..

해결 방법은 두 가지

<br>

## 프로젝트 내부에서 poetry virtual env를 만들도록 설정

```shell
poetry config virtualenvs.in-project true
poetry config virtualenvs.path "myproject/.venv"
```

위 명령어를 사용해 프로젝트 내부에 `.venv` 로 가상환경을 설정하게 하면 vscode에서 사용할 수 있다 

<br>

## vscode에서 poetry 가상환경 위치로 interpreter 설정

vsCode 에서 `Cmd + Shift + P` 를 눌러 `Python: select interpreter` 메뉴를 선택하면,

![](/assets/img/post/2025-01-26_vscode-select-interpreter.png)

내가 사용할 수 있는 파이썬 위치가 보인다 ( 왜 버전이 두개인지는 나중에 확인해봐야겠다 )
아무튼 저 메뉴에서 global python이 아닌 poetry 가상환경을 쓸 수 있게 경로를 입력해주면 된다.


```
poetry env info
```
명령어를 사용해 virtual env의 정보를 확인해보면

```
Virtualenv
Python:         3.12.1
Implementation: CPython
Path:           /Users/myname/Library/Caches/pypoetry/virtualenvs/cootte-api-3vJMG5Bj-py3.12
Executable:     /Users/myname/Library/Caches/pypoetry/virtualenvs/cootte-api-3vJMG5Bj-py3.12/bin/python
Valid:          True

Base
Platform:   darwin
OS:         posix
Python:     3.12.1
Path:       /Library/Frameworks/Python.framework/Versions/3.12
Executable: /Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12
```

이렇게 path를 확인할 수 있다.  
`Enter interpreter path` 를 선택 후 path를 복사해서 입력해주면 완료!


---

* issue Ref
	https://stackoverflow.com/questions/59882884/vscode-doesnt-show-poetry-virtualenvs-in-select-interpreter-option
	https://github.com/microsoft/vscode-python/issues/1871


