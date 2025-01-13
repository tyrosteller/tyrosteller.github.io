---
title: "[Trouble-Shooting] vscode 에서 prettier-plugin-tailwind not working 이슈 해결하기"
date: 2025-01-14 00:32:00 +0900
author: tyrosteller
categories: [Dev-Trouble-Shooting]
tags: [nextjs, tailwind, prettier, prettier-plugin-tailwindcss]
---

## Issue

바로 며칠 전 포스팅에서 설정했던 prettier 셋팅 중, prettier-plugin-tailwindcss가 적용되지 않고 있다는걸 이제야 눈치챘다.

무조건 알파벳 순서로 정렬되는게 아니라는걸 알고 있어서 그런가 방심했는데, 같은 className을 복사 붙여넣기 하다가 하나를 수정했는데 어째서인지 변화가 없는것!
vscode의 prettier 포멧터 자체는 정상적으로 동작하고 있었는데, 해당 plugin만 문제가 있는 상황!

분명 문제가 있다면 vscode의 동작 결과를 보여주는 output 탭에서 에러를 열심히 뱉고 있겠지 라고 생각하고 열어봤으나..


![](/assets/img/post/2025-01-14_prettier_issue_check.png)

음.. error가 없다..? completed.. 

결국 늘 하던대로 google에 검색하기 시작했고, nextjs를 쓰면서 prettier 특정 버전을 사용했을 때 나와 동일한 현상이 생긴다는 여러가지 글을 보았다.
글들을 토대로 몇가지 테스트를 진행하며 일단 해결책은 찾았으니, 내가 해결한 방법 부터 적어두고 과정을 추가로 남긴다. ( 역시 글은 두괄식.. )

<br>

## How to fix it?


>결론은 나는 기존에 `prettierrc.json` 파일을 설정파일로 사용하고 있었는데, 해당 파일을 `prettier.config.mjs` 로 변환해주고서 동작해보니 해결되었다!
{: .prompt-tip }


### 기존 `prettierrc.json`

```json
{
	"semi": true,
	"singleQuote": true,
	"trailingComma": "es5",
	"tabWidth": 2,
	"useTabs": false,
	"printWidth": 80,
	"jsxBracketSameLine": false,
	"arrowParens": "always",
	"endOfLine": "lf",
	"plugins": ["prettier-plugin-tailwindcss"],
	"overrides": [
		{
			"files": "*.json",
			"options": {
				"tabWidth": 2
			}
		}
	]
}

```

### 수정한 `prettier.config.mjs`

```js
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

tailwind config의 위치를 못 잡나 싶어서 tailwind.config.ts의 위치도 지정해줬다.
해당 방식은 [prettier.io 의 config 설정](https://prettier.io/docs/en/configuration.html)에서 참고해 작성했다.

일단 이렇게 하고나니 


![](/assets/img/post/2025-01-14_prettier_issue_clear.png)

위 처럼 깔끔하게 tailwind class가 sort 되는 것을 확인할 수 있다!


## 뭔가 서순이지만 과정은?

일단 Prettier 자체는 동작하기 때문에 vscode setting 관련된 내용을 제외하고 해결책으로 보였던 몇 가지를 나열해보자면 

1. (2023년 중순 기준) prettier 3 버전을 사용할 때, `prettier-plugin-tailwindcss` 의 0.4 버전에서 이슈 발생 / prettier 2.x 버전으로 다운그레이드 진행하기
2. `prettierrc` 파일을 `prettierrc.json` 으로 정확하게 확장자 입력해주기
3. `"prettier-plugin-tailwindcss"` 를 plugin 가장 마지막에 로드하기

이렇게 3개 였던 것 같다.

<br>
<br>


**downgrade**

```json
"prettier": "^3.4.2",
"prettier-plugin-tailwindcss": "^0.6.9",
```

>prettier가 3.대인건 맞지만, plugin은 이미 0.6 버전을 쓰고 있기도 했고, 시작부터 해답으로 다운그레이드를 선택하기엔  prettier를 2.x 버전으로 내린다는건 조금 나에 대해 무책임한 선택 같은 느낌이 들어 마지막으로 미뤘기 때문에 실험해보지 않았다.
{: .prompt-error }

<br>

**prettierrc.json 확장자 지정**
prettierrc 파일을 확장자 없이 사용하고 있지 않아서 딱히 생각은 안했지만, 혹시나 확장자 없이 사용하는 프로젝트의 git을 clone받았다면 추후에 꼭 확인해보자

<br>

`**prettier-plugin-tailwindcss` 를 plugin 가장 마지막에 로드하기**

이것도 지금은 플러그인을  `prettier-plugin-tailwindcss` 단 하나만 쓰고 있어서 문제될 부분은 아니었지만, 많은 사람들이 해당 문제를 겪는 듯 하다
추후에 같은 문제를 발견하면 되돌아 볼 수 있도록 남겨두자

```json
"plugins": ["prettier-plugin-tailwindcss", "prettier-plugin-something~~"]

"plugins": ["prettier-plugin-something~~", "prettier-plugin-tailwindcss"]

```

후순위로 로드된 무언가에 의해 동작하지 않게 될 수 있다고한다 ( eslint 설정 당시에도 비슷한 글을 봤었지.. )


아무튼 간단한 issue 였지만 나중에 혹시나 하고 찾을 때를 위해 posting!

---

* 이슈 참고
	https://github.com/tailwindlabs/prettier-plugin-tailwindcss/issues/186
	https://github.com/tailwindlabs/prettier-plugin-tailwindcss/issues/191
	https://blueprint-12.tistory.com/423
	https://stackoverflow.com/questions/75628944/prettier-tailwind-plugin-isnt-working-as-expected-when-i-hit-save-in-vscode


