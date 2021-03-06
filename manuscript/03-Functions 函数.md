# 函数（Functions）

函数是所有编程语言里的一个重要部分，在ES6之前，JavaScript里的函数自创造以来就很少有变动。这曾遗留下了一大堆待解决的问题和有着细微差异的代码行为——容易犯错而且经常需要写更多的代码，仅仅是为了实现非常基本的行为。

考虑到来自JavaScript开发者们的抱怨和请求，ES6中的函数有了重大突破。这个结果就是在ES5的函数上所作出的一些渐进式改进使得JavaScript的编程更少出错和更健壮。

## 有默认参数值的函数

Javascript中的函数是唯一的，不管函数定义中声明了多少个参数，他们允许任意数量的参数传入到函数中。这允许你在定义函数时可以通过给不需要提供的参数填充默认值来处理不同数量的参数。本章顺着`arguments`对象的一些重要信息，使用表达式作为参数和另一种TDZ，介绍了默认参数在ES6中以及ES6之前是如何工作的。

### 在ES5中模拟参数默认值

在ES5及早期版本中，你很可能会用以下方式来创建带有默认值参数的函数：

```js
function makeRequest(url, timeout, callback) {

    timeout = timeout || 2000;
    callback = callback || function() {};

    // 函数的剩下部分

}
```

在这个例子中，如果没有提供任何参数，由于`timeout`和`callback`已赋了默认值，它们实际上是可选的。当第一个操作数为false值时，或逻辑运算符（`||`）永远都会返回第二个操作数。由于没有被明确提供的已命名参数会被设成`undefined`，或逻辑运算符经常被用来为缺失的参数提供默认值。然而，这种方式有个缺点，`timeout`的一个有效值实际上可能为`0`，但它会因`0`是false的而被替换成`2000`。

在那种情况下，一个更为安全的做法是用`typeof`检查参数的类型，如：

```js
function makeRequest(url, timeout, callback) {

    timeout = (typeof timeout !== "undefined") ? timeout : 2000;
    callback = (typeof callback !== "undefined") ? callback : function() {};

    // 函数的剩下部分

}
```

虽然这方法更安全，但对于一个很基本的操作，它仍需要很多额外的代码。因这是个常见的模式，很多常用的JavaScript库都使用了相类似的模式。

### ES6中的参数默认值

ES6为参数的默认值提供了更简便的方法，即为不常被传入的参数提供初始化。比如说：

```js
function makeRequest(url, timeout = 2000, callback = function() {}) {

    // 函数的剩下部分

}
```

这个函数只有第一个参数是一直需要被传入的。另外两个参数有默认值，因为你不需要添加任何代码检查空值，这就使得函数体更小。

当传入全部参数来调用`makeRequest()`，函数中的参数默认值不会被使用。比如说：

```js
// 使用默认的timeout和callback值
makeRequest("/foo");

// 使用默认的callback值
makeRequest("/foo", 500);

// 都不使用默认值
makeRequest("/foo", 500, function(body) {
    doSomething(body);
});
```

在以上三次调用中，之所以`"/foo"`一直被传入到`makeRequest()`里，是因为ES6认为`url`是必须的，而带有默认值的另外两个参数则是可选的。

为任何参数指定默认值都是可以的，包括那些在函数定义中出现在没有默认值的参数之前的参数。如以下例子是可行的：

```js
function makeRequest(url, timeout = 2000, callback) {

    // 函数的剩下部分

}
```

在这个例子中，`timeout`的默认值只会在没有第二个参数值传入或者第二个参数值被明确为`undefined`时被使用。如以下例子：

```js
// 使用timeout的默认值
makeRequest("/foo", undefined, function(body) {
    doSomething(body);
});

// 使用timeout的默认值
makeRequest("/foo");

// 不使用timeout的默认值
makeRequest("/foo", null, function(body) {
    doSomething(body);
});
```

在默认参数值的这些例子中，`null`值被认为是有效的，也就是说，在第三个`makeRequest()`的调用中，`timeout`的默认值不会被使用。

### 参数默认值是如何影响实参对象的

要记住，`arguments`对象的行为会因参数默认值的出现而变得不同。在ES5非严格模式下，`arguments`对象会反映出一个函数已命名参数的变化。以下代码便阐述了这是如何起作用的：

```js
function mixArgs(first, second) {
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d";
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a", "b");
```

输出：

```
true
true
true
true
```

在非严格模式下，为了体现已命名参数的变化，`arguments`对象会永远被更新。因此，当`first`和`second`被赋予新值时，`arguments[0]`和`arguments[1]`会被相应的更新，以致于所有的`===`比较结果都是`true`。

然而，ES5的严格模式消除了`arguments`对象这一令人疑惑的一面。在严格模式下，`arguments`对象并不会反映已命名参数的变化。以下是在严格模式下再一次执行`mixArgs()`函数的情况：

```js
function mixArgs(first, second) {
    "use strict";

    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d"
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a", "b");
```

调用`mixArgs()`的输出：

```
true
true
false
false
```

这一次，改变`first`和`second`并不会影响到`arguments`，因此输出结果如预期的一样。

然而，在使用了ES6参数默认值的函数中，不管函数是否声明为严格模式，`arguments`对象的表现行为都会和ES5严格模式下的行为一致。参数默认值的出现使得`arguments`对象和已命名参数保持了分离。如何使用`arguments`对象，这是一个很细微却也重要的细节。看看以下例子：

```js
// 非严格模式下
function mixArgs(first, second = "b") {
    console.log(arguments.length);
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d"
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a");
```

输出：

```
1
true
false
false
false
```

