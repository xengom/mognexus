---
layout: post
title:  Javascript Array Method 정리
date:   2025-02-16
last_modified_at: 2025-02-16
category: [TIL-JS]
tags: [Javascript array]
---
<br/>
# 사전지식 
## 성긴 배열(Sparse array)
반복문에 대해 설명하기 전에 먼저 성긴 배열(Sparse array) 개념에 대한 이해가 필요하다. 성긴 배열이란 일반적인 인덱스가 연속적인 배열(Dense array)과 다르게 **인덱스가 연속적이지 않은 배열**이다. 즉, 성긴 배열의 length는 요소 개수 보다 크며 주로 Array() 생성자를 사용하거나 현재 배열의 length보다 큰 인덱스에 요소를 할당하면 만들어진다.
~~~js
let a = new Array(5); // 요소는 없지만 a.length는 5
a = [];
a[1000] = 0; // a 배열의 1000번 째 인덱스에 0을 추가하여 length는 1001이 됨.
~~~
<br/>
## thisValue
후술하겠지만 함수를 호출하는 대부분의 Array Method(`find`, `filter`, `map` 등)는 thisValue라는 매개변수를 옵션으로 받을 수 있다. 기본적으로 thisValue는 메서드의 콜백함수의 this를 의미한다.
~~~ js
let army = {
	minAge: 18,
	maxAge: 27,
	canJoin(user) {
		return user.age >= this.minAge && user.age < this.maxAge;
	}
};

let users = [
	{age: 16},
	{age: 20},
	{age: 23},
	{age: 30}
];

// army.canJoin 호출 시 참을 반환해주는 user를 찾음
let soldiers = users.filter(army.canJoin, army);
alert(soldiers); // [{age: 20}, {age: 23}]
~~~
thisValue에 army를 지정하지 않고 단순히 `users.filter(army.canJoin)`을 사용했다면 `army.canJoin`은 단독 함수처럼 취급되고, 함수 본문 내 this는 undefined가 되어 에러가 발생한다.

<br/><br/>
# Array Methods
## forEach()
`forEach(function(currentValue, index, arr), thisValue)`
<br/>
forEach()메서드는 배열을 순회하며 각 요소에서 함수를 호출한다. 
currentValue, index, arr를 인자로 전달해 함수를 호출하는데 배열 요소의 값에만 관심이 있다면 인자 하나만 받는 함수를 작성하고 나머지 값을 무시해도 된다.
~~~ js
let a = [1,2,3,4,5], sum = 0;

a.forEach(e => { sum += a;}); // sum == 15

a.forEach((v,i,a) { a[i] = v + 1; }); // a == [2,3,4,5,6]
~~~
forEach()에서 **모든 요소를 함수에 전달하기 전에 반복을 멈추는 방법은 존재하지 않다.** 따라서 일반적인 for 루프에서 사용하는 break 문 등은 사용할 수 없다.
또한 for/of 와 달리 성긴 배열을 인식하고 존재하지 않는 요소에 대해서는 호출하지 않는 특징을 가진다.
<br/><br/>
### for 문과의 차이
#### 일반 for, for/of 문
~~~ js
// for 문
for(let i = 0; len = letters.length; i++){
	//로직
}

// for/of 문
for(let [index, letter] of letters.entries()) {
	//로직
}
~~~
배열이 빽빽하며 모든 요소에 유효한 데이터가 들어있다(Dense Array)고 가정하기 때문에 성긴 배열에서 정의되지 않았거나 존재하지 않는 요소를 건너뛰려면 다음과 같은 로직이 추가되야 한다.

~~~js
for(let i = 0; i < a.length; i++) {
	if (a[i] === undefined) continue; // 유효하지 않은 요소 skip
}
~~~
<br/>
#### forEach() 메서드
~~~js 
letters.forEach(letter => {
	//로직
})
~~~
반면 forEach() 메서드는 상술한 대로 존재하지 않는 요소를 스킵한다. 단, 명시적으로 undefined를 할당하는 경우에는 함수를 호출한다. 

~~~ js
let a = [1,2,3];
delete a[1]; // [1, ,3]
a.forEach(n => console.log(n)); // 1, 3 출력

let b = [1,2,3];
b[1] = undefined;
b.forEach(n => console.log(n)); // 1, undefined, 3 출력
~~~

