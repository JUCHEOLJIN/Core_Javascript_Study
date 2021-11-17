# 명시적으로 this를 바인딩하는 방법

## this를 바인딩 해야하는 이유
기본적으로 this는 상황에 따라서 바인딩이 된다. 하지만, 규칙과 상관 없이 별도의 대상을 바인딩하는 방법도 있다. 필요한 이유는 아래와 같은 예시가 있다.
```jsx
class People {
  constructor() {
    this.name = '철진';
    this.age = 20;
  }

  printName  ()  {
    console.log(this.name);
  };

  onPop(callback) {
    callback();
  }
}

const people = new People();

people.onPop(people.printName);

// 어떻게 될까요?
// TypeError: Cannot read property 'name' of undefined
```

## call / apply 메서드

```jsx
Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])

Function.prototype.apply(thisArg[, argsArray])
```
call / apply 메서드는 메서드의 인자를 하나씩 전달하는지, 배열로 전달하는 지의 차이만 있다.
바인딩 하는 예시는 아래와 같다.

```jsx
// call

let func = function(a, b, c){
	console.log(this, a, b, c);
};

func(1,2,3); 
func.call({x: 1}, 4, 5, 6); // {x: 1} 4 5 6

// apply

func.apply({x:1}, [4,5,6]); // {x: 1} 4 5 6
```

## call / apply 메서드의 활용
### 유사배열객체에 배열 메서드를 적용
**키가 0 또는 양의 정수인 프로퍼티가 존재하고 length 프로퍼티의 값이 0 또는 양의 정수인 객체,** 즉 유사배열객체의 경우에는 call 또는 apply 를 통해서 배열 메서드를 차용할 수 있다.
- 유사배열객체
```jsx
let obj = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3
};
Array.prototype.push.call(obj, 'd');
console.log(obj); // {0: 'a', 1: 'b', 2: 'c', 3: 'd', length: 4}

let arr = Array.prototype.slice.call(obj);
console.log(arr); // ['a', 'b', 'c', 'd']

```

활용법의 예시를 이해하기 편하게 작성하면 이렇게 된다.

```jsx
func.apply(obj, [args...])
1. func : 빌려올 메서드
2. apply : 알죠?
3. obj : 활용할 객체
4. [args...] : 인자
```

- arguments
arguments 도 유사배열객체이기 때문에 활용할 수 있다.
```jsx
function a () {
  let args = Array.prototype.slice.call(arguments);
  args.forEach(function(arg){
    console.log(arg);
  });
}
a(1, 2, 3); // 1 2 3
```
arguments의 경우에 활용 방안을 하나 제시해보자면, 인자의 수를 정확히 알 수 없는 경우가 있다. 

```jsx
function checkNum() {
    const array = Array.prototype.slice.call(arguments); // 배열로 변형
    const check = array.every((value) => {
        return typeof value === "number"
    });
    console.log(check); // false
}
 
const numbers = checkNum(11, 22, 99, "66");
```

- NodeList
```jsx
document.body.innerHTML = '<div>a</div><div>b</div><div>c</div>';
let nodeList = document.querySelectorAll('div');
let nodeArr = Array.prototype.slice.call(nodeList);
nodeArr.forEach(function (node) {
  console.log(node); 
  // <div>a</div>
  // <div>b</div>
  // <div>c</div>
});
```

- 문자열
문자열의 경우에도 `length`와 `index` 를 가지고 있기 때문에 `call`과 `apply`를 활용하여 배열 메서드를 사용할 수 있다. 
다만, `length` 의 경우에는 읽기전용속성이기 때문에 이를 변경해야하는 `pop`, `push` 등의 메서드는 에러가 발생한다.

문자열의 경우에는 참조형인 배열 객체와는 다르게 불변값이기 때문에 길이가 변할 수 없다. 

```jsx
let str = 'cheol jin';

Array.prototype.push.call(str, ', pushed string');
// Error: Cannot assign to read only property 'length' of object [object String]

Array.prototype.concat.call(str, 'string'); // [String {"cheol jin"}, "string" ]
// 원본에 변경을 가하지 않는 경우
Array.prototype.every.call(str, function(char){return char !== ' ';}); // false

Array.prototype.some.call(str, function(char){return char === ' ';}); // true

let newArr = Array.prototype.map.call(str, function(char) { return char + '!';});

console.log(newArr); // ["c!", "h!", "e!", "o!", "l!", " !", "j!", "i!", "n!"]

let newStr = Array.prototype.reduce.apply(str, [ function(string, char, i){return string + char + i; }, '']);
console.log(newStr); // c0h1e2o3l4 5j6i7n8
```

