# 迭代器和生成器

许多编程语言已经将数据迭代从`for`循环转向了迭代器，`for`需要初始化变量来追踪集合里的位置，而使用迭代器对象在程序里可动态返回集合的下一个元素。迭代器使得处理集合的数据变得更简单，ES6为JavaScript加入了迭代器。当新的数组方法和新的集合类型（如`Set`和`Map`）结合在一起，迭代器对于高效的数据处理很关键，而且你会发现它们在很多地方都有用到。使用迭代器时，还有一个新的循环`for-of`，而展开操作符（`...`）便是使用了迭代器，迭代器甚至能使异步编程变得更容易。

本章将涉及迭代器的许多用法，但首先，理解将迭代器加入JavaScript的历史原因也是很重要的。

## 循环的问题

如果你写过JavaScript代码，你很可能写过这样的代码：

```js
var colors = ["red", "green", "blue"];

for (var i = 0, len = colors.length; i < len; i++) {
    console.log(colors[i]);
}
```

标准的`for`循环通过变量`i`来追踪数组`colors`。如果变量`i`的值比数组的长度（存储在`len`中）小，`i`的值会在每一次循环执行中增加。

这种循环非常简单，可在循环内嵌和需要追踪多个变量时，循环的复杂度便会增加。额外的复杂度可能会导致错误，而且类似的代码写在不同的地方，`for`循环的这个标准化语法也使得它更易出错。迭代器意在解决这个问题。

## 什么是迭代器？

迭代器是一个专门为了迭代而设计的专用接口对象。所有的迭代器对象都有一个返回结果对象的`next()`方法。返回的对象有两个属性：下一个值`value`，和一个布尔值`done`，当没有更多的值需要返回时该值为`true`。迭代器有一个内在指针追踪着一个集合的值的位置，在每一次的`next()`方法调用中它都会恰当地返回下一个值。

如果在最后一个值返回后调用`next()`，该方法会返回`true`值的`done`和包含迭代器*返回值*的`value`。该返回值不是数据集的一部分，而是相关数据的最后一部分，如果没有那样的数据则返回`undefined`。迭代器的返回值和函数返回值很相似，都是把信息返回给调用者的最后方法。

记住这一点，用ES5创建一个迭代器就相当简单明了了：

```js
function createIterator(items) {

    var i = 0;

    return {
        next: function() {

            var done = (i >= items.length);
            var value = !done ? items[i++] : undefined;

            return {
                done: done,
                value: value
            };

        }
    };
}

var iterator = createIterator([1, 2, 3]);

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"

// for all further calls
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

函数`createIterator()`返回一个包含`next()`方法的对象。`items`数组的下一个值作为`value`被返回。当`i`为3时，`done`变成了`true`，三元条件操作则把`value`设成了`undefined`。这两个结果满足了ES6中迭代器的最后那个特殊的情况，即在最后一部分数据被使用后迭代器的`next()`再次被调用。

就如这个例子所展示的，根据ES6中的规则来写同样行为的迭代器会有些复杂。

好在ES6还提供了生成器，有了它，创建迭代器对象就变得更简单了。

## 什么是生成器？

*生成器（generator）*是一个返回迭代器的函数。生成器函数由在关键字`function`后使用一个星号（`*`）来表示，它使用新关键字`yield`。星号是直接跟在`function`后或者两者间有空格，都没关系，如这个例子：

```js
// generator
function *createIterator() {
    yield 1;
    yield 2;
    yield 3;
}

// 生成器像普通函数一样调用，但返回的是一个迭代器
let iterator = createIterator();

console.log(iterator.next().value);     // 1
console.log(iterator.next().value);     // 2
console.log(iterator.next().value);     // 3
```

`createIterator()`前的`*`把这个函数变成了一个生成器。ES6里新加的关键字`yield`指明了所得到的迭代器在调用`next()`时应该按顺序返回的值。这个例子中生成的迭代器在成功调用`next()`方法时有三个不同的值返回：第一个是`1`，然后是`2`，最后是`3`。就跟创建`iterator`时展示的那样，生成器的调用和其它函数是一样的。

也许生成器函数最有趣的一个点在于它们会在每一个`yield`语句后停止执行。比如说，上述代码中`yield 1`执行之后，该函数不会执行其它任何东西，直到迭代器的`next()`方法被调用。在那个阶段，`yield 2`执行了。在一个函数的中间停止执行的这种能力是非常强大的，它带来了生成器函数一些有趣的用法（在“高级迭代器功能”部分详述）。

`yield`关键字可与任何值或者表达式一起使用，因此你可以不需一个个列出迭代器的元素，而直接写生成器函数给迭代器添加元素。比如，你可以在一个`for`循环里使用`yield`：

```js
function *createIterator(items) {
    for (let i = 0; i < items.length; i++) {
        yield items[i];
    }
}

let iterator = createIterator([1, 2, 3]);

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"

// 之后的所有调用
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

这个例子把一个数组`items`传给了`createIterator()`生成器函数。在这个函数里，一个`for`循环随着循环到进行从数组中将元素输出到了迭代器。每一次遇到`yield`，循环就会停止，并在下一个`yield`语句继续执行。

