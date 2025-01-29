---
title: "[postgreSQL] macOS postgreSQL 설치하기"
date: 2025-01-20 22:38:00 +0900
author: tyrosteller
categories: [Dev-PostgreSQL]
tags: [project-COOTTE, postgreSQL, database]
---

## 서론

일단 현재까지 Next.js 와 shadcn UI를 사용해 적당한 로그인폼과 dashboard를 그려뒀다.
물론 앞으로 jest를 사용한 간단한 unit test나 zustand를 사용한 상태관리를 진행하겠지만 그러려면 일단 API 서버가 하나 필요하고 ( 물론 mock-up 을 해서 대충 json만 받아와도 가능하겠지만 ), 그 API 서버를 위해 database가 필요하다

> 이걸 어디에 배포해서 테스트할지는 일단 뭐라도 돌아가게 만들고 생각하자

   
일단 나는 python 의 fast api 프레임워크를 도입할 예정이고 postgreSQL을 데이터베이스로 선택했다! 

   
## PostgreSQL 설치

[postgreSQL 공식 홈페이지](https://www.postgresql.org/download/) 에서 각자 OS에 맞는 버전을 설치해 주면 된다.
지금 나는 m1 이전 세대의 맥북을 사용해서 개발을 진행하고 있으니, 그냥 brew를 사용해서 설치가 가능하다.

```shell
brew search postgresql

brew install postgresql
brew install postgresql@16 
```

당연하게도 version 명시하지 않고 install 하면 latest stable 버전을 설치해줄 것이라 생각했다.
`postgres -V` 명령어로 버전을 확인한 결과

`postgres (PostgreSQL) 14.13 (Homebrew)`

14.13 버전이 설치되었다. 흠.. 2021년 게시글에서 다운받은것도 14.7버전인데 3년간 해당 버전이 스테이블일까.. 공식 홈페이지는 16까지 릴리즈 되어있었는데.. 
뭔가 찝찝해서 결국 설치한 postgresql을 지우고 latest 16버전을 재 설치하기로 한다.

   

### 환경변수 설정

![](/assets/img/post/2025-01-20_postgresql_cmd_error.png)

처음 시도했던 버전 명시를 하지않은 postgres의 경우 alias가 기본적으로 postgres로 설정되어 명령어가 동작했지만 
16버전을 명시하여 설치하니 명령어를 찾지 못 해서 `command not found: postgres` 또는 `command not found: postgresql@16` 등 에러가 발생한다

대신 postgresql을 설치하고 나면 아래와 같이 가이드를 명시해준다...

   

```
This formula has created a default database cluster with:
  initdb --locale=C -E UTF-8 /usr/local/var/postgresql@16
For more details, read:
  https://www.postgresql.org/docs/16/app-initdb.html

postgresql@16 is keg-only, which means it was not symlinked into /usr/local,
because this is an alternate version of another formula.

If you need to have postgresql@16 first in your PATH, run:
  echo 'export PATH="/usr/local/opt/postgresql@16/bin:$PATH"' >> ~/.zshrc

For compilers to find postgresql@16 you may need to set:
  export LDFLAGS="-L/usr/local/opt/postgresql@16/lib"
  export CPPFLAGS="-I/usr/local/opt/postgresql@16/include"

To start postgresql@16 now and restart at login:
  brew services start postgresql@16
Or, if you don't want/need a background service you can just run:
  LC_ALL="C" /usr/local/opt/postgresql@16/bin/postgres -D /usr/local/var/postgresql@16
```

>*흔히들 install 후에 화면에 나오는 글자들을 그냥 지나치는 경우가 많다. 꼭 한번은 읽어보자 나처럼 고생하지말고..*
>{:. prompt-tip}


위 가이드에 나온대로 명령어를 복사해 입력해주자

```shell
echo 'export PATH="/usr/local/opt/postgresql@16/bin:$PATH"' >> ~/.zshrc
export LDFLAGS="-L/usr/local/opt/postgresql@16/lib"
export CPPFLAGS="-I/usr/local/opt/postgresql@16/include"

source ~/.zshrc
```

이후 버전을 확인해보면 16.4버전이 설치된 걸 확인할 수 있다.
```
postgre -V
postgres (PostgreSQL) 16.4 (Homebrew)
```

   

### 계정 설정

일단 먼저 db 서버를 실행 시켜주자.

```
brew services start postgresql@16

==> Successfully started `postgresql@16` (label: homebrew.mxcl.postgresql@16)
```
위 명령어를 사용해 일단 brew에서 service를 시작해준다!

   

기본적으로 postgresql은 설치 시, 계정을 자동으로 생성해준다.

```
 psql postgres
 postgres=# \du
```
psql postgres로 콘솔에 접근 후, \du 명령어로 role 목록을 확인할 수 있다 ( # <- 슈퍼 유저 )


![](2025-01-23_postgresql_users.png)
>신기하게도 role name이 사용자명으로 설정되어있다, 어째서 한글 표기로 나오는지는 아직 모르겠지만..

일단 슈퍼유저가 아닌 개발자를 위한 계정 하나를 생성하고, 데이터베이스 추가 권한정도는 부여하자

```postgresql
CREATE USER cootte PASSWORD 'rPfksrPfks2@';
ALTER USER cootte createdb;
```

이제 위 생성한 계정으로 접속해서 새 데이터베이스를 만들어주자 테트를 위한 프로토타입 데이터베이스를!

```postgresql
CREATE DATABASE prototype_m1;
GRANT ALL PRIVILEGES ON DATABASE prototype_m1 TO cootte;
```



아래 계정 설정을 위한 postgre sql grant 명령어 확인을 위한 사이트를 참고해서 이후 계정 추가 시 좀더 세부적인 권한 부여를 해 볼 예정!

	 https://www.postgresql.org/docs/13/sql-grant.html - Grant 명령어 확인

   

### connecting

일단 무료 database tool 인 Dbeaver를 사용해서 Test connection을 진행해보자


![](/assets/img/post/2025-01-20_dbeaver_connect.png)

성공!