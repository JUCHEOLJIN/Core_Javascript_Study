## undefined와 null

- undefined, null 모두 '없음'을 나타내는 값이다.
- 의미는 같지만 미묘한 차이가 있다.

1. **자동으로 undefined를 부여하는 경우**

- **값을 대입하지 않은 변수**, 즉 데이터 영역의 메모리 주소를 지정하지 않은 식별자에 접급할 때
- 객체 내부의 존재하지 않는 프로퍼티에 접근하려고 할 때
- return 문이 없거나 호출되지 않는 함수의 실행 결과

2. **배열에서의 undefined**

- 값을 대입하지 않은 변수의 경우 undefined를 부여했지만, 배열에서는 조금 특이하게 동작한다.

```jsx
let arr1 = [];
arr1.length = 3;
console.log(arr1); // [empty x 3]

let arr2 = new Array(3);
console.log(arr2); // [empty x 3]

let arr3 = [undefined, undefined, undefined];
console.log(arr3); // [undefined, undefined, undefined]
```

배열에서는 값을 대입하지 않을 경우, 공간은 확보하지만 어떤 값도 할당되지 않음

**+배열을 순회할때 빈 요소는 순회 대상에서 제외된다.(명시적으로 undefined를 대입한 경우에는 값으로 취급해서 순회)**

3. **undefined와 null 비교**

```jsx
const n = null;
console.log(typeof n); // object

console.log(n == undefined); // true
console.log(n == null); // true
console.log(n === undefined); // false
console.log(n === null); // true
```

- '없음'을 표현 하고자 할 때는 null을 사용
- null의 타입이 object로 나오는 것은 JavaScript의 버그
- 사용자가 표현한 '없음'(null) 인지 확인하고자 할 때는 === 연산자를 사용해서 비교