生成器函数是ES6中一个重要到特性，而且由于它们只是函数，它们可以在所有的地方使用。接下来的内容主要集中在编写生成器的其它一些有用的方法。

W> `yield`关键字只能在生成器里使用，在其它地方包括在生成器内部的一些函数使用`yield`会导致语法错误。比如：
W>
W> ```js
W> function *createIterator(items) {
W>
W>     items.forEach(function(item) {
W>
W>         // 语法错误
W>         yield item + 1;
W>     });
W> }
W> ```
W>
W> 虽然从技术上来说`yield`是在`createIterator()`里边，但因为`yield`不能跨函数，这段代码会有语法错误。这种方式下，`yield`和`return`很相似，即内嵌函数不能为其所在函数返回值。

### 生成器函数表达式

你可以使用函数表达式来创建生成器，只需在关键字`function`和左括号之间加个星号（`*`）。比如：

```js
let createIterator = function *(items) {
    for (let i = 0; i < items.length; i++) {
        yield items[i];
    }
};

let iterator = createIterator([1, 2, 3]);

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"

// 之后的所有调用
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

在这段代码中，`createIterator()`是一个生成器函数表达式而不是函数声明。因为函数表达式是匿名的，星号在关键字`function`和左括号之间。此外，这个例子和上面使用`for`循环那个版本的`createIterator()`函数是一样的。

I> 创建箭头函数来作为生成器是不可能的。

### 生成器对象方法

由于生成器只是函数，它们也可以被加入到对象中。举个例子，你可以用函数表达式来创建在一个ES5风格的对象字面量里的生成器：

```js
var o = {

    createIterator: function *(items) {
        for (let i = 0; i < items.length; i++) {
            yield items[i];
        }
    }
};

let iterator = o.createIterator([1, 2, 3]);
```

你也可以使用ES6的方法速记法，在方法名前面加上一个星号（`*`）：

```js
var o = {

    *createIterator(items) {
        for (let i = 0; i < items.length; i++) {
            yield items[i];
        }
    }
};

let iterator = o.createIterator([1, 2, 3]);
```

这些例子和在“生成器函数表达式”中的例子功能上是一样的；它们只是使用里不一样的语法。在速记法的版本里，由于`createIterator()`定义时没有带关键字`function`，直接把星号放在里方法名前面，星号和方法名之间是可以有空格的。

## 迭代（Iterables）和for-of

*迭代*是指有着`Symbol.iterator`属性的一个对象，这和迭代器息息相关。大名鼎鼎的`Symbol.iterator`标记详细说明了为给定对象返回一个迭代器的一个函数。所有的集合对象（数组，`Set`和`Map`）以及字符串在ES6中都是可迭代的，因此它们都有一个指定的默认迭代器。迭代是被设计用来与ECMAScript中新加的`for-of`循环搭配使用的。

I> 由于生成器默认会分配属性`Symbol.iterator`，所有由生成器创建的迭代器也是可迭代的。

在本章开始的时候，我就提过在`for`循环里追踪索引的问题。迭代器是解决那个问题的第一部分。`for-of`循环是第二部分：它彻底移除了在一个集合里追踪一个索引的必要性，这样你就可以专心在集合的内容部分。

每一次循环执行，`for-of`都会调用一个可迭代参数，并且把结果对象中的`value`存到一个变量中。循环会持续这个过程，直到返回对象的`done`属性为`true`。看这个例子：

```js
let values = [1, 2, 3];

for (let num of values) {
    console.log(num);
}
```

这段代码会输出以下内容：

```
1
2
3
```

首先，这个`for-of`循环会调用`value`数组的`Symbol.iterator`方法来获取一个迭代器（`Symbol.iterator`的调用发生在背后的JavaScript引擎里）。然后调用`iterator.next()`，迭代器结果对象的`value`属性会被读取并赋给`num`。变量`num`一开始是1，然后是2，最后是3。当结果对象的`done`为`true`，退出循环，因此`num`永远不会被赋值`undefined`。

如果你只是想简单地轮询一个数组或者集合里的每一个值，使用`for-of`要好于`for`循环。因为`for-of`循环需要追踪的条件更少一些，它相对来说会没那么容易出错。但对于更复杂的控制条件，建议使用传统的`for`循环。

W> 在非迭代对象，`null`或者`undefined`中使用`for-of`语句将会报错。

### 访问默认的迭代器

你可以使用`Symbol.iterator`来访问一个对象的默认迭代器，像这样：

```js
let values = [1, 2, 3];
let iterator = values[Symbol.iterator]();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

这段代码获取了`values`默认的迭代器并用来迭代该数组的元素。这和使用`for-of`循环时背后的操作是一样的。

由于`Symbol.iterator`声明了默认的迭代器，你也可以用它来检测一个对象是否是可迭代的：

```js
function isIterable(object) {
    return typeof object[Symbol.iterator] === "function";
}

console.log(isIterable([1, 2, 3]));     // true
console.log(isIterable("Hello"));       // true
console.log(isIterable(new Map()));     // true
console.log(isIterable(new Set()));     // true
console.log(isIterable(new WeakMap())); // false
console.log(isIterable(new WeakSet())); // false
```

