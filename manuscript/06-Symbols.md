# Symbol以及Symbol属性

symbol是ES6中引入的一个基本类型，加入到现有的基本类型中：字符串（String），数值（number），布尔值（boolean），`null`和`undefined`。symbol作为创建私有对象成员的一个方法开始，这也是JavaScript开发者长时间以来想要的一个功能。在symbol之前，不管属性字符串名字有多含糊，它都很容易被访问，而这“私有名字”功能也意味着让开发者创建非字符串的属性名。这样的方式，检测这些私有名字的常规技巧都是不可行的。

私有名字的提议终于逐渐发展成了ES6的symbol，而本章将 教你如何有效地使用symbol。虽然实现的细节是一样的（即他们为属性名增加了非字符串值），可私有的目标被放弃了 。相反地，symbol属性与其它属性被分开分类。

## 创建Symbols

在JavaScript基本类型中，symbol是很独特的，它们没有像布尔值`true`或者数字`42`那样的文字形式。你可以通过全局的`Symbol`函数来创建一个symbol。就像这个例子：

```js
let firstName = Symbol();
let person = {};

person[firstName] = "Nicholas";
console.log(person[firstName]);     // "Nicholas"
```

这里创建了一个symbol`firstName`，并用在`person`对象上来指定一个新属性。每次你想要访问同一个属性时都必须使用该symbol。合理地命名symbol变量是个不错的主意，因此你可以很轻易地说出该symbol代表的是什么。

W> 因为symbol值是基本值，所以调用`new Symbol()`会抛出错误。你也可以通过`new Object(yourSymbol)`来创建一个`Symbol`的实例，可目前还不清楚这种能力何时有用。

`Symbol`函数同样接收一个描述该symbol的可选参数。该描述本身是不能用来访问属性的，但对调试还是挺有用的。比如：

```js
let firstName = Symbol("first name");
let person = {};

person[firstName] = "Nicholas";

console.log("first name" in person);        // false
console.log(person[firstName]);             // "Nicholas"
console.log(firstName);                     // "Symbol(first name)"
```

一个symbol的描述会被存在内部的`[[Description]]`属性中。该属性在显式或隐式调用symbol的`toString()`方法时被读取。这个例子中，symbol`firstName`的`toString()`方法被`console.log()`隐式调用，所以描述被打印在了日志中。其他时候想要从代码中直接访问`[[Description]]`是不可能的。我一直建议在创建symbol时提供描述，这样可以让读取和调试symbol变得更容易。

A> ### 识别symbol
A>
A>由于symbol是基本值，你可以使用`typeof`运算符来确认一个变量是否包含symbol。ES6扩展了`typeof`，它用在symbol上时会返回`"symbol"`。比如：
A>
A>```js
A>let symbol = Symbol("test symbol");
A>console.log(typeof symbol);         // "symbol"
A>```
A>
A>虽然还有一些非直接方式来确认变量是否为symbol，可`typeof`运算符是最准确和首选的技术。

## 使用symbol

你可以在任何使用计算属性名的地方使用symbol。虽然你已经在本章中看见用在symbol上的括号记法，你还可以在对象字面量的计算属性上使用symbol，也可以与`Object.defineProperty()`和`Object.defineProperties()`调用一起使用，比如：

```js
let firstName = Symbol("first name");

// 使用一个对象字面量的计算属性
let person = {
    [firstName]: "Nicholas"
};

// 设属性为只读
Object.defineProperty(person, firstName, { writable: false });

let lastName = Symbol("last name");

Object.defineProperties(person, {
    [lastName]: {
        value: "Zakas",
        writable: false
    }
});

console.log(person[firstName]);     // "Nicholas"
console.log(person[lastName]);      // "Zakas"
```

这个例子首先使用了一个对象字面量的计算属性来创建symbol属性`firstName`。接下来的一行代码把该属性设成只读的。然后，一个只读的symbol属性`lastName`通过`Object.defineProperties()`方法被创建。一个对象字面量的计算属性再次被使用，但这一次，它是`Object.defineProperties()`调用的第二个参数的一部分。

虽然symbol可以被用在计算属性名被允许的任何地方，为了更有效地使用它们，你将需要有一个体制来在代码不同的地方共享这些symbol。