在这个例子中，因为只有一个实参被传到了`mixArgs()`中，`arguments.length`是1。那也意味着`arguments[1]`是`undefined`，这是预期的行为。那也意味着`first`等于`arguments[0]`。改变`first`和`second`对`arguments`没有影响。在严格和非严格模式下这行为一样出现，因此你可以依赖`arguments`来体现最开始的调用情况。

### 参数的默认表达式

也许参数默认值最有趣的特性是默认值不一定要是基本类型的值。譬如，你可以像这样执行一个函数去获取参数默认值：

```js
function getValue() {
    return 5;
}

function add(first, second = getValue()) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 6
```

此处，如果最后一个参数没有被提供，函数`getValue()`被调用读取正确的默认值。要记得`getValue()`只会在没有第二个实参的情况下被调用，而不是在解析函数声明的时候。那就是说，如果`getValue()`被写得不同，它就有可能返回不一样的值。比如：

```js
let value = 5;

function getValue() {
    return value++;
}

function add(first, second = getValue()) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 6
console.log(add(1));        // 7
```

在此例中，`value`一开始是5，在每一次调用`getValue()`后递增。因为`value`是递增的，所以第一次调用`add(1)`返回6，而第二次则返回了7。因为`second`的默认值只在函数执行时被计算，该函数值随时都会变。

W> 使用函数调用作为参数默认值时一定要小心谨慎。如果你忘记了那对括号，比如在上例中`second = getValue`，你把一个引用传给了函数，而不是函数调用的结果。

这种行为引进了另一个有趣的能力。你可以用前一个参数作为后面参数的默认值。比如：

```js
function add(first, second = first) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 2
```

在这代码中，参数`second`用了`first`作为默认值，也就是说，只传入一个参数会导致两个参数有一样的值。因此，`add(1, 1)`会和`add(1)`一样返回2。更进一步，你可以像以下例子一样，把`first`传入到一个函数中去获取`second`的值。

```js
function getValue(value) {
    return value + 5;
}

function add(first, second = getValue(first)) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 7
```

这例子把`second`的值设成了`getValue(first)`的返回值，所以`add(1, 1)`还是返回了2而`add(1)`返回了7 (1 + 6)。

从参数默认赋值来引用参数的这种能力仅仅适用于其前面的参数，所以更早出现的参数无法访问其后来的参数。比如：

```js
function add(first = second, second) {
    return first + second;
}

console.log(add(1, 1));         // 2
console.log(add(undefined, 1)); // 抛出错误
```


`second`是在`first`之后定义的，因而它不能用来做默认值，所以`add(undefined, 1)`的调用抛出了错误。 重温暂时性死区内容对理解这情况发生的原因很重要。

### 参数默认值的暂时性死区（TDZ）

因其涉及到`let`和`const`，我们在第一章便介绍了TDZ，而参数默认值同样有参数不能被访问的TDZ。与`let`声明类似，每一个参数都会创建一个新的标识符绑定，且不能在初始化之前被引用，否则会抛出错误。参数初始化发生在函数被调用之时，要么是给参数传入一个值，要么使用参数默认值。

为了观察参数默认值的TDZ，我们再看看来自“参数默认表达式”中的例子：

```js
function getValue(value) {
    return value + 5;
}

function add(first, second = getValue(first)) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 7
```

`add(1, 1)`和`add(1)`的调用实际上执行了以下代码来创建`first`和`second`的参数值：

```js
// add(1, 1)调用的JavaScript描述
let first = 1;
let second = 1;

// add(1)调用的JavaScript描述
let first = 1;
let second = getValue(first);
```

当函数`add()`第一次被执行时，`first`和`second`的绑定就被加入到了特定的参数TDZ（类似于`let`的行为表现）。因此当`first`一直都会被初始化的时候，`second`可以以`first`的值来初始化，可反之不然。现在，假设我们重写`add()`函数：

```js
function add(first = second, second) {
    return first + second;
}

console.log(add(1, 1));         // 2
console.log(add(undefined, 1)); // 抛出错误
```

在这个例子中，`add(1, 1)`和`add(undefined, 1)`的调用现在背地里匹配了以下代码：

```js
// add(1, 1)调用的JavaScript描述
let first = 1;
let second = 1;

// add(undefined, 1)调用的JavaScript描述
let first = second;
let second = 1;
```

此例中，当`first`初始化时，`second`还没有被初始化，所以`add(undefined, 1)`的调用会抛出错误。在那时，`second`在TDZ里，因而任何对`second`的引用都会抛出错误。这也反映了在第一章中讨论的`let`的行为。

I> 函数的参数有着它们自己的作用域和TDZ，这和函数体的作用域是分开的。也就意味着，一个参数的默认值不能访问函数体里的任意变量。

## 使用未命名参数

目前为止，本章的所有例子都只涵盖了已在函数定义里命名的参数。然而，JavaScript函数对于可传递参数的数量和已定义的命名参数的数量都没有限制。你可以一直传入比正式指定的参数个数更少或者更多的参数。参数默认值很清晰地说明了一个函数可以接收更少的参数，而ES6同样力图更好地解决比定义参数传入更多参数的问题。

### ES5中的未命名参数

早期的时候，JavaScript提供了一个方式来检验有没有必要单独定义每一个参数传给函数参数。经过检验，虽然大多数情况下`arguments`都可以良好地工作，但该对象用起来会显得有些笨重。比如说，检查检查以下用来检验`arguments`对象的代码：

```js
function pick(object) {
    let result = Object.create(null);

    // 从第二个参数开始
    for (let i = 1, len = arguments.length; i < len; i++) {
        result[arguments[i]] = object[arguments[i]];
    }

    return result;
}

let book = {
    title: "Understanding ECMAScript 6",
    author: "Nicholas C. Zakas",
    year: 2015
};

let bookData = pick(book, "author", "year");

console.log(bookData.author);   // "Nicholas C. Zakas"
console.log(bookData.year);     // 2015
```

