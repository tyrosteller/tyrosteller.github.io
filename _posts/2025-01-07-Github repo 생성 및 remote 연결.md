---
title: "[Github] git repository 생성, remote 연결 하기 ( with. ssh-keygen )"
date: 2025-01-07 10:51:00 +0900
author: tyrosteller
categories: [Dev-Git, Github]
tags: [Infra, git, github, repository, ssh-key]
---

## 기록은 기초부터

언제나 개발을 시작할 때 함께 했던 툴 Git, 정규직으로 일할 땐 별 생각 없이 GitLab을 통해 repository를 관리하고 CI/CD를 구성하곤 했는데
이번 개인 프로젝트를 진행하면서 Github의 장점들을 여럿 보게되어 Github으로 repo 관리를 시작해보려 한다



## **SSH key 생성 및 Github 에 SSH 등록 하기**

지난번 프로젝트를 종료하면서.. macbook을 포멧을 하는 절차를 거쳤는데.. ssh키를 생각안하고 있었다
새로 ssh key도 생성하고 github에 추가로 등록해보자

### **1. SSH 공개키 생성**

일단 macOS ( linux도 동일 ) 기준으로, openSSH 를 기본적으로 포함하기 때문에 따로 설치할 것 없이 명령어로 ssh키 생성이 가능하다.


```shell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

* -t 
	* 생성할 SSH 키의 알고리즘(타입)을 지정한다, 우리는 가장 널리 쓰이는 rsa 공개키 알고리즘을 사용할 것
* -b
	* 생성하는 키의 비트 길이를 지정. 키가 길수록 보안이 강화되지만, 처리 속도는 느려질 수 있다고 한다.
	* 기본이 3072인가 그렇다는데 4096으로 설정하면 보안에 좋다하니 써보자!
* -C
	* ssh 키에 comment를 추가할 수 있다
	* github 같은 걸 쓸 때, 누구의 key인지 식별하기 쉬워진다하니 내 git 계정을 넣어줄 생각


위 명령어를 입력하면 아래와 같은 프롬프트가 진행된다

```
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/tyros/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

key를 저장할 file 명은 default 이름인 id_rsa를 쓰기위해 아무것도 입력하지 않고 넘어갔다.
물론 비밀번호도 어차피 개인 컴퓨터를 사용하고 ssh키를 노출할 일은 없어야하고 없을테니까 따로 설정하지 않았다 ( 그냥 아무것도 입력하지 않고 enter를 치면 암호 없이 진행 )
> 비밀번호를 지정해두면.. 나중에 git 에서 동작을 할 때 비밀번호 계속 쳐야되서 귀찮았다.. 물론 ssh-agent 같은걸 쓴다면 설정해도 좋다
{: .prompt-info }

```
Your identification has been saved in /Users/tyros/.ssh/id_rsa
Your public key has been saved in /Users/tyros/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:~~~~~~~~~~ tyros.dev@gmail.com
```

그럼 이제 .ssh 디렉토리에 **id_rsa** 와 **id_rsa.pub** 파일이 생성 되었을 것 이다.

<br>   

### **2. Github에 ssh key 등록**

이제 github에 settings 메뉴에서 [SSH and GPG keys](https://github.com/settings/keys) 를 찾아가서 [New SSH key](https://github.com/settings/ssh/new) 버튼을 눌러주자

![](/assets/img/post/2025-01-07_Git-ssh_setting.png)

**Title** 은 내가 ssh키를 생성한 기기를 식별할 수 있도록 적당한 이름을 넣어주자 나는 소중한 맥북프로를 적어줬다
그리고 아까 만든 SSH키를 복사해서 **Key** 항목에 입력해주면 된다.


```shell
pbcopy < ~/.ssh/id_rsa.pub
```

위 명령어를 사용하면 간단하게 복사 할 수 있다~
그리고 Add SSH key 를 눌러 주면 완료!

<br>   

## **Repository 생성부터 remote 연결 까지 시작해보자**
---

### 1. Github 에서 new repository 생성하기

일단 가장 먼저 github 에서 [new repository](https://github.com/new) 를 생성해주자


![](/assets/img/post/2025-01-07_Git-create_new_repo.png)

원하는 **repository name** 을 적고, 원한다면 간단한 설명을 **Description**에 추가한다. 
> 나는 기존에 만든 next project의 이름을 그대로 적고, 말 그대로 이거저거 시도해 볼 playground 겸 backoffice가 될 곳이라 간단하게 설명을 적었다

딱히 기업이나 다른 보안이 필요한 내용이 들어갈 곳은 아니라 **Public** 으로 설정했고, 굳이 **Read.me** 는 만들지 않았다.
그리고 **.gitignore** 도 이미 작성해 둔 파일이 있어서 굳이 추가하지 않았다



![](/assets/img/post/2025-01-07_Git-repo-guide.png)

Create repository를 하고나니 따로 설명 덧 붙일 것도 없이 가이드를 바로 보여 준다.

<br>

### 2. Github repository 에 remote 연결하기

일단 먼저 내 프로젝트에서 `.git` 디렉토리가 있는지 확인하자, 나는 next 로 프로젝트를 시작해서 .git이 자동으로 생성되어 있었다.
만약 `.git`이 없는 경우 `git init` 명령어를 통해 이니셜라이즈 해주자
> 연결 된 remote repository가 없고, 새 repository에 연결한다는 가정하에 따라가자
{: .prompt-info }

<br>

#### 2-1. main으로 사용할 branch 지정

아마 따로 설정하지 않으면 메인으로 사용 할 브랜치 이름이 `main` 일 것 이다. 원한다면 그대로 사용해도 무방하나 나는 이름을 바꾸기로 했다.


```
git branch -m production
```
위 명령어를 사용하여 현재 브랜치 이름을 `production`으로 수정 해줬다.

<br>

#### 2-2. remote add

이제 진짜 git repository에 연결해보자 명령어 한 줄 이면 된다.

```shell
git remote add origin git@github.com:username/repository-name.git
```

따로 성공 메시지는 나오지 않으니 에러가 없다면 성공 했다고 생각하자.

<br>

#### 2-3. remote branch로 push 하기

```shell
git push -u origin user-branch-name
```
본인이 설정한 branch로 push를 진행하자! 

> 참고로 -u 옵션은 `--set-upstream` 의 단축어다
> 공식 문서에 보면 잘 나와있다.
>  
> -u (--set-upstream)
> For every branch that is up to date or successfully pushed, add upstream (tracking) reference, used by argument-less [git-pull[1]](https://git-scm.com/docs/git-pull) and other commands. For more information, see `branch.<name>.merge` in [git-config[1]](https://git-scm.com/docs/git-config).
>{: .prompt-info }

```
The authenticity of host 'github.com (**.***.***.***)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvv***********/*******
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

나는 한번도 ssh로 github을 연결 한 적이 없는 상태기 때문에 finger print가 없는게 정상! 


```
To github.com:tyrosteller/cootte-backoffice.git
 * [new branch]      production -> production
branch 'production' set up to track 'origin/production'.
```

이렇게 origin remote 에 new branch가 잘 만들어졌다면 끝!



