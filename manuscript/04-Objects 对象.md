# 扩展对象的功能

ES6重点关注对象功能的改善，几乎JavaScript的每一个值都是对象的某一种类型，因此这改善是有意义的。此外，在一个普通的JavaScript程序中使用的对象数量随着JavaScript应用程序的复杂度增加而增加，也就是说程序 总是会创建更多的对象。随着对象越来越多，更有效地使用对象就显得很有必要了。

ES6通过许多方式改善了对象，从简单的语法扩展，到操作和交互对象的选项。

## 对象类别

JavaScript使用了一个混合的术语来描述在标准中找到的对象，这与那些在执行环境如浏览器或者Node.js添加的对象相反。ES6规格中队对象的每一个类别都有明确的定义。为了对该语言整体上有个好的理解，理解这个术语是很重要的。对象的类别有：

* *普通对象* 拥有JavaScript中对象的所有内在固有的默认行为。
* *外来对象* 拥有与默认对象内在固有行为某种程度上不一样的行为。
* *标准对象* 是那些被ES6定义的对象，如`Array`，`Date`等等。标准对象可能是普通对象或外来的。
* *内置对象* 当一个脚本开始执行时，该对象存在于JavaScript执行环境中。所有标准对象都是内置对象。

这些术语将会在本书中使用，以解释ES6中定义的各种对象。

## 对象的字面量语法扩展

对象字面量是JavaScript中最普遍的模式之一。JSON是基于它的语法而创建的，而且它几乎存在于互联网上的每一个JavaScript文件。因对象字面量是一个并不需几行代码便可创建对象的简洁语法，使得对象字面量非常受欢迎。对开发者们来说幸运的是，ES6使得对象字面量更健壮，甚至通过不同的方式扩展该语法使得它更简洁。

### 简洁的属性初始化器

在ES5和早期版本中，对象字面量只是键值对的一个集合。那也意味着在初始化时可能会重复一些属性值。比如：

```js
function createPerson(name, age) {
    return {
        name: name,
        age: age
    };
}
```

函数`createPerson()`创建了一个对象，该对象的属性名与函数参数名相同。虽然一个是对象属性的名字，另一个提供了该属性的值，但结果好像重复了`name`和`age`。返回对象中的键`name`被指派了变量`name`中的值，而键`age`则被指派了变量`age`的值。

在ES6中，使用简洁的*属性初始化器*，你可以去除这种在属性名和本地变量之间的重复。当对象的一个属性名与本地变量名相同时，你可以简单地包括该名字，而不需提供冒号和值。比如，`createPerson()`可以在ES6改写成以下形式：

```js
function createPerson(name, age) {
    return {
        name,
        age
    };
}
```

当一个对象字面量中的属性只有名字时，JavaScript引擎会在周围的作用域中查找具有相同名字的变量。如果找到则将该变量的值赋给对象字面量中相同名字的属性。在这个例子中，对象字面量属性`name`便被赋予了本地变量`name`的值。

这个扩展使得对象字面量的初始化变得甚至更简洁，也帮助避免了命名错误。给属性指定和本地变量相同名字，这是JavaScript中非常常见的一个模式，这也使得这个扩展成了一个受欢迎的新增。

### 简洁方法

ES6同样改善了给对象字面量指派方法的语法。在ES5和早期版本中，为了给对象添加一个方法，你必须指定一个名字，然后把整个函数定义赋给该名字。如下：

```js
var person = {
    name: "Nicholas",
    sayName: function() {
        console.log(this.name);
    }
};
```

ES6中，通过去除冒号和关键字`function`，该语法变得更简洁。也就是说，你可以像这样改写前面的例子：

```js
var person = {
    name: "Nicholas",
    sayName() {
        console.log(this.name);
    }
};
```

这种速写的语法，也被称为*简洁方法*语法，就像前面例子一样，在`person`对象上创建了一个方法。属性`sayName()`被指派了一个匿名函数，并有着与ES5中函数`sayName()`一样的特性。其中一个区别便是简洁方法可能会使用在非简洁方法中可能不会使用的`super`（稍后将在“超引用的简单原型访问”部分讨论）。