`isIterable()`函数只是简单地检查默认的迭代器是否存在于一个对象中且是一个函数。`for-of`循环在执行前也有一个类似的检查。

目前为止，这一部分中的例子展示了和内置可迭代类型一起使用`Symbol.iterator`的方法，然而你也可以使用`Symbol.iterator`来创建自己的可迭代对象。

### 创建可迭代对象

开发者定义的对象默认是不可迭代的，不过你可以通过创建一个包含生成器的`Symbol.iterator`属性来使得它们可迭代。比如：

```js
let collection = {
    items: [],
    *[Symbol.iterator]() {
        for (let item of this.items) {
            yield item;
        }
    }

};

collection.items.push(1);
collection.items.push(2);
collection.items.push(3);

for (let x of collection) {
    console.log(x);
}
```

这段代码会输出以下内容：

```
1
2
3
```

首先，这个例子为一个叫`collection`的对象定义了一个默认的迭代器。该默认迭代器是由`Symbol.iterator`方法创建的，它是一个生成器（注意星号还是加在名字前面）。然后生成器使用了一个`for-of`循环来轮询`this.items`里的值并使用`yield`来返回每一个值；而不是手动轮询来定义`collection`默认迭代器的返回值，该`collection`对象依赖`this.items`的默认迭代器来完成这件事。

I> 本章随后介绍的“委派生成器”将介绍另一个使用别的对象迭代器的方法。

此刻，你已经见识到了默认数组迭代器的一些用法，然而ES6里还有许多内置的迭代器来帮助更好地处理集合的数据。

## 内置迭代器

迭代器是ES6的一个重要部分，而且同样地，你不必要为许多内置类型创建自己的迭代器；该语言默认已经包括这些了。你只需要在内置迭代器不能满足你需求时创建迭代器，这很多时候发生在定义你自己的对象或者类时。否则你可以依赖内置迭代器来完成你的工作。或许最常用的迭代器是集合里的那些迭代器。

### 集合迭代器

ES6的集合对象有三种：数组，`Map`和`Set`。它们都有以下内置的迭代器来帮助你操作它们的内容：

* `entries()` - 返回一个值为键值对的迭代器
* `values()` - 返回一个值为集合值的迭代器
* `keys()` - 返回一个值为集合所包含的键的迭代器

你可以通过这些方法中任意一个来获取一个集合的迭代器。

#### entries()迭代器

每一次调用`next()`，`entries()`迭代器都会返回一个包含两个元素的数组。这个数组代表该集合中每一个元素的键和值。对于数组，第一元素是数字索引；对于`Set`，第一个元素同样是值（因为`Set`里的值兼作键）；对于`Map`，第一个元素是它的键。

且看使用这个迭代器的一些例子：

```js
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "ebook");

for (let entry of colors.entries()) {
    console.log(entry);
}

for (let entry of tracking.entries()) {
    console.log(entry);
}

for (let entry of data.entries()) {
    console.log(entry);
}
```

The `console.log()` calls give the following output:

```
[0, "red"]
[1, "green"]
[2, "blue"]
[1234, 1234]
[5678, 5678]
[9012, 9012]
["title", "Understanding ECMAScript 6"]
["format", "ebook"]
```

这段代码在每一个类型的集合上使用了`entries()`方法来获取迭代器，并且使用了`for-of`循环来轮询元素。控制台输出展示了每一个对象中键和值是如何成对返回的。

#### values()迭代器

`values()`迭代器只是简单地返回集合里存储的值。比如：

```js
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "ebook");

for (let value of colors.values()) {
    console.log(value);
}

for (let value of tracking.values()) {
    console.log(value);
}

for (let value of data.values()) {
    console.log(value);
}
```

这段代码输出以下内容：

```
"red"
"green"
"blue"
1234
5678
9012
"Understanding ECMAScript 6"
"ebook"
```

如此例所示，调用`values()`迭代器会返回每一个集合所包含的真实数据，它不会返回任何关于数据在集合里的位置信息。

#### keys()迭代器

`keys()`迭代器返回集合里出现的每一个键。对于数组，它只返回数字键，而不是数组自身的其它属性。对于`Set`，键和值是一样的，所以`keys()`和`values()`都返回同样的迭代器。而对于`Map`，`keys()`迭代器返回的是每一个唯一键。比如：

```js
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "ebook");

for (let key of colors.keys()) {
    console.log(key);
}

for (let key of tracking.keys()) {
    console.log(key);
}

for (let key of data.keys()) {
    console.log(key);
}
```

这个例子输出以下内容：

```
0
1
2
1234
5678
9012
"title"
"format"
```

`keys()`迭代器获取了`colors`，`tracking`和`data`里的每一个键，并且所有的键都从那三个`for-of`循环里被打印出来。对于数组对象，即使你在数组里添加里命名属性，也只有数字索引会被打印。这和数组的`for-in`的行为有些不一样，因为`for-in`循环迭代的是属性而不只是数字索引。

