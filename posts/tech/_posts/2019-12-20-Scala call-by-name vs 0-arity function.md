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