<br/><br/>
## map()
`map(function(currentValue, index, arr), thisValue)`
<br/>
각 배열 요소를 함수에 전달에 호출하며, 그 함수가 반환한 값으로 이루어진 **배열을 반환한다.** 
forEach()에 전달하는 함수와 같은 방법으로 호출되지만 map()메서드는 값을 반환해야 한다.
map()은 **새 배열을 반환하며 기존 배열은 수정하지 않는다.** 성긴 배열이라면 존재하지 않는 요소에 대해서는 함수를 호출하지 않지만, 반환된 배열 역시 같은 위치에 갭이 있으며 length도 같다.
~~~ js
let a =[1,4,9];
a.map(Math.sqrt); // [1,2,3]

// 주로 배열 속 객체를 재구성하는데 사용한다. (경험)
let testArr = [
	{ key: 1, value: 10 },
	{ key: 2, value: 20 },
	{ key: 3, value: 30 }
];
const resultArr = testArr.map((obj) => {
	let rst = {};
	rst[obj.key] = obj.value;
	return rst;
}); // [{1:10}, {2:20}, {3:30}]
~~~

<br/><br/>
## filter()
`filter(function(currentValue, index, arr), thisValue)`
<br/>
filter() 메서드는 기존 배열의 일부분만 포함하는 부분 집합을 반환한다. 전달하는 함수를 기준으로 하며 이 함수는 true 또는 false를 반환한다. 

~~~ js
let a = [5,4,3,2,1];
a.filter(x => x < 3); // [2,1] 값이 3 미만인 경우
a.filter((x, i) => i%2 === 0); // [5,3,1] 인덱스가 짝수인 경우
~~~

마찬가지로 성긴 배열에서 존재하지 않는 값은 건너뛰며 반환하는 배열은 항상 빽빽한 배열이다.

~~~ js
// 성긴 배열에서 갭을 제거하는 방법
let dense = sparse.filter(() => ture);
~~~

<br/><br/>
## find(), findIndex()
`find(function(currentValue, index, arr), thisValue)`
<br/>
`findIndex(function(currentValue, index, arr), thisValue)`
<br/>
find()와 findIndex() 메서드는 filter() 메서드와 비슷하지만, 기준을 만족하는 첫 번째 요소를 찾는 즉시 순회를 종료한다. 순회가 종료되면 find()는 해당 요소를, findIndex()는 요소의 인덱스를 반환하며 만족하는 요소를 찾지 못하면 각각 undefined와 -1을 반환한다.

~~~ js
let a = [1,2,3,4,5];
a.findIndex(x => x === 3); // 인덱스 값을 return하므로 2 출력
a.find(x => x % 5 === 0); // 5
~~~

<br/><br/>
## every(), some()
`every(function(currentValue, index, arr), thisValue)`
<br/>
`some(function(currentValue, index, arr), thisValue)`
<br/>
every()와 some() 메서드는 배열 요소에 판별 함수를 적용하고 결과에 따라 true/false를 반환한다. every()는 배열의 모든 요소에 대해 true일 경우에만 true를 반환하며 some()은 배열 요소 중, 판별 함수가 true를 반환하는 것이 하나라도 있으면 true를 반환한다.

두 메서드 모두 자신이 어떤 값을 반환할지 확실해지는 순간 순회를 멈춘다. 
만일 빈 배열에 호출한 경우 every()는 true를, some()은 false를 반환한다.

<br/><br/>
## reduce(), reduceRight()
`reduce(function(acc, curVal, curIdx, arr), initialValue`
* acc: accumulator(누산기)로, 콜백의 반환값을 누적한다. 
* curVal: 처리할 현재 요소
* curIdx: 처리할 현재 요소의 인덱스로 initialValue를 제공하면 0, 아니면 1부터 시작
* arr: reduce()를 호출한 배열
* initialValue: 콜백 최초 호출 시 acc에 제공하는 값이며, 초기값이 없는 경우 배열의 첫 번째 요소를 initialValue로 사용한다.

reduce() 메서드는 제공하는 함수를 사용해 배열 요소를 값 하나로 만든다.
~~~ js
let a = [1,2,3,4,5];
a.reduce((x,y) => x+y, 0); // 15
a.reduce((x,y) => x*y, 1); // 120
a.reduce((x,y) => (x>y) ? x : y); // 5 
~~~

reduceRight() 메서드는 reduce()와 동일하나 오른쪽에서 왼쪽으로 진행한다.
~~~ js
let a = [2,3,4];
a.reduceRight((acc, val) => Math.pow(val,acc)) // 2^(3^4)

/**
 *  initialValue가 없으므로 가장 오른쪽 값인 4가 acc, 3이 val에 들어감
 *  > Math.pow(3, 4)이므로 3^4가 새로운 acc값이 됨
 *  > Math.pow(2, 3^4)이므로 2^(3^4)가 최종 값으로 출력됨
 */
~~~


