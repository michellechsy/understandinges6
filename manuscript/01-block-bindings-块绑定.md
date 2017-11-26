# 块绑定

习惯上，变量声明的工作方式已经成为了JavaScript编程里很棘手的一部分。在大多数基于C的编程语言里，变量（或者说绑定）都是在声明出现的时候创建的。然而在JavaScript中，并非如此。变量何时真正被创建取决于你定义它们的方式，而为了更好的控制作用域，ES6提供了一些方式。本章解释了为什么传统的var声明会令人费解，介绍了ES6里引入的块级的绑定，以及提供了一些相应的最佳实践。

## 变量声明和提升

Variable declarations using `var` are treated as if they are at the top of the function (or global scope, if declared outside of a function) regardless of where the actual declaration occurs; this is called *hoisting*. For a demonstration of what hoisting does, consider the following function definition:
不管实际的声明出现在哪里，通过 `var`的变量声明都会被认为是在函数的顶部的（如果在函数外声明则是全局的作用域）-- 即所谓的“提升”。为解释提升的作用，我们从以下函数定义来考虑：
```js
function getValue(condition) {

    if (condition) {
        var value = "blue";

        // other code

        return value;
    } else {

        // 此处value存在，但值为undefined

        return null;
    }

    // 此处value存在，但值为undefined
}
```

如果你不熟悉JavaScript，你可能会认为变量`value`仅会在`condition`条件判断为true的时候创建。然而不管什么情况，实际上变量`value`都会被创建。从底层实现来看，JavaScript引擎会像下边这样改变函数`getValue`：

```js
function getValue(condition) {

    var value;

    if (condition) {
        value = "blue";

        // 其他代码

        return value;
    } else {

        return null;
    }
}
```

`value`的声明被提升到了顶部而它的初始化还是在原来的位置。这意味着变量`value`实际上还是可以从`else`子句被访问的。若从`else`访问，因为该变量还没有被初始化，它的值会是`undefined`。

对于JavaScript的开发新手来说，他们往往需要花费一些时间去习惯声明提升，也会误解这独特的行为会引起缺陷。正因如此，ES6引入了块级作用域配置来更好地控制一个变量的生命周期。

## 块级声明

块级声明是指那些变量在给定块作用域之外不能被访问的声明。块级作用域，或称词法范围（lexical scopes），可以通过以下方式创建：

1. 函数内
1. 块内(由`{`和`}`标识)

块级作用域跟许多基于C的编程语言的工作方式一样，而ES6引入的块级声明是为了给JavaScript带来同样的灵活性（一致性）。

### Let声明

`let`的声明语法和`var`的一样。声明一个变量时基本上你都可以直接把`var`换成`let`，但其作用域会被限制成只在当前的代码块（在后面的内容还会介绍其他的一些区别）。因为`let`声明不会被提升到代码块的顶部，你可能要把`let`声明一直放在代码块的首行，这样他们才能在整个代码块里可用。比如说这个例子：

```js
function getValue(condition) {

    if (condition) {
        let value = "blue";

        // 其他代码

        return value;
    } else {

        // 此处value不存在

        return null;
    }

    // 此处value不存在
}
```

这个版本的`getValue`函数已经很接近那些你期待的基于C的编程语言的工作方式。因为变量`value`是通过`let`而不是`var`声明的，这个声明不会被提到该函数定义的顶部，而且只要`if`代码块被执行完毕，该变量`value`将不能再被访问。如果`condition`返回false，那`value`将永远不会被声明或者初始化。

### 没有再声明

如果一个标识符已经在一个作用域里被定义，那么在那个作用域里的`let`声明里使用该标识符将会报错。比如：

```js
var count = 30;

// 语法错误
let count = 40;
```


在这个例子中，`count`被定义了两次：一次是通过`var`，一次是`let`。因为`let`不会重新定义一个已经存在于同一个作用域的标识符，所以它会抛出错误。然而如果`let`声明创建了一个新的跟在它的所包含域里同名的变量，它不会抛出错误，就像以下代码所展示的：

```js
var count = 30;

// 不会抛出错误
if (condition) {

    let count = 40;

    // 更多代码
}
```

这个`let`声明不会抛出错误，因为它是在`if`语句里创建了一个新的变量`count`，而不是在外层代码块创建。在`if`代码块里，这个新的变量会盖住全局变量`count`，以避免访问它，直到执行离开这个代码块。


