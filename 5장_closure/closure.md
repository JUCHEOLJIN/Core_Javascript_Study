# Clouser란?

MDN 직역 : 함수와 그 함수가 선언된 당시의 Lexical Environment(outerEnvironmentReference)의 상호관계에 따른 현상

어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 B를 **외부로 전달\*\***할 경우, A의 실행 컨텍스트가 종료된 이후에도 변수a가 사라지지 않는 현상.

➡️ 즉, 함수의 실행 컨텍스트가 종료된 후에도 LexicalEnvironment가 가비지 컬렉터의 수집 대상에서 제외\*되는 현상을 말한다.

- 예시 보기
  ```jsx
  var outer = function () {
    var a = 1;
    var inner = function () {
      return ++a;
    };
    return inner; // inner()는 실행 결과를 반환하고, inner는 함수째로 반환한다
  };
  var outer2 = outer();
  console.log(outer2());
  console.log(outer2());
  ```
  `inner`함수 자체를 반환하는 case기에, `outer2`변수는 `outer`의 실행 결과인 `inner` 함수를 참조하게 된다.<br>
  **`inner` 함수의 실행 context 분석해보기**
  - `environmentRecord` : 아무것도 없다
  - `outer-EnvironmentReference` : `outer` 함수의 LexicalEnvironment
    <br>
    > 💡 **inner 함수의 실행 시점에 outer함수는 이미 실행이 종료된 상태인데, 어떻게 outer함수의 LexicalEnvironment에 접근할 수 있나요?**<br>
    > → 가비지 컬렉터는 어떤 값을 참조하는 변수가 하나라도 있다면 그 값은 수집 대상에 포함시키지 않습니다.<br>
    > 위의 경우도 언젠가 `inner` 함수의 실행 컨텍스트가 활성화되면 `outerEnvironmentReference` 가 `outer` 함수의 `LexicalEnvironment` 를 필요로 할 것이므로 해당 내역이수집 대상에서 제외됩니다.
  **LexicalEnvironment가 GC의 수집 대상에서 제외되는 경우는 지역변수를 참조하는 내부 함수가 외부로 전달된 경우가 유일**
  <br>
- return 이외에 외부로 전달하는 예
  ```jsx
  // (1) setInterval/setTimeout
  function() {
  	var a = 0;
  	var intervalId = null;
  	var inner = function() {
  		if(++a >= 10) {
  			clearInterval(intervalId);
  		};
  		console.log(a);
  	};
  	intervalId = setInterval(inner, 1000); // 외부 객체인 window의 메서드에 전달할 콜백 함수 내부에서 지역변수 참조
  };

  // (2) eventListener
  function() {
  	var count = 0;
  	var button = document.createElement("button");
  	button.innerText = "click";
  	button.addEventListener("click", function(){
  		console.log(++count, "times clicked"); // 외부 객체인 DOM의 메서드에 등록할 콜백 함수 내부에서 지역변수 참조
  	});
  	document.body.appendChild(button);
  };

  ```
  지역변수를 참조하는 내부함수를 외부에 전달했기 때문에 클로저이다.

# Closure와 메모리 관리

closure의 본질적 특성 중 하나는 **메모리 소모**가 발생한다는 것이다
→ 그래서 불필요한 메모리 소모가 발생하지 않도록 클로저의 필요성이 사라진 시점에는 더는 메모리를 소모하지 않게 해줘야 한다
참조 count를 0으로 만들면 GC가 수거해 가도 되는 요소로 인식하게 되기에 `null` 또는 `undefined` 를 할당하여 count를 0으로 맞춰줘야 한다.

