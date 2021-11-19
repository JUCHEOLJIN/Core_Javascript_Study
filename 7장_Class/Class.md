## 1. 클래스와 인스턴스 개념 이해하기

**클래스**

: 공통 요소를 지니는 집단을 분류하기 위한 개념

`ex. 동물 > 포유류`

- 동물, 포유류는 어떤 것들의 공통 속성을 모아 정의한 개념으로, 직접 만지거나 볼 수 없는 추상적인 개념
- 두 관계에서 동물은 상위, 포유류는 하위 개념
  ⇒ 동물 : 상위클래스(superclass), 포유류 : 하위클래스(subclass)
- 클래스는 하위로 갈수록 상위클래스의 속성을 상속하면서 더 구체적인 요건 추가 혹은 변경
  (하위클래스가 아무리 구체화되더라도 결국 추상적인 개념)

**인스턴스**

: 어떤 클래스의 속성을 지니는 실존하는 개체

`ex. 강아지, 햄스터`

### 현실과 프로그래밍 언어에서의 클래스 정의 비교

**현실**

- 이미 존재하는 개체들을 구분짓기 위해 클래스 도입
- 한 개체가 같은 레벨의 여러 클래스의 인스턴스일 수 있음

ex. `나`는 `한국인`이며 `여성`이며 `20대`이다.

**프로그래밍 언어**

- 사용자가 직접 클래스를 정의하고, 이 클래스를 바탕으로 인스턴스를 만들 때 해당 개체는 클래스의 속성을 지님
- 한 인스턴스는 하나의 클래스만을 바탕으로 만들어짐

### 2. 자바스크립트의 클래스

**다시 복습!**

자바스크립트는 **\_\_\_\_** 기반의 언어이다. 따라서 클래스의 개념이 **\_\_\_\_**.

- 정답은? 찾아라 지식의 별!
  자바스크립트는 **프로토타입** 기반의 언어이다. 따라서 클래스의 개념이 **없다.**

하지만 프로토타입을 일반적인 클래스 의미로 접근해보면 비슷하게 해석할 수도 있다

**static member, instance member**

- 분류 기준 : 인스턴스에 상속되는지 (인스턴스가 참조하는지) 여부

```jsx
// 생성자
var Rectangle = function (width, height) {
  this.width = width;
  this.height = height;
};

// static method
**Rectangle**.isRectangle = function (instance) {
  return instance instanceof Rectangle && instance.width > 0 && instance.height > 0;
};

// prototype method
**Rectangle.prototype**.getArea = function () {
  return this.width * this.height;
};

var rect1 = new Rectangle(3, 4);
console.log(rect1.getArea());  // 12
console.log(rect1.isRectangle(rect1));  // error
console.log(Rectangle.isRectangle(rect1));  // true
```

- static member
  - 인스턴스에서 직접 접근할 수 없는 메서드
  - 생성자 함수를 this로 해야 호출가능
- instance member (⇒ prototyle method)
  - 프로토타입 메서드라고 사용해 혼란을 줄임.
  - 인스턴스에서 직접 호출할 수 있는 메서드

<br />

### 3. 클래스 상속

**클래스 상속을 흉내내기 위한 세 가지 방법**

1. SubClass.prototype에 SuperClass의 인스턴스를 할당한 다음 프로퍼티를 모두 삭제하는 방법
2. 빈 함수(Bridge)를 활용하는 방법
3. Object.create를 이용하는 방법

### 4. ES6의 클래스 및 클래스 상속

**ES5와 ES6의 클래스 문법 비교 **

- 클래스 문법에서 constructor는 ES5의 생성자 함수와 동일한 역할 수행.
- static 키워드는 해당 메서드가 static 메서드임을 알리는 내용으로 생성자 함수(클래스) 자신만 호출 가능
- method()는 prototype 객체 내부에 할당되는 메서드로, 인스턴스가 프로토타입 체이닝을 통해 자신의 것처럼 호출할 수 있는 메서드

**ES6의 클래스 상속**

- ES6의 클래스 문법에서의 상속받는 SubClass를 만들기 위해 class 명령 뒤 extends 키워드와 상속받고 싶은 SuperClass를 적으면 상속 관계 설정 완료
- constructor 내부에는 super라는 키워드를 함수처럼 사용할 수 있는데,
  이는 SuperClass의 constructor를 실행
