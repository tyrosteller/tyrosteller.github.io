---
title: "[Next.js] Shadcn UI 적용하기!"
date: 2025-01-09 17:19:00 +0900
author: tyrosteller
categories: [Dev-React]
tags: [project-COOTTE, frontend, react, nextjs, tailwind, shadcnui]
---

## shadcn UI?

결론부터 말하자면 새로 도입 하기로 결정한 [shadcnUI](https://ui.shadcn.com/)! 

![](/assets/img/post/2025-01-08_Shadcn_UI_home.png)

이전에도 설명했지만 Shadcn UI는Tailwind CSS와 React의 생태계에서 많은 경험을 쌓은 개발자인 **Shadcn**이라는 개발자 님이 주도적으로 개발했으며, 기존 UI 라이브러리들의 제약을 해결하고자 만들어진 현대적 UI 라이브러리다!

   

## why shadcn UI?

* Next가 권장하는 tailwind에 친화적인 UI 킷 
	* 그 중 하나를 고른 것
	* 모든 컴포넌트가 Tailwind CSS를 기반으로 스타일링되어, Tailwind 유틸리티 클래스를 자유롭게 활용할 수 있다는 장점이 있다.
* Tailwind를 좀 다룰 수 있다면 편하게 스타일링이 가능하다
	* 반대로 말하면 필요 시 shadcn이 아닌 내가 만든 tailwind로 만든 컴포넌트와 통합이 쉽다는 것
* 필요한 컴포넌트를 설치해서 사용하는 방식이다.
	* vuetify나 MUI가 불필요한 컴포넌트들을 모두 설치해야되서 무거운 편인 것에 비해, 내가 원하는 컴포넌트만 설치해서 사용할 수 있는게 뭐랄까 새로운건 아닌데 새롭게 다가왔다.
	* 컴포넌트를 설치하면 로컬에 복사되어서, 필요하다면 컴포넌트 코드를 직접 수정할 수 있다
* radix ui 기반으로 설계되어 키보드 네비게이션이나 스크린 리더 지원 등 접근성 표준을 충족한다는 장점도 있음!

물론 신생에 가까운 라이브러리라 생태계가 크진 않다고 하지만, 그래서 그런지 빠르게 변화하는 Next@15에 대한 빠른 대응과 monorepo에 대한 공식문서가 있는 것 또한 맘에들었다.
~~monorepo는 아직 시도해본 적 없지만..~~

> 물론 도입을 위한 준비가 좀 더 많고, 특히나 컴포넌트에 업데이트가 있을 때 마다 수동으로 변경해줘야 한다는 점은 좀 걱정이지만, 어차피 다른 애들도 업데이트 하면 깨지는건 비슷하지 않은가!
{: .prompt-info }

<br>

그리고 가장 끌렸던 내용은 사실, **v0** 사용성이다!
내가 원하는건 예쁜 화면 UI를 가진 프로젝트를 빨리 만드는 것 인데, 디자인은 기본적으로 감각이 부족하고.. ux를 생각하는 퍼블리싱까지 (완벽하게) 공부하려면 커브가 너무 크기 때문에 v0를 활용을 종종 하고 있었다.

shadcn ui는 공식문서에서도 [open in v0](https://ui.shadcn.com/docs/v0) 로 쉽게 커스텀도 가능하다고 하니 끌릴 수 밖에..! 

아 참 먼저 디자인에 관한 설명은 [shadcn 디자인 블로그](https://www.shadcndesign.com/blog/difference-between-default-and-new-york-style-in-shadcn-ui)에서 찾았다, `new york` 과 `default` 타입이 있고 색상도 `zinc`라는 팔레트가 있다는데 자세한건 블로그를 참고하자

   
---

   

## How to shadcn UI?

Next@15에 React 19로 프로젝트를 시작한 나는 일단 [공식문서](https://ui.shadcn.com/docs/react-19#upgrade-status) 를 찬찬히 살펴보기로 했다.
일단 title 자체가 `Next.js15 + React 19` 지만 React 19로 업그레이드를 하는 모두를 위한 글 이었다
> Next 15 업그레이드를하는 사람들을 위해 타이틀을 이렇게 만들었다고 한다

<br> 

>패키지매니저로 pnpm, yarn, bun을 사용하는 경우엔 install 시 **별다른 flag를 필요로 하지 않는다** 하지만 npm을 사용한다면 peer deependency issue를 해결하기 위한 옵션을 선택해야한다. ( `--force || --legacy-peer-deps` )
,물론 component를 추가할 때도 동일하다! 
{: .prompt-warning }

### Install Shadcn UI


```bash
npx shadcn@latest init
```

> 원한다면 -d 옵션을 사용해서 default 옵션으로 구성할 수 있다
> default style 은 `new york` 이고 default base color는 `Zinc`로 설정된다고 한다.



```bash
✔ Preflight checks.
✔ Verifying framework. Found Next.js.
✔ Validating Tailwind CSS.
✔ Validating import alias.
✔ Which style would you like to use? › Default
✔ Which color would you like to use as the base color? › Zinc
✔ Would you like to use CSS variables for theming? … no / yes
✔ Writing components.json.
✔ Checking registry.
✔ Updating tailwind.config.ts
✔ Updating src/app/globals.css
  Installing dependencies.

It looks like you are using React 19.
Some packages may fail to install due to peer dependency issues in npm (see https://ui.shadcn.com/react-19).

✔ How would you like to proceed? › Use --force
✔ Installing dependencies.
✔ Created 1 file:
  - src/lib/utils.ts

Success! Project initialization completed.
You may now add components.
```

#### **select style**

두 가지 스타일 중 하나를 선택할 수 있다, `default` 와 `new york` 위에서 안내한 shadcn 디자인 페이지에서 각 특징을 확인할 수 있다
뭐 new york 쪽이 좀더 패딩이 적고 눈에 띄는 폰트와 두께감 같은 것들을 기본적으로 설정해 준다고 하는데
확실히 조금 더 예쁘긴 하지만 내눈엔 그렇게 큰 차이는 못 느껴서 ( 미적감각 제로 ) default로 선택했다


#### **base color**

base color는 tailwindCss에서 제공하는 [customization colors](https://tailwindcss.com/docs/customizing-colors) 중 5가지를 제공하는 듯 하다. ( Slate, Gray, Zinc, Neutral, Stone )
원하는 어떤 것을 선택해도 좋다! 이 때 선택한 색상은 custom이 불가능 한 내용도 아니라, 단순히 방금 선택한 색상의 layer base를 `global.css`에 추가해주는 내용으로 보인다


#### **option for peer dependency issues clear**

아마 프로젝트를 진행하면서 deprecate 된 모듈을 업데이트하는 일을 하다보면 종종 만나는 issue고 그 해결법으로 --force, --legacy-peer-deps 를 찾게 될 텐데 어쨌든 지금은 react19 도 rc가 아닌 latest가 올라온 상태이니 --force를 사용하기로 한다.

<br>

---

### **추가 셋팅**

이렇게 설치를 완료하고 나면, 프로젝트에 몇 가지 변경점이 생긴걸 확인 할 수 있다.

#### tailwind.config.ts  수정

일단 tailwind eslint를 나와 같이 설정했던 사람이라면 require를 사용하는 방식 때문에 lint error가 발생한 것을 볼 수 있다.
간단하게 수정을 해두자.

```js
import type { Config } from "tailwindcss";
import tailwindcssAnimate from "tailwindcss-animate"; // import tailwindcss-animate

export default {
	darkMode: ["class"],
	content: [
		"./src/pages/**/*.{js,ts,jsx,tsx,mdx}",
		"./src/components/**/*.{js,ts,jsx,tsx,mdx}",
		"./src/app/**/*.{js,ts,jsx,tsx,mdx}",
	],
	
	//...
	// plugins: [require("tailwindcss-animate")], 
	plugins: [tailwindcssAnimate],
```


#### color customize

물론 black & white 의 UI도 예쁘지만 base color를 좀 눈에 띄는 색상으로 변경하고 싶어서,
[shadcn/themes](https://ui.shadcn.com/themes) 에 가서 원하는 커스터마이징을 진행한 후, copy code를 진행해 `global.css`를 변경해주기로 한다

![](2025-01-09_Shadcn_Ui_custom.png)

보라색이 마음에들어서 `Copy code` 버튼을 눌러 코드를 복사해서 `global.css`에 적용해 주었다.

<br>

---

<Br>

## Use case - Components 사용법 그리고 Building Blocks

shadcn UI의 components들은 설치할 때 한번에 모두 추가되는게 아니라, 개발자가 사용할 것 들만 add해서 local에 복사된 컴포넌트를 사용하는 것
일단 가장 심플한 button component를 추가해서 정상적으로 동작하는지 사용해보자!

### Components add

```bash
npx shadcn@latest add button
```

button components를 add 하고 나면 `src/app/components/ui` 디렉토리가 생기고 `button.tsx` 파일이 생성 된다


```tsx
import { Button } from "@/components/ui/button"

export function ButtonDemo() {
  return <Button>Button</Button>
}
```
위는 공식홈페이지에서 제공하는 use case 다
import 하는 경로만 봐도 알 수 있듯, lib에서 가져오는게 아닌 local에 방금 add하면서 생성 된 파일을 import해서 사용하게 된다.


### Blocks

나와 같이 빠른 스타트를 위해 템플릿을 쓰고 싶은 사람을 위한 [building blocks](https://ui.shadcn.com/blocks) 도 제공한다!
( 개인적으로 참 마음에 드는 부분이다. )

제공하는 마음에 드는 login form 블록을 찾아서 적용해 보았다.

```
npx shadcn@latest add login-04
```

어떻게 Next.js에서 app-router를 사용하는걸 인식했는진 모르겠지만, `src/app/login/page.tsx` 가 생성되어 별다른 노력 없이 `localhost:port/login` 에서 확인 할 수 있었다!

![](2025-01-09_Shadcn_UI_block.png)
> 이미지는 따로 넣지 않아서 깨지지만..

이제 부턴 간단하게 내 서비스 라고 하긴 애매하고 나의 웹을 만들어보자.