该函数模仿了*Underscore.js*库的方法`pick()`，其用于返回一个给定对象的一个拷贝，该对象包含原始对象属性的一个特定子集。该例只定义了一个参数，且期望第一个参数就是用来拷贝属性的对象。其它每一个参数都是应当被用来拷贝到结果上的属性的名字。

对于`pick()`函数，还有一些东西需要注意。首先，该函数可以处理不止一个参数，这一点完全不明显。你可以定义多几个参数，但你会一直缺乏指示说这个函数可以接收任意个数的参数。第二，因为第一个参数是已命名且可以直接使用的，当你查询要拷贝的属性时，你必须从索引1而不是0开始轮询`arguments`对象。要知道虽然在`arguments`上使用适当的索引并不一定很难，但它是另一件需要记住的事。

ES6引入了剩余参数来解决这些问题。

### 剩余参数（Rest Parameters）

一个*剩余参数*由三个点（`...`）紧跟着一个命名参数来表示。该命名参数变成了一个包含了所有传入函数的剩余参数的`Array`———这便是“剩余”的由来。比如，`pick()`可以用剩余参数来改写，像这样：

```js
function pick(object, ...keys) {
    let result = Object.create(null);

    for (let i = 0, len = keys.length; i < len; i++) {
        result[keys[i]] = object[keys[i]];
    }

    return result;
}
```

在这个版本的函数中，`keys`是一个包含了所有在`object`之后传入的所有参数的剩余参数（不像`arguments`那样包含了全部参数）。那意味着，你可以放心的从头开始迭代`keys`。而作为意外收获，你可以很清楚地从函数中看出该函数可以处理任意个数的参数。

I> 剩余参数并不影响一个函数用来表示函数参数个数的`length`属性。此例中`pick()`的`length`值是1，因为只有`object`被计入该值。

#### 剩余参数的局限性

剩余参数有两大局限性。其一，只能有一个剩余参数，而且必须是最后一个参数。比如，以下代码不会生效：

```js
// 语法错误：剩余参数后面不能再有命名参数
function pick(object, ...keys, last) {
    let result = Object.create(null);

    for (let i = 0, len = keys.length; i < len; i++) {
        result[keys[i]] = object[keys[i]];
    }

    return result;
}
```

此处，参数`last`跟在了剩余参数`keys`后面，导致了语法错误。

第二个局限性，剩余参数不能用于一个对象的字面setter。也就是说，这段代码也会导致语法错误：

```js
let object = {

    // 语法错误：剩余参数不能用于setter
    set name(...value) {
        // do something
    }
};
```

对象的字面setter限制了只能有一个参数，因此导致了该局限性。从定义上来说，剩余参数是无数个参数，因为这个原因，它不能用于对象的setter。

#### 剩余参数是如何影响arguments对象的

在ECMAScript中剩余参数一开始是被设计用于取代`arguments`的。起初，ES4确实是去掉了`arguments`，并且加入了剩余参数来允许无限个参数可以传入到函数里。ES4没有被发布，，但这个想法却被保留了并且在ES6中重新被引入，尽管`arguments`在该语言中并没有被去掉。

`arguments`对象和剩余参数一起使用来表示函数被调用时所传入的参数。一如以下程序所阐述的：

```js
function checkArgs(...args) {
    console.log(args.length);
    console.log(arguments.length);
    console.log(args[0], arguments[0]);
    console.log(args[1], arguments[1]);
}

checkArgs("a", "b");
```

`checkArgs()`调用的输出：

```
2
2
a a
b b
```

且不说剩余参数的用法，`arguments`对象总能正确地表示被传入一个函数的所有参数。 

以上便是你在开始使用剩余参数之前需要知道的。


## 函数构造器更强的能力

`Function`构造函数是JavaScript中不常用的一部分，它允许你动态地创建一个新函数。构造函数中的参数便是函数和函数体中的参数，都是字符串。请看例子：

```js
var add = new Function("first", "second", "return first + second");

console.log(add(1, 1));     // 2
```

ES6扩大了`Function`构造函数的能力，它允许参数默认值和剩余参数。你只需要在参数名后加上一个等号和值。比如：

```js
var add = new Function("first", "second = first",
        "return first + second");

console.log(add(1, 1));     // 2
console.log(add(1));        // 2
```

此例中，当只传入一个参数时，`second`会被赋予`first`的值。语法和不用`Function`的函数声明式一样的。

对于剩余参数，只需在最后一个参数前加上`...`即可。像这样：

```js
var pickFirst = new Function("...args", "return args[0]");

console.log(pickFirst(1, 2));   // 1
```

这代码创建了一个只使用剩余参数且返回传入的第一个参数的函数。

默认值和剩余参数的加入保证了`Function`拥有所有和创建函数的声明形式一样的能力。

## 散布操作符（The Spread Operator）

散布操作符和剩余参数密切相关。剩余参数允许你指定多个相互独立的参数应该组合成一个数组，而散布操作符则允许你指定一个数组应该被拆分且它的元素应作为参数分别传给函数。相信看，方法`Math.max()`接收任意个数的参数，然后返回最大值。以下是该方法的一个简单例子：

```js
let value1 = 25,
    value2 = 50;

console.log(Math.max(value1, value2));      // 50
```

如此例所示，当你要处理的只是两个值，`Math.max()`使用起来是很简单的。俩值传入后返回了最大值。但若你正在追踪一个数组里的值，现在你想要找到最大值？方法`Math.max()`并不允许你传入一个数组，所以在ES5及早期版本中，你会被卡住，要么你自己手动搜索该数组，要么如下方法使用`apply()`：