- 메모리 관리 예시
  ```jsx
  // (1) return에 의한 클로저의 메모리 해제
  var outer = (function () {
    var a = 1;
    var inner = function () {
      return ++a;
    };
    return inner;
  })();
  console.log(outer());
  console.log(outer());
  outer = null; // outer 식별자의 inner 함수 참조를 끊음

  // (2) setInterval에 의한 클로저의 메모리 해제
  (function () {
    var a = 0;
    var intervalId = null;
    var inner = function () {
      if (++a >= 10) {
        clearInterval(intervalId);
        inner = null; // inner 식별자의 함수 참조를 끊음
      }
      console.log(a);
    };
    intervalId = setInterval(inner, 1000);
  })();

  // (3) eventListener에 의한 클로저의 메모리 해제
  (function () {
    var count = 0;
    var button = document.createElement("button");
    button.innerText = "click";

    var clickHandler = function () {
      console.log(++count, "times clicked");
      if (count >= 10) {
        button.removeEventListener("click", clickHandler);
        clickHandler = null; // clickHanlder 식별자의 함수 참조를 끊음
      }
    };
    button.addEventListener("click", clickHandler);
    document.body.appendChild(button);
  })();
  ```

> 💡 **클로저의 메모리 소모는 메모리 누수인가?**<br>
> 메모리가 소모되기에 클로저가 메모리 누수의 위험이 있다고 생각할 수 있다. 허나 이는 틀린 생각이다.<br>
> "메모리 누수"라는 표현은 **개발자의 의도와 달리 어떤 값의 참조 카운트가 0이 되지 않아 GC의 수거 대상이 되지 않는 경우**에 사용하는 표현이다.<br>
> 클로저같이 개발자가 의도적으로 참조 카운트를 0이 되지 않게 설계한 경우, "누수"라는 표현을 사용하는 건 적절하지 않다.

# Closure 활용 사례

## 1. 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때

콜백 함수 내부에서 외부변수를 참조하는 방법은 3가지가 있다.

1. 콜백 함수를 내부 함수로 선언해서 외부 변수를 직접 참조하는 방법
2. bind 메서드로 값을 직접 넘겨주는 방법

   → 함수 내부에서의 this가 원래의 것과 달라지기에 유의

3. **콜백함수를 고차함수로 바꿔서 클로저를 활용하는 방법**
   <br>

- 각 예시 보기
  ### 1. 콜백 함수를 내부 함수로 선언해서 외부 변수를 직접 참조
  ```jsx
  // 공통 코드
  var fruits = ["apple", "banana", "peach"];
  var ul = document.createElement("ul");

  fruits.forEach((fruit) => {
    // (A)
    var li = document.createElement("li");
    li.innerText = fruit;
    li.addEventListener("click", () => {
      // (B)
      alert("your choice is " + fruit);
    });
    ul.appendChild(li);
  });

  document.body.appendChild(ul);
  ```
  A는 fruits의 개수 만큼 실행되며, 그때마다 새로운 실행 context가 활성화 된다. A의 실행 종료 여부와 무관하게 클릭 이벤트에 의해 각 컨텍스트의 B가 실행될 때는 B의 `outerEnvironmentReference` 가 A의 `LexicalEnvironment` 를 참조하게 된다. 따라서, B함수가 참조할 예정인 변수 fruit에 대해서는 A가 종료된 후에도 GC 대상에서 제외되어 계속 참조할 수 있다.
  ### 2. bind 메서드로 값을 직접 넘겨주기
  ```jsx
  // 공통 코드 참고
  // alert 함수를 외부로 옮김
  var alertFruit = fruit => {
  	alert("your choice is " + fruit);
  };

  fruits.forEach(fruit => {
  	var li = document.createElement("li");
  	li.innerText = fruit;
  	li.addEventListener("click", alertFruit.bind(null, fruit)); // alertFruit 호출 시 bind로 값을 넘겨줌
  	ul.appendChild(li);
  };
  ```
  `bind` 없이 넘기는 경우 콜백 함수의 인자에 대한 제어권을 `addEventListener` 가 가지게 된다.
  이 `addEventListener` 는 콜백함수를 호출 할 때 첫번째 인자에 "이벤트 객체"를 주입하기에 li를 클릭 할 경우 `[object MouseEvent]` 라는 값이 출력된다.
  그래서 이벤트 객체 대신에 우리가 원하는 fruit가 먼저 넘어올 수 있도록 하려면 bind메서드를 사용해서 처리해야 한다.
  ![스크린샷 2021-11-15 오후 8.15.26.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/30b97b3f-4488-4d6d-ad99-822dd97c8202/스크린샷_2021-11-15_오후_8.15.26.png)
  ![위와 같이 event 객체만 전달된다.](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7ec5aa76-1b9f-4c1e-8f19-d5ffa2ebc1bc/스크린샷_2021-11-15_오후_8.15.41.png)
  위와 같이 event 객체만 전달된다.
  ### 3. 콜백 함수를 고차함수로 바꿔서 클로저를 활용하기
  ```jsx
  var alertFruitBuilder = fruit => {
  	return function(){
  		alerert("your choice is + " + fruit);
  	};
  };

  fruits.forEach(fruit => {
  	var li = document.crerateElement("li");
  	li.innerText = fruit;
  	li.addEventListener("click", alertFruitBuilder(fruit));
  	ul.appendChild(li);
  ```
  `alertFruitBuilder` 함수를 실행하면 **실행 결과가 다시 함수가 된다.** 이 함수를 이벤트리스너에 콜백 함수로서 전달한 형태이다.
  이후 언젠가 클릭 이벤트가 발생하면, 비로소 반환되는 익명 함수의 실행 컨텍스트가 열리면서 `alertFruitBuilder` 의 인자로 넘어온 fruit를 `outerEnvironmentReference` 에 의해 참조할 수 있게 된다.
  즉, `alertFruitBuilder` 의 실행 결과로 반환된 함수에는 클로저(fruit)가 존재하게 된다.