## 共享Symbols

你可能会发现你想要在代码的不同部分使用同样的symbol。比如说，在你的应用程序里，你有两个不同的对象类型，它们应该使用同一个symbol来表示一个唯一标识。在文件或者大型代码库之间追踪symbol是很难而且容易出错的。那也是ES6提供一个全局的symbol注册表的原因，它能让你在任何地方都能适时地访问到symbol。


如果你想要创建一个用以共享的symbol，你需要调用`Symbol.for()`而不是`Symbol()`方法。`Symbol.for()`方法只接收一个参数，即你要创建的symbol的字符串标识。该参数也用作symbol的描述。比如：
```js
let uid = Symbol.for("uid");
let object = {};

object[uid] = "12345";

console.log(object[uid]);       // "12345"
console.log(uid);               // "Symbol(uid)"
```

`Symbol.for()`方法一开始先查找全局symbol注册表，看带有键`"uid"`的symbol是否存在。如果存在该方法将返回已经存在的symbol。否则创建一个新的symbol并且使用指定的键将它注册到全局symbol注册表中，然后返回新symbol。那就是说，使用相同键调用后续的`Symbol.for()`将返回同一个symbol，如下：

```js
let uid = Symbol.for("uid");
let object = {
    [uid]: "12345"
};

console.log(object[uid]);       // "12345"
console.log(uid);               // "Symbol(uid)"

let uid2 = Symbol.for("uid");

console.log(uid === uid2);      // true
console.log(object[uid2]);      // "12345"
console.log(uid2);              // "Symbol(uid)"
```

此例中`uid`和`uid2`包含了同样的symbol，所以它们可以交换使用。第一次调用`Symbol.for()`创建了这个symbol，而第二次调用则从全局symbol注册表中获取了该symbol。

共享symbol另一个独特之面在于你可以通过`Symbol.keyFor()`方法的调用从全局symbol注册表中获取一个symbol所关联的键。

```js
let uid = Symbol.for("uid");
console.log(Symbol.keyFor(uid));    // "uid"

let uid2 = Symbol.for("uid");
console.log(Symbol.keyFor(uid2));   // "uid"

let uid3 = Symbol("uid");
console.log(Symbol.keyFor(uid3));   // undefined
```

要注意`uid`和`uid2`都返回了键`"uid"`。而symbol`uid3`并不存在全局symbol注册表里，所以它没有关联的键，因此`Symbol.keyFor()`返回了`undefined`。

W> 全局symbol注册表是一个共享的环境，就像全局作用域一样。那意味着你不能假定那个环境中有什么是已经存在或不存在的。使用第三方组件时可以使用symbol键的命名空间来减少命名冲突的可能性。比如，jQuery代码可能会使用`"jquery."`来给所有的键加前缀，像键`"jquery.element"`或者类似的。

## Symbol的类型强制转换

类型强制转换是JavaScript一个重要的部分，这使得该语言在将一个数据类型强制转换另一个的方面有许多灵活性。然而说到强制转换，symbol是非常不灵活的，因为其它的类型缺乏与symbol的逻辑相等性。尤其不能将symbol强制转换成string或者number，这样它们就不能被意外地用作期望有和symbol一样表现行为的属性。

本章中的例子都用了`console.log()`来显示symbol的输出，这是可行的，因为`console.log()`在symbol上调用了`String()`来创建有用的输出。你可以直接使用`String()`来取得同样的结果。比如：

```js
let uid = Symbol.for("uid"),
    desc = String(uid);

console.log(desc);              // "Symbol(uid)"
```

`String()`函数调用了`uid.toString()`且返回了symbol的字符串描述。然而，若你尝试直接将symbol和字符串拼接在一起，则会抛出错误：

```js
let uid = Symbol.for("uid"),
    desc = uid + "";            // error!
```

将`uid`和一个空字符串拼接在一起要求先将`uid`强制转换成一个string。为了预防这种行为上的使用，错误会在检测到强制转换时抛出。

类似地，你不能将一个symbol强制转换成number。所有的数学运算符应用在symbol上时都会导致错误。比如：

```js
let uid = Symbol.for("uid"),
    sum = uid / 1;            // error!
```

