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

在这段代码里，常量`maxItems`是在`if`语句里定义的。一旦该语句执行完毕，`maxItems`就不能在该代码块外被访问。

和`let`另一个相似之处，当一个已经在同一个作用域里被定义过的标识符在`const`声明中被使用时会抛出异常，不管该变量是被`var`（全局的或者函数作用域的）还是被`let`（块作用域）定义。比如这段代码：

```js
var message = "Hello!";
let age = 25;

// 以下每一个都会抛出异常。
const message = "Goodbye!";
const age = 30;
```

单独使用时，这两个`const`声明是有效的。但在这个例子中，因为前面已经给定了`var`和`let`声明，这两个都不会如你所期望的那样生效。

除了这些相似之处，`let`和`const`之间还有一个很大的区别需要留意。在严格和非严格模式下，企图给一个已经定义过的常量通过`const`再次赋值都会抛出异常：

```js
const maxItems = 5;

maxItems = 6;      // 抛出异常
```

跟其他编程语言的常量一样，`maxItems`变量不能再次被赋新值。但是，跟其他编程语言不一样的是，如果该常量是一个对象（object），它的值是有可能被修改的。

#### 用Const声明对象

`const`声明不允许绑定（binding）被修改，并不是指值本身。也就是说，通过`const`声明的对象并不会预防被修改。比如：

```js
const person = {
    name: "Nicholas"
};

// 可行
person.name = "Greg";

// 抛出异常
person = {
    name: "Greg"
};
```

在这个例子里，`person`的绑定被创建时被赋予了初始值，一个只有一个属性的对象。修改`person.name`的值是不会引起错误的，因为`person`所包含的改变并没有改变`person`所绑定的值。而当尝试给`person`赋新值时（即尝试改变绑定），代码会抛出异常。`const`对于对象的实现原理的这一巧妙之处是很容易引起误解的。只要记住：`const`预防的是对绑定的修改，而不是被綁定的值。

### 暂时性死区（dead zone）

被`let`或者`const`声明的变量在声明之后才能被访问。若在声明之前企图去访问该变量，哪怕是一些安全操作，都会导致引用错误。比如这个例子里的`typeof`操作：

```js
if (condition) {
    console.log(typeof value);  // ReferenceError（引用错误）!
    let value = "blue";
}
```

在这个例子中，`value`通过`let`定义并初始化了，但该语句永远不会被执行，因为它上面一行代码抛出了异常。`value`存在哪里的这个问题在JavaScript社区里被成为“暂时性死区”(TDZ).TDZ一直都没有很明显地写在ECMAScript规范里，但这个术语经常被用来描述为何`let`和`const`声明不能在它们的声明语句之前被使用。这一部分将会介绍一些TDZ引起的声明位置的微妙之处，虽然例子中用的都是`let`，但对于`const`同样适用。

当JavaScript引擎轮询新进的代码块并且找到一个变量声明时，它要么把声明提升到函数或者全局（对于`var`）作用域的顶部，要么把声明放到TDZ（对于`let`和`const`）中。而且任何企图访问TDZ里的变量的行为都会导致运行时错误。该变量只能从TDZ里移除，因此当代码执行到该变量声明，对它的使用是可靠安全的。

无论何时，一个通过`let`或者`const`声明的变量被定义之前，如果你尝试去调用它，以上理论都是成立的。就像前面例子所展示的，这理论同样适用于可靠的`typeof`操作。但是 ，尽管你不一定能拿到想要的值，你还是可以在变量被声明的代码块以外对变量使用`typeof`。比如：

```js
console.log(typeof value);     // "undefined"

if (condition) {
    let value = "blue";
}
```

当`typeof`操作执行时，变量`value`并不在TDZ里，因为它出现在`value`被定义的块之外。也就是说，没有`value`的绑定，因此`typeof`只是简单地返回了`"undefined"`

TDZ只是块绑定的一个特点。另一个特点跟循环中的使用有关。

## 循环中的块绑定

对于变量的块级作用域的使用，开发者比较需要的一点也许是在`for`循环中使用的那些用完即弃的计数变量。比如说，以下代码在JavaScript中并不常见：

