# 실행컨텍스트 & Variable Environment

```
2장 실행 컨텍스트
2-1 실행 컨텍스트란?
2-2 VariableEnvironment
p36~41
```

## 실행 컨텍스트란?

1. **사전 지식**

 - 스택 
    - 출입구가 하나 뿐인 우물 같은 데이터 구조
    - first in last out 
    
  - 큐 
    - 양쪽이 모두 열려있는 파이프 같은 데이터 구조
    - 보통 한 쪽은 입력, 다른 한 쪽은 출력을 담당하는 구조
    - first in first out 

2. **실행 컨텍스트**
- 개념
    - 실행할 코드에 제공할 환경 정보들을 모아 놓은 객체
    - 각 생성된 실행 컨텍스트는 스택에 쌓이게 된다.

3. **예시**
- `var` 을 사용한 경우

```jsx
// 전역 컨텍스트 실행
var a = 1;
function 바깥() {
	function 안(){
		console.log(a); //  
		var a = 3;
  }
	안(); 
	console.log(a);	//
}
바깥();
console.log(a); //

// 결과값을 예상해  보세요.
```

- `let`을 사용한 경우

```jsx
// 전역 컨텍스트 실행
let a = 1;
function 바깥() {
	function 안(){
		console.log(a); // 
		let a = 3;
  }
	안(); 
	console.log(a);	//
}
바깥();
console.log(a); //

// 결과값을 예상해  보세요. 
```

## Variable Environment

현재 context 내의 식별자들에 대한 정보 및 외부 환경 정보
