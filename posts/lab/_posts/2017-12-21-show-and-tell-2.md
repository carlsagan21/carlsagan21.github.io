---
title: Introduction to Javascript type systems
---

<https://www.slideshare.net/SooKim17/sookim-171221>

기본 문법 비교

```javascript
// @flow

// bool
const isDone: boolean = false

// number
const decimal: number = 6

// string
const color: string = "blue"

// array
const list: Array<number> = [1, 2, 3]

// tuple
const x: [string, number] = ["hello", 10]

// object, no dynamic binding
let obj2: { bar: number } = { bar: 1 }
obj2.foo = 2 // Property not found in object literal

// structural typing
interface Named {
  name: string
}
class Person {
  name: string
}
let p: Named
p = new Person() // OK, because of structural typing

// subtype
let a: Named
const y = { name: "Alice", location: "Seattle" } // y's inferred type is { name: string; location: string; }
a = y

// generic
interface NotEmpty<T> {
  data: T
}
let b: NotEmpty<number>

// intersection
type A = { a: number }
type B = { b: boolean }
type C = { c: string }

function method(value: A & B & C) {
  // ...
}

// literal & union
type Color = "Red" | "Green"
type Result = "success" | "danger"
// function getColor(name: Result): Color {
function getColor(name: Result) {
  switch (name) {
    case "success": return "Green"
    case "danger": return "Red"
  }
  // throw new Error("ss")
  name
}
getColor("success") // Works!
getColor("danger")  // Works!
getColor("error")   // Error!

// union refinement
type Success = {kind: "success", value: 1}
type Danger = {kind: "danger", message: "e"}
type Result2 = Success | Danger
function getColor2(name: Result2): Color {
  if (name.kind === "success") {
    return "Green"
  } else if (name.kind === "danger"){
    return "Red"
  }
  else {
    // if ask type of name, not covered -> likely be unreachable
    // but if else do not exist, return type is Color | void. refinement is not working.
    // can make a guess using "not covered" as refinement
    // to ways to solve this
    // 1. return unreacable in else
    // 2. throw error in else
    name
    throw new 
    return "Red"
    // throw new Error("unreachable")
  }
}

type Nat =
  null
  | () => Nat

const zero : Nat = null
const succ : Nat => Nat = n => () => n
const succ1 : Nat => null = n => () => n
const one : Nat = () => zero
const two : Nat = () => one
const two1 : Nat = () => 1

const nat2Int : Nat => number = n => {
  if (n === null) {
    n()
    return 0
  } else {
    n
    return 1 + nat2Int(n())
  }
}

console.log(nat2Int(two))
console.log(nat2Int(succ(succ(zero))))
```

```typescript
// typescript

// bool
const isDone: boolean = false

// number
const decimal: number = 6

// string
const color: string = "blue"

// array
const list: Array<number> = [1, 2, 3]

// tuple
const x: [string, number] = ["hello", 10]

// object, no dynamic binding
let obj2: { bar: number } = { bar: 1 };
obj2.foo = 2; // Property not found in object literal

// structural typing
interface Named {
  name: string;
}
class Person {
  name: string;
}
let p: Named;
p = new Person(); // OK, because of structural typing

// subtype
let a: Named;
let y = { name: "Alice", location: "Seattle" }; // y's inferred type is { name: string; location: string; }
a = y;

// generic
interface NotEmpty<T> {
  data: T;
}
let b: NotEmpty<number>;

// intersection
type A = { a: number };
type B = { b: boolean };
type C = { c: string };

function method(value: A & B & C) {
  // ...
}

// literal & union
type Color = "Red" | "Green"
type Result = "success" | "danger"
// function getColor(name: Result): Color {
function getColor(name: Result) {
  switch (name) {
    case "success": return "Green";
    case "danger": return "Red";
  }
  // throw new Error("ss")
  name
}
getColor("success"); // Works!
getColor("danger");  // Works!
getColor("error");   // Error!

// union refinement
type Success = { kind: "success", value: 1 }
type Danger = { kind: "danger", message: "e" }
type Result2 = Success | Danger
function getColor2(name: Result2): Color {
  name
  if (name.kind === "success") {
    name
    return "Green"
  } else if (name.kind === "danger") {
    name
    return "Red"
  } else {
    // if ask type of name, not covered -> likely be unreachable
    // but if else do not exist, return type is Color | void. refinement is not working.
    // can make a guess using "not covered" as refinement
    name
    return "Red"
  }
}

type Nat =
  null
  | (() => Nat)

const zero: Nat = null
const succ: (n: Nat) => Nat = n => () => n
const succ1: (n: Nat) => null = n => (() => n)
const one: Nat = () => zero
const two: Nat = () => one
const two1: Nat = () => 1

const nat2Int: (n: Nat) => number = n => {
  if (n === null) {
    n()
    return 0
  } else {
    n
    return 1 + nat2Int(n())
  }
}

console.log(nat2Int(two))
console.log(nat2Int(succ(succ(zero))))
```

```diff
diff --git a/points.ts b/points.ts
index fe11aad..895c042 100644
--- a/points.ts
+++ b/points.ts
@@ -1,4 +1,4 @@
-// typescript
+// @flow
 
 // bool
 const isDone: boolean = false;
@@ -31,7 +31,7 @@ p = new Person(); // OK, because of structural typing
 
 // subtype
 let a: Named;
-let y = { name: "Alice", location: "Seattle" }; // y's inferred type is { name: string; location: string; }
+const y = { name: "Alice", location: "Seattle" }; // y's inferred type is { name: string; location: string; }
 a = y;
 
 // generic
@@ -78,23 +78,27 @@ function getColor2(name: Result2): Color {
     // if ask type of name, not covered -> likely be unreachable
     // but if else do not exist, return type is Color | void. refinement is not working.
     // can make a guess using "not covered" as refinement
-    name;
-    return "Red";
+    // to ways to solve this
+    // 1. return unreacable in else
+    // 2. throw error in else
+    name
+    return "Red"
+    // throw new Error("unreachable")
   }
 }
 
 type Nat =
   null
-  | (() => Nat)
+  | () => Nat
 
 const zero: Nat = null
-const succ: (n: Nat) => Nat = n => () => n
-const succ1: (n: Nat) => null = n => (() => n)
+const succ: Nat => Nat = n => () => n
+const succ1: Nat => null = n => () => n
 const one: Nat = () => zero
 const two: Nat = () => one
 const two1: Nat = () => 1
 
-const nat2Int: (n: Nat) => number = n => {
+const nat2Int: Nat => number = n => {
   if (n === null) {
     n()
     return 0
```