#### 集合类型的默认迭代器

当迭代器没有被显式指定时，每一个集合类型同样有一个默认的迭代器，可供`for-of`使用。`values()`方法是数组和`Set`的默认迭代器，而`entries()`方法则是`Map`的默认迭代器。这些默认迭代器使得在`for-of`循环中使用集合对象变得更容易。看看这个例子：

```js
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "print");

// 和使用colors.values()是一样的
for (let value of colors) {
    console.log(value);
}

// 和使用tracking.values()是一样的
for (let num of tracking) {
    console.log(num);
}

// 和使用data.entries()是一样的
for (let entry of data) {
    console.log(entry);
}
```

因为没有指定迭代器，所以会使用默认的迭代器函数。数组，`Set`和`Map`的默认迭代器是被设计用来反映这些对象是如何被初始化的，所以这段代码会输出以下内容：

```
"red"
"green"
"blue"
1234
5678
9012
["title", "Understanding ECMAScript 6"]
["format", "print"]
```

数组和`Set`默认返回它们的值，而`Map`则返回一个数组，该数组的格式与传给`Map`构造器的数组格式一样。另一方面，弱`Set`和弱`Map`没有内置的迭代器。管理弱引用就意味着没有办法知道这些集合里到底有多少值，这也表示没有办法迭代它们。

A> ### 解构和for-of循环
A>
A> `Map`默认迭代器的行为在`for-of`结合解构一起使用时也显得很有用，如此例：
A>
A> ```js
A> let data = new Map();
A>
A> data.set("title", "Understanding ECMAScript 6");
A> data.set("format", "ebook");
A>
A> // same as using data.entries()
A> for (let [key, value] of data) {
A>     console.log(key + "=" + value);
A> }
A> ```
A>
A> 这段代码里`for-of`循环使用了一个解构数组来给`Map`每一个元素的`key`和`value`赋值。通过这种方式，你可以很容易地同时处理键和值，而不需要访问一个有两个元素的数组或者回到`Map`来获取键或者值。利用`Map`的解构数组，让`for-of`对于`Map`来说变得相当有用，就跟它在`Set`和数组中的用途一样。

### 字符串迭代器

ES5发布以来，JavaScript字符串渐渐变得和数组很像。举个例子，ES5使得通过括号法来访问字符串中的字符成为了标准（即，使用`text[0]`来获取第一个字符，以此类推）。但括号法是在代码单元而非字符上使用的，因此它在双字节字符的访问上并不管用。就像这样：

```js
var message = "A 𠮷 B";

for (let i=0; i < message.length; i++) {
    console.log(message[i]);
}
```

这段代码使用了括号法和`length`属性来轮询并且打印包含了一个Unicode字符的字符串。输出有点令人意外：

```
A
(blank)
(blank)
(blank)
(blank)
B
```

由于双字节字符会被当作是两个独立的代码单元，所以输出的时候`A`和`B`之间有四个空行。

值得庆幸的是，ES6意在完全支持Unicode的（请看第二章），而且默认的字符串迭代器是解决字符串迭代问题的一个尝试。字符串的默认迭代器本身是作用于字符而非代码单元上的。把这个例子修改一下，在`for-of`里使用默认的字符串迭代器会输出更恰当的结果。且看调整过后的代码：

```js
var message = "A 𠮷 B";

for (let c of message) {
    console.log(c);
}
```

This outputs the following:

```
A
(blank)
𠮷
(blank)
B
```

这个结果更符合你在操作字符时所期待的结果：循环成功打印出Unicode字符，以及其它字符。

### NodeList迭代器

文档对象模型（DOM）有一个`NodeList`类型，它表示一个文档里所有元素的集合。在web浏览器上编写JavaScript的人一直都有些难理解`NodeList`对象和数组之间的区别。`NodeList`对象和数组都使用`length`属性来表示元素的个数，并且都使用括号法来访问每一个元素。然而，一个`NodeList`和数组的内部表现行为还是非常不一样的，这也引起了一堆困惑。

有了ES6中新加的默认迭代器，`NodeList`的DOM定义（包含在HTML规格而不是ES6自己的规格中）包含了一个默认迭代器，其表现形式与数组的默认迭代器一样。那意味着你可以在`for-of`或者其它任何使用对象默认迭代器的地方使用`NodeList`。比如：

```js
var divs = document.getElementsByTagName("div");

for (let div of divs) {
    console.log(div.id);
}
```

这段代码调用了`getElementsByTagName()`来获取一个表示`document`对象中所有`<div>`元素的`NodeList`。然后`for-of`循环轮询每一个元素并且打印元素的ID，实际上就使得这段代码跟在一个标准数组中循环一样。

## 展开操作符和非数组迭代器

回忆一下第七章中提到的，展开操作符（`...`）可以用来将一个`Set`转换成一个数组。比如：

```js
let set = new Set([1, 2, 3, 3, 3, 4, 5]),
    array = [...set];

console.log(array);             // [1,2,3,4,5]
```