```js
for (var i = 0; i < 10; i++) {
    process(items[i]);
}

// i仍然可以被访问
console.log(i);                     // 10
```

在其他编程语言中，默认都是块级作用域的。这个例子理应也如此，理应只有`for`循环可以访问到变量`i`。然而在JavaScript中，因为`var`声明被提升了，变量`i`在循环结束后仍然可以被访问。而当替换成`let`之后，它应该会是预期的行为。如下例：

```js
for (let i = 0; i < 10; i++) {
    process(items[i]);
}

// i不能被访问 - 抛出异常
console.log(i);
```

在这个例子中，变量`i`只存在于`for`循环中。一旦循环结束，该变量不能再被访问。

### 循环中的函数

长期以来，`var`的特性给在循环中创建的函数带来不少问题，主要是因为循环中的变量在循环作用域外被访问。比如说：

```js
var funcs = [];

for (var i = 0; i < 10; i++) {
    funcs.push(function() { console.log(i); });
}

funcs.forEach(function(func) {
    func();     // 输出十次数字“10”
});
```

一般情况下，你会希望这段代码打印0到9的数字，可它却在同一行输出了十次数字10。那是因为`i`是跨循环遍历共享的，也就是说，所有在循环里创建的函数都指向了同一个变量的引用。当循环结束时`i`的值是10，所以当`console.log(i)`被调用时，每一次都会被打印这个值。

为了解决这个问题，开发者们在循环中使用了立即调用的函数表达式（IIFEs），以便在遍历时可以强制性地创建该变量的一个副本。就像以下例子：

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
    func();     // 输出0, 然后是1, 2, 直到9
});
```

这个版本在循环中用了一个IIFE。变量`i`被传到IIFE时，会创建一个副本并且把副本赋给了`value`。在迭代遍历时函数将使用这个值，因此当循环计数从0计到9时，每一个函数的调用都会返回我们所期望的值。庆幸的是，在ES6中，`let`和`const`的块级绑定为我们简化了这个循环。

### 循环中的Let声明

`let`声明通过有效的模拟上述例子中IIFE的工作方式简化了循环。在每一次遍历中，循环都会创建一个跟上一次遍历所使用的变量同名的新变量，并以其值做初始化。也就是说，你可以忽略IIFE，还能得到你想要的结果。就像这样：

```js
var funcs = [];

for (let i = 0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}

funcs.forEach(function(func) {
    func();     // 输出0, 然后是1, 2, 直到9
})
```

这个循环的效果跟用`var`和IIFE的那个循环是一样的，而且更简洁。每一次遍历循环，`let`声明都创建了一个新的变量`i`，因此在循环中创建的每一个函数都会拿到一个属于它自己的`i`的副本。每一个`i`的副本都会有自己的值，而且这个值是在循环遍历开始时副本被创建的时候赋予的。对于`for-in`和`for-of`也一样。比如：

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
    func();     // 输出"a","b","c"
});
```

这个例子中，`for-in`循环有着和`for`一样的行为。每一次遍历循环，都会创建一个新的`key`绑定，因此每一个函数都会有属于它自己的变量`key`的副本。最后每一个函数都会输出不一样的值。但如果用`var`来定义`key`，所有的函数都会输出`”c“`。

I> 需要特别注意理解的是，在ES6的规范里，循环中`let`声明的行为是一种特别定义的行为，它与`let`的不提升特性不一定有关联。实际上，`let`的早期实现并没有这种行为，毕竟它是在后来才加上的。

### 循环中的常量声明

ES6规范并没有明确地不允许在循环中使用`const`声明；使用不同的循环类型，它们的行为也会不一样。对于普通的`for`循环，你可以在初始化代码里使用`const`，但当你企图改变它的值时循环将会抛出警告。比如：

```js
var funcs = [];

// 在一次遍历后将抛出异常
for (const i = 0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}
```

在这段代码中，变量`i`被定义成了一个常量。循环第一次遍历时，`i`为0，可以执行成功。而当执行`i++`时，因为它企图改变一个常量，这时候就会抛出异常。同样地，如果你不打算修改一个变量，你可以在循环的初始化代码中用`const`定义一个变量。

