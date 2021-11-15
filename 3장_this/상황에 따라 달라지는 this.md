## this의 개념

---

`this` 는 **'자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 참조 변수'**.

`this`를 통해 자신이 속한 객체 또는 자신이 생성할 인스턴스의 **프로퍼티나 메서드를 참조**할 수 있다.

<aside>
💡 **여기서 '또는' 이란 표현은 어떤 고정된 값에 바인딩되지 않기 때문!  `this`는 함수가 호출되는 방식에 따라 동적으로 결정된다!**

</aside>

- 예시
  할머니: 나는 즐겁다 (나 === 할머니)
  엄마: 나는 행복하다 ( 나 === 엄마)
  아빠: 나는 기분이 좋다 ( 나 === 아빠)
  **자바스크립트에서 this 란 문장들에서 나타난 '나'라는 개념과 비슷! 어떤 문맥이냐에 따라 그 의미(값)가 달라진다.**

## this의 이해

---

`this`의 값은 어떻게 변화할까? → 바인딩을 통해 `this` 가 어떤 값과 연결되는 지 확인할 수 있다.

- 바인딩?
  `this`의 호출 방식에 따라 `this`가 특정 **'객체'**에 연결되는 것

### 핵심 내용

1. **전역 공간에서의 this는 전역 객체와 바인딩된다.**
2. **메서드 내부의 this는 메서드를 호출한 객체와 바인딩된다.**
3. **일반 함수 내부의 this는 전역 객체와 바인딩된다.**
4. **생성자 함수 내부에서의 this는 생성자 함수가 생성할 인스턴스와 바인딩된다.**

## 1. 전역 공간에서의 this

---

전역 공간에서 this는 전역 객체를 가리킨다. **개념상 전역 컨텍스트를 생성하는 주체가 바로 전역 객체이기 때문!**

```jsx
console.log(this); // {alert: f(), atob: f(), ...}
console.log(window); // {alert: f(), atob: f(), ...}
console.log(global); // {alert: f(), atob: f(), ...}

console.log(this === global); // true
console.log(this === window); // true
```

\*_브라우저 환경에서는 `window` 이고 Node.js 환경에서는 `global`_

### 여기서 질문!

```jsx
// 1번 질문

var a = 1;

console.log(a); // (1)
console.log(window.a); // (2)
console.log(this.a); // (3)

// 2번 질문

let a = 1;

console.log(a); // (1)
console.log(window.a); // (2)
console.log(this.a); // (3)
```

- 1번 답
  (1), (2), (3) 모두 **1** 이 나온다.
  **그 이유는 자바스크립트의 모든 변수는 특정 객체의 프로퍼티로서 동작하기 때문!** → 여기서 특정 객체는 L.E(LexicalEnv)
  그래서 전역공간에서 `var`로 선언된 변수는 전역 객체의 프로퍼티로 인식되는 것!
- 2번 답
  (1) → 1
  (2),(3) → undefined
    <aside>
    💡 `**let`, `const`  두 개 모두  전역공간에서(최상위 스코프) 선언을 해도 전역 객체에 새로운 프로퍼티를 추가하지 않는다!**
    
    </aside>

대부분의 경우에서 `var` 로 선언한 변수가 `window`의 프로퍼티에 직접 할당하는 것과 거의 똑같다고 할 수 있지만 몇 가지의 예외 경우가 있다.

```jsx
var a = 1;
delete window.a; // false
console.log(a); // 1

window.b = 2;
delete window.b; // true
console.log(b); // Uncaught ReferenceError : b is not defined
```

- var로 선언한 전역변수와 전역객체의 프로퍼티는 **configurable 속성(변경 및 삭제 가능성)과 호이스팅 여부에서 차이가 난다.**

## 2. 메서드 내부에서의 this

---

this에는 호출한 주체에 대한 정보가 담긴다. **점 표기법이나 대괄호 표기법의 경우 마지막 점 앞에 명시된 객체가 곧 this!**

### 함수와 메서드의 구분법

- **독립성** → 메서드는 클래스 및 객체와 연관되어 있고 함수는 상관없이 독립적으로 존재
- 메서드는 클래스 내 선언되어 있는 함수
- 간단하게 **함수 앞에 점(.)이 있으면 메서드! 없으면 함수!**

### 메서드 내부에서의 this 예시

```jsx
let obj = {
  methodA: function () {
    console.log(this);
  },
  inner: {
    methodB: function () {
      console.log(this);
    },
  },
};

obj.methodA(); // { methodA: ƒ methodA(), inner: { methodB: ƒ methodB() }} === obj
obj["methodA"](); // { methodA: ƒ methodA(), inner: { methodB: ƒ methodB() }} === obj

obj.inner.methodB(); // { methodB: ƒ methodB() } === obj.inner
obj.inner["methodB"](); // { methodB: ƒ methodB() } === obj.inner
```

## 3. 함수 내부에서의 this

---

자바스크립트에서 함수를 호출하면, **해당 함수의 내부에서의 this는 전역 객체에 바인딩된다.**

<aside>
⚠️ 이건 명백한 설계상의 오류!

</aside>

### 그렇다면, 메서드의 내부함수에서의 this?

```jsx
global.value = 100;

const myObject = {
  value: 1,

  //메서드
  func1: function () {
    this.value += 1;
    console.log(this.value); //2

    //내부함수
    func2 = function () {
      this.value += 1;
      console.log(this.value); //101

      //내부함수
      func3 = function () {
        this.value += 1;
        console.log(this.value); //102
      };
      func3();
    };
    func2(); //함수는 점(.)이 없고!
  },
};

myObject.func1(); //메서드는 앞에 점(.)이 있고!
```