I> 通过简洁方法速写创建的方法的`name`属性值便是括号前使用的名字。在最后一个例子中，`person.sayName()`的`name`属性值是`"sayName"`。

### 计算属性名

若对象实例的属性用方括号[]而不是点记法来设置时，ES5和早期版本都可以计算这些属性名。方括号允许你使用变量，字符串字面量来指定属性名，而用在标识中时，这些属性可能会包含一些会引发语法错误的字符。比如这些例子：

```js
var person = {},
    lastName = "last name";

person["first name"] = "Nicholas";
person[lastName] = "Zakas";

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas"
```

由于`lastName`被指派了`"last name"`的值，这个例子中的两个属性名都用了空格，使得它们都不能通过点记法来引用。然而，括号记法允许任何字符串值都可被用作属性名，所以分配`"first name"`给`"Nicholas"`和分配`"last name"`给`"Zakas"`都是可以的。

此外，你还可以在对象字面量中直接使用字符串字面量作为属性名，像这样：

```js
var person = {
    "first name": "Nicholas"
};

console.log(person["first name"]);      // "Nicholas"
```

对于提前知道的以及可以用字符串字面量来表示的属性名，这模式是可行的。然而，如果该属性名`"first name"`被包含在一个变量里（如前面的例子那样）或者必须被计算，那么在ES5中将没有办法使用对象字面量来定义该属性。

在ES6中，计算属性名是对象字面量语法的一部分，而且它们使用和在对象实例中引用计算的属性名一样的方括号记法。比如：

```js
var lastName = "last name";

var person = {
    "first name": "Nicholas",
    [lastName]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas"
```

对象字面量里的方括号表明了该属性名是计算而来的，所以它的内容被估算成一个字符串。那意味着你也可以包括表达式，比如：

```js
var suffix = " name";

var person = {
    ["first" + suffix]: "Nicholas",
    ["last" + suffix]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person["last name"]);       // "Zakas"
```

这些属性估算成`"first name"`和`"last name"`，而且那些字符串随后可以被用来引用这些属性。在对象实例上使用方括号记法时，你放入方括号的任何东西同样也会对对象字面量内的属性名生效。

## 新方法

从ES5开始ECMAScript的设计目标之一便是避免在`Object.prototype`上创建新的全局函数或者方法，取而代之地是尝试找到那些新方法可行的对象。结果，当没有其他更合适的对象时，全局的`Object`获取了越来越多的方法。ES6在全局的`Object`上引入了一些新方法，它们旨在让某一些特定的任务变得更容易。

### Object.is()方法

在JavaScript中，若你想要比较两个值，你可能习惯性地会使用等于运算符（`==`）或者全等运算符（`===`）。为了避免比较中的强制类型，许多开发者都倾向于后者。但是，就算是全等运算符也不是完全精确的。比如，就算值+0和-0在JavaScript引擎中代表着不一样的意义，可在`===`下它们被认为是相等的。同样地，需要使用`isNaN()`来检测`NaN`属性的`NaN === NaN`会返回`false`。

ES6引入了`Object.is()`方法来弥补全等运算符的这一残余的怪癖。该方法接收两个参数，并且在值相等时返回`true`。当两个值的类型和值都都相等时，这两个值才被认为是相等的。这有一些例子：

```js
console.log(+0 == -0);              // true
console.log(+0 === -0);             // true
console.log(Object.is(+0, -0));     // false

console.log(NaN == NaN);            // false
console.log(NaN === NaN);           // false
console.log(Object.is(NaN, NaN));   // true

console.log(5 == 5);                // true
console.log(5 == "5");              // true
console.log(5 === 5);               // true
console.log(5 === "5");             // false
console.log(Object.is(5, 5));       // true
console.log(Object.is(5, "5"));     // false
```

