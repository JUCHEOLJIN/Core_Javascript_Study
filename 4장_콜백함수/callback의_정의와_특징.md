# 콜백 함수의 정의

함수의 매개변수를 통해 다른 함수의 내부로 전달되는 함수 (수동적인 함수)

> **현실 세상에서의 예 - 일어날 시간을 알람 맞춰놓기 vs 알람 설정하지 않고 제때에 일어나기**
> **알람을 맞춰놓는 경우**
> 핸드폰(콜백함수)으로 알람을 설정해놓으면, 알아서 알람이 시간이 되면 울리고, 울리기 전까지 신경쓰지 않아도 된다.
> 이 경우, **핸드폰에서 원하는 방식**으로 알람을 설정해주면, 핸드폰이 알아서 알람을 울려준다.
> **알람을 설정하지 않은 경우**
> 15분 뒤에 일어나려고 한다면, 좀 누워있다가 '15분이 지난거같은데..?'라는 생각과 함께 일어나서 시간을 확인한다. 시간은 고작 3분 지나있다. '아니네..더 자야지..'라고 마음을 먹고 다시 좀 누워있다가 '이번엔 진짜 15분이 지난거 같은데..?' 라는 생각과 함께 다시 시간을 보면 10분을 막 지나있다. 이제 한 5분 내외 밖에 안남으니 불안해져서 1분마다 잠에서 깨서 시간을 확인한다. 그렇다 결국 잠을 잔 시간은 15분이 훨 안되게 된다.

### callback

callback은 call + back의 합성어로, 어떤 함수 X(callback "함수")를 호출하면서 "특정 조건일 때 함수 Y를 실행해서 나에게 알려달라"는 요청을 함께 보내는 명령이다.

### callback 함수와 고차함수

함수의 매개변수를 통해 다른 함수의 내부로 전달되는 함수 → 콜백 함수
매개변수를 통해 함수 외부에서 콜백함수를 전달받은 함수 → 고차함수
즉, 고차함수는 콜백함수를 자신의 일부분으로 합성한다

**고차함수**
매개변수를 통해 전달받은 콜백 함수의 **호출 시점을 결정**해서 호출한다.
콜백함수는 고차함수를 통해 호출되며, 이때 고차함수는 필요에 따라 콜백함수에게 인자를 전달할 수 있다.
대표적인 예) setTimeout, setInterval, map, filter...

# callback 함수와 관련된 가장 큰 특징 : "제어권"

**고차함수는 콜백 함수에 대한 제어권을 가지고 있다.**
크게 3가지를 제어하는데, 아래를 통해 자세히 알아보자.

## 1. 호출 시점 제어

고차함수의 대표주자인 `setInterval` 이 대표적으로 callback의 호출 시점 제어를 보여준다.

```jsx
let count = 0;

const cbFunc = () => {
  console.log(count);
  if (++count > 4) clearInterval(timer);
};

let timer = setInterval(cbFunc, 300);
```

위의 `timer` 변수를 호출했을 때와 `cbFunc` 그 자체를 호출했을 때의 통제권을 비교하면 아래와 가다

**setInterval vs cbFunc**

| code                     | 호출 주체                                                                        | 제어권                                                         |
| ------------------------ | -------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| cbFunc                   | 사용자                                                                           | 사용자                                                         |
| setInterval(cbFunc, 300) | setInterval<br> (3초가 지나면 setInterval이 콜백함수인 cbFunc를 알아서 호출한다) | setInterval<br> (3초에 한번 cbFunc가 호출되도록 직접 제어한다) |

## 2. 인자 제어

고차함수 `map`을 사례로 살펴보자

```jsx
const originArr = [10, 20, 30];
const newArr = originArr.map((item, idx) => {
	console.log(item, idx);
	return item + 5;
}));

const newArr2 = originArr.map((idx, item)) => {
	console.log(idx, item);
	return item + 5;
}));

```

위의 경우에 `newArr` 와 `newArr2` 두개가 map이 실행되는 동안에 console.log가 다르게 찍히지 않는다는 것을 우리는 안다. 왜냐면 map의 경우 첫번째 인자는 무조건 해당 turn의 value로, 두번째 인자는 무조건 해당 value의 index로 고정되어있기 때문이다.

이런식으로 **map을 사용할 때 약속된 인자의 순서 또한 고차함수로서 callback에 대한 인자를 제어한다**는 것으로 볼 수 있다.

> **현실 세상에서의 예 (이어서)**
> 아까 핸드폰으로 알람을 맞추는 과정으로도 대입해서 볼 수 있다.
> 핸드폰으로 맞추는 경우, 핸드폰이 원하는 방식에 따라 알람을 맞춰야된다 (날짜, 시간, 반복횟수, 진동여부 설정 등...) 이것 또한 일종의 "인자 제어"로 볼 수 있다..!

## 3. this 제어

고차함수를 통해 callback이 참조할 this도 제어할 수 있다

