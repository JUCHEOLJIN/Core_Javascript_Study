# Lexical Environment, this

```
2장 실행 컨텍스트
2-3 LexicalEnvironment
2-4 this
p41~63
```

## 실행 context에 저장되는 환경 정보 3가지

1. **Variable Environment**

   현재 context 내의 식별자들에 대한 정보 및 외부 환경 정보
   선언 시점의 Lexical Environment의 스냅샷 저장

   **변경 사항은 저장되지 않는다**

2. **Lexcial Envrionment**

   variable environment와 동일한 정보가 저장되어 있으나, **변경사항이 실시간으로 반영된다**

3. **This binding**

   식별자가 바라봐야할 대상 객체.
   실행 context를 활성화 하는 당시에 지정된 this가 저장된다
   함수를 호출하는 방법에 따라 그 값이 달라지는데, 지정되지 않은 경우, 전역객체가 this가 된다.

## Lexical Environment

**정의**

사전적인 환경 (정의를 내린 환경)
해당 context 내부의 식별자들을 일람 해 둔 공간. 식별자들과 참조내용을 사전에서 접하는 느낌으로 모아둔 것

**구성**

- Environment Record
  - 현재 context와 관련된 code와 식별자 정보\*가 저장되는 공간
    **함수에 저장된 매개변수, 선언한 함수 자체, `var` 로 선언된 변수의 식별자**들을 context 내부를 처음부터 끝까지 순서대로 수집해서 쌓아놓는다
  - hoisting이 이 record를 참고해서 일어난다
- Outer Environment Reference
  - (쉽게 말하면) 내 부모의 context와 관련된 환경 정보가 저장된 공간
  - scope, scope-chain이 이 reference를 참고해서 일어난다

## Hoisting

변수, 함수를 실제로 끌어올리지는 않지만, 편의상 모두 끌어올린 것으로 **"간주"**하는 것

- 변수 : **변수명만** 끌어올리고, 할당과정은 원래 자리에 그대로 둔다
- 함수

  - 선언문의 경우 : **함수 전체**를 끌어올린다
      <aside>
      💡 주의) 전역 context가 활성화 될 때 전역 공간에 선언된 함수들이 가장 위로 끌어올려지니 주의해야 한다
      </aside>
      
      ```jsx
      // 선언문의 형태
      function a () {};
      a();
      ```

  - 표현식의 경우 : **변수 선언부만** hoisting 한다

    ```jsx
    // 익명 함수 표현식
    var a = function(){}...

    // 기명 함수 표현식
    var b = function c(){}...
    ```

## Scope, Scope-chain

### scope

식별자에 대한 유효 범위

예 ) A 내부에서 선언한 변수는 오직 A 내부에서만 접근할 수 있다
특이사항 ) ES5에서는 함수만이 scope를 생성할 수 있다

### scope-chain

식별자의 유효범위를 **안에서부터 바깥으로** 차례대로 검색해 나가는 것
부모의 Lexical Environment를 연어처럼 거슬러 올라가면서 추척한다..
이를 가능하게 하는 요인이 `Outer-Environment Refernce` 이다

- `Outer-Environment Reference` 는 선언될 당시\*의 부모의 Lexical Environment를 참조한다
    <aside>
    💡 선언될 당시란 콜 스택 상에서 **어떤 실행 context가 활성화된 상태**를 말한다
    (모든 code는 실행 context가 활성화 상태일때만 실행된다)
    </aside>

이러한 구조적 특성 덕분에 동일 식별자를 선언한 경우, 무조건 scope-chain 상에서 **가장 먼저 발견된 식별자에만 접근 가능**하게 된다 → 이로 인해서 **"변수 은닉화"**가 일어나기도 한다

<aside>
💡 변수 은닉화 ) inner 함수에서 선언한 변수와 전역에서 선언한 변수명이 동일한 경우, inner에서는 전역 변수에 접근할 수 없다
</aside>

# Context의 특징

depth가 깊어질수록 context의 규모는 작아지는 반면, 깊어진 depth만큼 scope-chain을 타고 접근 가능한 변수의 수는 늘어난다

# 전역변수와 지역변수

전역변수 : 전역공간에서 선언한 변수
지역변수 : 함수 내부에서 선언한 변수
