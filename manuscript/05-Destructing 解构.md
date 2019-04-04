# 更容易的数据访问解构（Destructuring）

对象和数组字面量是JavaScript中用得罪频繁的记法，而且多亏了广受喜爱的JSON数据格式，它们变成了该语言里特别重要的一部分。定义对象和数组，然后系统地从这些结构中抽取相关信息，这是非常常见的。ES6增加了“解构”来简化这一任务，它是将一个数据结构拆分成更小部分的一个过程。本章将为你展示对对象和数据如何利用解构。

## 解构为何有用？

在ES5和早期版本中，从对象和数组中获取信息的需要会导致一堆看起来是一样的代码，而这只是为了获取某些数据存进本地变量里。比如：

```js
let options = {
        repeat: true,
        save: false
    };

// 从对象中提取数据
let repeat = options.repeat,
    save = options.save;
```

这段代码从`options`对象中提取了`repeat`和`save`的值，并把它们存到了具有相同名字的本地变量中。虽然这段代码看起来很简单，但想象一下如果你有一大堆变量需要赋值；你可能需要一个一个地给它们赋值。而且如果需要遍历内嵌的数据结构来找到你所需的信息，你可能需要翻遍整个数据结构，只为了找到某一个数据。

那也是ES6为对象和数组增加解构的原因。当你需要将一个数据结构拆分成小部分，从中提取出你需要的信息就变得更容易了。许多语言为了让过程使用更简单，都用了很少量的语法实现解构。ES6的实现实际上只是利用了你已熟悉的语法：对象和数组字面量的语法。

## 对象解构

对象的解构语法就是在赋值运算符的左边使用对象字面量。比如：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
```

这段代码中，`node.type`的值被存到了一个叫`type`的 变量中，而`node.name`的值则存到了`name`变量中。这语法和第四章中讲述的对象字面量的简洁的属性初始化器一样。标识`type`和`name`都是本地变量的声明和从`node`对象读取值的属性。

A> #### 不要忘了初始化器
A>
A>当通过解构使用`var`、`let`或者`const`来声明变量时，你必须提供一个初始化器（等号后面的值）。以下几行代码会由于初始化器的缺失而报错：
A>
A>```js
A>// 语法错误！
A>var { type, name };
A>
A>// 语法错误！
A>let { type, name };
A>
A>// 语法错误！
A>const { type, name };
A>```
A>
A>`const`总是需要初始化器的，即便是在使用非解构的变量时，而`var`和`let`只在解构中要求初始化器。

#### 解构赋值

目前看到的对象解构例子都用了变量声明。然而，在赋值时也是有可能使用解构的。比如，在变量定义了之后，你可能决定改变它们的值，像这样：

```js
let node = {
        type: "Identifier",
        name: "foo"
    },
    type = "Literal",
    name = 5;

// 利用解构来赋不同的值
({ type, name } = node);

console.log(type);      // "Identifier"
console.log(name);      // "foo"
```

这个例子中，在声明时`type`和`name`都被初始化了值，然后相同名字的两个变量被两个不同的值初始化。下一行代码则通过解构赋值读取`node`对象来改变那两个值。值得注意的是你必须用括号包着解构赋值的语句。那是因为左花括号是一个块语句的开始，而块语句不能出现在赋值语句的左边。括号表示接下来的花括号不是一个块语句，它应该被解析成允许赋值完成的一个表达式。

一个解构赋值表达式等值于表达式右边（`=`后面）。也就是说，不管在哪，你都可以用解构赋值来获取你要的值。比如，给函数传入一个值：

```js
let node = {
        type: "Identifier",
        name: "foo"
    },
    type = "Literal",
    name = 5;

function outputInfo(value) {
    console.log(value === node);        // true
}

outputInfo({ type, name } = node);