이렇게 출력되는 이유는 자바스크립트에서는 **내부 함수 호출 패턴을 정의해놓지 않기 때문!**
내부 함수도 결국 함수이므로, 이를 호출할 때는 함수 호출로 취급된다.

### 내부함수에서 this를 우회하는 방법

이런 한계를 극복하려면 this를 내부함수가 접근 가능한 **다른 변수에 저장**하면 된다.

```jsx
global.value = 100;

const myObject = {
  value: 1,
  func1: function () {
    //this, 즉 myObject를 that에 저장
    const self = this;
    this.value += 1;
    console.log(this.value); //2

    func2 = function () {
      //that으로 접근
      that.value += 1;
      console.log(self.value); //3

      func3 = function () {
        that.value += 1;
        console.log(self.value); //4
      };
      func3();
    };
    func2();
  },
};

myObject.func1();
```

그저 상위 스코프의 `this`를 저장해서 내부함수에서 활용하려는 수단으로 변수를 만드는 것!

\*_변수명은 대체로 `self`를 많이 쓰고 `that` , `_this` 정도를 쓰기도 한다._

또는 ES6에서 나온 **화살표 함수를 이용**할 수 있다.

```jsx
global.value = 100;

const myObject = {
  value: 1,
  func1: function () {
    this.value += 1;
    console.log(this.value); //2

    //화살표 함수 사용
    func2 = () => {
      this.value += 1;
      console.log(this.value); //3

      //화살표 함수 사용
      func3 = () => {
        this.value += 1;
        console.log(this.value); //4
      };
      func3();
    };
    func2();
  },
};

myObject.func1();
```

왜 화살표 함수는 상위 스코프의 `this`를 바라보는 것일까?

## 콜백 함수 호출 시 그 함수 내부에서의 this

---

콜백함수도 **함수**이기 때문에,어떤 객체의 메서드로 있던 함수가 인자로 전달될 경우에는 객체의 메서드가 아닌 **함수자체로 전달**된다.

콜백함수 내부에서의 this는 해당 콜백함수의 제어권을 넘겨받은 함수가 정의한 바에 따르며, **정의하지 않은 경우에는 전역객체를 참조한다.**

### 여기서 질문!

```jsx
let obj = {
  foo: function (a, b) {
    console.log(this, a, b);
  },
};

obj.foo(4, 5); // (1)
[10, 20, 30].forEach(obj.foo); // (2)
```

- 답
  **(1) : { foo: ƒ foo() } 4 5**
  - 점(.) 앞에 obj라는 객체가 있으니 foo함수는 obj의 메서드 foo의 자격으로 호출이 된 것
  - 따라서 this는 점(.) 앞의 객체인 obj를 가리키게 됨
  ***
  **(2) : 전역 객체를 바라봄**
  - `obj.foo`가 통째로 forEach 함수의 인자로 들어가 있다. 생긴건 점이 붙은 메서드의 모양으로 들어가 있지만, 메서드 그대로가 아닌 그냥 콜백함수의 역할을 하는 함수로서 있는 것 뿐! 결국엔 이런 식인 것!
  ```jsx
  [10, 20, 30].forEach(function (a, b) {
    console.log(this, a, b);
  });
  ```

## 생성자 함수 내부에서의 this

---

### 생성자 함수란?

new 연산자와 함께 어떤 공통된 성질을 지닌 객체들을 생성하는 함수

생성자 = 클래스(class) / 클래스를 통해 만든 객체를 인스턴스(instance)

(일반 함수에도 new를 붙여버리면 생성자 함수가 되기 때문에 생성자 함수 이름의 **첫 문자를 대문자**로 쓰기를 권하고 있다.)

```jsx
// 생성자
function User(name) {
  this.name = name;
  this.isAdmin = false;
}

// new 연산자를 사용한 함수 호출 -> user라는 인스턴스 생성
let user = new User("보라");

console.log(user.name); // 보라
console.log(user.isAdmin); // false
```

### **생성자 함수가 동작하는 방식**

1. 빈 객체 생성 및 `this` 바인딩
   - 생성자에 명시적으로 빈 객체를 생성하고, this로 바인딩.
   - 이 객체는 생성자의 prototype 객체를 자신의 프로토타입 객체로 설정한다.
     (`__proto__` 라는 프로퍼티나 메서드가 있는 객체를 만들어 this로 바인딩된 빈 객체에 부여)
2. `this`를 통한 프로퍼티 생성
   - 함수 코드 내부에서 this를 사용해서, 빈 객체에 동적으로 프로퍼티나 메서드를 생성할 수 있다.

```jsx
function User(name) {
  // this = {};  (빈 객체가 암시적으로 만들어짐)

  // 새로운 프로퍼티를 this에 추가함
  this.name = name;
  this.isAdmin = false;

  // return this;  (this가 암시적으로 반환됨)
  // return { name: "고릴라" };  // <-- this가 아닌 새로운 객체를 반환함
}
```

리턴문이 없을 경우, `**this`로 바인딩된 새로 생성한 객체가 리턴된다.\*\*
하지만 다른 객체를 반환하는 경우는 생성자 함수를 호출했다고 하더라도 `this`가 아닌 해당 객체가 리턴된다.