### 常量声明

在ES6里你也可以通过`const`语法定义变量。通过`const`声明的变量会被当作“常量”，也就是他们的值一旦被赋予就不能被改变。因为这个原因，每一个`const`变量都必须在声明时初始化，如下例子：

```js
// 有效的常量
const maxItems = 30;

// 语法错误：没有初始化
const name;
```

`maxItems`变量被初始化了，所以它的`const`声明应该可以正常工作。然而当你尝试去执行包含`name`变量的代码程序时，它会引起语法错误，因为`name`没有被初始化。


#### 常量 vs Let声明

常量是块级声明，跟`let`声明一样。那意味着一旦执行离开了常量被声明的代码块，常量就不能再被访问，而且声明也不会被提升。如下例子：

```js
if (condition) {
    const maxItems = 5;

    // 更多代码
}

// 此处maxItems不能被访问
```

In this code, the constant `maxItems` is declared within an `if` statement. Once the statement finishes executing, `maxItems` is not accessible outside of that block.

In another similarity to `let`, a `const` declaration throws an error when made with an identifier for an already-defined variable in the same scope. It doesn't matter if that variable was declared using `var` (for global or function scope) or `let` (for block scope). For example, consider this code:

```js
var message = "Hello!";
let age = 25;

// Each of these would throw an error.
const message = "Goodbye!";
const age = 30;
```

The two `const` declarations would be valid alone, but given the previous `var` and `let` declarations in this case, neither will work as intended.

Despite those similarities, there is one big difference between `let` and `const` to remember. Attempting to assign a `const` to a previously defined constant will throw an error, in both strict and non-strict modes:

```js
const maxItems = 5;

maxItems = 6;      // throws error
```

Much like constants in other languages, the `maxItems` variable can't be assigned a new value later on. However, unlike constants in other languages, the value a constant holds may be modified if it is an object.

#### Declaring Objects with Const

A `const` declaration prevents modification of the binding and not of the value itself. That means `const` declarations for objects do not prevent modification of those objects. For example:

```js
const person = {
    name: "Nicholas"
};

// works
person.name = "Greg";

// throws an error
person = {
    name: "Greg"
};
```

Here, the binding `person` is created with an initial value of an object with one property. It's possible to change `person.name` without causing an error because this changes what `person` contains and doesn't change the value that `person` is bound to. When this code attempts to assign a value to `person` (thus attempting to change the binding), an error will be thrown. This subtlety in how `const` works with objects is easy to misunderstand. Just remember: `const` prevents modification of the binding, not modification of the bound value.

### The Temporal Dead Zone

A variable declared with either `let` or `const` cannot be accessed until after the declaration. Attempting to do so results in a reference error, even when using normally safe operations such as the `typeof` operation in this example:

```js
if (condition) {
    console.log(typeof value);  // ReferenceError!
    let value = "blue";
}
```

Here, the variable `value` is defined and initialized using `let`, but that statement is never executed because the previous line throws an error. The issue is that `value` exists in what the JavaScript community has dubbed the *temporal dead zone* (TDZ). The TDZ is never named explicitly in the ECMAScript specification, but the term is often used to describe why `let` and `const` declarations are not accessible before their declaration. This section covers some subtleties of declaration placement that the TDZ causes, and although the examples shown all use `let`, note that the same information applies to `const`.

When a JavaScript engine looks through an upcoming block and finds a variable declaration, it either hoists the declaration to the top of the function or global scope (for `var`) or places the declaration in the TDZ (for `let` and `const`). Any attempt to access a variable in the TDZ results in a runtime error. That variable is only removed from the TDZ, and therefore safe to use, once execution flows to the variable declaration.

This is true anytime you attempt to use a variable declared with `let` or `const`  before it's been defined. As the previous example demonstrated, this even applies to the normally safe `typeof` operator. You can, however, use `typeof` on a variable outside of the block where that variable is declared, though it may not give the results you're after. Consider this code:

```js
console.log(typeof value);     // "undefined"

if (condition) {
    let value = "blue";
}
```

The variable `value` isn't in the TDZ when the `typeof` operation executes because it occurs outside of the block in which `value` is declared. That means there is no `value` binding, and `typeof` simply returns `"undefined"`.

The TDZ is just one unique aspect of block bindings. Another unique aspect has to do with their use inside of loops.