console.log(type);      // "Identifier"
console.log(name);      // "foo"
```

`outputInfo()`函数被调用时传入了一个解构赋值表达式。由于那便是表达式右边的值，所以该表达式等值于`node`。对`type`和`name`的赋值和正常的行为一样，`node`被传入了`outputInfo()`。

W> 若解构赋值表达式（`=`后面的表达式）的右边部分值为`null`或者`undefined`，将会抛出错误。由于读取值为`null`或者`undefined`的属性的这种尝试总会导致运行时错误，所以这种情况是会发生的。

#### 默认值

当你在使用解构赋值语句时，若你指定了一个不存在于对象里的属性名的本地变量，那么该本地变量会被赋值`undefined`。比如：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name, value } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
console.log(value);     // undefined
```

这段代码定义了一个额外的本地变量`value`并尝试给它赋值。然而，`node`对象里并没有相应的`value`属性，所以该变量不出所料地会被赋予`undefined`。

你也可以在指定的属性不存在时，选择性地给它定义一个默认值以供使用。在属性后插入等号（`=`）并指明默认值便可，像这样：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name, value = true } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
console.log(value);     // true
```

这个例子中，变量`value`被赋予默认值`true`。只有当该属性不存在于`node`中或者其值为`undefined`时才会使用该默认值。由于没有`node.value`属性，变量`value`使用了默认值。这与第三章中讲述的函数的参数默认值类似。

#### 指定到不同的本地变量名

到目前为止，每一个例子中解构赋值都用了对象属性名作为本地变量名；比如，`node.type`的值存在了变量`type`中。当你想要使用同样的名字时这是可行的，可若你不想要一样的时候会怎样呢？ES6有一个扩展语法，它允许你把值指定到一个不同名字的本地变量中，而且该语法看起来跟对象字面量的简洁属性初始化器语法一样。看例子：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type: localType, name: localName } = node;

console.log(localType);     // "Identifier"
console.log(localName);     // "foo"
```

这段代码使用了解构赋值来声明变量`localType`和`localName`，分别存了来自属性`node.type`和`node.name`的值。语法`type: localType`是说读取名为`type`的属性并将其值存入变量`localType`。这种语法实际上与传统的冒号左边为名而右边为值的对象字面量语法相反。在这种情况下，冒号右边的才是名字，而左边是要读取的值所在的位置。

在使用不同变量名时，你也可以添加默认值。等号和默认值仍放在本地变量名后面。比如：

```js
let node = {
        type: "Identifier"
    };

let { type: localType, name: localName = "bar" } = node;

console.log(localType);     // "Identifier"
console.log(localName);     // "bar"
```

此处变量`localName`有一个默认值`"bar"`。因为属性`node.name`，该变量被赋予了默认值。

目前为止，你已看到怎么处理一个属性值为原始值的对象的解构。对象解构也可用来获取内嵌对象结构中的值。

#### 内嵌对象的解构

通过使用与对象字面量相似的语法，你可以导航到一个内嵌对象结构中去获取你要的信息。比如这个例子：

```js
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        }
    };

let { loc: { start }} = node;

console.log(start.line);        // 1
console.log(start.column);      // 1
```

这例子中的解构表达式使用了花括号来指明该表达式应该进入到`node`中的`loc`属性并从中查找`start`属性。回忆一下上一部分的内容，当解构表达式中有冒号时，它意味着冒号前面的标识给定了用来检测的位置，而右边则赋了一个值。而冒号后面的花括号则表示目标被内嵌在对象的另一个层级。

你还可以更进一步，给本地变量使用不一样的名字：

```js
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        }
    };

// 提取node.loc.start
let { loc: { start: localStart }} = node;

console.log(localStart.line);   // 1
console.log(localStart.column); // 1
```

这个版本的代码中，`node.loc.start`被存在了一个新的变量`localStart`里。解构表达式可以被内嵌到任意深度，在每一级深度上所有的功能都是可行的。

对象解构是很强大的，而且有许多选择。而数组解构则提供了一些独特的能力允许你从数组中提取信息。

