---
layout: post
title: 암호걸린 급여명세서를  파싱해서 자동입력하기
date:   2025-02-14
last_modified_at: 2025-02-20
category: [Toy Project]
tags: [Javascript, Hono, html, GoogleAPI]
---
# Overview
## Purpose
자산 관리를 하면서 매달 들어오는 Income을 정리하기 위해 매달 급여명세서를 수기로 Google SpreadSheet에 입력하였으나, 휴먼에러로 인해 연말정산 계산이 틀어짐. 따라서 급여명세서를 파싱하여 자동으로 Google SpreadSheet에 저장하는 것으로 휴먼에러를 방지하기 위함.

<br/><br/>

## Key Features
1. 매달 `html`파일로 오는 급여명세서를 `POST`요청으로 서버에 송신하여 `json`파일로 return
2. return 받은 `json`은 Google SpreadSheet에 저장하여 활용

<br/><br/>

## Technology Stack
* Backend
	* Javascript
	* Hono
* Deploy
	* Cloudflare Workers

<br/><br/><br/>


# Details
## Hono를 선택한 이유
처음에는 익숙하고 레퍼런스가 많은 `Express.js`를 사용하려 했으나 Cloudflare Workers에서 `Express.js`를 지원하지 않아 부득이 V8 엔진에서도 돌아가는 `Hono`를 이용하여 서비스를 만들었다. 

~~~ js
// index.js
import { Hono } from 'hono';
import { cors } from 'hono/cors';
import { payroll } from './payroll/payroll.js';

const app = new Hono();

// CORS 미들웨어 설정
app.use('*', cors());

// 루트 경로
app.get('/', async (c) => {
  return c.json({ message: 'Hello World' });
});

// 급여 경로
app.post('/payroll', async (c) => {
  const response = await payroll(c.req, c.env);
  return response;
});

// 404 처리
app.notFound((c) => {
  return c.json({ message: 'Not Found' }, 404);
});

// 에러 처리
app.onError((err, c) => {
  console.error(`${err}`);
  return c.json({ message: err.message }, 500);
});

export default app;
~~~

소스를 보면 알겠지만 `Express.js`와 크게 다르지 않아 간단하게 구현이 가능했다.

<br/><br/>
## 1. 급여명세서 구조 파악
먼저 이번 달 급여명세서를 IDE로 열어본 결과 다음과 같은 구조를 가지고 있었다.
~~~html
<html>
 ...
 <script>
  ...
  function ViewPayPaper() {
	try {
		var cliperText;
		var plainText;
		var pwd;
		...
		// 비밀번호 확인 로직
        ...
		var bin = unescape(cliperText).split(',');
		var text = '';
		var strKey = pwd;
		for (var i = 0; i < bin.length; i++) {
			text = text + 
				String.fromCharCode(Number(bin[i]) + 
				strKey.charCodeAt(i % strKey.length));
		}
		document.write(text);
	} catch (err) {
		alert(err.description);
		return;
	}
  }
  ...
 </script>
 ...
    <form>
    ...
        <input type="hidden" name="_viewData"
			value="3,52,68,68,3441,2,-6442,....."
        </input>
    </form>
 ...
</html>
~~~
`ViewPayPaper()`에서 비밀번호(생년월일)을 확인하고, 비밀번호가 맞으면 해당 비밀번호를 기준으로 최하단 `_viewData` input태그의 value값을 파싱해서 `document.write()`하고 있다. 즉, `급여명세서.html`파일은 비밀번호를 치지 않아도 이미 급여명세서의 내용을 html파일로 가지고 있었다. 실제로 Chrome DevTools를 켜놓고 비밀번호를 입력했을 때, 별도의 네트워크 통신 없이 월급명세서 화면을 띄울 수 있다. 

<br/><br/>

## 2. 파싱로직 구현
급여명세서는 암호화된 급여명세서를 비밀번호(생년월일)을 이용해 복호화하기만 하면, 원하는 내용을 추출가능하므로 다음과 같은 순서로 파싱로직을 구현해야 한다. 
1. `급여명세서.html`파일에서 `_viewData` input태그의 value 값 추출
2. value 값을  `ViewPayPaper()`로 복호화해 `평문html`을 추출
3. 2 번의 `평문html`파일에서 급여명세에 해당하는 부분을 추출
상세 구현은 다음과 같다.