## 2. 접근 권한 제어(정보 은닉)

> **정보 은닉(information hiding)이란**<br>

     어떤 모듈의 내부 로직에 대해 외부로의 노출을 최소화해서 모듈간의 결합도를 낮추고 유연성을 높이고자 하는 개념이다.<br>
     접근 권한은 `public`, `private`, `protected` 이렇게 3 종류가 있으며 `public` 은 외부에서 접근 가능한 것이고, `private` 는 내부에서만 사용 가능하며 외부에 노출되지 않는 것을 의미한다.

clouser를 활용하면 `return` 을 활용하여 함수 내부의 변수들 중 선택적으로 접근 권한 제어가 가능하다.

외부에 제공하고자 하는 정보들은 모아서 return 하고, 내부에서만 사용할 정보들은 return 하지 않는 형태로 접근 권한을 제어한다. 이 경우, return되는 변수들은 공개 멤버(public member)가 되고, 그렇지 앟은 변수들은 비공개 멤버(private member)가 된다.

- 예시 보기
  ```jsx
  var car = {
    fuel: Math.ceil(Math.random() * 10 + 10), // 연료(L)
    power: Math.ceil(Math.random() * 3 + 2), // 연비(km/L)
    moved: 0, //총 이동 거리
    run: function () {
      var km = Math.ceil(Math.random() * 6);
      var wasteFuel = km / this.power;
      if (this.fuel < wasteFuel) {
        console.log("이동 불가");
        return;
      }
      this.fuel -= wasteFuel;
      this.moved += km;
      console.log(`${km}km 이동 ( 총 ${this.moved}km)`);
    },
  };
  ```
  위의 경우처럼 car 변수에 객체를 직접 할당한 경우 어뷰징이 가능하다 (`car.fuel = 1000` , `car.power = 10000` 이런 식으로..) 이렇게 값을 함부로 바꾸지 못하도록 객체가 아닌 함수로 만들고, 필요한 멤버만을 return 해서 대응하게 되며, 이를 정보 은닉이라고 한다.
  ```jsx
  var createCar = function(){
  	var fuel = Math.ceil(Math.random() * 10 + 10);
  	var power = Math.ceil(Math.random() * 3 + 2);
  	var moved = 0;
  	return {
  		get moved(){ // getter를 부여하여 읽기 전용 속성 부여
  			return moved;
  		},
  		run: function(){
  			var km = Math.ceil(Math.random() * 6);
  			var wasteFuel = km / power;
  			if(fuel < wasteFuel){
  				console.log("이동불가");
  				return;
  			};
  			fuel -= wasteFuel;
  			moved += km;
  			console.log(`${km}km 이동 (총 ${moved}km). 남은 연료: ${fuel}`);
  		};
  	};
  };

  var car = createCar();
  ```
  위와 같이 함수로 반환하는 경우, return되는 moved와 run function만 접근이 가능하다.
  ```jsx
  car.run(); // 3km 이동 (총 3km), 남은 연료: 17.4;
  console.log(car.moved); // 3
  console.log(car.fuel); // undefined
  console.log(car.power); // undefined
  ```
  - 응용 ) 더 안전한 코드로 만들기
    ```jsx
    var createCar = funcion(){
    ...
    	var publicMembers = {
    ...
    	};
    	Object.freeze(publicMembers);
    	return publicMembers;
    };
    ```
    `freeze` 를 객체에 걸면, 해당 객체는 변경이 불가능한 상태가 된다.