该例子尝试用symbol来除1，这就导致了错误。不管使用什么数学运算符都会抛出错误（就像JavaScript的其它非空值一样，所有的symbol都被认为等同于`true`，所以逻辑运算符不会抛出错误）。

## 检索symbol属性

方法`Object.keys()`和`Object.getOwnPropertyNames()`都可以检索一个对象中的所有属性名。前者返回所有可列举的属性名，而后者则返回全部属性，不管属性是否可列举。然而，为了保护ES5的功能性，这二者都不能返回symbol属性。相反地，ES6中加入了方法`Object.getOwnPropertySymbols()`，它允许你从一个对象中检索属性的symbol。

`Object.getOwnPropertySymbols()`的返回值是一个数组，它包含了一个对象的自有属性symbol。比如：

```js
let uid = Symbol.for("uid");
let object = {
    [uid]: "12345"
};

let symbols = Object.getOwnPropertySymbols(object);

console.log(symbols.length);        // 1
console.log(symbols[0]);            // "Symbol(uid)"
console.log(object[symbols[0]]);    // "12345"
```

此代码中`object`只有一个symbol属性`uid`。`Object.getOwnPropertySymbols()`返回的数组是一个只包含了该symbol的数组。

所有的对象都从第0个自有symbol属性开始，而对象可以从它们的原型中继承symbol属性。ES6使用了well-known symbols预定义了一些symbol属性。

## 用well-known symbols暴露内部操作

ES5的一个中心主题是暴露和定义JavaScript的一些“神奇”部分，即开发者在当时无法仿效的部分。ES6继续了这种传统，甚至暴露了更多该语言以前的内部逻辑，ES6主要使用了symbol原型属性来定义某些对象的基本行为。

ES6有一些叫做*well-known symbol*的预定义symbol，表示JavaScript的一些常见行为，而这些行为在ES6之前都被认为是内部专有的操作。每一个well-known symbol在`Symbol`对象上都用一个属性来表示，比如`Symbol.create`。

well-known symbols有:

* `Symbol.hasInstance` - 一个使用`instanceof`来确定一个对象的继承。
* `Symbol.isConcatSpreadable` - 一个布尔值，表明当一个集合被当作参数传给`Array.prototype.concat()`时，`Array.prototype.concat()`是否应该将该集合的元素规整到同一个层级。
* `Symbol.iterator` - 一个返回迭代器的方法 （迭代器将在第七章讲解）。
* `Symbol.match` - 一个被`String.prototype.match()`调用的方法，用以比较字符串。
* `Symbol.replace` - 一个被`String.prototype.replace()`调用的方法，用以替换字符串的子串。
* `Symbol.search` - 一个被`String.prototype.search()`调用的方法，用以定位字符串的子串。
* `Symbol.species` - 用于创建派生类的构造函数（派生类将在第八章讲解）。
* `Symbol.split` - 一个被`String.prototype.split()`调用的方法，用以拆分字符串。
* `Symbol.toPrimitive` - 一个返回对象原始值的方法。
* `Symbol.toStringTag` - 一个被`Object.prototype.toString()`使用的字符串，用以创建对象描述。
* `Symbol.unscopables` - 一个定义了不能被`with`语句引用的对象属性名称的对象。

接下来将会讲述其中一些常用的well-known symbols，其余的则会在本书剩下的相应内容中讲述。

I> 改写一个定义在一个well-known symbol上的方法会使得一个普通的类变成异类，因为这改变了一些内部默认的行为。它只是改变了规范里关于该类的描述，对代码并没有什么特别的影响。

### Symbol.hasInstance属性

每一个函数都有一个`Symbol.hasInstance`方法，用以确定一个给定的对象是否为该函数的一个实例。该方法被定义在`Function.prototype`上，因此所有的函数都继承`instanceof`属性的默认行为，而且为了避免该方法被错误的改写，它是只读的、不可配置的。

`Symbol.hasInstance`方法只接收一个参数：用以检查的值。如果传入的值是该函数的一个实例，该方法将返回true。以下代码将展示`Symbol.hasInstance`是如何工作的：

```js
obj instanceof Array;
```

这段代码相当于：

```js
Array[Symbol.hasInstance](obj);
```

作为该方法的一个简洁语法，ES6从本质上重定义了`instanceof`操作符。而且既然涉及了方法调用，实际上你可以改变`instanceof`的行为。