这段代码在一个数组字面量里使用了展开操作符将`set`里的值填充到该数组中。
This code uses the spread operator inside an array literal to fill in that array with the values from `set`. The spread operator works on all iterables and uses the default iterator to determine which values to include. All values are read from the iterator and inserted into the array in the order in which values were returned from the iterator. This example works because sets are iterables, but it can work equally well on any iterable. Here's another example:

```js
let map = new Map([ ["name", "Nicholas"], ["age", 25]]),
    array = [...map];

console.log(array);         // [ ["name", "Nicholas"], ["age", 25]]
```

Here, the spread operator converts `map` into an array of arrays. Since the default iterator for maps returns key-value pairs, the resulting array looks like the array that was passed during the `new Map()` call.

You can use the spread operator in an array literal as many times as you want, and you can use it wherever you want to insert multiple items from an iterable. Those items will just appear in order in the new array at the location of the spread operator. For example:

```js
let smallNumbers = [1, 2, 3],
    bigNumbers = [100, 101, 102],
    allNumbers = [0, ...smallNumbers, ...bigNumbers];

console.log(allNumbers.length);     // 7
console.log(allNumbers);    // [0, 1, 2, 3, 100, 101, 102]
```

The spread operator is used to create `allNumbers` from the values in `smallNumbers` and `bigNumbers`. The values are placed in `allNumbers` in the same order the arrays are added when `allNumbers` is created: `0` is first, followed by the values from `smallNumbers`, followed by the values from `bigNumbers`. The original arrays are unchanged, though, as their values have just been copied into `allNumbers`.

Since the spread operator can be used on any iterable, it's the easiest way to convert an iterable into an array. You can convert strings into arrays of characters (not code units) and `NodeList` objects in the browser into arrays of nodes.

Now that you understand the basics of how iterators work, including `for-of` and the spread operator, it's time to look at some more complex uses of iterators.

## Advanced Iterator Functionality

You can accomplish a lot with the basic functionality of iterators and the convenience of creating them using generators. However, iterators are much more powerful when used for tasks other than simply iterating over a collection of values. During the development of ECMAScript 6, a lot of unique ideas and patterns emerged that encouraged the creators to add more functionality. Some of those additions are subtle, but when used together, can accomplish some interesting interactions.

### Passing Arguments to Iterators

Throughout this chapter, examples have shown iterators passing values out via the `next()` method or by using `yield` in a generator. But you can also pass arguments to the iterator through the `next()` method. When an argument is passed to the `next()` method, that argument becomes the value of the `yield` statement inside a generator. This capability is important for more advanced functionality such as asynchronous programming. Here's a basic example:

```js
function *createIterator() {
    let first = yield 1;
    let second = yield first + 2;       // 4 + 2
    yield second + 3;                   // 5 + 3
}

let iterator = createIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next(4));          // "{ value: 6, done: false }"
console.log(iterator.next(5));          // "{ value: 8, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

The first call to `next()` is a special case where any argument passed to it is lost. Since arguments passed to `next()` become the values returned by `yield`, an argument from the first call to `next()` could only replace the first yield statement in the generator function if it could be accessed before that `yield` statement. That's not possible, so there's no reason to pass an argument the first time `next()` is called.

On the second call to `next()`, the value `4` is passed as the argument. The `4` ends up assigned to the variable `first` inside the generator function. In a `yield` statement including an assignment, the right side of the expression is evaluated on the first call to `next()` and the left side is evaluated on the second call to `next()` before the function continues executing. Since the second call to `next()` passes in `4`, that value is assigned to `first` and then execution continues.

The second `yield` uses the result of the first `yield` and adds two, which means it returns a value of six. When `next()` is called a third time, the value `5` is passed as an argument. That value is assigned to the variable `second` and then used in the third `yield` statement to return `8`.

It's a bit easier to think about what's happening by considering which code is executing each time execution continues inside the generator function. Figure 8-1 uses colors to show the code being executed before yielding.

![Figure 8-1: Code execution inside a generator](images/fg0601.png)

The color yellow represents the first call to `next()` and all the code executed inside of the generator as a result. The color aqua represents the call to `next(4)` and the code that is executed with that call. The color purple represents the call to `next(5)` and the code that is executed as a result. The tricky part is how the code on the right side of each expression executes and stops before the left side is executed. This makes debugging complicated generators a bit more involved than debugging regular functions.

So far, you've seen that `yield` can act like `return` when a value is passed to the `next()` method. However, that's not the only execution trick you can do inside a generator. You can also cause iterators throw an error.

### Throwing Errors in Iterators

It's possible to pass not just data into iterators but also error conditions. Iterators can choose to implement a `throw()` method that instructs the iterator to throw an error when it resumes. This is an important capability for asynchronous programming, but also for flexibility inside generators, where you want to be able to mimic both return values and thrown errors (the two ways of exiting a function). You can pass an error object to `throw()` that should be thrown when the iterator continues processing. For example:

```js
function *createIterator() {
    let first = yield 1;
    let second = yield first + 2;       // yield 4 + 2, then throw
    yield second + 3;                   // never is executed
}

let iterator = createIterator();