## 3. 부분 적용 함수

> **부분 적용 함수(partially applied function)이란**<br>

    n개의 인자를 받는 함수에 미리 m개의 인자만 넘겨 기억시켰다가, 나중에 (n - m)개의 인자를 넘기면 비로소 원래 함수의 실행 결과를 얻을 수 있게 하는 함수

- 구현 예제
  ```jsx
  Object.defineProperty(window, "_", {
  // 빈 자리라는 것을 표시하기 위해 미리 전역객체에 _라는 프로퍼티를 준비하면서
  // 삭제, 수정이 불가능하도록 여러가지 속성을 설정했다.
  	value: "EMPTY_SPACE",
  	writable: false,
  	configurable: false,
  	enumerable: false,
  });

  var partial = function(){{
  	var originalPartialArgs = arguments;
  	var func = originalPartialAargs[0];
  	if(typeof func !== "function"){[
  		throw new Error("첫 번째 인자가 함수가 아닙니다");
  	};
  	return function(){
  		var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1);
  		var restArgs = Array.prototype.slice.call(arguments);
  		for(var i = 0; i < partialArg.length; i++){
  			if(partialArgs[i] === _){
  				partialArgs[i] = restArgs.shift();
  				// _로 비워놓은 공간마다 나중에 넘어온 녀석들로 채워지도록 작업
  			};
  		};
  		return func.apply(this, partialArgs.concat(restArgs));
  	};
  };

  var add = function(){
  	var result = 0;
  	for(var i = 0; i < arguments.length; i++){
  		result += arguments[i];
  	};
  	return result;
  };

  var addPartial = partial(add, 1, 2, _, 4, 5, _, _, 8, 9);
  console.log(addPartial(3, 6, 7, 10)); // 55

  var dog = {
  	name: "강아지",
  	greet: partial((prefix, suffix)) => {
  		return prefix + this.name + suffix;
  	}, "왕왕, ")
  };
  dog.greet("입니다!"); // 왕왕, 강아지입니다!
  ```
  미리 일부 인자를 넘겨두어 기억하게끔 하고, 추후 필요한 시점에 기억했던 인자들까지 함께 실행하게 하기에, 클로저를 핵심 기법으로 사용했다고 볼 수 있다.
  <br>
- 디바운스 예제
  > 💡 **디바운스(debounce)란?**<br>
  > 짧은 시간 동안 동일한 이벤트가 많이 발생할 경우 이를 전부 처리하지 않고 처음 또는 마지막에 발생한 이벤트에 대해 한 번만 처리하는 것으로 프론트엔드 성능 최적화에 큰 도움을 주는 기능 중 하나이다.<br>
  > scroll, wheel. mousemove, resize 등에 적용하기 좋다.
  ```jsx
  var debounce = function (eventName, func, wait) {
    var timeoutId = null;
    return function (event) {
      // setTimeout 사용을 위해 this를 별도의 변수에 담음
      var self = this;
      console.log(eventName, "이벤트 발생!");
      // 대기 큐 초기화
      clearTimeout(timeoutId);
      //  새로운 대기열 등록 - wait 시간 뒤에 func를 실행할 것
      // 각 이벤트가 바로 이전 이벤트로부터 wait시간 이내에 발생하는 한,
      // 마지막에 발생한 이벤트만이 초기화되지 않고 무사히 실행될 것이다
      timeoutId = setTimeout(func.bind(self, event), wait);
    };
  };
  var moveHandler = function (e) {
    console.log("move event 처리");
  };
  var wheelHandler = function (e) {
    console.log("wheel event 처리");
  };

  document.body.addEventListener("mousemove", debounce("move", moveHandler, 500));
  document.body.addEventListener("mousewheel", debounce("wheel", wheelHandler, 700));
  ```