比如，假设你想要定义一个函数，用以声明没有对象作为实例。你可以这样做，将`Symbol.hasInstance`的返回值写死为`false`，例如：

```js
function MyObject() {
    // ...
}

Object.defineProperty(MyObject, Symbol.hasInstance, {
    value: function(v) {
        return false;
    }
});

let obj = new MyObject();

console.log(obj instanceof MyObject);       // false
```

你必须使用`Object.defineProperty()`来改写一个只读的属性，因此该例子使用了该方法通过一个新函数改写了`Symbol.hasInstance`方法。而这新函数总是返`false`，所以即便`obj`实际上就是`MyObject`类的一个实例，`instanceof`操作符也会在`Object.defineProperty()`的调用后返回`false`。

当然，你也可以基于任意条件来检测一个值，并确定该值是否被当作是一个实例。比如，也许值在1-100之间的数字都被当作是一个特殊number类型的实例。为了实现该行为，你可能会像这样编写代码：

```js
function SpecialNumber() {
    // empty
}

Object.defineProperty(SpecialNumber, Symbol.hasInstance, {
    value: function(v) {
        return (v instanceof Number) && (v >=1 && v <= 100);
    }
});

let two = new Number(2),
    zero = new Number(0);

console.log(two instanceof SpecialNumber);    // true
console.log(zero instanceof SpecialNumber);   // false
```

该代码定义了一个`Symbol.hasInstance`方法，若一个值为`Number`的实例并且值在1-100之间，该方法则返回 `true`。因此，即使没有直接定义`SpecialNumber`和变量`two`之间的关系，`SpecialNumber`也会声明`two`为一个实例。要注意，`instanceof`左边的操作数必须是一个能触发`Symbol.hasInstance`调用的对象，因为非对象会导致`instanceof`一直只返回`false`。

W> 你也可以为所有的内置函数改写默认的`Symbol.hasInstance`属性，如`Date`和`Error`函数。但是不推荐这样做，因为这会影响到你的代码无法预料和令人困惑。最好是你自己的函数且有必要时才改写`Symbol.hasInstance`。

### Symbol.isConcatSpreadable Symbol

JavaScript数组有一个`concat()`方法，用以拼接两个数组。该方法是这样用的：

```js
let colors1 = [ "red", "green" ],
    colors2 = colors1.concat([ "blue", "black" ]);

console.log(colors2.length);    // 4
console.log(colors2);           // ["red","green","blue","black"]
```

该代码将一个新数组拼接到了`colors1`后面，从而创建了一个包含了两个数组的所有元素的数组`colors2`。但`concat()`方法也可以接收非数组参数，在这种情况下，那些参数将会被简单地添加到数组的尾部。比如：

```js
let colors1 = [ "red", "green" ],
    colors2 = colors1.concat([ "blue", "black" ], "brown");

console.log(colors2.length);    // 5
console.log(colors2);           // ["red","green","blue","black","brown"]
```

此处，额外的参数`"brown"`被传入了`concat()`并成为了`colors2`数组中的第五个元素。为什么一个数组参数和字符串参数会有所不同？JavaScript规范中提到，数组会被自动拆分成单独的元素，而其它所有类型都不会。ES6之前没有办法调整这种行为。

`Symbol.isConcatSpreadable`属性是一个布尔值，它表示一个对象有一个`length`属性和一些数字键，而这些数字键属性的值应该被单独的添加到`concat()`调用的结果中。跟其它well-known symbols不一样，这个symbol属性默认是不会出现在任何标准对象中。相反，绕过默认行为，实际上该symbol可用来增大在一些特定类型的对象上`concat()`方法的使用。你可以定义任意类型，像数组在`concat()`调用上的行为一样，像这样：

```js
let collection = {
    0: "Hello",
    1: "world",
    length: 2,
    [Symbol.isConcatSpreadable]: true
};

let messages = [ "Hi" ].concat(collection);

console.log(messages.length);    // 3
console.log(messages);           // ["Hi","Hello","world"]
```