## Block Binding in Loops

Perhaps one area where developers most want block level scoping of variables is within `for` loops, where the throwaway counter variable is meant to be used only inside the loop. For instance, it's not uncommon to see code like this in JavaScript:

```js
for (var i = 0; i < 10; i++) {
    process(items[i]);
}

// i is still accessible here
console.log(i);                     // 10
```

In other languages, where block level scoping is the default, this example should work as intended, and only the `for` loop should have access to the `i` variable. In JavaScript, however, the variable `i` is still accessible after the loop is completed because the `var` declaration gets hoisted. Using `let` instead, as in the following code, should give the intended behavior:

```js
for (let i = 0; i < 10; i++) {
    process(items[i]);
}

// i is not accessible here - throws an error
console.log(i);
```

In this example, the variable `i` only exists within the `for` loop. Once the loop is complete, the variable is no longer accessible elsewhere.

### Functions in Loops

The characteristics of `var` have long made creating functions inside of loops problematic, because the loop variables are accessible from outside the scope of the loop. Consider the following code:

```js
var funcs = [];

for (var i = 0; i < 10; i++) {
    funcs.push(function() { console.log(i); });
}

funcs.forEach(function(func) {
    func();     // outputs the number "10" ten times
});
```

You might ordinarily expect this code to print the numbers 0 to 9, but it outputs the number 10 ten times in a row. That's because `i` is shared across each iteration of the loop, meaning the functions created inside the loop all hold a reference to the same variable. The variable `i` has a value of `10` once the loop completes, and so when `console.log(i)` is called, that value prints each time.

To fix this problem, developers use immediately-invoked function expressions (IIFEs) inside of loops to force a new copy of the variable they want to iterate over to be created, as in this example:

```js
var funcs = [];

for (var i = 0; i < 10; i++) {
    funcs.push((function(value) {
        return function() {
            console.log(value);
        }
    }(i)));
}

funcs.forEach(function(func) {
    func();     // outputs 0, then 1, then 2, up to 9
});
```

This version uses an IIFE inside of the loop. The `i` variable is passed to the IIFE, which creates its own copy and stores it as `value`. This is the value used by the function for that iteration, so calling each function returns the expected value as the loop counts up from 0 to 9. Fortunately, block-level binding with `let` and `const` in ECMAScript 6 can simplify this loop for you.

### Let Declarations in Loops

A `let` declaration simplifies loops by effectively mimicking what the IIFE does in the previous example. On each iteration, the loop creates a new variable and initializes it to the value of the variable with the same name from the previous iteration. That means you can omit the IIFE altogether and get the results you expect, like this:

```js
var funcs = [];

for (let i = 0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}

funcs.forEach(function(func) {
    func();     // outputs 0, then 1, then 2, up to 9
})
```

This loop works exactly like the loop that used `var` and an IIFE but is, arguably, cleaner. The `let` declaration creates a new variable `i` each time through the loop, so each function created inside the loop gets its own copy of `i`. Each copy of `i` has the value it was assigned at the beginning of the loop iteration in which it was created. The same is true for `for-in` and `for-of` loops, as shown here:

```js
var funcs = [],
    object = {
        a: true,
        b: true,
        c: true
    };

for (let key in object) {
    funcs.push(function() {
        console.log(key);
    });
}

funcs.forEach(function(func) {
    func();     // outputs "a", then "b", then "c"
});
```

In this example, the `for-in` loop shows the same behavior as the `for` loop. Each time through the loop, a new `key` binding is created, and so each function has its own copy of the `key` variable. The result is that each function outputs a different value. If `var` were used to declare `key`, all functions would output `"c"`.

I> It's important to understand that the behavior of `let` declarations in loops is a specially-defined behavior in the specification and is not necessarily related to the non-hoisting characteristics of `let`. In fact, early implementations of `let` did not have this behavior, as it was added later on in the process.

### Constant Declarations in Loops

The ECMAScript 6 specification doesn't explicitly disallow `const` declarations in loops; however, there are different behaviors based on the type of loop you're using. For a normal `for` loop, you can use `const` in the initializer, but the loop will throw a warning if you attempt to change the value. For example:

```js
var funcs = [];

// throws an error after one iteration
for (const i = 0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}
```

