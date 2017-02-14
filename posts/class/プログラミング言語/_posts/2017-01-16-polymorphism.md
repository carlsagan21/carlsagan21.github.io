---
title: 다형성(in Java, OCaml, C)
---

polymorphism. 多相性.

하나의 정의를 다양한 타입에서 사용가능 한 것을 의미한다.

Java에서의 오버로딩/제네릭이 가장 먼저 떠오른다.

거기에 더해, `+` 등의 연산자는 좌, 우에 정수/실수/문자열 등이 올 수 있으므로 다형성을 가진다고 할 수 있다.

오버로딩/제네릭 등 함수의 파라미터에 대해서 다형성이 성립할 경우, 그것을 parametric polymorphism이라고 한다.

형 파라미터, type parameter, 型パラメータ 를 인터페이스/클래스에 정의하여 그것을 구체화 함으로서 다형성을 구현할 수 있다.

참고) 자바 제네릭에는 프리미티브 타입을 넣을 수 없다.

```java
Tree<int> t = ...;  // illegal!
Tree<Integer> t = ...; // 대신에.
```

---

OCaml

코드로 보자

타입변수는 `'`가 앞에 붙는다.

```ocaml
type 'elm tree =
  | Lf
  | Br of {
    left: 'elm tree;
    value: 'elm;
    right: 'elm tree;
  }

(* Constructing a sample tree holding integers *)
let t1 = Br {left = Lf; value = 10; right = Lf}
(* Now let's construct a tree holding strings *)
let t11 = Br {left = Lf; value = "I"; right = Lf}
```

정의할 때는 `'elm`을 적지만, 실제로 사용할 때는 타입을 명시할 필요 없다. 컴파일러가 추론을 해주기 때문이다. 위의 예에서 정수와 문자열의 예를 확인할 수 있다.


타입변수는 여러 부분에서 찾아볼 수 있다. 아래 코드에서 `'a`는 트리에 원하는 어떤 종류의 타입도 사용될 수 있음을 의미한다. 정확히는 이것을 타입스킴(type scheme, 型スキーム)이라고 한다.

```ocaml
# let rec size t = ...;;
val size : 'a tree -> int = <fun>
```

このような，最も一般的な型を 主要型(principal type) と呼び，型推論が主要型を推論してくれることを 主要型性(principal type property) がある，という．

결국 자바와의 차이는, 정의가 아니라 구현에서 타입을 그때마다 적어줄 필요 없이, 컴파일러가 추론해준다는 점이다.

---

C의 경우 타입변수를 지정해줄 수 없다. 그럼 다형성을 어떻게 구현해야하는가?

`void *`를 활용한다.

`void *`는 다른 포인터를 암묵적 형변환을 통해 저장할 수 있다.

즉,

```c
int *i = ...;
void *p = i;   // assignment from int * to void *
```

이런게 가능하다. 주의할 점은, 이것을 포인트로만 활용하지 않고 내용물을 꺼내려 한다면, 그 타입은 void이기 때문에 문제가 될 것이다.

```c
struct tree {
  enum nkind { LEAF, BRANCH } tag;
  union {
    struct leaf { int dummy; } lf;
    struct branch {
      struct tree* left;
      void* value;        // a pointer to something
      struct tree* right;
    } br;
  } dat;
};

// Prepare elements in the heap
int *e1 = (int *)malloc(sizeof(int)); *e1 = 10;
// Construct a tree
// newbranch 함수 생략.
struct tree *t1 = newbranch(newleaf(), e1, newleaf());
```

이런 식으로 구현에 활용된다. 이전까지의 구현들은 `value`로는 정수나 문자열 등을 직접 활용했지만, 이 경우는 그 자리에 포인터를 놓고, 어떤 값을 가리키는 형태로 하여서 다형성을 흉내낸다.

실제 사용은, 형변환을 명시하여 이용한다.

```c
// Functions whose behavior depends on the element type have to be defined separately
void print_inttree(struct tree* t) {
  if (t->tag == LEAF) {
    printf("leaf");
  } else {
    struct branch* b = &t->dat.br;
    int* v = (int*)b->value;
    printf("branch(");
    print_inttree(b->left);
    printf(", %d, ", * v);
    print_inttree(b->right);
    printf(")");
  }
  return;
}
```

포인트는 `int* v = (int*)b->value;`이다. 타입이 다르다고해서 컴파일러가 문제를 일으키지는 않으므로, 코드 작성자가 주의를 기울일 수밖에 없다.
