---
title: 꼬리재귀
---

핵심은, 불린 함수의 입장에서 부른 함수의 위치가 어디인가를 저장한 레퍼런스 주소가 쌓이고 쌓여서 스텍오버플로우가 나는 문제를 해결하기 위함

깊게 깊게 불려 들어가는데, 돌아가는 위치를 자신이 돌아가는 위치가 아닌, 최초 함수가 호출된 위치로 지정하면 스텍이 쌓이지 않음. 즉, 따로 하나하나 함수의 불린 위치를 저장할 필요가 없음.

따라서 이전의 함수의 메모리는(로컬 변수, 호출위치 등을 저장하는) 필요없으므로 다음 함수가 불리는 순간 해방됨.

즉, 제일 깊은 곳의 함수에서 리턴된 값이 최초 함수 호출 위치로 다이렉트로 넘어감.


따라서 필요한 것은? 함수가 불린 위치가, 더이상의 작업을 하지 않고 다이렉트로 리턴으로 결과값을 넘겨줘야 함.

즉 이전의 함수를 새롭게 불린 함수로 대체할 수 있으면, 됨.

꼬리호출
꼬리재귀
꼬리재귀함수

반복문으로 변환 가능.

```c
// 꼬리재귀
#include <stdio.h>
#include <assert.h>

int gcd(int n, int m) {
  assert(n>0 && m>0);
  if (n == m) { return n; }
  else if (n > m) { return gcd(n-m, m); }
  else { return gcd(m-n, n); }
}

int main(void) {
  int a = 540000001;
  int b = 2;
  printf("gcd(%d, %d) = %d\n", a, b, gcd(a, b));
  return 0;
}
```

```cpp
// 꼬리재귀 아님
#include <stdio.h>

int sum(int n) {
  if (n == 1) then return 1
  else return n + sum(n - 1);
}

int main(void) {
  int n = 260000;
  printf("sum(%d) = %d\n", n, sum(n));

  return 0;
}
```

```c
// 꼬리재귀 변환
int sum2(int n, int m) {
  if (n == 1) then { return m + 1; }
  else { return sum2(n-1, m + n); }
}

...

```