很多情况下`Object.is()`像`===`运算符一样工作。唯一的区别在于+0和-0被认为是不相等的，而`NaN`和`NaN`则相等。但没有必要停止使用相等运算符。基于那些特殊情况会怎样影响你的代码，你可以选择是否适用`Object.is()`而不用`==`或者`===`。

### 方法Object.assign()

在JavaScript里，*混入（Mixins）*是对象组合里最受欢迎的模式。在一个混入中，一个对象从另一个对象中获取属性和方法。许多JavaScript库中都有类似这样一个的混入方法：

```js
function mixin(receiver, supplier) {
    Object.keys(supplier).forEach(function(key) {
        receiver[key] = supplier[key];
    });

    return receiver;
}
```

该`mixin()`函数迭代了`supplier`自己的属性并且把它们拷贝到`receiver`上（一个浅拷贝，属性值为对象时，对象引用是共享的）。这允许`receiver`可以不经继承而获得新属性，如下所示：

```js
function EventTarget() { /*...*/ }
EventTarget.prototype = {
    constructor: EventTarget,
    emit: function() { /*...*/ },
    on: function() { /*...*/ }
};

var myObject = {};
mixin(myObject, EventTarget.prototype);

myObject.emit("somethingChanged");
```

此处，`myObject`从`EventTarget.prototype`对象中获取行为。这就给了`myObject`分别通过`emit()`和`on()`来发布和订阅事件的能力。

这个模式变得足够普遍，ES6增加了同样行为的方法`Object.assign()`，来接收一个接收者和任意的供应者，然后返回接收者。名字从`mixin()`改成了`assign()`反映了实际发生的运算。由于`mixin()`函数使用的是赋值运算符（`=`），所以它不能拷贝接收器的访问器属性。而`Object.assign()`的名字就是被选择用来反映这个区别的。

I> 各种各样的库里都有类似的方法，它们对同一个基本功能可能有着不一样的名字；普遍的一些方法包括`extend()`和`mix()`方法。简而言之，除了`Object.assign()`方法，ES6中也有一个`Object.mixin()`方法。主要区别是`Object.mixin()`还拷贝访问器属性，但出于对`super`使用的忧虑，该方法被移除了（稍后将在本章的“超引用的简单原型访问”部分讨论）。

在任何使用`mixin()`函数的地方你都可以使用`Object.assign()`，且看这个例子：

```js
function EventTarget() { /*...*/ }
EventTarget.prototype = {
    constructor: EventTarget,
    emit: function() { /*...*/ },
    on: function() { /*...*/ }
}

var myObject = {}
Object.assign(myObject, EventTarget.prototype);

myObject.emit("somethingChanged");
```

方法`Object.assign()`接收任意个供应者，接收者按供应站被指定时的顺序来接收属性。那意味着在接收者中第二个供应者可能会改写来自第一个供应者的某一个值，就像这代码片段里发生的：

```js
var receiver = {};

Object.assign(receiver,
    {
        type: "js",
        name: "file.js"
    },
    {
        type: "css"
    }
);

console.log(receiver.type);     // "css"
console.log(receiver.name);     // "file.js"
```

因为第二个供应者改写了第一个里的值，所以`receiver.type`是`"css"`。

`Object.assign()`方法不是ES6里的一个很大的加入，可它却将在许多JavaScript库中发现的一个常见函数整合成了一个正式的方法。

A> ### 使用访问器属性
A>
A> 要记得，若一个供应者有访问器属性时，`Object.assign()`不会在接收者上创建访问器属性。由于`Object.assign()`使用赋值运算符，供应者上的一个访问器属性会成为接收者的一个数据属性。比如：
A>
A> ```js
A> var receiver = {},
A>     supplier = {
A>         get name() {
A>             return "file.js"
A>         }
A>     };
A>
A> Object.assign(receiver, supplier);
A>
A> var descriptor = Object.getOwnPropertyDescriptor(receiver, "name");
A>
A> console.log(descriptor.value);      // "file.js"
A> console.log(descriptor.get);        // undefined
A> ```
A>
A> 在这段代码中，`supplier`有一个访问器属性`name`。在使用了方法`Object.assign()`后，`receiver.name`便作为一个数据属性存在，因`Object.assign()`被调用时`supplier.name`返回了`"file.js"`，所以该数据属性的值为`"file.js"`。