## 4. 커링 함수

> **커링 함수(currying function)란**<br>

    여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수로 나눠서 순차적으로 호출될 수 있게 체인 형태로 구성한 것을 칭한다.<br>
    커링은 한 번에 하나의 인자만 전달하는 것을 원칙으로 한다. 또한, 중간 과정상의 함수를 실행한 결과는 그 다음 인자를 받기 위해 대기만 할 뿐으로, 마지막 인자가 전달되기 전까지는 원본 함수가 실행되지 않는다.<br>
    (부분 적용 함수는 여러개의 인자를 전달할 수 있고, 실행 결과를 재실행 할 때 원본함수가 무조건 실행된다)

- 예제
  ```jsx
  var curry3 = function(func){
  	return function(a){
  		return function(b){
  			return func(a, b);
  		};
  	};
  };

  var getMaxWith10 = curry3(Math.max)(10);
  console.log(getMaxWith10)(8)); // 10
  console.log(getMaxWith10)(25)); // 25

  // ES6
  var curry3 => func => a => b => c => func(a, b, c);
  ```
  각 단계에서 받은 인자들은 모두 마지막 단계에서 참조할 것이므로 GC되지 않고 메모리에 차곡 차곡 쌓였다가, 마지막 호출로 실행 컨텍스트가 종료된 후에야 비로소 한꺼번에 GC의 수거 대상이 된다.

**커링 함수가 유용한 경우 = 지연 실행(lazy execution)**

커링 함수는 마지막 인자가 넘어갈 때까지 함수 실행을 미루는 형태이므로, 원하는 시점까지 지연시켰다가 실행하는 것이 요긴한 상황이라면 커링을 쓰기에 적절하다.

또한, 프로젝트 내에서 자주 쓰이는 함수의 매개변수가 항상 비슷하고 일부만 바뀌는 경우에도 유용하다.

- 지연 실행 예시 (with fetch)
  ```jsx
  var imageURL = "http://imageAddress.com/";
  var productURL = "http://productAddress.com/";

  var getInformation = (baseURL) => {
    return function (path) {
      return function (id) {
        return fetch(baseURL + path + "/" + id);
      };
    };
  };
  // ES6 : var getInformation = baseURL => path => id => fetch(baseURL + path + "/" + id);

  // 이미지 타입별 요청 함수 형태
  var getImage = getInformation(imageURL); // http://imageAddress.com/
  var getEmoticon = getImage("emoticon"); // http://imageAddress.com/emoticon
  var getIcon = getImage("icon"); // http://imageAddress/icon

  // 실제 요청
  var emoticon1 = getEmoticon(100); // http://imageAddress.com/emoticon/100
  var icon2 = getIcon(40); // http://imageAddress.com/icon/40
  ```
  공통적인 요소는 먼저 기억시켜두고, 특정한 값(id)만으로 서버 요청을 수행하는 함수를 만들어두는 편이 개발 효율성이나 가독성 측면에서 훨 좋다.
  이런 이유로 최근 여러 프레임워크나 라이브러리 등에서 커링을 상당히 광범위하게 사용하고 있다
  <br>
  **Redux middleware로 본 커링함수**
  ```jsx
  // Redux "Logger"
  const logger = (store) => (next) => (action) => {
    console.log("dispatching", action);
    console.log("next state", store.getState());
    return next(action);
  };

  // Redux "thunk"
  const thunk = (store) => (next) => (action) => {
    return typeof action === "function" ? action(dispatch, store.getState) : next(action);
  };
  ```
  두 미들웨어 모두 store, next, action 순서로 인자를 받는다.
  이 중 store는 프로젝트 내에서 한 번 생성된 이후로는 바뀌지 않는 속성이고, disptach의 의미를 가지는 next 또한 바뀌지 않는 속성이다. 허나 action의 경우 매번 달라진다. 즉, store와 next 값이 결정되면 Redux 내부에서 logger 또는 thunk에 store, next를 미리 넘겨서 반환된 함수를 저장시켜놓고, 이후에는 action만 받아서 처리할 수 있게끔 한다.