A> #### 语法问题
A>
A> 使用内嵌解构时一定要小心，因为你可能不经意地就创建了一条不生效的语句。空的花括号在对象解构中是合法的，但它们什么都不做。比如：
A>
A>```js
A>// 没有声明变量！
A>let { loc: {} } = node;
A>```
A>
A>此语句没有声明任何绑定。由于花括号在右边，`loc`被用作检测的位置而不是要创建的绑定。在这样一种情况下，很有可能原本的意图是要使用`=`来定义一个默认值而不是用`:`来定义一个位置。这语法将来可能会被标记成非法的，可目前来说这都是一个要注意的点。

## 数组解构

数组解构的语法与对象解构的非常相似；只是它使用的是数组字面量语法而不是对象字面量的。解构操作在一个数组内的位置上而不是对象上存在的命名属性。比如：

```js
let colors = [ "red", "green", "blue" ];

let [ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

这里，数组解构从数组`colors`中提取出值`"red"`和`"green"`，并把它们存入变量`firstColor`和`secondColor`中。那些值是由于它们在数组中的位置而选择的；实际上变量名可以是任意名字。 没有在解构表达式中显示提及的所有元素都会被忽略。要记住，无论如何数组本身都是不变的。

你也可以在解构表达式上省略一些元素，只为你感兴趣的元素提供变量名。比如，如果你只想要数组的第三个值，你不需要给第一二个元素提供变量名。它是这样工作的：

```js
let colors = [ "red", "green", "blue" ];

let [ , , thirdColor ] = colors;

console.log(thirdColor);        // "blue"
```

这段代码使用了解构赋值来获取`colors`中的第三个元素。该表达式中`thirdColor`前面的逗号是在所取元素之前的数组元素的占位符。通过这种方式，你可以很容易地从数组中间任意数量的位置上提取值，而不需要为其它元素提供变量名。

W> 与对象解构类似，数组解构与`var`、`let`、`const`一起使用时，你总是要提供初始化器的。

#### 解构赋值

你可以在一个赋值上下文中使用数组解构，但与对象解构不一样的是，表达式不需要包在括号中。比如：

```js
let colors = [ "red", "green", "blue" ],
    firstColor = "black",
    secondColor = "purple";

[ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

这段代码中的解构赋值和上一个数组解构的例子有着相似的行为。唯一的区别在于`firstColor`和`secondColor`都已经被定义。大多数时候，那大概就是所有你需要知道的关于数组解构赋值的内容。不过还有一点点你可能会发现很有用的内容。

数组解构赋值有一个非常独特的用例，它使得交换两个变量值变得更简单。在排序算法中值交换式一个常见的操作，而在ES5中交换变量值的操作需要引用一个临时的第三变量，如下例：

```js
// ES5中交换变量
let a = 1,
    b = 2,
    tmp;

tmp = a;
a = b;
b = tmp;

console.log(a);     // 2
console.log(b);     // 1
```

为了交换`a`和`b`的值，中间变量`tmp`是必要的。然而，使用数组解构赋值不需要额外的变量。你可以看看ES6中是怎样交换变量的：

```js
// ES6中交换变量
let a = 1,
    b = 2;

[ a, b ] = [ b, a ];

console.log(a);     // 2
console.log(b);     // 1
```

这个例子中的数组解构赋值看起来就像一个镜像。赋值的左边（等号前面）是一个解构表达式，就跟其它数组解构例子中的一样。而右边则是为交换临时创建的数组字面量。解构发生在临时数组上，该数组有着从`b`和`a`复制到它第一二位置上的两个值。这效果便是变量交换了值。

W> 跟对象解构赋值一样，当数组解构赋值表达式右边得出的值为`null`或者`undefined`时将会抛出错误。

#### 默认值

数组解构赋值同样允许你为数组任意位置上的值赋予默认值。当给定位置上的属性不存在或者值为`undefined`时，便会使用默认值。比如：

```js
let colors = [ "red" ];

let [ firstColor, secondColor = "green" ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

这段代码中，`colors`数组只有一个元素，所以`secondColor`匹配不到任何值。但因为它有一个默认值，所以`secondColor`值为`"green"`而非`undefined`。

#### 内嵌解构

跟内嵌对象的解构一样，你可以解构内嵌数组。通过将另一个数组表达式插入到整一个表达式中，解构将下降到一个内嵌的数组中，像这样：

```js
let colors = [ "red", [ "green", "lightgreen" ], "blue" ];

// later

let [ firstColor, [ secondColor ] ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

此处，变量`secondColor`引用了`colors`数组的值`"green"`。该元素被包含在第二个数组，因此解构表达式中`secondColor`周围的方括号是很有必要的。跟内嵌对象解构一样，你可以解构任意深度的内嵌数组。

#### 剩余元素

第三章介绍了函数的剩余参数，而数组解构也有一个类似的概念，叫*剩余元素*。剩余元素使用`...`语法将一个数组中剩下的元素赋给一个特殊变量。且看例子：

```js
let colors = [ "red", "green", "blue" ];

let [ firstColor, ...restColors ] = colors;

console.log(firstColor);        // "red"
console.log(restColors.length); // 2
console.log(restColors[0]);     // "green"
console.log(restColors[1]);     // "blue"
```

`colors`的第一个元素被赋给了`firstColor`，而剩下的都赋给了一个新数组`restColors`。因此，`restColors`数组有两个元素：`"green"`和`"blue"`。从一个元素中提取一些特定元素并保留剩下的元素备用，剩余元素对于这种情况是很有帮助的。不过它还有另一个有帮助的用法。

JavaScript数组有一个明显的功能缺漏是简单地创建一个复制。在ES5中，开发者经常使用`concat()`作为一个简单的方式来复制一个数组。比如：

```js
// cloning an array in ECMAScript 5
var colors = [ "red", "green", "blue" ];
var clonedColors = colors.concat();

console.log(clonedColors);      //"[red,green,blue]"
```

`concat()`原意是拼接两个数组，而不带参数调用它时会返回该数组的一个复制。在ES6中，你可以使用剩余元素来实现同样的东西，其语法就跟函数的那样。它是这样工作的：

```js
// ES6中复制一个数组 
let colors = [ "red", "green", "blue" ];
let [ ...clonedColors ] = colors;

console.log(clonedColors);      //"[red,green,blue]"
```

这个例子中，剩余元素被用来将`colors`数组中的值拷贝到`clonedColors`中。虽然这种技巧是否让开发者的意图更清晰是一个感知问题，但还是有必要知道这是个有用的能力。

W> 在解构数组中，剩余元素必须是最后一项，其后不能有逗号。在剩余元素后加逗号会导致语法错误。

## 混合解构

对象解构和数组解构可以被一起用来创建更复杂的表达式。这样能够让你从任何对象和数组的混合中提取你要的信息片段。比如：

```js
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        },
        range: [0, 3]
    };

let {
    loc: { start },
    range: [ startIndex ]
} = node;

console.log(start.line);        // 1
console.log(start.column);      // 1
console.log(startIndex);        // 0
```

这段代码将`node.loc.start`和`node.range[0]`分别提取到了`start`和`startIndex`中。要记住解构表达式中的`loc:`和`range:`只是`node`对象中相应的属性。使用对象和数组解构的混合，没有哪个部分是不能从`node`中提取的。这种方式在从JSON配置结构中取值是特别有用的，这样你就不需要操作整个结构。

## 解构参数

解构还有一个特别好的用例，那就是给函数传参。当一个JavaScript函数有一大堆可选参数时，一个常见的做法便是创建一个对象`options`，该对象的属性标明了额外参数。像这样：

```js
// 、options的属性代表另外的参数
function setCookie(name, value, options) {

    options = options || {};

    let secure = options.secure,
        path = options.path,
        domain = options.domain,
        expires = options.expires;

    // code to set the cookie
}

// 第三个参数映射到options
setCookie("type", "js", {
    secure: true,
    expires: 60000
});
```
许多包含`setCookie()`函数的JavaScript库都跟这个很像。在这个函数中，参数`name`和`value`都是必须的，但`secure`、`path`、`domain`和`expires`不是。而且因为其它数据没有优先级顺序，定义只包含了命名属性的对象`options`是很高效的。这方法是可行的，但是这样单看函数定义你就无法看出该函数期待的输入是什么了；你需要查阅函数体。

解构参数提供了一个可行方法让函数所需的参数变得更清晰。解构参数使用一个对象或数组解构表达式来替代一个命名参数。我们来看看上一个例子中`setCookie()`函数的改写版本：

```js
function setCookie(name, value, { secure, path, domain, expires }) {

    // code to set the cookie
}

setCookie("type", "js", {
    secure: true,
    expires: 60000
});
```

这个函数跟前一个例子功能是类似的，不过现在第三个参数使用了解构来取出必要的数据。解构参数外的参数需要什么是很清晰的，同时，当要使用`setCookie()`时，什么选项也就是额外参数是存在的，也变得清晰了。当然，如果第三个参数是必要的，它应该包含的值也十分清楚。解构参数和一般的参数一样，它们不被传入时会被设为`undefined`。

A> 解构参数拥有所有你目前在本章中学习到的解构能力。你可以使用默认值，混合使用对象和数组表达式以及使用跟所读取属性不一样的变量名。

### 解构参数是必要的

使用解构参数的一个怪事是，函数调用时如果不传入解构参数默认会抛出错误。比如上一个例子中，调用函数`setCookie()`将会抛出错误：

```js
// 错误!
setCookie("type", "js");
```

第三个参数缺失了，因此它的值会是`undefined`。因为解构参数其实只是解构声明的一个简写，所以这会导致错误。当函数`setCookie()`被调用时，JavaScript引擎实际上做了这些事情：

```js
function setCookie(name, value, options) {

    let { secure, path, domain, expires } = options;

    // code to set the cookie
}
```

当右边表达式为`null`或`undefined`时解构会抛出错误，所以当第三个参数没有传给函数`setCookie()`时也会一样。

若解构参数是必要的，这行为也不会那么麻烦。但若解构参数是可选的，你可以通过给解构参数一个默认值来临时解决这问题。像这样：

```js
function setCookie(name, value, { secure, path, domain, expires } = {}) {

    // ...
}
```

该例子为第三个参数提供了一个新对象作为默认值。该默认值说明如果`setCookie()`的第三个参数没有提供时，`secure`、`path`、`domain`和`expires`都是`undefined`，且不会出错。

### 解构参数的默认值

你可以像解构赋值一样为解构参数指定默认值。在参数后加上等号和指明默认值便可。比如：

```js
function setCookie(name, value,
    {
        secure = false,
        path = "/",
        domain = "example.com",
        expires = new Date(Date.now() + 360000000)
    } = {}
) {

    // ...
}
```

这代码中，解构参数中的每一个属性都有一个默认值，因此你可以避免为了使用正确的值而检查一个属性是否存在。同样的，以空对象作为默认值，整个解构参数便认为是可选的。这确实让函数声明看起来比平常的要复杂一点，但那只是为了保证每一个参数都有可用值而付出的小代价而已。

## 总结

解构使得JavaScript中对象和数组的使用更简单。使用熟悉的对象字面量和数组字面量语法，你可以将数据结构拆解，然后只取你感兴趣的信息。对象表达式允许你从对象中提取数据，而数组表达式则让你从数组中提前数据。

对象解构和数组解构都可以为任意属性或者`undefined`的元素指定默认值；当赋值表达式右边值为`null`或者`undefined`时，二者皆抛出错误。你也可以通过对象解构和数组解构以任意深度进入到内嵌的数据结构中。

使用`var`、`let`或者`const`的解构声明来创建变量必须有初始化器。使用的是解构赋值而非其它赋值，而且允许你解构对象属性和已存在的变量。

解构参数使用解构语法，使“可选”对象在用作函数参数时变得透明。你感兴趣的实际数据可以连同其它命名参数一起罗列出来。解构参数可以是数组表达式，对象表达式，或者混合，而且你可以使用解构的所有功能。