这个例子设置了一个看起来跟数组一样的对象`collection`：它有一个`length`属性和两个数字键。`Symbol.isConcatSpreadable`属性设成了`true`，来表示属性值应该被添加成独立的数组元素当把`collection`传给`concat()`方法时，结果数组在`"Hi"`元素后会有分开的元素`"Hello"`和`"world"`。

I> 在数组子类中你也可以设置`Symbol.isConcatSpreadable`为`false`，来避免元素在`concat()`调用中被拆分。子类将在第八章讨论。

### Symbol.match, Symbol.replace, Symbol.search和 Symbol.split

在JavaScript中字符串和正则表达式总有着密切关系。特别是字符串类型中有着几个接收正则表达式为参数的方法：

* `match(regex)` - 判断给定的字符串是否匹配某个正则表达式
* `replace(regex, replacement)` - 替换`replacement`匹配到正则表达式的字符串子串
* `search(regex)` - 定位字符串中的正则表达式匹配的索引
* `split(regex)` - 在正则表达式匹配的索引处将字符串拆分成一个数组

在ES6之前，对于开发者而言，这些方法与正则表达式的互动方式都是隐藏的，导致无法利用开发者定义的对象来模拟正则表达式。ES6为这四个方法定义了相应的symbol，实际上是把本地行为外包给了内置对象`RegExp`。

symbol `Symbol.match`，`Symbol.replace`，`Symbol.search`和`Symbol.split`分别表示应该被当作正则表达式第一个参数被调用的方法`match()`，`replace()`，`search()`以及`split()`。这四个symbol属性是`RegExp.prototype`上定义的默认实现，字符串方法都应该使用的。

知道了这一点，你可以通过与正则表达式相似的一个方式，使用字符串方法来创建一个对象。你可以在代码里使用以下函数来实现这一点：

* `Symbol.match` - 接受一个字符串参数并返回匹配到的子串数组，若无匹配则返回`null`。
* `Symbol.replace` - 接受一个字符串参数和替换字符串，并返回字符串。
* `Symbol.search` - 接受一个字符串参数并返回匹配到的索引位置，若无匹配则返回-1。
* `Symbol.split` - 接受一个字符串参数并返回包含被匹配拆分的字符串片段的数组。

在对象上定义这些属性的这个能力允许你不需实现正则表达式匹配便可创建对象，还可以在接受正则表达式的方法中使用。以下是使用这些symbol的一些例子：

```js
// 相当于 /^.{10}$/
let hasLengthOf10 = {
    [Symbol.match]: function(value) {
        return value.length === 10 ? [value] : null;
    },
    [Symbol.replace]: function(value, replacement) {
        return value.length === 10 ? replacement : value;
    },
    [Symbol.search]: function(value) {
        return value.length === 10 ? 0 : -1;
    },
    [Symbol.split]: function(value) {
        return value.length === 10 ? ["", ""] : [value];
    }
};

let message1 = "Hello world",   // 11 characters
    message2 = "Hello John";    // 10 characters


let match1 = message1.match(hasLengthOf10),
    match2 = message2.match(hasLengthOf10);

console.log(match1);            // null
console.log(match2);            // ["Hello John"]

let replace1 = message1.replace(hasLengthOf10, "Howdy!"),
    replace2 = message2.replace(hasLengthOf10, "Howdy!");

console.log(replace1);          // "Hello world"
console.log(replace2);          // "Howdy!"

let search1 = message1.search(hasLengthOf10),
    search2 = message2.search(hasLengthOf10);

console.log(search1);           // -1
console.log(search2);           // 0

let split1 = message1.split(hasLengthOf10),
    split2 = message2.split(hasLengthOf10);

console.log(split1);            // ["Hello world"]
console.log(split2);            // ["", ""]
```

对象`hasLengthOf10`意在实现像一个正则表达式一样的功能，只要字符串长度为10就匹配。`hasLengthOf10`的四个方法每一个都是通过适当的symbol来实现，随后在两个字符串中调用相应的方法。第一个字符串`message1`有11个字符，因此它不会匹配；而第二个字符串`message2`有10个字符，因此匹配。虽然不是一个正则表达式，`hasLengthOf10`被传到每一个字符串方法中，由于这些额外的方法，它们都被正确地使用。

虽然这只是一个简单的例子，但它能处理复杂的匹配，比正则表达式目前所可能匹配的还要复杂，这种能力为自定义表达式匹配提供了更多的可能性。