console.log(iterator.next());                   // "{ value: 1, done: false }"
console.log(iterator.next(4));                  // "{ value: 6, done: false }"
console.log(iterator.throw(new Error("Boom"))); // error thrown from generator
```

In this example, the first two `yield` expressions are evaluated as normal, but when `throw()` is called, an error is thrown before `let second` is evaluated. This effectively halts code execution similar to directly throwing an error. The only difference is the location in which the error is thrown. Figure 8-2 shows which code is executed at each step.

![Figure 8-2: Throwing an error inside a generator](images/fg0602.png)

In this figure, the color red represents the code executed when `throw()` is called, and the red star shows approximately when the error is thrown inside the generator. The first two `yield` statements are executed, and when `throw()` is called, an error is thrown before any other code executes.

Knowing this, you can catch such errors inside the generator using a `try-catch` block:

```js
function *createIterator() {
    let first = yield 1;
    let second;

    try {
        second = yield first + 2;       // yield 4 + 2, then throw
    } catch (ex) {
        second = 6;                     // on error, assign a different value
    }
    yield second + 3;
}

let iterator = createIterator();

console.log(iterator.next());                   // "{ value: 1, done: false }"
console.log(iterator.next(4));                  // "{ value: 6, done: false }"
console.log(iterator.throw(new Error("Boom"))); // "{ value: 9, done: false }"
console.log(iterator.next());                   // "{ value: undefined, done: true }"
```

In this example, a `try-catch` block is wrapped around the second `yield` statement. While this `yield` executes without error, the error is thrown before any value can be assigned to `second`, so the `catch` block assigns it a value of six. Execution then flows to the next `yield` and returns nine.

Notice that something interesting happened: the `throw()` method returned a result object just like the `next()` method. Because the error was caught inside the generator, code execution continued on to the next `yield` and returned the next value, `9`.

It helps to think of `next()` and `throw()` as both being instructions to the iterator. The `next()` method instructs the iterator to continue executing (possibly with a given value) and `throw()` instructs the iterator to continue executing by throwing an error. What happens after that point depends on the code inside the generator.

The `next()` and `throw()` methods control execution inside an iterator when using `yield`, but you can also use the `return` statement. But `return` works a bit differently than it does in regular functions, as you will see in the next section.

### Generator Return Statements

Since generators are functions, you can use the `return` statement both to exit early and  specify a return value for the last call to the `next()` method. In most examples in this chapter, the last call to `next()` on an iterator returns `undefined`, but you can specify an alternate value by using `return` as you would in any other function. In a generator, `return` indicates that all processing is done, so the `done` property is set to `true` and the value, if provided, becomes the `value` field. Here's an example that simply exits early using `return`:

```js
function *createIterator() {
    yield 1;
    return;
    yield 2;
    yield 3;
}

let iterator = createIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

In this code, the generator has a `yield` statement followed by a `return` statement. The `return` indicates that there are no more values to come, and so the rest of the `yield` statements will not execute (they are unreachable).

You can also specify a return value that will end up in the `value` field of the returned object. For example:

```js
function *createIterator() {
    yield 1;
    return 42;
}

let iterator = createIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 42, done: true }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

Here, the value `42` is returned in the `value` field on the second call to the `next()` method (which is the first time that `done` is `true`). The third call to `next()` returns an object whose `value` property is once again `undefined`. Any value you specify with `return` is only available on the returned object one time before the `value` field is reset to `undefined`.

I> The spread operator and `for-of` ignore any value specified by a `return` statement. As soon as they see `done` is `true`, they stop without reading the `value`. Iterator return values are helpful, however, when delegating generators.

### Delegating Generators

In some cases, combining the values from two iterators into one is useful. Generators can delegate to other iterators using a special form of `yield` with a star (`*`) character. As with generator definitions, where the star appears doesn't matter, as long as the star falls between the `yield` keyword and the generator function name. Here's an example:

```js
function *createNumberIterator() {
    yield 1;
    yield 2;
}

function *createColorIterator() {
    yield "red";
    yield "green";
}

function *createCombinedIterator() {
    yield *createNumberIterator();
    yield *createColorIterator();
    yield true;
}

var iterator = createCombinedIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: "red", done: false }"
console.log(iterator.next());           // "{ value: "green", done: false }"
console.log(iterator.next());           // "{ value: true, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

In this example, the `createCombinedIterator()` generator delegates first to the iterator returned from `createNumberIterator()` and then to the iterator returned from `createColorIterator()`. The iterator returned from `createCombinedIterator()` appears, from the outside, to be one consistent iterator that has produced all of the values. Each call to `next()` is delegated to the appropriate iterator until the iterators created by `createNumberIterator()` and `createColorIterator()` are empty. Then the final `yield` is executed to return `true`.

Generator delegation also lets you make further use of generator return values. This is the easiest way to access such returned values and can be quite useful in performing complex tasks. For example:

```js
function *createNumberIterator() {
    yield 1;
    yield 2;
    return 3;
}

function *createRepeatingIterator(count) {
    for (let i=0; i < count; i++) {
        yield "repeat";
    }
}

function *createCombinedIterator() {
    let result = yield *createNumberIterator();
    yield *createRepeatingIterator(result);
}

var iterator = createCombinedIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

Here, the `createCombinedIterator()` generator delegates to `createNumberIterator()` and assigns the return value to `result`. Since `createNumberIterator()` contains `return 3`, the returned value is `3`. The `result` variable is then passed to `createRepeatingIterator()` as an argument indicating how many times to yield the same string (in this case, three times).

Notice that the value `3` was never output from any call to the `next()` method. Right now, it exists solely inside the `createCombinedIterator()` generator. But you can output that value as well by adding another `yield` statement, such as:

```js
function *createNumberIterator() {
    yield 1;
    yield 2;
    return 3;
}

function *createRepeatingIterator(count) {
    for (let i=0; i < count; i++) {
        yield "repeat";
    }
}

function *createCombinedIterator() {
    let result = yield *createNumberIterator();
    yield result;
    yield *createRepeatingIterator(result);
}

var iterator = createCombinedIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

In this code, the extra `yield` statement explicitly outputs the returned value from the `createNumberIterator()` generator.

Generator delegation using the return value is a very powerful paradigm that allows for some very interesting possibilities, especially when used in conjunction with asynchronous operations.

I> You can use `yield *` directly on strings (such as `yield * "hello"`) and the string's default iterator will be used.

## Asynchronous Task Running

A lot of the excitement around generators is directly related to asynchronous programming. Asynchronous programming in JavaScript is a double-edged sword: simple tasks are easy to do asynchronously, while complex tasks become an errand in code organization. Since generators allow you to effectively pause code in the middle of execution, they open up a lot of possibilities related to asynchronous processing.

The traditional way to perform asynchronous operations is to call a function that has a callback. For example, consider reading a file from the disk in Node.js:

```js
let fs = require("fs");

fs.readFile("config.json", function(err, contents) {
    if (err) {
        throw err;
    }

    doSomethingWith(contents);
    console.log("Done");
});
```

The `fs.readFile()` method is called with the filename to read and a callback function. When the operation is finished, the callback function is called. The callback checks to see if there's an error, and if not, processes the returned `contents`. This works well when you have a small, finite number of asynchronous tasks to complete, but gets complicated when you need to nest callbacks or otherwise sequence a series of asynchronous tasks. This is where generators and `yield` are helpful.

### A Simple Task Runner

Because `yield` stops execution and waits for the `next()` method to be called before starting again, you can implement asynchronous calls without managing callbacks. To start, you need a function that can call a generator and start the iterator, such as this:

```js
function run(taskDef) {

    // create the iterator, make available elsewhere
    let task = taskDef();

    // start the task
    let result = task.next();

    // recursive function to keep calling next()
    function step() {

        // if there's more to do
        if (!result.done) {
            result = task.next();
            step();
        }
    }

    // start the process
    step();

}
```

The `run()` function accepts a task definition (a generator function) as an argument. It calls the generator to create an iterator and stores the iterator in `task`. The `task` variable is outside the function so it can be accessed by other functions; I will explain why later in this section. The first call to `next()` begins the iterator and the result is stored for later use. The `step()` function checks to see if `result.done` is false and, if so, calls `next()` before recursively calling itself. Each call to `next()` stores the return value in `result`, which is always overwritten to contain the latest information. The initial call to `step()` starts the process of looking at the `result.done` variable to see whether there's more to do.

With this implementation of `run()`, you can run a generator containing multiple `yield` statements, such as:

```js
run(function*() {
    console.log(1);
    yield;
    console.log(2);
    yield;
    console.log(3);
});
```

This example just outputs three numbers to the console, which simply shows that all calls to `next()` are being made. However, just yielding a couple of times isn't very useful. The next step is to pass values into and out of the iterator.

### Task Running With Data

The easiest way to pass data through the task runner is to pass the value specified by `yield` into the next call to the `next()` method. To do so, you need only pass `result.value`, as in this code:

```js
function run(taskDef) {

    // create the iterator, make available elsewhere
    let task = taskDef();

    // start the task
    let result = task.next();

    // recursive function to keep calling next()
    function step() {

        // if there's more to do
        if (!result.done) {
            result = task.next(result.value);
            step();
        }
    }

    // start the process
    step();

}
```

Now that `result.value` is passed to `next()` as an argument, it's possible to pass data between `yield` calls, like this:

```js
run(function*() {
    let value = yield 1;
    console.log(value);         // 1

    value = yield value + 3;
    console.log(value);         // 4
});
```

This example outputs two values to the console: 1 and 4. The value 1 comes from `yield 1`, as the 1 is passed right back into the `value` variable. The 4 is calculated by adding 3 to `value` and passing that result back to `value`. Now that data is flowing between calls to `yield`, you just need one small change to allow asynchronous calls.

### Asynchronous Task Runner

The previous example passed static data back and forth between `yield` calls, but waiting for an asynchronous process is slightly different. The task runner needs to know about callbacks and how to use them. And since `yield` expressions pass their values into the task runner, that means any function call must return a value that somehow indicates the call is an asynchronous operation that the task runner should wait for.