```js
let values = [25, 50, 75, 100]

console.log(Math.max.apply(Math, values));  // 100
```

此方案是可行的，但在这种方式上使用`apply()`却有点令人费解。实际上，这额外增加的语法看起来好像有点混淆代码真正的意图。

ES6的散布操作符可以让这种情况变得非常简单。除了调用`apply()`，你还可以像在剩余参数中使用的模式一样，在数组前加上`...`,然后将数组直接传入`Math.max()`。JavaScript引擎会将数组拆分成独立的参数并传入到函数中。就像这样：

```js
let values = [25, 50, 75, 100]

// 相当于
// console.log(Math.max(25, 50, 75, 100));
console.log(Math.max(...values));           // 100
```

此时，`Math.max()`的调用看起来更常见一些，也避免了对一个简单的数学上的操作指定`this`绑定的复杂性（上例中`Math.max.apply()`的第一个参数）。

你也可以将散布操作符和其它参数混合搭配使用。假设你想从`Math.max()`返回的最小值为0（以防万一数组里混入了负数）。你可以单独传入该参数，而对其它参数仍使用散布操作符，如下：

```js
let values = [-25, -50, -75, -100]

console.log(Math.max(...values, 0));        // 0
```

在这个例子中，传入`Math.max()`的最后一个参数为`0`，而其前的其它参数都用散布操作符传入。

对所传参数使用散布操作符使得对函数参数使用数组变得更容易。你很有可能会发现在大多数情况下它都会是`apply()`的一个很好的替代品。

目前为止，你已看见参数默认值和剩余参数的用法。除此之外，在ES6中，你还可以将这两个参数类型应用到JavaScript的`Function`构造函数中。

## ES6的name属性

定义一个函数的方式是多种多样的，因此要在JavaScript中识别一个函数可能是很有挑战性的。此外，匿名函数表达式的普遍性使得调试有些更困难，它经常导致堆栈信息难以读懂和解析。基于这些原因，ES6给所有的函数都增加了`name`属性。

### 选择合适的名字

在一个ES6的程序中，所有的函数都会有一个合适的`name`属性值。以下例子展示了该属性在实际应用中的效果，该例子展示了一个函数和函数表达式，并打印两者的`name`属性：

```js
function doSomething() {
    // ...
}

var doAnotherThing = function() {
    // ...
};

console.log(doSomething.name);          // "doSomething"
console.log(doAnotherThing.name);       // "doAnotherThing"
```

这段代码中，因为`doSomething()`是一个函数声明，它的`name`属性值为`"doSomething"`。而对于匿名函数表达式`doAnotherThing()`，其`name`属性值则为其对应的变量的名字`"doAnotherThing"`。

### name属性的特殊情况

合适函数声明和函数表达式的名字都很容易找到，然后ES6更进一步地保证*所有*函数都有合适的名字。以下程序便很好地阐述了这一点：

```js
var doSomething = function doSomethingElse() {
    // ...
};

var person = {
    get firstName() {
        return "Nicholas"
    },
    sayName: function() {
        console.log(this.name);
    }
}

console.log(doSomething.name);      // "doSomethingElse"
console.log(person.sayName.name);   // "sayName"

var descriptor = Object.getOwnPropertyDescriptor(person, "firstName");
console.log(descriptor.get.name); // "get firstName"
```

此例中，因函数表达式有它自己的一个名字，而且该名字优先级高于该函数所被分配的变量名，所以`doSomething.name` 值为`"doSomethingElse"`。而`person.sayName()`的`name`属性则是从对象字面量解释而来的值`"sayName"`。类似地，`person.firstName`实际上是一个getter函数，所以它的名字是用来体现区别的`"get firstName"`。setter函数同样地以`"set"`开头（getter和setter函数都必须通过`Object.getOwnPropertyDescriptor()`来获取）。

函数的名字还有其它一些特别情况。通过`bind()`创建的函数名字以`"bound"`开头，而`Function`构造函数创建的函数名字则以`"anonymous"`开头。如这个例子：

```js
var doSomething = function() {
    // ...
};

console.log(doSomething.bind().name);   // "bound doSomething"

console.log((new Function()).name);     // "anonymous"
```

一个绑定函数的`name`总是用以绑定的函数的`name`，并以字符串`"bound "`前缀修饰。因此，`doSomething()`绑定的版本是`"bound doSomething"`。

记住，任何函数的`name`值不一定都指向具有相同名字的变量。`name`属性只是为帮助调试提供信息，所以无法通过`name`的值来获取一个函数的引用。

## 理清函数的双重目的

在ES5及早期版本中，函数是为通过或不通过`new`调用而服务的。当通过`new`使用时，函数内部的`this`值是一个新的对象，且此新对象会被返回。如下例阐述：

```js
function Person(name) {
    this.name = name;
}

var person = new Person("Nicholas");
var notAPerson = Person("Nicholas");

console.log(person);        // "[Object object]"
console.log(notAPerson);    // "undefined"
```

创建`notAPerson`时，不通过`new`调用`Person()`导致了`undefined`（而且在非严格模式下它会在global对象中设一个`name`属性）。一如JavaScript程序中常见的，`Person`的大写才是真正地表明了该函数旨在通过`new`来调用。而这令人混乱的函数的双重角色使得ES6中引入了一些改变。

JavaScript中对函数有两个不同内部专用的方法：`[[Call]]`和`[[Construct]]`。当一个函数不通过`new`调用时，`[[Call]]`会被执行，就好像它本来就在代码中，它会执行函数体。而当一个函数通过`new`来调用时，`[[Construct]]`方法就会被调用。`[[Construct]]`负责创建一个新对象，也就是*新目标*，然后对新目标设置`this`并调用函数体。带有`[[Construct]]`方法的函数被称为*构造函数*。