### Symbol.toPrimitive方法

当应用某些特定的操作时，JavaScript经常尝试隐式地将对象转换成基本值。比如当你使用双等号（`==`）来比较一个字符串和一个对象时，该对象会在比较之前被转换成基本值。在ES6之前到底什么基本值会被使用是一个内部操作，但ES6通过`Symbol.toPrimitive`方法将其暴露了出来（可变的）。

`Symbol.toPrimitive`方法定义在每一个标准类型的原型上，并规定了当对象被转换成基本值时应该发生什么。若一个基本类型转换是必要的，`Symbol.toPrimitive`将被调用，它只包含一个参数，说明书中把它称为`hint`。`hint`参数是三个字符串值中的一个。如果`hint`是`"number"`，那么`Symbol.toPrimitive`应该返回一个数值。如果`hint`是`"string"`，则返回一个字符串，而如果是`"default"`则表示该操作没有该类型的引用。

对于大多数标准对象来说，数值模式有以下行为，按优先次序：

1. 调用`valueOf()`方法，如果结果是一个基本值，则返回该值。
2. 否则，调用`toString()`方法，如果结果是一个基本值，则返回该值。
3. 否则，抛出错误。

类似地，对于大多数标准对象，字符串模式的行为会有以下优先：

1. 调用`toString()`方法，如果结果是一个基本值，则返回该值。
2. 否则，调用`valueOf()`方法，如果结果是一个基本值，则返回该值。
3. 否则，抛出错误。

在很多情况下，标准对象把默认模式视为数值模式（除了`Date`，它会把默认模式视为字符串模式）。通过定义一个`Symbol.toPrimitive`方法，你可以改写这些默认的强制行为。

I> 默认模式只应用在操作`==`，`+`上以及当只有一个参数传给`Date`构造函数时。大部分操作都是用字符串或者数值模式。

通过使用`Symbol.toPrimitive`并以一个函数来赋值，你可以改写默认的转换行为。比如：

```js
function Temperature(degrees) {
    this.degrees = degrees;
}

Temperature.prototype[Symbol.toPrimitive] = function(hint) {

    switch (hint) {
        case "string":
            return this.degrees + "\u00b0"; // degrees symbol

        case "number":
            return this.degrees;

        case "default":
            return this.degrees + " degrees";
    }
};

let freezing = new Temperature(32);

console.log(freezing + "!");            // "32 degrees!"
console.log(freezing / 2);              // 16
console.log(String(freezing));          // "32°"
```

这个脚本定义了一个`Temperature`构造函数，并在原型上改写了`Symbol.toPrimitive`方法的默认行为。取决于`hint`参数是否表示字符串，数值或者默认模式，将会返回不同的值（`hint`会被JavaScript引擎填充）。在字符串模式下，`Symbol.toPrimitive`方法返回带有度的Unicode符号的温度值。在数值模式下，它只返回该数值，而默认模式下，它在数值后加上了单词"degrees"。

每一个log语句都会触发一个不同的`hint`参数值。`+`操作通过把`hint`设为`default`触发了默认模式，而`/`操作通过把`hint`设为`"number"`触发了数值模式，而`String()`函数则把`hint`设为了`"string"`从而触发字符串模式。三个模式返回不同的值都是有可能的，更为常见的做法是把默认模式设成和字符串或者数值模式一样的。

### Symbol.toStringTag Symbol

JavaScript中最有趣的问题之一是多全局执行环境的可用性。当一个页面包含一个内联框架（iframe）时，由于该页面和内联框架有着它们各自的执行环境，浏览器便会出现多全局执行环境的情况。大多数情况下这都不是问题，因为数据可以在这些环境中来回传递，所以没什么好担心的。但若是一个对象在不同对象之间传递之后，你尝试去识别你正在处理的这个对象的类型，这时候便会引发问题。

这个问题的一个典型例子便是将一个数组从内联框架传给其所在的页面或者反过来。在ES6术语中，该内联框架和其所在的页面分别表示一个不同的JavaScript执行环境*范围*。每个范围都有着它们自己的带有全局对象拷贝的全局作用域。不管在哪个范围里创建数组，它肯定是一个数组。然而，当把数组传给另一个不同的范围时，`instanceof Array`的调用会返回`false`，这是因为该数组是在另一个不同的范围内通过构造函数创建的，而在当前范围内`Array`表示一个构造函数。