<br/>
### 2-1. input 태그의 value 값 추출
먼저 정규식을 사용하여 `_viewData` input태그의 value값을 추출하여 return한다.
~~~ js
function extractViewData(htmlContent) {
	const viewDataMatch = htmlContent.match(/<input[^>]*name=['"]_viewData['"][^>]*value=['"]([^'"]*)['"]/i);
	if (!viewDataMatch?.[1]) {
		throw new Error('_viewData를 찾을 수 없습니다.');
	}
	return viewDataMatch[1];
}
~~~

<br/>
### 2-2. 실제 급여명세서 html 추출
그 다음 추출한 `암호화 html`파일을 `ViewPayPaper()`로직에 따라 복호화한다.
~~~ js
function decodeHtml(viewData) {
	const bin = unescape(viewData).split(',');
	const strKey = `${생년월일}`; // 급여명세서 조회시  입력하는 비밀번호(생년월일)
	return bin.reduce((decodedHtml, value, index) => {
		return decodedHtml + String.fromCharCode(
		Number(value) + strKey.charCodeAt(index % strKey.length));
	}, '');
}
~~~
for문을 reduce method로 바꾸긴 했지만 동일하게 동작하는 코드다.

<br/>
### 2-3. HTMLRewriter를 사용해 원하는 부분의 데이터를 추출한다.
** 필자의 경우 모든 값이 table로 구성되어 있어 필요한 부분의 table 매핑 정보를 Chrome을 통해 묵시로 확인하여 해당 위치의 값을 가져오도록 구현하였다.**

~~~js
async function parseHtml(decodedHtml) {
	let tempKeys = [];
	let tempValues = [];
	let currentTable = 0;
	let currentRow = 0;

// 테이블과 행 매핑 정의, 원하는 급여명세 내역이 몇 번째 table에 있는지 직접  찾아야 한다.
const tableMapping = {
	지급내역: { table: 5, keyRow: 7, valueRow: 8 },
	공제내역1: { table: 5, keyRow: 16, valueRow: 17 },
	공제내역2: { table: 5, keyRow: 18, valueRow: 19 },
	총액정보: { table: 7, keyRow: 25, valueRow: 26 },
};

const categories = ['지급내역', '공제내역', '합계'];
const rewriter = new HTMLRewriter()
.on('table', {
	element(element) {
		currentTable++;
	},
})
.on('tr', {
	element(element) {
		currentRow++;
	},
})
.on('th, td', {
	text(text) {
		const content = text.text.trim();
		if (!content || categories.includes(content)) return;
		// 각 섹션별 데이터 수집
		Object.values(tableMapping)
		.forEach(({ table, keyRow, valueRow }) => {
			if (currentTable === table) {
				if (currentRow === keyRow) {
					tempKeys.push(content);
				} else if (currentRow === valueRow) {
					tempValues.push(content.replace(/,/g, ''));
				}
			}
		});
	},
});

const response = new Response(decodedHtml, {
	headers: { 'Content-Type': 'text/html' },
});

await rewriter.transform(response).text();
const result = {};
tempKeys.forEach((key, index) => {
	if (tempValues[index] && key) {
		result[key] = tempValues[index];
	}
});

return result;

}
~~~

<br/><br/>
## 3. 파싱로직 테스트
postman으로 body에 `급여명세서.html`를 binary로  첨부하여 `POST`요청하고 return 값을 살펴본다.
<br/>

<br/>
![payroll_parsing](/assets/images/payroll_parsing.png)

정상적으로 파싱된 급여명세가 json으로 return 된 것을 확인 할 수 있다.

<br/><br/>
## 4. SpreadSheet로 저장
![payroll_parsing_spreadsheet](/assets/images/payroll_parsing_spreadsheet.png)
이 부분이 별도의 글로 적을만큼 상당히 긴 내용이기에, 이 글에 적기에는 주제에서 벗어나는 듯 하여 상세내용은 생략하고, 간단히 설명하자면 다음과 같은 작업이 필요하다.
1. GCP(Google Cloud Platform)에 프로젝트 생성
2. SpreadSheet에 해당 프로젝트 연동
3. 프로젝트에 OAuth 설정
4. GCP 프로젝트의 Key값 다운로드
5. 해당 키값을 ArrayBuffer로 변환
6. Web Crypto API로 서명(SHA-256)해서 최종 JWT 액세스 토큰 생성
7. 6에서 생성한 토큰으로 SpreadSheet에 접근 (Bearer ${accessToken})
8. 원하는 Sheet에 급여명세 추가

추후 이 부분에 대한 상세 내용은 별도 POST로 작성할 예정이다.

<br/><br/>

## 향후 개발과제
사실 가장 하고 싶은 것은 급여명세서가 메일로 오면, 자동으로 이 첨부파일을 다운받아서, POST 요청을 보내는 것이긴 한데, Apple 단축어에 `특정 메일이 왔을 때 작동하는 트리거`는 존재하지만, `메일에서 첨부파일을 다운`받는 액션은 존재하지 않아 완벽한 자동화는 아직 달성하지 못했다.
<br/>
어쩔 수 없이 급여명세서가 오면 `파일을 요청하고, 입력받은 파일을 body에 실어서 POST 요청 보내는 액션`을 실행시키도록 해서 수동으로 파일을 첨부하도록 하고 있지만, 추후 이 부분을 `AppleScript` 등으로 개발하거나 별도의 다른 코드를 작성해 완전자동화 하려고 한다. IMAP을 통해 메일을 주기적으로 읽어와서, 급여메일이 있는 경우 자동으로 POST요청하도록 하면 되지 않을까

<hr>
<br/><br/><br/>
## 240220 추가
GMAIL API를 사용해서 서버에서 매일 한 번씩 송신자가  월급명세서가 발송되는 email 주소인 메일을 IMAP으로 읽은 다음, 상기 로직을 활용해 자동으로 첨부파일을 파싱해서 Google SpreadSheet에 저장하도록 만들었다. 조금 난관에 부딪혔던 부분은 사용자 서비스 계정으로 Auth가 끝나는 SpreadSheet API와 다르게 별도의 OAuth2 계정을 만들어서, 인증한다는 점이였다. 즉, 소스 작성시 구글 인증관련 소스를 하나 더  작성해야하는 점. <br/> 귀찮긴 했지만 인증부분을 제외하면 API자체는 상당히 직관적이고 메일 찾는 쿼리도 상당히 알아보기 쉬우니 메일연동한다면 gmail이 가장 나을 듯 싶다. 게다가 필자는  `Cloudflare Workers`에 올려놓고 사용할 예정이라 OAuth2 부분을 전부 구현했지만, 보통의 Node.js 환경이라면 관련 package를 install하는 것 만으로 간단히 끝나니 강추.