In this code, the `i` variable is declared as a constant. The first iteration of the loop, where `i` is 0, executes successfully. An error is thrown when `i++` executes because it's attempting to modify a constant. As such, you can only use `const` to declare a variable in the loop initializer if you are not modifying that variable.

When used in a `for-in` or `for-of` loop, on the other hand, a `const` variable behaves the same as a `let` variable. So the following should not cause an error:

```js
var funcs = [],
    object = {
        a: true,
        b: true,
        c: true
    };

// doesn't cause an error
for (const key in object) {
    funcs.push(function() {
        console.log(key);
    });
}

funcs.forEach(function(func) {
    func();     // outputs "a", then "b", then "c"
});
```

This code functions almost exactly the same as the second example in the "Let Declarations in Loops" section. The only difference is that the value of `key` cannot be changed inside the loop. The `for-in` and `for-of` loops work with `const` because the loop initializer creates a new binding on each iteration through the loop rather than attempting to modify the value of an existing binding (as was the case with the previous example using `for` instead of `for-in`).

## Global Block Bindings

Another way in which `let` and `const` are different from `var` is in their global scope behavior. When `var` is used in the global scope, it creates a new global variable, which is a property on the global object (`window` in browsers). That means you can accidentally overwrite an existing global using `var`, such as:

```js
// in a browser
var RegExp = "Hello!";
console.log(window.RegExp);     // "Hello!"

var ncz = "Hi!";
console.log(window.ncz);        // "Hi!"
```

Even though the `RegExp` global is defined on `window`, it is not safe from being overwritten by a `var` declaration. This example declares a new global variable `RegExp` that overwrites the original. Similarly, `ncz` is defined as a global variable and immediately defined as a property on `window`. This is the way JavaScript has always worked.

If you instead use `let` or `const` in the global scope, a new binding is created in the global scope but no property is added to the global object. That also means you cannot overwrite a global variable using `let` or `const`, you can only shadow it. Here's an example:

```js
// in a browser
let RegExp = "Hello!";
console.log(RegExp);                    // "Hello!"
console.log(window.RegExp === RegExp);  // false

const ncz = "Hi!";
console.log(ncz);                       // "Hi!"
console.log("ncz" in window);           // false
```

Here, a new `let` declaration for `RegExp` creates a binding that shadows the global `RegExp`. That means `window.RegExp` and `RegExp` are not the same, so there is no disruption to the global scope. Also, the `const` declaration for `ncz` creates a binding but does not create a property on the global object. This capability makes `let` and `const` a lot safer to use in the global scope when you don't want to create properties on the global object.

I> You may still want to use `var` in the global scope if you have a code that should be available from the global object. This is most common in a browser when you want to access code across frames or windows.

## Emerging Best Practices for Block Bindings

While ECMAScript 6 was in development, there was widespread belief you should use `let` by default instead of `var` for variable declarations. For many JavaScript developers, `let` behaves exactly the way they thought `var` should have behaved, and so the direct replacement makes logical sense. In this case, you would use `const` for variables that needed modification protection.

However, as more developers migrated to ECMAScript 6, an alternate approach gained popularity: use `const` by default and only use `let` when you know a variable's value needs to change. The rationale is that most variables should not change their value after initialization because unexpected value changes are a source of bugs. This idea has a significant amount of traction and is worth exploring in your code as you adopt ECMAScript 6.

## Summary

The `let` and `const` block bindings introduce lexical scoping to JavaScript. These declarations are not hoisted and only exist within the block in which they are declared. This offers behavior that is more like other languages and less likely to cause unintentional errors, as variables can now be declared exactly where they are needed. As a side effect, you cannot access variables before they are declared, even with safe operators such as `typeof`. Attempting to access a block binding before its declaration results in an error due to the binding's presence in the temporal dead zone (TDZ).

In many cases, `let` and `const` behave in a manner similar to `var`; however, this is not true for loops. For both `let` and `const`, `for-in` and `for-of` loops create a new binding with each iteration through the loop. That means functions created inside the loop body can access the loop bindings values as they are during the current iteration, rather than as they were after the loop's final iteration (the behavior with `var`). The same is true for `let` declarations in `for` loops, while attempting to use `const` declarations in a `for` loop may result in an error.

The current best practice for block bindings is to use `const` by default and only use `let` when you know a variable's value needs to change. This ensures a basic level of immutability in code that can help prevent certain types of errors.
