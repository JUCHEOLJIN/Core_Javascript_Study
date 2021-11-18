# 프로토타입

자바스크립트는 **'프로토타입 기반 언어!'**.

어떤 객체를 **원형(프로토타입)으로 삼고 이를 복제(참조)함으로써** 상속과 비슷한 효과를 얻는다

> 💡 클래스 기반 언어에서는 '상속'을 사용함

<br>

---

## 프로토타입의 이해

### 1. constructor, prototype, instance

```jsx
let instance = new Constructor();
```

- **생성자 함수 내 prototype 구조 흐름**
  - 어떤 생성자 함수(Constructor)를 new 연산자와 함께 호출하면
  - Constructor에서 정의된 내용을 바탕으로 새로운 인스턴스가 생성된다.
  - 이때 instance에는 `__proto__` 라는 프로퍼티가 자동을 부여되는데
  - 이 프로퍼티는 **Constructor의 prototype라는 프로퍼티를 참조**한다.

> 💡 prototype와 이를 참조하는 `__proto__`는 모두 **객체**이다.
> prototype 객체 내부에는 인스턴스가 사용할 메서드를 저장한다.

Person이라는 생성자 함수의 prototype에 getName이라는 메서드를 지정했다.

```jsx
let Person = function (name) {
  this._name = name;
};
Person.prototype.getName = function () {
  return this._name;
};
```

아래와 같이 Person의 인스턴스는 proto 프로퍼티로 getName을 호출할 수 있다.

```jsx
let suzi = new Person("Suzi");
suzi.__proto__.getName(); // undefined

Person.prototype === suzi.__proto__; // true
```

- **왜 undefined가 출력될까?**
  `undefined`가 출력되는 이유는 함수를 메서드로서 호출하면 바로 앞의 객체가 곧 `this`가 되기 때문이다. `__proto__` 객체에는 name 프로터티가 없기 때문에 `undefined`가 반환된다.

### 여기서 주목할 점! → '에러가 발생하지 않았다!'

> 어떤 변수를 실행해서 `undefined`가 나왔다는 것은 '호출할 수 있는 함수' 에 해당.
> 그러므로 getName은 함수가 맞다.

```jsx
let suzi = new Person("Suzi");
suzi.__proto__._name = "SUZI_proto";
suzi.__proto__.getName(); // 'SUZI_proto'
```

**그렇다면 `__proto__`가 아닌 `this` 를 인스턴스로 사용하고 싶다면?**
**`__proto__`를 생략하면 된다.** 원래부터 생략 가능하도록 정의되어있다.
이런 점 때문에 **생성자 함수의 prototype에 어떤 메서드나 프로퍼티가 있다면 인스턴스에서도 마치 자신의 것처럼 해당 메서드나 프로퍼티에 접근할 수 있게 된다.**

```jsx
let suzi = new Person("Suzi");
suzi.getName(); // Suzi
// = suzi(.__proto__).getName();
```

> 💡 new연산자로 생성자를 호출하면 인스턴스가 만들어지는데, **이 인스턴스의 생략 가능한 프로퍼티인 `__proto__` 는 생성자의 `prototype`을 참조한다!**

<br>

### 2. `__proto__` **vs prototype 프로퍼티**

모든 객체는 자신의 프로토타입 객체를 가리키는 `__proto__`(=[[prototype]])를 가지며 **상속**을 위해 사용된다.

함수도 객체이므로 `__proto__`를 갖는다.
**그런데 함수 객체는 일반 객체와는 달리 prototype 프로퍼티도 소유하게 된다.**

주의해야 할 것은 prototype 프로퍼티는 프로토타입 객체를 가리키는 `__proto__` 는 다르다는 것!

```jsx
function Person(name) {
  this.name = name;
}

const foo = new Person("Lee");

console.dir(Person); // prototype 프로퍼티가 있다.
console.dir(foo); // prototype 프로퍼티가 없다.
```

- `**__proto__**`

  - 함수를 포함한 모든 객체가 가지고 있다.
  - 객체의 입장에서 자신의 부모 역할을 하는 프로토타입 객체를 가리키며 함수 객체의 경우 Function.prototype를 가리킨다.

  ```jsx
  console.log(Person.__proto__ === Function.prototype); // true
  ```

- **prototype 프로퍼티**
  - 함수 객체만 가지고 있는 프로퍼티이다.
  - 함수 객체가 생성자로 사용될 때 이 함수를 통해 생성될 객체의 부모 역할을 하는 객체(프로토타입 객체)를 가리킨다.
  ```jsx
  console.log(Person.prototype === foo.__proto__); // true
  ```

그런 측면에서 내장 생성자 함수인 Array를 살펴보면

```jsx
let arr = [1, 2]; // (==  new Array(1,2);)
arr.forEach(function () {});
Array.isArray(arr);
arr.isArray(); //Error
```

prototype프로퍼티 내부에 있지 않은 `from`, `isArray`등의 메소드들을 인스턴스가 직접 호출 할수 없기 때문에 **Array 생성자 함수에서 직접 접근해야한다.**

### 3. Constructor 프로퍼티

생성자 함수의 프로퍼티인 `prototype` 내부에는 `constructor`라는 프로퍼티가 있다. 인스턴스의 `__proto__` 객체에도 마찬가지이다. 원래의 **생성자 함수(자기 자신)을 참조**하는데, **인스턴스로부터 그 원형을 알 수 있는 수단이기 때문!**

```jsx
function Person(name) {
  this.name = name;
}

const foo = new Person("Lee");

// Person() 생성자 함수에 의해 생성된 객체를 생성한 객체는 Person() 생성자 함수이다.
console.log(Person.prototype.constructor === Person);

// foo 객체를 생성한 객체는 Person() 생성자 함수이다.
console.log(foo.constructor === Person);

// Person() 생성자 함수를 생성한 객체는 Function() 생성자 함수이다.
console.log(Person.constructor === Function);
```

`constructor`는 읽기 전용 속성(기본형 리터럴 변수 - number, string, boolean)이 부여된 예외적인 경우를 제외하고는 값을 바꿀 수 있다.

```jsx
let NewConstructor = function () {
  console.log("this is new constructor!");
};
let dataTypes = [
  1, // Number & false
  "test", // String & false
  true, // Boolean & false
  {}, // NewConstructor & false
  [], // NewConstructor & false
  function () {}, // NewConstructor & false
  /test/, // NewConstructor & false
  new Number(), // NewConstructor & false
  new String(), // NewConstructor & false
  new Boolean(), // NewConstructor & false
  new Object(), // NewConstructor & false
  new Array(), // NewConstructor & false
  new Function(), // NewConstructor & false
  new RegExp(), // NewConstructor & false
  new Date(), // NewConstructor & false
  new Error(), // NewConstructor & false
];

dataTypes.forEach(function (d) {
  d.constructor = NewConstructor;
  console.log(d.constructor.name, "&", d instanceof NewConstructor);
});
```

모든 데이터가 `d instanceof NewConstructor` 명령어에 대해 false를 반환하는데, constructor를 변경하더라도 참조하는 대상이 변경될 뿐

**이미 만들어진 인스턴스의 원형이 바뀐다거나 데이터 타입이 변하는 것은 아님**을 알 수 있다.

어떤 인스턴스의 생성자 정보를 알아내기 위해 constructor 프로퍼티에 의존하는 것이 항상 안전하지는 않다는 것을 알 수 있다.
