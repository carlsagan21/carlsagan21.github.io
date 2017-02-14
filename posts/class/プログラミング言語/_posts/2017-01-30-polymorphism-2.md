---
title: 다형성2(in Java, OCaml, C)
---

다형성으로 구현된 자료구조에서 비교 및 함수들의 구현방법

`OCaml`의 경우

```ocaml
type 'elm tree =
    Lf       (* Leaf *)
  | Br of {  (* Branch *)
      left:  'elm tree;
      value: 'elm;
      right: 'elm tree;
    }

(* (Recursive) function find, which returns whether given integer n exists in BST t *)
let rec find cmp t n =
  match t with
    Lf -> false
  | Br {left=l; value=v; right=r} ->
     if cmp n v = 0 then true
     else if cmp n v < 0 then find cmp l n
     else (* n > v *) find cmp r n
```

어떤 종류의 데이터가 저장되는지 불명(`'elm`)이므로, `cmp` 함수를 명시적으로 넘겨준다.

`insert`, `delete`도 유사하게 구현하면 된다.

`min` 이 문제일 수 있는게, 만일 트리가 `leaf` 인 경우 아무것도 리턴할 수 없다. 예외처리를 해주자.

```ocaml
(* Function min, which, given BST t, returns the minimum value stored in t.
   If t is empty, it fails. *)
let rec min t =
  match t with
    Lf -> invalid_arg "Input can't be a leaf!"
  | Br {left=Lf; value=v; right=_} -> v
  | Br {left=l; value=_; right=_} -> min l
```

그렇다면, 다형성으로 구현된 것에 고차함수를 적용하는 것은 어떻게 되어야 하는가?

`map`이나 `fold`를 구현해보자.

```ocaml
let rec treemap f t =
  match t with
    Lf -> Lf
  | Br {left=l; value=v; right=r} ->
      Br {left = treemap f l;
        value = f v;
        right = treemap f r}

let rec treefold e f t =
  match t with
    Lf -> e
  | Br {left=l; value=v; right=r} ->
      f (treefold e f l)
        v
        (treefold e f r)
```


---

Java

크게 두 가지 구현법이 있다. 먼저 Ocaml처럼, 자료형에 따라 `cmp`함수를 넘겨줘서 비교하는 것이 가능하다.

```java
import java.util.function.*;  // for ToIntBiFunction

public interface BinarySearchTree<Elm> {
    boolean find(Elm e, ToIntBiFunction<Elm,Elm> compare);
    ...
}

or

// import は特に要らない
public interface BinarySearchTree<Elm> {
    boolean find(Elm e, Comparator<Elm> c);
    ...
}
```

이렇게하면, `compare` 함수를 구현마다 정의해줄 필요가 있다.

...

다른 방법으로는 `Elm`이 가지고 있는 비교 함수를 이용하는 방법이 있다. 많은 객체에는 `compareTo` 함수가 내장되어 있다.

`Elm`이 비교가능하다는 것을 명시적으로 알려주기 위해서, `extends Comparable<Elm>`이 필요해진다.

```java
public interface BinarySearchTree<Elm extends Comparable<Elm>> {
  ...
}

public class Leaf<Elm extends Comparable<Elm>> implements BinarySearchTree<Elm> {
  ...
}

public class Branch<Elm extends Comparable<Elm>> implements BinarySearchTree<Elm> {
  ...
  public boolean find(Elm e) {
    if (e.compareTo(v) == 0) { return true; }
    else if (e.compareTo(v) < 0) { return left.find(e); }
    else /* e > v */ { return right.find(e); }
  }
  ...
}
```

트리의 고차함수를 자바로 구현해보자.

```java
public interface Tree<Elm> {
  ...
  Tree<Elm> map(Function<Elm, Elm> f);
  ...
}
```

이렇게 했을 때 문제는, 맵 합수는 `Elm -> Elm` 이기 때문에, 다른 타입으로 변환하는 것은 불가능하다. 가령 `string -> int` 같은 함수를 map으로 할 수 없다.

그러면 Elm, Elm2로 해주면 되지 않나? 다음 처럼.

```java
public interface Tree<Elm, Elm2> {
  ...
  Tree<Elm2> map(Function<Elm, Elm2> f);
  ...
}
```

문제가 2가지 있다. 먼저, 트리를 정의하는 순간부터 map이 무엇으로부터 무엇을 바꾼다는 것을 밝혀야만 한다. 가령

```java
Tree<Integer,String> t = new Branch<Integer,String> ...
```

이 되어버린다. 너무 빠르다.

이걸 해결하기 위해서, 다형메소드라는 걸 사용한다.

```java
public interface Tree<Elm> {
  ...
  <Elm2> Tree<Elm2> map(Function<Elm,Elm2> f);
  ...
}
```

이렇게 하면 `map` 을 사용할 때 목표 타입을 명시할 수 있게 된다.

구현과 사용은 다음과 같다.

```java
// Leaf クラスに追加
public <Elm2> Tree<Elm2> map(Function<Elm,Elm2> f) {
  return new Leaf<Elm2>();
}
// Branch クラスに追加
public <Elm2> Tree<Elm2> map(Function<Elm,Elm2> f) {
  Tree<Elm2> newLeft = left.map(f);
  Tree<Elm2> newRight = right.map(f);
  Elm2 newVal = f.apply(v);
  return new Branch<Elm2>(newLeft, newVal, newRight);
}

...

Tree<Integer> t9 = t18.<Integer>map(s -> s.length());
// "<Integer>" before map specifies what Elm2 is in this invocation
// It can be omitted and written t18.map(s -> s.length())
// This is another form of type inference in Java!
```
