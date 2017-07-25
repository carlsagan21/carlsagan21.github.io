---
title: 고차함수(in OCaml, Java, C)
---

수열이 함수라는 사실을 떠올리면,

시그마(합계산)는 고차함수(higher order function)라는걸 알 수 있다.

이렇게 사용할 수 있는 함수(수열)을 일급함수(first class function)라고 한다.

```ocaml
let rec sigma(f, n) =
  if n < 1 then f 0
  else f n + sigma(f, n-1)

/* 타입추론결과 */
val sigma : (int -> int) * int -> int = <fun>
```

커링 예시

```ocaml
let curried_greeting gen = fun name ->
  match gen with
    Male -> "Hello, Mr. " ^ name
  | Female -> "Hello, Ms. " ^ name

/* 타입추론결과 */
val : curried_greeting : gender -> string -> string = <fun>
```

자바에서는 이런 기능이 약하다. 가장 큰 문제는, `->`에 해당하는게 없다. 임의의 무엇을 받아서 임의의 무엇을 돌려주는 것이 없다.

이를 해결하려면
1. `interface를` 만든다
2. `Function<T, R>` or `BiFunction<T, U, R>`을 쓴다.(3항 이상은 직접 구현해야)
3. `Java8`에 들어온 람다함수를 사용한다.

```java
// 1번
public interface IntToInt {
  int apply(int x);
}
public class Dbl implements IntToInt {
  // no instance variables
  public Dbl() {
    // nothing to initialize
  }
  public int apply(int n) { return n * 2; }
}

// 2번
Function<Integer, Integer>

// 3번
ThreeIntsToInt f = (n, m, p) -> n + m + p;
```

C의 경우. 복잡함.

```c
// 함수 포인터인 (*f)가 int를 가리킨다는 것을 의미.
int sigma(int (*f)(int), int n) {
  if (n < 1) {
    return (*f)(0);
  } else {
    return (*f)(n) + sigma(f, n - 1);
  }
}
```