## 重复的对象字面量属性

ES5严格模式下若发现重复的对象字面量属性则会抛出错误，ES5严格模式为此引入了一个检查。比如，这段代码是有问题的：

```js
"use strict";

var person = {
    name: "Nicholas",
    name: "Greg"        // ES5严格模式下语法错误
};
```

当在ES5严格模式下运行时，第二个`name`属性引起了语法错误。而在ES6中，重复属性的检查被去掉了。严格和非严格模式下代码都不会再检查重复属性。相反地，给定名字的最后一个属性值将会成为该属性的实际值。如下所示：

```js
"use strict";

var person = {
    name: "Nicholas",
    name: "Greg"        // ES6下没有语法错误
};

console.log(person.name);       // "Greg"
```

这个例子中，`person.name`的值是`"Greg"`，因为它是赋予该属性的最后一个值。

## 自有属性列举的顺序

ES5并没有为对象属性定义列举顺序，它把这个交给了JavaScript引擎供应商来决定。然而，ES6严格定义了该顺序，即自有的属性必须按照它们在列举时的返回顺序。这就影响了属性在使用`Object.getOwnPropertyNames()`和`Reflect.ownKeys`返回时的情况（将在12章讲解）。它也影响了被`Object.assign()`处理的属性的顺序。

自有属性列举的基本顺序是：

1. 所有的数字键都按升序排序
2. 所有的字符串键按照它们被添加到对象时的顺序排序
3. 所有的符号键（将在第六章中讲解）按照它们被添加到对象时的顺序排序

例子：

```js
var obj = {
    a: 1,
    0: 1,
    c: 1,
    2: 1,
    b: 1,
    1: 1
};

obj.d = 1;

console.log(Object.getOwnPropertyNames(obj).join(""));     // "012acbd"
```

方法`Object.getOwnPropertyNames()`按顺序`0`, `1`, `2`, `a`, `c`, `b`, `d`返回了`obj`里的所有属性。要注意，就算所有的数字键乱序出现在一个对象字面量里，它们都会被组到一起并且排好序。字符串键紧随其后，且按它们被添加到`obj`的顺序排序。对象字面量本身的键排在第一位，然后是随后加入的任意动态键（在这情况下则是指`d`）。

W> `for-in`循环仍有一个没有指明的列举顺序，因为并不是所有的JavaScript引擎都用同样的方式来实现循环。方法`Object.keys()`和`JSON.stringify()`都被指明使用和`for-in`一样的（未指明的）列举顺序。

列举顺序是JavaScript如何工作中的一个微小变化，然而要找到依赖指明的列举顺序来正确运行的程序是不常见的。通过定义列举顺序，ES6保证了不管在哪执行，依赖列举的JavaScript代码都会正确运行。

## 更健壮的原型

原型是JavaScript继承的基础，而ES6继续让它更健壮。JavaScript的早期版本都严格限制了原型可以完成的东西。然而，开发者们对原型想要有更多的控制和更简单的方式来使用它们，这一点随着该语言越来越成熟以及开发者们对原型的工作越发熟悉而变得清晰。结果ES6对原型也引入了一些改善。

### 改变对象的原型

一般情况下，一个对象的原型在对象被创建时要么通过构造函数，要么通过`Object.create()`方法来指明。ES5里JavaScript程序的最大假设之一，便是对象的原型在实例化之后保持不变。ES5的确也增加了`Object.getPrototypeOf()`方法来获取任意给定对象的原型，但它仍缺乏一个标准方法，以便在实例化后改变对象的原型。

ES6通过增加`Object.setPrototypeOf()`方法改变了这一假设，该方法允许你改变任意给定对象的原型。`Object.setPrototypeOf()`方法接收两个参数：要改变原型的对象，以及用作第一个参数的原型的对象。比如：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

