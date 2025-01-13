---
title: "[Next.js] nextjs에 Tailwindcss 추가하기 ( with. prettier-plugin-tailwindcss )"
date: 2025-01-08 17:19:00 +0900
author: tyrosteller
categories: [Dev-React]
tags: [project-COOTTE, frontend, react, nextjs, tailwind, shadcnui]
---

## 이럴거면 처음부터 tailwind 추가 할 걸..

지난 번 next-create-app 을 진행하면서 vue의 vuetify와 유사해 보였던 chakra-ui를 쓰겠다고 마음먹고, tailwind를 추가하지 않았는데..
본격적으로 리서치를 해보니..  Next가 지향하는 방향과 최근의 추세를 보고 새로운 길을 찾기로 했다.
> 뭐 결론적으로는 tailwindCss를 다시 도입하기로 했다는 이야기
> 악 저거 class 겁나 많아서 가독성 지짜 별론데.. 익숙해지면 나아지겠지.. 부트스트랩처럼..

<br>

## shadcn UI

결론부터 말하자면 새로 도입 하기로 결정한 [shadcnUI](https://ui.shadcn.com/)! 
일단 공식 홈페이지 첫 화면의 직관적이고 심플 그 자체인 UI에 반해서 이리저리 찾아봤다 

![](/assets/img/post/2025-01-08_Shadcn_UI_home.png)

Shadcn UI는Tailwind CSS와 React의 생태계에서 많은 경험을 쌓은 개발자인 **Shadcn**이라는 개발자 님이 주도적으로 개발했으며, 기존 UI 라이브러리들의 제약을 해결하고자 만들어진 현대적 UI 라이브러리다!

이걸 도입하기 위해 일단 다시 tailwind를 추가하기로 했다, shadcnUI에 대해서는 다음에 다시 이야기해보자.

<br>

---


## Next 프로젝트에 tailwind 추가하기

next-create-app을 할 때 tailwind를 선택했으면 좋았겠지만, 그 땐 그게 정답이었다 
지금은 새로운 준비를 위해 이미 만들어진 next 프로젝트에 tailwind를 추가 할 것 이다. [next 공식문서](https://tailwindcss.com/docs/guides/nextjs) 에 가이드가 있기 때문에 큰 문제는 아니리라 생각한다

> 말 그대로 tailwind를 설치 하지 않은 next 프로젝트에 tailwind를 추가하는 내용

<br>

### `tailwindcss` `postcss` `autoprefixer` 설치하기

```shell
npm install -D tailwindcss postcss autoprefixer
```

<br>

tailwindcss는 당연히 설치해야 되는 거고 postcss도 이미 자주 사용해서 알고 있는데, autoprefixer는 무엇인가?

> [PostCSS](https://github.com/postcss/postcss) plugin to parse CSS and add vendor prefixes to CSS rules using values from [Can I Use](https://caniuse.com/). It is recommended by Google and used in Twitter and Alibaba.
{: .prompt-info }
위처럼 css 규칙에 vendor 접두사를 자동으로 붙여주는 기능을 가지고 있다고 한다.


```shell
npx tailwindcss init -p
```

설치가 완료되고 나면 위 명령어로 initialize를 진행하면 tailwind.config.js와 postcss.config.js 파일이 생성된다

```shell
Created Tailwind CSS config file: tailwind.config.js
Created PostCSS config file: postcss.config.js
```

<br>

### config 수정하기

일단 tailwind와 postcss 관련 config.js 파일이 만들어졌는데, 내 프로젝트는 ts를 쓰기 때문에 조금 수정을 해 줄 예정이다.

#### postcss.config.js 파일 수정
```js

// postcss.config.js
module.exports = {
	plugins: {
		tailwindcss: {},
		autoprefixer: {},
	},
}

// postcss.config.mjs 로 수정

/** @type {import('postcss-load-config').Config} */
const config = {
	plugins: {
		tailwindcss: {},
		autoprefixer: {},
	},
};

export default config;

```

<br>

#### tailwind.config.js 수정

```js

// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
	content: [],
	theme: {
		extend: {},
	},
	plugins: [],
}

// tailwind.config.ts 로 수정
import type { Config } from "tailwindcss";

export default {
	content: [
		"./src/pages/**/*.{js,ts,jsx,tsx,mdx}",
		"./src/components/**/*.{js,ts,jsx,tsx,mdx}",
		"./src/app/**/*.{js,ts,jsx,tsx,mdx}",
	],
	theme: {
		extend: {
			colors: {
				background: "var(--background)",
				foreground: "var(--foreground)",
			},
		},
	},
	plugins: [],
} satisfies Config;
```

next create를 ts, tailwind를 모두 사용하고 src 디렉토리도 사용한다고 선택했을 때 기본으로 나오는 구조를 참조했다.
<br>

#### global.css 수정

```css
/* global.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* ... */
```
global.css에 위 3개 라인을 추가해주면 tailwind 설치는 끝났다

#### tailwindcss를 위한 prettier 설정

마지막으로 prettier에 tailwind 관련 플러그인을 설치해, tailwind 클래스들도 formatting 해주도록 만들자
```bash
npm install -D prettier-plugin-tailwindcss
```

이후 prettierrc.json 파일에 방금 설치한 플러그인을 사용하도록 추가 해주자
```json
{
	//...
	"plugins": ["prettier-plugin-tailwindcss"],
}
```

> eslint에서 tailwindcss plugin을 도입하지 않은건, custom class를 아직까진 사용할 일이 잘 없고, 퍼블리싱까지 많이 손댈 예정은 아니라 굳이 따로 추가하지 않았다.

<br>

---
2025-01-14 추가

위 `prettierrc.json`을 사용하는 방법으로는 tailwindcss plugin이 동작하지 않는 상황이 생겼다.
[트러블슈팅](https://tyrosteller.github.io/posts/prettier-plugin-tailwindcss-not-working) 포스팅에 남겨두었던 내용이지만 아래 파일형식으로 변경을 해주었더니 정상 동작했으니, 같은 문제를 겪는 다면 참고하시라!


```js
// prettier.config.mjs

/** @type {import("prettier").Config} */
const config = {
	semi: true,
	singleQuote: true,
	trailingComma: 'es5',
	tabWidth: 2,
	useTabs: false,
	printWidth: 80,
	jsxBracketSameLine: false,
	arrowParens: 'always',
	endOfLine: 'lf',
	plugins: ['prettier-plugin-tailwindcss'],
	tailwindConfig: './tailwind.config.ts', // add this
	overrides: [
		{
			files: '*.json',
			options: {
				tabWidth: 2,
			},
		},
	],
};

export default config;
```

---


이렇게 tailwind 설치 까지 끝났으니, 다음 이야기는 shadcn UI 적용기가 되시겠다.