- Array.from

위의 방법들의 경우에 본래의 목적과는 다소 동떨어진 활용이기 때문에 ES6에서 새롭게 `Array.from` 을 도입했다.
```jsx
let obj = {
	0: "a",
	1: "b",
	2: "c",
	length: 3,
};
let arr = Array.from(obj);
console.log(arr); // ["a", "b", "c"]
```

### 생성자 내부에서 다른 생성자를 호출

생성자에 공통된 내용이 있을 경우 `call` 또는 `apply` 를 이용해 다른 생성자를 호출하면 간단하게 반복을 줄일 수 있다.

```jsx
function Person(name, gender){
  this.name = name;
  this.gender = gender;
}
function Student(name, gender, school){
  Person.call(this, name, gender);
  this.school = school;
}
function Employee(name, gender, company){
  Person.apply(this, [name, gender]);
  this.company = company;
}
let jy = new Student("주영", "male", "전남대");
let cj = new Employee("철진", "male", "구글");
```

### 여러 인수를 묶어 하나의 배열로 전달하고 싶을 때

여러 개의 인수를 받는 메서드에게 하나의 배열로 인수들을 전달하고 싶을 때 `apply` 메서드를 이용하면 좋다. 최대 / 최솟값을 구해야 하는 경우를 보자.

```jsx
// 직접 구현하는 코드
let numbers = [10, 20, 3, 16, 45];
let max = min = numbers[0];
numbers.forEach(function(number){
  if(number > max){
    max = number;
  }
  if(number < min){
    min = number;
  }
});
console.log(max, min); // 45, 3

// apply 활용하는 코드
let numbers = [10, 20, 3, 16, 45];
let max = Math.max.apply(null, numbers);
let min = Math.min.apply(null, numbers); 

// 펼치기 연산자 사용
const numbers = [10, 20, 3, 16, 45];
const max = Math.max(...numbers);
const min = Math.min(...numbers);
console.log(mx, min); // 45 3
```

## bind 메서드
`bind` 메서드는 ES5에서 추가된 기능으로, `call`과 비슷하지만 즉시 호출하지는 않고 넘겨 받은 `this` 및 인수들을 바탕으로 새로운 함수를 반환하기만 하는 메서드이다.
```jsx
let func = function(name, age){
  console.log(this, name, age);
};
func("cheoljin", 20); // window{...} "cheoljin" 20 
// this를 지정
let bindFunc1 = func.bind("kind people");
bindFunc1("cheoljin", 20); // {"kind people"} "cheoljin" 20
// this를 지정하고 부분 적용 함수를 구현
let bindFunc2 = func.bind("kind people", "cheoljin"); 
bindFunc2("20"); // {"kind people"} "cheoljin" 20
```

### name 프로퍼티
`bind` 메서드를 사용해서 만든 함수는 독특한 성질이 있다. 바로 `name` 프로퍼티에 동사 bind의 수동태인 bound라는 접두어가 붙는다는 점이다. 이를 통해 `call` 이나 `apply`에 비해 추적하기 좋아졌다.

```jsx
console.log(func.name); // func
console.log(bindFunc1.name); // bound func
```
## 화살표 함수
화살표 함수의 경우에는 함수 내부에 `this` 자체가 없다. 그렇기 때문에 `this` 에 접근하려고 하는 경우에 스코프 체인상 가장 가까운 `this` 에 접근하게 된다.
```jsx

handleViewCount = () => {
    this.setState({ isClickedView: !this.state.isClickedView });
  };

  getViewCount = id => {
    this.setState({ view: Number(id), isClickedView: false });
  };

  getPageOption = (value, setTarget) => {
    this.setState({ [setTarget]: value });
  };

  handleLike = product => {
    const newList = this.state.list.map(item => {
      if (product.id === item.id) {
        return { ...item, isLiked: !item.isLiked };
      } else {
        return item;
      }
    });
    this.setState({ list: newList });
  };
```