I> 要记住，并不是所有的函数都有`[[Construct]]`，因此并不是所有的函数都可以通过`new`来调用。如在“箭头函数”一章中介绍的，箭头函数没有`[[Construct]]`方法。

### 决定ES5中一个函数是如何被调用的

在ES5中决定一个函数是否通过`new`（即通过构造函数）来调用，最普遍的一个做法是使用`instanceof`，如：

```js
function Person(name) {
    if (this instanceof Person) {
        this.name = name;   // 使用new
    } else {
        throw new Error("Person必须与new一起使用。")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person("Nicholas");  // 抛出错误
```

此处，`this`值被检查是否是该构造函数的一个实例，如果是，则正常继续执行。如果`this`不是`Person`的一个实力，则抛出一个错误。`[[Construct]]`方法创建了`Person`的一个新实例并把它指派给了`this`，因此这是可行的。遗憾的是，这方法并不完全可靠，因为不通过`new`，`this`也可以是`Person`的一个实例。如下例：

```js
function Person(name) {
    if (this instanceof Person) {
        this.name = name;   // 使用new
    } else {
        throw new Error("Person必须与new一起使用。")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person.call(person, "Michael");    // 起作用！
```

`Person.call()`的调用把变量`person`作为第一个参数传入，即`this`被设为了`person`而不是`Person`函数。对该函数而言，无法判断这是通过`new`调用而来的。

### 元属性new.target

为了解决这个问题，ES6引入了*元属性*`new.target`。元属性是一个非对象的一个属性，它提供了关于它的目标（如`new`）的额外信息。当一个函数的`[[Construct]]`方法被调用，`new.target`便会被`new`操作符的目标所填充。典型地，那目标一般都是最近创建的对象实例的构造函数，该构造函数将成为函数体内的`this`。如果`[[Call]]`被执行，那么`new.target`则为`undefined`。

这个新的元属性允许你通过`new.target`是否被定义来安全地检测一个函数是否通过`new`来调用。如下：

```js
function Person(name) {
    if (typeof new.target !== "undefined") {
        this.name = name;   // 使用new
    } else {
        throw new Error("Person必须与new一起使用。")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person.call(person, "Michael");    // 错误!
```

通过使用`new.target`而不是`this instanceof Person`，此时若不通过`new`来调用`Person`，其构造函数便会正确地抛出一个错误。

你也可以检查`new.target`是否被一个特殊的构造函数调用。比如这个例子：

```js
function Person(name) {
    if (new.target === Person) {
        this.name = name;   // 使用new
    } else {
        throw new Error("Person必须与new一起使用。")
    }
}

function AnotherPerson(name) {
    Person.call(this, name);
}

var person = new Person("Nicholas");
var anotherPerson = new AnotherPerson("Nicholas");  // 错误!
```

在这段代码中，为了正确运行，`new.target`必须是`Person`。当`new AnotherPerson("Nicholas")`被调用时，随后对`Person.call(this, name)`的调用将会抛出一个错误，因为在`Person`构造函数中`new.target`是`undefined`（它没有通过`new`来调用）。

W> 警告：在函数外使用`new.target`会报语法错误。

通过增加`new.target`，ES6帮助澄清了关于函数调用的一个歧义。跟着这个主题继续，ES6同样指出了该语言中以前就存在的另一个歧义部分：在块中声明函数。

## 块级函数

在ES3及早期版本，在一个块（*块级函数*）内出现函数声明，从技术上来说会出现语法错误，然而所有的浏览器仍然支持。遗憾的是，每一个允许该语法的浏览器表现都有点不一样。因此，在块内避免函数声明被认为是最佳实践（最好的做法是使用函数表达式）。

在一次控制这种不兼容行为的尝试中，ES5严格模式引入了一个错误，如下方式，不管何时，只要在块内使用函数声明便抛出错误：

```js
"use strict";

if (true) {

    // ES5会抛出语法错误，ES6则不会
    function doSomething() {
        // ...
    }
}
```

在ES5中，该代码抛出了语法错误。而在ES6中，函数`doSomething()`被认为是一个块级声明，且在其被声明的同一个块内可以被访问和调用。比如：

```js
"use strict";

if (true) {

    console.log(typeof doSomething);        // "function"

    function doSomething() {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);            // "undefined"
```

块级函数会被提升到其定义所在的块的头部，因此此代码中即便它出现在函数声明前，`typeof doSomething`也会返回`"function"`。一旦`if`块执行完毕，`doSomething()`便不再存在。

### 决定何时使用块级函数

块级函数与`let`函数表达式类似，一旦执行流离开了函数定义所在的代码块，该定义便会被移除。主要区别在于块级函数会被提升到所在块的顶部，而使用`let`定义的函数表达式则不会被提升。如例阐述：

```js
"use strict";

if (true) {

    console.log(typeof doSomething);        // 抛出错误

    let doSomething = function () {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);
```

此处，因为`let`语句还没有执行，以致于`doSomething`还在TDZ里，所以当执行`typeof doSomething`时代码执行便停止了。得知这些区别后，基于你是否想要提升的行为，你可以选择是否适用块级函数或者`let`表达式。

### 非严格模式下的块级函数

ES6同样允许非严格模式下使用块级函数，但行为会有些许不同。这些声明会被一直提升至包含函数或者全局环境，而不是把它们提升到块的顶部。比如：

```js
// ES6行为
if (true) {

    console.log(typeof doSomething);        // "function"

    function doSomething() {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);            // "function"
```

此例中，`doSomething()`被提升至全局作用域，因此它仍存在于`if`代码块外部。为了移除这种从前就存在了的不兼容的浏览器行为，ES6规范了这种行为。所以，所有ES6的 运行时都应该有同样的行为。