#### 标识问题的一个临时解决方案

对于这个问题，开发者们很快就找到了一个好方法来识别数组。他们发现通过在对象上调用标准的`toString()`方法，总会返回一个可预测的字符串。因此，许多JavaScript库都开始包括这么一个函数：

```js
function isArray(value) {
    return Object.prototype.toString.call(value) === "[object Array]";
}

console.log(isArray([]));   // true
```

这看起来可能有点绕，但它可以很好地在所有浏览器中识别数组。数组上的`toString()`方法对于识别一个对象不是很有用，因为它返回一个表示对象包含的所有元素的字符串。但在`Object.prototype`上的`toString()`有一个怪癖：它在返回结果里包括了叫做`[[Class]]`的内部定义的名字。开发者们可以在对象上使用这个方法来获取JavaScript环境认为的该对象的数据类型。

开发者们很快就意识到因为这个行为无法改变，利用相同的方法来区分原生对象和开发者们创建的对象是有可能的。其中最重要的例子是ES5的`JSON`对象。

在ES5之前，许多开发者使用Douglas Crockford的*json2.js*来创建一个全局的`JSON`对象。随着浏览器开始实现全局的`JSON`对象，弄清楚全局的`JSON`是由JavaScript环境本身提供，还是由其它一些库提供，就变得很有必要了。利用跟我所展示的`isArray()`函数相同的技术，许多开发者这样创建函数：

```js
function supportsNativeJSON() {
    return typeof JSON !== "undefined" &&
        Object.prototype.toString.call(JSON) === "[object JSON]";
}
```

`Object.prototype`允许开发者们跨过内联框架的边界来识别数组的特性也提供了一个方法来指明`JSON`是不是一个原生的`JSON`对象。非原生的`JSON`对象会返回`[object Object]`，而原生的版本则返回`[object JSON]`。这种方法也成为了识别原生对象的实际标准。

#### ES6的回答

ES6通过`Symbol.toStringTag` symbol重新定义了这种行为。这个symbol表示一个属性，该属性定义了在每个对象调用`Object.prototype.toString.call()`时应该产生什么样的一个值。对于一个数组，函数返回的值被解释为在`Symbol.toStringTag`属性上存放`"Array"`。

同样地，你可以为你自己的对象定义`Symbol.toStringTag`的值：

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Person";

let me = new Person("Nicholas");

console.log(me.toString());                         // "[object Person]"
console.log(Object.prototype.toString.call(me));    // "[object Person]"
```

这个例子中，`Symbol.toStringTag`被定义在`Person.prototype`上，它提供了创建一个字符串表示的默认行为。因为`Person.prototype`继承了`Object.prototype.toString()`方法，`Symbol.toStringTag`的返回值在调用`me.toString()`方法时也被使用。然而，你仍然可以定义你自己的`toString()`方法，来提供一个不一样的行为，它不会影响到`Object.prototype.toString.call()`方法的使用。看起来可能是这样的： 

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Person";

Person.prototype.toString = function() {
    return this.name;
};

let me = new Person("Nicholas");

console.log(me.toString());                         // "Nicholas"
console.log(Object.prototype.toString.call(me));    // "[object Person]"
```

这段代码定义了`Person.prototype.toString()`返回`name`属性的值。由于`Person`实例不再继承`Object.prototype.toString()`方法，调用`me.toString()`展示了另一种行为。

I> 除非特别声明，否则所有对象都从`Object.prototype`上继承`Symbol.toStringTag`。字符串`"Object"`是属性默认值。

开发者定义的对象的`Symbol.toStringTag`应该用什么值并没有限制。比如，没有任何东西阻止你使用`"Array"`作为`Symbol.toStringTag`的属性值。如：

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Array";

Person.prototype.toString = function() {
    return this.name;
};

let me = new Person("Nicholas");

