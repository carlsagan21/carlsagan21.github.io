---
title: 타입과 수학 사이
---

<https://www.youtube.com/watch?v=ojZbFIQSdl8>를 다시 기술함.

임의의 갯수의 원소를 가지는 리스트를 타입으로 기술해보자.

```ocaml
type T = ELEM
type list
    = NULL
    | T of list
```

OCaml로 재귀적으로 정의된 타입니다. 하나씩 나열해보면 이런 식이다.

```c++
NULL, T NULL, T T NULL, T T T NULL, ...
// in c++ vector
std::vector<T> : {}, {T}, {T, T}, {T, T, T}, ...
```

이걸 일반적인 기호로 옮기면 다음과 같다.

```ocaml
list = 1 + T * list
(* 1 은 NULL, + 는 or, * 는 원소를 붙이는 기호 대신 쓴 것이다. *)
```

여기까지가 타입이었다. 이제 똑같은 기호를 수학으로 생각해보자. 단순한 중학교 수학을 해보는 것이다.

```ocaml
list = 1 + T * list
=>
list - T * list = 1
=>
(1 - T) * list = 1
=>
list = 1 / (1 - T)
```

이게 무슨 의미일까? 우리가 만든 임의의 갯수의 원소를 가지는 리스트를 수학적으로 변조했을 때, 그것이 `1 / (1 - T)`와 같다는 게 뭐지? 울프람알파에 물어보자.
<http://www.wolframalpha.com/input/?i=1%2F(1-T)>
쭉 내리다보면, 중간에 T=0에서 테일러전개(Series expansion at T=0)가 있다.

```ocaml
(* 테일러 급수 *)
1 + T + T*T + T*T*T + T*T*T*T + T*T*T*T*T + ...
```

이제 이걸 다시 타입 기호로 읽어보자. 앞에서 말했듯이, 1 은 NULL, + 는 or, * 는 원소를 붙이는 기호이다. 따라서,

```ocaml
type list = NULL | T NULL | T T NULL | T T T NULL | T T T T NULL | T T T T T NULL | ...
```

따라서, `재귀적 타입 기호 -> 수학 기호화 -> 수학 연산 -> 테일러 급수 -> 타입 기호화`를 거쳤더니... 재귀적 타입 기호가 동일한 나열식 타입 기호로 변하고 만 것이다! 타입이라는 개념의 미학.