另一方面，在`for-in`或者`for-of`循环中，一个`const`变量的行为和一个`let`变量是一样的。因此，以下代码不应该引起错误：

```js
var funcs = [],
    object = {
        a: true,
        b: true,
        c: true
    };

//不会有错误
for (const key in object) {
    funcs.push(function() {
        console.log(key);
    });
}

funcs.forEach(function(func) {
    func();     // 输出“a“，”b“，”c“
});
```

这段代码跟我们在”循环中的Let声明“中列举的第二个例子的行为差不多。唯一的区别是在循环中`key`的值不能被改变。而在`for-in`和`for-of`循环中是可以使用`const`的，因为循环中的初始化代码会在每一次遍历中创建一个新的绑定，而不是企图改变一个现有的绑定的值（如同上一个例子中用`for`而不是`for-in`一样）。

## 全局的块绑定

`let`和`const`和`var`另一个不一样的方面是它们在全局作用域中的行为。当在全局作用域中使用`var`时，它会创建一个新的全局变量，该变量还是全局对象（浏览器中的`window`）的一个属性。那意味着你可以通过`var`不经意地就重写一个现有的全局变量。比如：

```js
// 在浏览器里
var RegExp = "Hello!";
console.log(window.RegExp);     // "Hello!"

var ncz = "Hi!";
console.log(window.ncz);        // "Hi!"
```

即便是定义在`window`中的全局`RegExp`，通过`var`声明重写变量是不可靠的。这个例子定义了一个新的全局变量`RegExp`，它重写了原来的`RegExp`。类似地，`ncz`被定义为一个全局变量，并且立即就被定义成`window`的一个属性。这种方式在JavaScript里是一直都生效的。

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

此处，`REgExp`的一个新的`let`声明创建了一个覆盖了全局`RegExp`的绑定。那意味着`window.RegExp`和`RegExp`是不一样的，因此全局作用域并没有被破坏。同样的，`ncz`的`const`声明创建了一个绑定，但并没有在全局对象上创建属性。当你并不想在全局对象上创建属性时，这种性能使得`let`和`const`在全局作用域下可以更安全地使用。

I> 如果你希望某一些代码在全局对象中生效，你可能仍希望在全局作用域里使用`var`。这在浏览器跨框架或者窗口访问代码中是很常见的。
## 块绑定的一些最佳实践（Best Practices）

在ES6还处于开发阶段的时候，普遍的看法认为你应该默认使用`let`而不是`var`来声明变量。对于大多数的JavaScript开发者来说，`let`的确如他们所想的那样工作，因此逻辑上来说直接替换是行得通的。在这种情况下，当变量需要避免被修改时你可能需要用`const`。
然而，随着越来越多的开发者升级到ES6后，另一种方式越来越受欢迎：默认使用`const`，当且仅当你知道一个变量的值是需要改变的时候才使用`let`。理论上来看大部分的变量在初始化之后都不应该被改变值的，毕竟不可预期的值变化是bugs的一个来源。这想法有一个很重要的牵引作用，而且当你接受ES6时也应该在代码里这么做。

## 总结

`let`和`const`的块绑定为JavaScript引入了文本化范围的作用域。这些声明不会被提升，而且只存在于它们被声明的代码块里。这提供和大多数编程语言相似的行为，因为变量可以在被需要的时候才声明，这也减少了无意中引起错误的可能性。随之而来的问题是，你不能在变量被声明之前访问它们，即使是用像`typeof`那样安全可靠的操作符来访问。由于块绑定存在于暂时性死区（TDZ），在声明之前企图访问一个块绑定将会导致错误。

很多情况下，`let`和`const`的行为很大程度上都和`var`相似；但在循环中并不如此。对于`let`和`const`，`for-in`和`for-of`循环会在每一次遍历中创建一个新的绑定。那意味着在循环体内创建的函数在当前遍历中可以访问循环绑定的值，而不是在循环的最后一次遍历（`var`的行为方式）。在`for`循环里使用`let`声明会和`const`一样都会导致错误。

目前块绑定的最佳实践是默认使用`const`，并且在你知道一个变量的值是需要变化的时候使用`let`。这可以在基础级别上保证代码里的不变性，也可以预防某些类型的错误。