let dog = {
    getGreeting() {
        return "Woof";
    }
};

// 原型是person
let friend = Object.create(person);
console.log(friend.getGreeting());                      // "Hello"
console.log(Object.getPrototypeOf(friend) === person);  // true

// 把原型设为dog
Object.setPrototypeOf(friend, dog);
console.log(friend.getGreeting());                      // "Woof"
console.log(Object.getPrototypeOf(friend) === dog);     // true
```

这段代码定义了两个基本对象：`person`和`dog`。两个对象都有一个返回一个字符串的`getGreeting()`方法。对象`friend`首先继承了`person`对象，也就是说它的`getGreeting()`会输出`"Hello"`。而当原型变成`dog`对象时，`person.getGreeting()`输出`"Woof"`，因为原来跟`person`的关系已经被打断了。

对象原型的实际值被保存在一个内置属性`[[Prototype]]`中。`Object.getPrototypeOf()`方法返回存在`[[Prototype]]`里的值，而`Object.setPrototypeOf()`会改变存在`[[Prototype]]`里的值。然后，这些并不是使用`[[Prototype]]`里的值的唯一方法。

### 超引用的简单原型访问

就像前面所提的，对JavaScript来说原型是很重要的，而且ES6中很多工作都使得它们的使用变得更简单。另一个改善便是`super`引用的引入，这使得一个对象原型的访问功能变得更容易。比如，为了改写一个对象实例上的一个方法，使得它也可以调用相同名字的原型方法，你可以这样做：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

let dog = {
    getGreeting() {
        return "Woof";
    }
};


let friend = {
    getGreeting() {
        return Object.getPrototypeOf(this).getGreeting.call(this) + ", hi!";
    }
};

// 把原型设为person
Object.setPrototypeOf(friend, person);
console.log(friend.getGreeting());                      // "Hello, hi!"
console.log(Object.getPrototypeOf(friend) === person);  // true

// 把原型设为dog
Object.setPrototypeOf(friend, dog);
console.log(friend.getGreeting());                      // "Woof, hi!"
console.log(Object.getPrototypeOf(friend) === dog);     // true
```

这个例子中，`friend`中的`getGreeting()`调用了同样名字的原型方法。`Object.getPrototypeOf()`方法保证了正确的原型被调用，然后一个额外的字符串会被附加到输出中。额外的`.call(this)`确保了原型方法里的`this`值设置正确。

要知道使用`Object.getPrototypeOf()`和`.call(this)`来调用原型上的一个方法是有些复杂的，所以ES6引入了`super`。简单为之，`super`是当前对象原型的一个指针，也就是`Object.getPrototypeOf(this)`的值。知道了这一点，你可以像下面这样简化`getGreeting()`方法：

```js
let friend = {
    getGreeting() {
        // 在前面的例子中，这是一样的：
        // Object.getPrototypeOf(this).getGreeting.call(this)
        return super.getGreeting() + ", hi!";
    }
};
```

`super.getGreeting()`的调用与`Object.getPrototypeOf(this).getGreeting.call(this)`在上下文环境中是一样的。类似地，只要是在一个简洁的方法里，你都可以通过`super`引用来调用一个对象原型里的任意方法。在简洁方法之外尝试使用`super`会导致语法错误。如此例：

```js
let friend = {
    getGreeting: function() {
        // 语法错误
        return super.getGreeting() + ", hi!";
    }
};
```

这个例子在一个命名属性上使用一个函数，因为`super`在此上下文环境中是无效的，`super.getGreeting()`的调用会导致语法错误。

当你有多级继承时，`super`引用是非常强大的。因为在那种情况下，`Object.getPrototypeOf()`在任何情况下都不再可行。比如：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