虽然允许块级函数能改善你在JavaScript中定义函数的能力，但ES6也引入了一种全新的方式来声明函数。

## 箭头函数

ES6中最有趣的新功能之一便是*箭头函数*。顾名思义，箭头函数就是使用“箭头”（`=>`）这种新语法来定义的函数。然而，箭头函数的表现行为与传统的JavaScript函数有着重大差别：

* **没有`this`，`super`，`arguments`和`new.target`绑定** - 一个函数内的`this`，`super`，`arguments`和`new.target`的值是由最近的包含非箭头函数得到的（`super`将在第四章讲解）。
* **不能通过`new`调用** - 箭头函数没有`[[Construct]]`方法，因此不能用作构造函数。当和`new`一起使用时，箭头函数将会抛出错误。
* **没有原型** - 因在箭头函数中不能使用`new`，原型便显得没有必要了。箭头函数中不存在着`prototype`属性。
* **不能改变`this`** - 函数内部的`this`值不能被改变。在整个函数的生命周期中它总是保持一致的。
* **没有`arguments`对象** - 因为箭头函数没有`arguments`绑定，你必须依赖命名参数和剩余参数来访问函数参数。
* **没有重复命名的参数** - 非箭头函数只在严格模式下不能有重复命名的参数。与此相反，箭头函数在严格和非严格模式下都不能有重复命名的参数。

造成这些差异的原因有好几个。首先，也是最重要的一点，在JavaScript中`this`绑定是造成错误最常见的源码。在函数内极其容易丢失`this`值的追踪，这就可能导致不预期的程序行为。而箭头函数消除了这困惑。其次，与可能会被用作构造函数或者更改的普通函数不一样，为了JavaScript引擎可以更容易地优化这些操作，箭头函数被限制了通过一个`this`来简单执行代码。

剩下的那些差异主要专注于减少箭头函数里的错误和歧义。通过这种方式，JavaScript引擎可以更好的优化箭头函数的执行。

I> 注意:箭头函数跟其它函数一样，也有`name`属性。

### 箭头函数语法

箭头函数的语法来自于大家想要达到的行为风格。所有的变异始于函数参数，然后是箭头，而后是函数体。取决于不同的用法，参数和函数体都可能采取不同的形式。比如，以下箭头函数只包含一个参数，而且简单返回它：

```js
var reflect = value => value;

// 实际上等同于：

var reflect = function(value) {
    return value;
};
```

当一个箭头函数只有一个参数时，该参数可以不带其它语法直接使用。然后是箭头，求箭头右边的表达式的值并且返回。即使没有显示的`return`语句，这个箭头函数将会返回所传入的第一个参数。

如果你传入的参数多于一个，那么你必须用括号来包住那些参数。像这样：

```js
var sum = (num1, num2) => num1 + num2;

// 实际上等同于：

var sum = function(num1, num2) {
    return num1 + num2;
};
```

函数`sum()`简单地添加了两个参数并返回结果。这个箭头函数和`reflect()`函数之间的唯一差别是那些参数被括号包着，并以逗号分隔（就像传统的函数一样）。

如果函数没有参数，那你必须在声明中包括一个空的括号。如下：

```js
var getName = () => "Nicholas";

// 实际上等同于

var getName = function() {
    return "Nicholas";
};
```
若你想要有一个更传统的函数体，或许是由多个表达式组成，那么你需要用花括号来包住函数体，并且显示地定义返回值。比如这个版本的`sum()`：

```js
var sum = (num1, num2) => {
    return num1 + num2;
};

// 实际上等同于

var sum = function(num1, num2) {
    return num1 + num2;
};
```

除了`arguments`不可用，你几乎可以认为花括号里的内容和传统函数的一样。

如果你想创建一个什么都不做的函数，那么你需要包含花括号。像这样：

```js
var doNothing = () => {};

// 实际上等同于

var doNothing = function() {};
```

如你目前所见，花括号被用来表示函数体。但若要在箭头函数里返回一个函数体外的对象字面量，则必须用花括号包住字面量。比如：

```js
var getTempItem = id => ({ id: id, name: "Temp" });

// 实际上等同于

var getTempItem = function(id) {

    return {
        id: id,
        name: "Temp"
    };
};
```

用花括号来包住对象字面量表示该花括号是一个对象字面量而不是函数体。

### 创建立即调用的函数表达式

JavaScript中函数的一个普遍用法是创建立即调用的函数表达式（IIFEs）。IIFEs允许你定义一个匿名函数并且立即调用而不需保存引用。当你想要创建一个隐匿于程序剩余部分的作用域时，这种模式便派上用场了。

```js
let person = function(name) {

    return {
        getName: function() {
            return name;
        }
    };

}("Nicholas");

console.log(person.getName());      // "Nicholas"
```

这段代码中，IIFE被用作创建一个包含方法`getName()`的对象。该方法用`name`参数作为返回值，实际上是把`name`变成了所返回对象的一个私有成员。

只要你用括号包住箭头函数，你也可以实现用箭头函数实现同样的功能:

```js
let person = ((name) => {

    return {
        getName: function() {
            return name;
        }
    };

})("Nicholas");

console.log(person.getName());      // "Nicholas"
```

要注意括号只包住了箭头函数的定义，并没有把`("Nicholas")`包住。这便有别于正式函数——括号可以放在传入的参数外边，也可以用来包住函数定义。

### 没有this绑定

JavaScript中最常见的一个错误便是函数中的`this`绑定。依靠函数被调用的上下文环境，`this`值在一个函数内部可能会改变，因此当你想改变一个对象时，很有可能会错误地影响到另一个对象。