> **사전지식 (메서드 만드는 원리 - call, apply)**
> **call과 apply는 어디에 사용되나요?**
> 👉 주어진 this 값 및 각각 전달된 인수와 함께 함수를 호출할 때 사용됩니다.
> 즉, this를 "특정 대상"으로 지정하여 함수에게 넘겨줄 때 사용하는 것으로, 해당 함수의 this 말고 다른 객체를 할당할 수도 있습니다.
> `function.prototype.call()` , `function.prototype.apply()` 본래 형태에서 보면 알 수 있듯이 이 두가지는 모두 메서드입니다.
> **call과 apply의 차이점**
> 👉 둘은 기능이 같습니다! 다만, 인자의 형태가 다릅니다.
> call의 경우 `call(호출에 사용되는 this값, 필요 인자1, 필요 인자2)` 이런 식으로 처음에 this를 지정한 후, call을 호출한 함수에서 필요한 인자를 **나열**해서 전달합니다.
> 반면 apply의 경우 `apply(호출에 사용되는 this값, [필요인자들])` 이런 형태로, apply를 호출한 함수에서 필요로 하는 인자를 **배열로 묶어** 전달합니다.

<br>
<br>

```jsx
// #1
setTimeout(function () {
  console.log(this);
}, 300);
// Window{...}

// #2
[1, 2, 3, 4, 5].forEach((item) => {
  console.log(this);
});
// Window{...}

// #3
document.body.innerHTML += `<button id="a">클릭</button>`;
document.body.querySelector("#a").addEventListener("click", (e) => {
  console.log(this, e);
});
// <button id="a">클릭</button>, MouseEvent{isTrusted: true, ...}
```

고차함수인 `setTimeout`, `forEach`, `addEventListener` 의 콜백함수들의 this들이 모두 일관적이지 않게 나온다.
왜냐면 각각의 고차함수들이 서로 다른 this에 대한 기준을 가지고 있기 때문이다.
`setTimeout` 은 콜백함수를 호출할 때 call 매서드의 첫번째 인자에 전역객체를 넘기기 때문에 콜백함수의 this가 전역객체를 가리킨다
`forEach` 는 별도의 인자(3번째 인자)로 this를 넘겨 받아야되는데 위의 경우 넘겨받지 않았기 때문에 전역객체를 가리킨다
`addEventListener` 는 메서드의 this를 그대로 넘기도록 정의되어있어, `document.body.querySelector("#a")` 부분이 그대로 넘어가게 된다.

# 주의사항 - callback 함수는 함수이다!

**콜백함수로 어떤 객체의 메서드를 전달하더라도 그 메서드는 메서드가 아닌 함수로서 호출된다!**
함수로서 호출되기에 메서드를 콜백함수로 사용하면 함수기에 메서드와 원래 연결되어있던 객체와 연결이 사라진다 → 고차함수에서 별도로 this를 지정하지 않은 경우 this는 전역 객체가 된다

- 예시

  ```jsx
  var obj = {
    vals: [1, 2, 3],
    logValues: function (v, i) {
      console.log(this, v, i);
    },
  };

  obj.logValues(1, 2); // {vals: [1, 2, 3], logValues: f()} 1, 2
  [4, 5, 6].forEach(obj.logValues); // Window{...} 4 0
  ```

## callback으로 지정된 메서드를 메서드처럼 사용하는 법 (콜백 함수의 this를 다른 값으로 바인딩하기)

(모든 고차함수가 별도의 인자로 this를 받지 않으니까요..!)

### 과거에는

this를 다른 변수에 담아 콜백 함수로 활용할 함수에서는 그 새로만든 변수를 사용하도록 했었다.

- like this! (변수에 새로 할당)

  ```jsx
  var obj1 = {
    name: "obj1",
    func: function () {
      // 아래 부분을 주의깊게 봐주세요! //
      var self = this;
      return function () {
        // 함수를 리턴해서 사용하도록 했습니다. self라는 변수로 this를 지정해주고요
        console.log(self.name);
        // 위 부분을 주의깊게 봐주세요! //
      };
    },
  };

  var callback = obj1.func();
  setTimeout(callback, 1000);
  ```

- or like this! (call 메서드 사용)
  ```jsx
  var obj2 = {
    name: "obj2",
  };
  var callback2 = obj1.func.call(obj2); // call 메서드를 사용해서 this를 지정해줌
  // call의 맨 첫번째 인자는 this를 지정하죠?
  setTimeout(callback2, 1500);
  ```

### 요즘은

bind를 사용해서 이를 해결합니다

- 예시

  ```jsx
  var obj1 = {
    name: "obj1",
    func: function () {
      // bind를 사용하면 this를 따로 가공하지 않고 바로 사용할 수 있습니다
      console.log(this.name);
    },
  };

  var obj2 = { name: "obj2" };
  setTimeout(obj1.func.bind(obj2), 1500);
  ```
