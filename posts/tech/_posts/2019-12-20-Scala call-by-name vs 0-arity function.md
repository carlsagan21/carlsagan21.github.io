Call by name and 0 arity functions are interchangable. i.e.

```scala
object CallByNameVsFunction0 {
  def f0Arity(f: () => Int): Int = f()
  def fCallByName(f: => Int): Int = f
  f0Arity(() => 1)
  fCallByName(1)
}
```

Behaves same.

But is there performance difference or bytecode diff?

I decompiled compiled bytecode in Java to make things clear.

```java
public final class CallByNameVsFunction0$ {
   public static final CallByNameVsFunction0$ MODULE$ = new CallByNameVsFunction0$();

   static {
      MODULE$.f0Arity(() -> {
         return 1;
      });
      MODULE$.fCallByName(() -> {
         return 1;
      });
   }

   public int f0Arity(final Function0 f) {
      return f.apply$mcI$sp();
   }

   public int fCallByName(final Function0 f) {
      return f.apply$mcI$sp();
   }

   private CallByNameVsFunction0$() {
   }

   // $FF: synthetic method
   private static Object $deserializeLambda$(SerializedLambda var0) {
      return Class.lambdaDeserialize<invokedynamic>(var0);
   }
}
```

So, they are identical for this case.
But it is a super simple case, so it is possible that compiler do some more magics with complicated cases. But the case shows it can be compiled to identical bytecode.
This make me understand how call-by-name actually works.


---

제가 원래 Call-by-name이라는 개념에 익숙지 않아서 그런 것도 있겠습니다만, 코드를 읽다가 다음과 같은 코드를 만났다고 가정해보면

def naiveFunction(futureVal : Future[T]): IO[T]
def cbnFunction(futureExp : => Future[T]): IO[T]
def arity0Function(futureThunk : () => Future[T]): IO[T]

// lets assume onErrorRetry is defined. retry when exception happens. And lets assume given future(or whatever exp with side effect) can throw error

val a = naiveFunction(future).onErrorRetry() // future explicitly excuted only once

val b = cbnFunction(future).onErrorRetry() // if the function parameter is call-by-name, can be evaluated multiple times. but if is not(like naiveFunction), only once it will be evaluated.

val c = arity0Function(() => future).onErrorRetry() // explicitly can be excuted multiple times

A와 B가 함수를 부르는 입장에서 형태가 동일하지만, 실제 동작은 다릅니다. 즉 호출하는 함수가 Call-by-name인지 아닌지에 따라서 Retry 동작이 달라지게 된다고 생각합니다. 따라서 함수 호출시 이런 문제를 피하기위해 side-effect가 있는 파라미터를 넘길때에는 불리는 함수의 타입을 매번 확인해야 한다고 생각하고 있었습니다.