```js
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click", function(event) {
            this.doSomething(event.type);     // 错误
        }, false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

在这段代码中，对象`PageHandler`被设计用来处理页面的交互。调用`init()`方法建立交互，而那方法进而指派一个时间处理器来调用`this.doSomething()`。然而，这代码并不完全如预期地工作。

因为`this`是事件的目标对象（这种情况下为`document`）的一个引用，而不是被绑定在`PageHandler`，所以`this.doSomething()`的调用失败了。如果你尝试执行这段代码，当时间处理器触发时，因`this.doSomething()` 不存在与目标对象`document`中，你将得到一个错误。

你可以显示地通过在函数上使用`bind()`来将`this`值绑定到`PageHandler`上来解决这个问题。像这样：

```js
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click", (function(event) {
            this.doSomething(event.type);     // 没有错误
        }).bind(this), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

此时该代码便可如预期地工作，但看起来却有点奇怪。调用`bind(this)`，实际上你是在创建一个新函数，且将它的`this`绑定在当前`this`上，即`PageHandler`。为了避免创建额外的函数，更好的一个办法是使用箭头函数是解决这段代码的问题。

箭头函数没有`this`绑定，也就是说，箭头函数内部的`this`值只取决于作用域链的向上查找。若箭头函数被包含在一个非箭头函数中，`this`将与包含函数一样；否则`this`等同于全局作用域中的`this`值。你可以通过这种方式以箭头函数来写这段代码：

```js
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click",
                event => this.doSomething(event.type), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

这个例子中的事件处理器是一个调用`this.doSomething()`的箭头函数。`this`的值和`init()`中的一样，因此，这个版本的代码和用`bind(this)`的代码运行类似。就算`doSomething()`方法没有返回值，它仍是函数体里执行的唯一语句，因此不必要加花括号。

箭头函数被设计成是“抛弃型”的函数，因此不能用于定义新类型；从一般函数都有的`prototype`属性的缺失可以明显看出这一点。若通过`new`操作数来使用箭头函数，你将得到一个错误。如下例：

```js
var MyType = () => {},
    object = new MyType();  // 错误 - “new”不能和箭头函数一起使用
```

这段代码中，因为`MyType`是一个箭头函数，它没有`[[Construct]]`的行为，因此`new MyType()`的调用会失败。箭头函数不能与`new`一起使用，理解了这一点可以使JavaScript引擎进一步地优化它们的行为。

同样地，因`this`的值取决于箭头函数被定义所在的包含函数，你不能通过`call()`，`apply()`，或者`bind()`来改变`this`值。

### 箭头函数和数组

箭头函数的简洁语法也使得它们很适合处理数组。例如，若你要用一个自定义的比较器来对数组排序，很典型的一个写法是：

```js
var result = values.sort(function(a, b) {
    return a - b;
});
```

有很多语法可以用于一个简单的步骤。与一个更简洁的箭头函数版本来比较：

```js
var result = values.sort((a, b) => a - b);
```

数组中像`sort()`，`map()`和`reduce()`这些接收回调函数的方法都可以从更简单的箭头函数中得益，它把看起来很复杂的流程变成了简单的代码。

### 没有arguments绑定

虽然箭头函数没有它们自己的`arguments`对象，但访问包含函数的`arguments`对象还是有可能的。而后不管箭头函数在何处被执行，那`arguments`对象都是可以访问的。比如：

```js
function createArrowFunctionReturningFirstArg() {
    return () => arguments[0];
}

var arrowFunction = createArrowFunctionReturningFirstArg(5);

console.log(arrowFunction());       // 5
```

在`createArrowFunctionReturningFirstArg()`里，元素`arguments[0]`被已创建的箭头函数引用着。该引入包含了传入`createArrowFunctionReturningFirstArg()`函数的第一个参数。随后当箭头函数被执行时，它会返回`5`，也就是传入`createArrowFunctionReturningFirstArg()`的第一个参数。就算该箭头函数不再在创建它的函数的作用域，由于`arguments`标识符的作用域链，`arguments`仍可被访问。

### 识别箭头函数

忽略其不同的语法，箭头函数仍然是函数，它也被认为如此。且看以下代码：

```js
var comparator = (a, b) => a - b;

console.log(typeof comparator);                 // "function"
console.log(comparator instanceof Function);    // true
```

`console.log()`的输出表明了`typeof`和`instanceof`在箭头函数中的行为表现和在其它函数中是一样的。

和其它函数一样，即便函数的`this`绑定不会受影响，你仍可在箭头函数上使用`call()`，`apply()`，和`bind()`。比如这些例子：

```js
var sum = (num1, num2) => num1 + num2;

console.log(sum.call(null, 1, 2));      // 3
console.log(sum.apply(null, [1, 2]));   // 3

var boundSum = sum.bind(null, 1, 2);

console.log(boundSum());                // 3
```

就像调用任何函数一样，`sum()`函数是通过`call()`和`apply()`传入参数来调用的。`bind()`方法被用来创建`boundSum()`，并且两个参数绑定在`1`和`2`，因此它们没有必要直接传入。

在你正使用匿名函数表达式的所有地方，使用箭头函数都是一个适合的用法，比如回调。下一章将会介绍ES6中另一个大的改良，但都是针对内部的，并没有新语法。

## 尾调用的优化

也许在ES6中最有趣的关于函数的变动便是引擎的优化，它改变了尾调用系统。*尾调用*是指在一个函数作为另一个函数的最后一个语句被调用。像这样：

```js
function doSomething() {
    return doSomethingElse();   // tail call
}
```

ES5引擎中尾调用的实现就像其它函数调用一样被处理：一个新的栈帧会被创建，并被加入到调用栈中来表示该函数的调用。那就是说，之前的每一个栈帧都被保留在内存中，这在调用栈越来越大时容易出问题。

### 区别在哪？

ES6力图在严格模式下为某些尾调用减小调用栈的大小（非严格模式下尾调用是不受影响的）。有了这个优化，只要符合以下条件，当前栈帧会被清空和重用，而不是为尾调用创建一个新的栈帧：

1. 尾调用不需要访问当前栈帧中的变量（即函数不是一个闭包）；
2. 使用尾调用的函数在尾调用返回后没有下一步操作；
3. 尾调用的结果作为函数值返回
 
作为一个例子，因为它符合以上三个条件，该代码可以很容易地被优化：

```js
"use strict";