console.log(me.toString());                         // "Nicholas"
console.log(Object.prototype.toString.call(me));    // "[object Array]"
```

在这代码中`Object.prototype.toString()`的结果是`"[object Array]"`，这和你在真正的数组中得到的结果一样。这也强调了一个事实，即`Object.prototype.toString()`已经不再是一个识别对象类型的完全可靠的方法。

为原生对象改变该字符串标签也是可能的。只要把`Symbol.toStringTag`分配在对象的原型上，像这样：

```js
Array.prototype[Symbol.toStringTag] = "Magic";

let values = [];

console.log(Object.prototype.toString.call(values));    // "[object Magic]"
```

在这个例子中，虽然数组的`Symbol.toStringTag`被改写了，`Object.prototype.toString()`的调用返回了`"[object Magic]"`。虽然这编程语言并不禁止你这么做，但我建议不要用这种方式改变内建对象。

### Symbol.unscopables Symbol

`with`语句是JavaScript中最具争议之一。设计之初是为了避免重复输入，但后来`with`语句饱受批评，它使得代码更难理解，对性能的负面影响以及更容易出错。

结果，在严格模式下不允许使用`with`语句；这限制同样影响了类和模块，它们默认便是严格模式，而且没有选择退出。

虽然无疑地未来的代码不会再使用`with`语句，为了向后兼容以及保证使用`with`的代码仍能正常工作，ES6仍然在非严格模式下支持`with`。

为了理解这个任务的复杂性，请看以下代码：

```js
let values = [1, 2, 3],
    colors = ["red", "green", "blue"],
    color = "black";

with(colors) {
    push(color);
    push(...values);
}

console.log(colors);    // ["red", "green", "blue", "black", 1, 2, 3]
```

这个例子中，因为`with`语句把`push`加为了本地的一个绑定，它里面的两个`push()`调用相当于`colors.push()`。引用`color`指向`with`语句外创建的变量，引用`values`也一样。

但是ES6为数组添加了一个`values`方法（`values`方法的细节将在第七章讨论）。那可能意味着，在ES6环境中在`with`语句里的`values`引用应该指向数组的`values`方法，而不是变量`values`，这可能会破坏代码。这便是`Symbol.unscopables` symbol存在的原因。

`Symbol.unscopables` symbol被用在`Array.prototype`上，表明哪些属性不该在`with`语句里创建绑定。如果存在，`Symbol.unscopables`会是一个对象，它的键是会被`with`语句绑定省略的标识，值则为`true`，从而强制代码块。数组默认的`Symbol.unscopables`属性如示：

```js
// built into ECMAScript 6 by default
Array.prototype[Symbol.unscopables] = Object.assign(Object.create(null), {
    copyWithin: true,
    entries: true,
    fill: true,
    find: true,
    findIndex: true,
    keys: true,
    values: true
});
```

对象`Symbol.unscopables`有一个`null`原型，它由`Object.create(null)`调用创建，而且包含了ES6新数组的全部方法（这些方法将在第七章"Iterators and Generators,"以及第九章"Arrays"中讨论）。这些方法的绑定不会在`with`语句里创建，这就保证旧代码可以继续工作而不出任何问题。

一般而言，除非你要用`with`语句或者在你的代码里改变一个现有的对象，否则你不应该需要为你的对象定义`Symbol.unscopables`。

## 总结

Symbols是JavaScript中的一个新的基本类型，它被用来创建不引用该symbol就不能访问的属性。

虽然这些属性不是真的私有，但它们更难意外改变或改写，因此，它们很适合那些需要从开发者那里得到一定程度的保护的功能。

你可以为symbols提供详细描述，这能更容易地识别symbol的值。有一个全局的symbol注册，通过使用相同的描述，它可以让你在代码的不同地方使用共享的symbols。用这种方法，基于同样的原因，同样的symbol在多个地方便可以被使用。

像`Object.keys()`或者`Object.getOwnPropertyNames()`这样的方法不会返回symbols，所以为了获取symbol属性，一个新的叫做`Object.getOwnPropertySymbols()`的方法被加进了ES6。但你仍可通过调用`Object.defineProperty()`和`Object.defineProperties()`方法来改变symbol属性。

Well-known symbols以前为标准对象定义了内部专用的功能，它使用全局可用的symbol常量，比如`Symbol.hasInstance`属性。这些symbols在规格说明里都使用前缀`Symbol.`，并允许开发者们通过各种方法来改变标准对象的行为。