// 原型是person
let friend = {
    getGreeting() {
        return Object.getPrototypeOf(this).getGreeting.call(this) + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);


// 原型是friend
let relative = Object.create(friend);

console.log(person.getGreeting());                  // "Hello"
console.log(friend.getGreeting());                  // "Hello, hi!"
console.log(relative.getGreeting());                // error!
```

当调用`relative.getGreeting()`时，`Object.getPrototypeOf()`导致了错误。那是因为`this`是`relative`，`relative`的原型`friend`对象。当以`relative`为`this`调用`friend.getGreeting().call()`，整个过程重新开始，并且持续回归调用直到出现栈溢出错误。

ES5中这问题是很难解决的，但有了ES6的`super`，它变得很容易：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

// 原型是person
let friend = {
    getGreeting() {
        return super.getGreeting() + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);


// 原型是friend
let relative = Object.create(friend);

console.log(person.getGreeting());                  // "Hello"
console.log(friend.getGreeting());                  // "Hello, hi!"
console.log(relative.getGreeting());                // "Hello, hi!"
```

因为`super`引用不是动态的，所以它们总是引用正确的对象。在这种情况里，不管有多少个对象继承了该方法，`super.getGreeting()`总是引用`person.getGreeting()`。

## 一个正式的方法定义

ES6之前，“方法”的概念并没有正式地定义。方法就是包含了函数而非数据的对象属性。ES6正式定义了方法就是一个函数，它有一个内在的包含了该方法所属对象的`[[HomeObject]]`属性。且看：

```js
let person = {

    // 方法
    getGreeting() {
        return "Hello";
    }
};

// 不是一个方法
function shareGreeting() {
    return "Hi!";
}
```

这个例子定义了一个只有一个方法`getGreeting()`的`person`。通过直接给对象分配函数的方式，`getGreeting()`的`[[HomeObject]]`是`person`。另一方面，`shareGreeting()`函数没有指定`[[HomeObject]]`，那是因为它在创建时并没有赋给对象。在大多数情况下这种区别并不是很重要，但在使用`super`引用时却非常重要。

任何对`super`的引用都使用`[[HomeObject]]`来决定要做什么。第一步是调用在`[[HomeObject]]`上`Object.getPrototypeOf()`来获取原型的引用。然后，原型被用来搜索具有相同名字的一个函数。最后，设置`this`绑定，然后调用该方法。且看这个例子：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

// 原型是person
let friend = {
    getGreeting() {
        return super.getGreeting() + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);

console.log(friend.getGreeting());  // "Hello, hi!"
```

`friend.getGreeting()`的调用返回了一个字符串，该字符串值为`person.getGreeting()`的值和`", hi!"`的组合。`friend.getGreeting()`的`[[HomeObject]]`是`friend`，而`friend`的原型是`person`，所以`super.getGreeting()`等同于`person.getGreeting.call(this)`。

## 总结

对象是JavaScript程序的中心，ES6为对象做出了一些有帮助的改变，使得它们处理起来更容易和更健壮。

ES6对对象字面量也作出了几个改变。简洁的属性定义使为属性赋予作用域内变量相同的名字变得更容易。计算属性名允许你指定非字面量值作为属性名，就像你已在该语言其他方面做的那样。简洁方法完全省略了冒号和`function`关键字，让你在定义对象字面量的方法上可以输入更少的字符。ES6在重复对象字面量属性名方面也放宽了严格模式的检查，也就是说，你可以在一个对象字面量里有两个名字相同的属性，这不会抛出错误。

`Object.assign()`方法让你可以更轻松地在 一个对象上一次性地改变多个属性。在你使用混入模式时这会是非常有用的。`Object.is()`方法在任何值上执行严格的相等，在处理特殊的JavaScript值时它实际上就是`===`的一个更安全的版本。

自有属性的列举顺序如今在ES6中有了清晰的定义。在列举属性时，数字键永远都以升序排在最前面，然后是以插入顺序排序的字符串键和符号键。

现在，在对象创建后修改它的原型是有可能的。这多亏了ES6的`Object.setPrototypeOf()`方法。

最后，你可以使用`super`关键字来调用对象原型上的方法。通过`super`调用的方法内的`this`绑定会被自动设为`this`的当前值。