function doSomething() {
    // 已优化
    return doSomethingElse();
}
```

这个函数包含一个尾调用`doSomethingElse()`，立刻返回结果，且没有访问在本地作用域中的任何变量。一个小的改动，不返回结果，会导致一个未优化的函数：

```js
"use strict";

function doSomething() {
    // 没有优化 - 没有返回
    doSomethingElse();
}
```

类似地，如果你有一个函数在尾调用返回后仍有执行操作，那么该函数就不能被优化：

```js
"use strict";

function doSomething() {
    // 没有优化 - 必须在返回之后再加
    return 1 + doSomethingElse();
}
```

这个例子在值返回前将`doSomethingElse()`的结果和1相加，那足以关闭优化。

无意中关闭优化的另一种常见方法是把一个函数调用的结果存在一个变量中，然后返回结果。比如：

```js
"use strict";

function doSomething() {
    // 没有优化 - 调用不在尾部
    var result = doSomethingElse();
    return result;
}
```

因为`doSomethingElse()`的值没有立即返回，该例子并没有被优化。

最难避免的情况也许是使用闭包。在包含作用域中闭包可以访问变量，因此尾调用的优化可能会被关闭。比如：

```js
"use strict";

function doSomething() {
    var num = 1,
        func = () => num;

    // 没有被优化 - 函数是个闭包
    return func();
}
```

此例中闭包`func()`可以访问本地变量`num`。虽然`func()`的调用立即返回了结果，但因变量`num`仍被引用，不会出现优化。

### 如何利用尾调用优化

在实战中，尾调用优化发生在幕后，所以一般不需要考虑到它，除非你尝试优化一个函数。尾调用优化主要的用例在递归函数，优化在递归函数中可以取得最大的效果。且看这个计算阶乘的例子：

```js
function factorial(n) {

    if (n <= 1) {
        return 1;
    } else {

        // 不会被优化 - 必须要返回后相乘
        return n * factorial(n - 1);
    }
}
```

乘法必须在`factorial()`的递归调用返回之后发生，所以这个版本的函数不会被优化。如果`n`是一个很大的值，调用栈的大小便会增大，也很有可能导致栈溢出。

为了优化这个函数，你必须保证乘法不会发生在最后一个函数的调用之后。要做到这样，你可以使用参数默认值来将乘法操作移到`return`语句外边。创建具体相同功能却可被ES6引擎优化的一个函数，这个结果函数将临时结果带入到下一个迭代。且看修改后的代码：

```js
function factorial(n, p = 1) {

    if (n <= 1) {
        return 1 * p;
    } else {
        let result = n * p;

        // 已被优化
        return factorial(n - 1, result);
    }
}
```

在这个重写的`factorial()`版本中，加入了第二个参数`p`，并赋予默认值1。参数`p`用以保存上一个乘法的结果，这样下一个结果便不需要调用另一个函数来计算。当`n`大于1时，会先计算一次乘法，然后把结果作为第二个参数传入到`factorial()`中。这便使得ES6引擎得以优化该递归调用。

无论何时若你要写递归函数，你应该考虑一下尾调用优化。它可以带来重要的性能改善，尤其是在计算昂贵的函数应用中。

## 总结

ES6中函数并没有经历很大的改动，而是通过一系列增量式的改动让函数变得更容易使用。

函数的默认参数值使得你能够容易地明确当某一个参数没有传入时，该使用何值。而在ES6以前，这需要在函数里添加额外的代码来实现，一是检查参数是否存在，二是对其设一个不同的值。

剩余参数允许你指定一个数组，用来存放所有剩余的参数。使用一个真实的数组，让你指明该包含哪些参数，比起`arguments`，这些都使得剩余参数成为了一个更加灵活的解决方案。

散布操作符是剩余参数的一个好搭档，在调用函数时，它使得你能够将一个数组解构成独立的参数。在ES6之前，想要将数组里的值作为各自独立的参数，有两个办法：手动地指定每一个参数，或者使用`apply()`。有了散布操作符，你可以更容易地将一个数组传给任何函数，而不需担心函数的`this`绑定。

`name`属性的加入应该能使得你在调试中更容易识别函数和对目标求值。此外，ES6正式定义了块级函数的表现行为，使得它们在严格模式下不再有语法错误。

在ES6中，当一个函数通过`new`来调用时，该函数的行为是由`[[Call]]`，正常的函数执行，和`[[Construct]]`来定义的。`new.target`元属性也让你能够决定一个函数是否是通过`new`来调用的。

ES6中函数的最大改动是箭头函数的加入。箭头函数被设计用来替代匿名函数表达式的。箭头函数拥有更简洁的语法，`this`词汇绑定，没有`arguments`对象。此外，箭头函数不会改变它们的`this`绑定，因此不能用作构造函数。

尾调用优化使得一些函数调用能够得到优化，从而保持更小的调用栈，使用更少的内容，以及防止栈溢出错误。当引擎认为这样做是安全的时候，它会自动地进行优化。然而，为了利用这种优化，你可能决定重写递归函数。