Here's one way you might signal that a value is an asynchronous operation:

```js
function fetchData() {
    return function(callback) {
        callback(null, "Hi!");
    };
}
```

For the purposes of this example, any function meant to be called by the task runner will return a function that executes a callback. The `fetchData()` function returns a function that accepts a callback function as an argument. When the returned function is called, it executes the callback function with a single piece of data (the `"Hi!"` string). The `callback` argument needs to come from the task runner to ensure executing the callback correctly interacts with the underlying iterator. While the `fetchData()` function is synchronous, you can easily extend it to be asynchronous by calling the callback with a slight delay, such as:

```js
function fetchData() {
    return function(callback) {
        setTimeout(function() {
            callback(null, "Hi!");
        }, 50);
    };
}
```

This version of `fetchData()` introduces a 50ms delay before calling the callback, demonstrating that this pattern works equally well for synchronous and asynchronous code. You just have to make sure each function that wants to be called using `yield` follows the same pattern.

With a good understanding of how a function can signal that it's an asynchronous process, you can modify the task runner to take that fact into account. Anytime `result.value` is a function, the task runner will execute it instead of just passing that value to the `next()` method. Here's the updated code:

```js
function run(taskDef) {

    // create the iterator, make available elsewhere
    let task = taskDef();

    // start the task
    let result = task.next();

    // recursive function to keep calling next()
    function step() {

        // if there's more to do
        if (!result.done) {
            if (typeof result.value === "function") {
                result.value(function(err, data) {
                    if (err) {
                        result = task.throw(err);
                        return;
                    }

                    result = task.next(data);
                    step();
                });
            } else {
                result = task.next(result.value);
                step();
            }

        }
    }

    // start the process
    step();

}
```

When `result.value` is a function (checked with the `===` operator), it is called with a callback function. That callback function follows the Node.js convention of passing any possible error as the first argument (`err`) and the result as the second argument. If `err` is present, then that means an error occurred and `task.throw()` is called with the error object instead of `task.next()` so an error is thrown at the correct location. If there is no error, then `data` is passed into `task.next()` and the result is stored. Then, `step()` is called to continue the process. When `result.value` is not a function, it is directly passed to the `next()` method.

This new version of the task runner is ready for all asynchronous tasks. To read data from a file in Node.js, you need to create a wrapper around `fs.readFile()` that returns a function similar to the `fetchData()` function from the beginning of this section. For example:

```js
let fs = require("fs");

function readFile(filename) {
    return function(callback) {
        fs.readFile(filename, callback);
    };
}
```

The `readFile()` method accepts a single argument, the filename, and returns a function that calls a callback. The callback is passed directly to the `fs.readFile()` method, which will execute the callback upon completion. You can then run this task using `yield` as follows:

```js
run(function*() {
    let contents = yield readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
});
```

This example is performing the asynchronous `readFile()` operation without making any callbacks visible in the main code. Aside from `yield`, the code looks the same as synchronous code. As long as the functions performing asynchronous operations all conform to the same interface, you can write logic that reads like synchronous code.

Of course, there are downsides to the pattern used in these examples--namely that you can't always be sure a function that returns a function is asynchronous. For now, though, it's only important that you understand the theory behind the task running. Using promises offers more powerful ways of scheduling asynchronous tasks, and Chapter 11 covers this topic further.

## Summary

Iterators are an important part of ECMAScript 6 and are at the root of several key language elements. On the surface, iterators provide a simple way to return a sequence of values using a simple API. However, there are far more complex ways to use iterators in ECMAScript 6.

The `Symbol.iterator` symbol is used to define default iterators for objects. Both built-in objects and developer-defined objects can use this symbol to provide a method that returns an iterator. When `Symbol.iterator` is provided on an object, the object is considered an iterable.

The `for-of` loop uses iterables to return a series of values in a loop. Using `for-of` is easier than iterating with a traditional `for` loop because you no longer need to track values and control when the loop ends. The `for-of` loop automatically reads all values from the iterator until there are no more, and then it exits.

To make `for-of` easier to use, many values in ECMAScript 6 have default iterators. All the collection types--that is, arrays, maps, and sets--have iterators designed to make their contents easy to access. Strings also have a default iterator, which makes iterating over the characters of the string (rather than the code units) easy.

The spread operator works with any iterable and makes converting iterables into arrays easy, too. The conversion works by reading values from an iterator and inserting them individually into an array.

A generator is a special function that automatically creates an iterator when called. Generator definitions are indicated by a star (`*`) character and use of the `yield` keyword to indicate which value to return for each successive call to the `next()` method.

Generator delegation encourages good encapsulation of iterator behavior by letting you reuse existing generators in new generators. You can use an existing generator inside another generator by calling `yield *` instead of `yield`. This process allows you to create an iterator that returns values from multiple iterators.

Perhaps the most interesting and exciting aspect of generators and iterators is the possibility of creating cleaner-looking asynchronous code. Instead of needing to use callbacks everywhere, you can set up code that looks synchronous but in fact uses `yield` to wait for asynchronous operations to complete.