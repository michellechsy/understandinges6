# 字符串与正则表达式

字符串（Strings）大概是编程语言最重要的数据类型之一。它们几乎存在于每一个高级编程语言里，高效的使用字符串也是开发者创建应用程序的基础。延伸开来，因为正则表达式给开发者们带来了字符串的一些额外好处，正则表达式也显得很重要的。鉴于这些因素，ES6的创建者们通过增加新的性能和长期以来缺失的一些功能改善了字符串和正则表达式。

## 更好地支持Unicode

在ES6之前，JavaScript的字符串以16位的字符编码（UTF-16）为主。每一个16位序列都是一个代表一个字符的*代码单元*。所有字符串的属性和方法，比如说`length`属性和`charAt()`方法，都是基于16位代码单元的。当然，在之前16位是足够用来表示任何字符的。但自从Unicode（万国码）中引入了扩展字符集，这就不能够满足了。

### UTF-16字符码（Code Points）

对于Unicode的既定目标——为世界上的每一个字符都提供一个全球唯一的标识符，字符的长度限于16位是不可能的。这些被称为*字符码*的全球唯一标识符，只是一些从零开始的数字。你可能会觉得字符码就是字符编码——用一个数字来表示一个字符。一个字符编码必须把字符码编译成内部一致的编码单元。对于UTF-16，字符码可以由许多编码单元组成。

UTF-16里最初的2^16^字符码被用来表示单个的16位代码单元。这段范围被称为*基本多文种平面*(BMP)。剩下的那些已经不能只用16位来表示的字符码都被称为*辅助平面*。UTF-16引入了*代理编码对*（surrogate pairs）来解决这个问题，即一个字符码用两个16位的代码单元来表示。也就是说，对于字符串里的任何一个字符，在给定16位时它就是BMP字符里的一个代码单元，或者是在32位里的辅助平面字符里的两个单元。

在ES5里，所有的字符操作都是基于16位代码单元来工作的，也就是说对于用16位编译过包含了代理编码对的的字符串，你可以会得到意想不到的结果，就像以下例子：

```js
var text = "𠮷";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(text.charAt(0));        // ""
console.log(text.charAt(1));        // ""
console.log(text.charCodeAt(0));    // 55362
console.log(text.charCodeAt(1));    // 57271
```

Unicode的单个字符 `"𠮷"`是用代理编码对来表示的，就其本身而言，上述的JavaScript字符串运算符把这个字符串当成了两个16位的字符。那就是说：

* `text`的`length`是2，但它应该是1。
* 当一个正则表达式尝试去匹配单个字符时会失败，因为它认为是有两个字符。
* 方法`charAt()`不能返回一个有效的字符序列，因为这两个16位的组合都不是一个可打印的字符。

方法`charCodeAt()`同样不能正确地识别字符，它会根据每一个代码单元返回相应的16位数字，但那也是在ES5里你能拿到最接近`text`真实值的方法。

然而在ES6中，它会迫使UTF-16字符串编码解决这些问题。基于这个字符编码来标准化字符串意味着JavaScript可以支持特意为代理编码对而设计的功能。本章节接下来的内容将会围绕这个功能以一些关键例子来展开讨论。

### codePointAt()方法

ES6新增的用以支持UTF-16的一个方法是`codePointAt()`，它检索在字符串中匹配给定位置所得的Unicode字符码。这个方法接收的是代码单元的位置而不是字符的位置，并且返回一个整数，如以下`console.log()`例子所显示的：

```js
var text = "𠮷a";

console.log(text.charCodeAt(0));    // 55362
console.log(text.charCodeAt(1));    // 57271
console.log(text.charCodeAt(2));    // 97

console.log(text.codePointAt(0));   // 134071
console.log(text.codePointAt(1));   // 57271
console.log(text.codePointAt(2));   // 97
```

除非`codePointAt()`方法是在非BMP字符集中执行，否则它会返回和`charCodeAt()`方法一样的结果。`text`中的第一个字符是非BMP的，因此由两个代码单元组成，也就是说，`length`属性的值是3而不是2.`charCodeAt()`方法对于位置0只返回第一个代码单元，然而哪怕字符码跨越多个代码单元，`codePointAt()`也会返回整个字符码。对于位置1（第一个字符的第二个字符码）和位置2（字符`"a"`），两个方法都会返回同样的值。

在一个字符上调用`codePointAt()`方法是判断字符是由一个或两个字符码组成最容易的方式。以下是你可以写来检查的一个函数：

```js
function is32Bit(c) {
    return c.codePointAt(0) > 0xFFFF;
}

console.log(is32Bit("𠮷"));         // true
console.log(is32Bit("a"));          // false
```

16位字符的最大值用十六进制表示是`FFFF`，因此任何大于该值的字符码都必须用两个代码单元来表示，总共32位。

### String.fromCodePoint()方法

当ECMAScript提供一种方式去做某些事情时，它同样会有意地提供一种方式去做相反的事情。你可以用`codePointAt()`去检索一个字符串里某个字符的字符码，而`String.fromCodePoint()`会从一个给定的字符码生成一个单字符的字符串。例如：

```js
console.log(String.fromCodePoint(134071));  // "𠮷"
```

把`String.fromCodePoint()`当成是`String.fromCharCode()`方法的一个更完整的版本。对于BMP里的所有字符两者返回一样的结果。当给定BMP以外字符的字符码时它们会有些不一样。

### normalize()方法

Unicode另一个有趣的方面是不同的字符在排序或者别的基于比较的操作中可能会被认为是相同的。有两种方式去定义这些关系。第一，*规范相等性*意味着两个字符码的序列被认为在各方面都是内部可变的。例如，两个字符的一个组合可以是正则式地相等于一个字符。第二种关系是*兼容性*。两个兼容的序列看起来不一样，但在特定的情况下可以互换使用。

鉴于这些关系，从根本上表示同一个文本的两个字符串可以包括不同的字符码序列。例如， 字符"æ"和两个字符的字符串"ae"可能会被互换使用，但除非它们在一定意义上被规范化了，否则严格上来说它们是不相等的。

ES6支持对字符串给定`normalize()`方法的Unicode规范化形式。这方法可选地接收一个字符串参数，这个参数表明将会应用以下Unicode规范化形式的其中一个：

* 规范化形式规范组成(`"NFC"`),默认
* 规范化形式规范分解 (`"NFD"`)
* 规范化形式兼容性组成 (`"NFKC"`)
* 规范化形式兼容性分解 (`"NFKD"`)

这四种规范化形式的区别不在本书的讨论范围。需要铭记的是，当比较两个字符串时，它们必须用同一种规范化形式来规范。例如：

```js
var normalized = values.map(function(text) {
    return text.normalize();
});

normalized.sort(function(first, second) {
    if (first < second) {
        return -1;
    } else if (first === second) {
        return 0;
    } else {
        return 1;
    }
});
```

这段代码将`values`数组里的字符串都转化成一个规范化形式，这样该数组可以被更好地排序。你也可以通过调用作为比较器一部分的`normalize()`方法来对原始数组进行排序。如下：

```js
values.sort(function(first, second) {
    var firstNormalized = first.normalize(),
        secondNormalized = second.normalize();

    if (firstNormalized < secondNormalized) {
        return -1;
    } else if (firstNormalized === secondNormalized) {
        return 0;
    } else {
        return 1;
    }
});
```

再次提醒，这段代码中需注意的最重要的事情是`first`和`second`都是用同样的方式规范化的。这些例子用的是默认的NFC规范，不过你可以像以下这样简单地就指定其它规范中的一个：

```js
values.sort(function(first, second) {
    var firstNormalized = first.normalize("NFD"),
        secondNormalized = second.normalize("NFD");

    if (firstNormalized < secondNormalized) {
        return -1;
    } else if (firstNormalized === secondNormalized) {
        return 0;
    } else {
        return 1;
    }
});
```

如果你之前从未考虑过Unicode的规范化，那么现在你可能不会太常用这个方法。但如果你开发国际性的应用，你一定会发现`normalize()`方法很好用。

尽管这些方法不是ES6提供给Unicode字符串操作的唯一改善，该标准还是加了两个很有用的语法元素。

### 正则表达式标识

通过正则表达式，你可以完成许多常用的字符串操作。但请记住，正则表达式采用16位代码单元，每一个都代表一个字符。为了阐明这个问题，ES6为正则表达式定义了一个标记`u`来表示Unicode。

#### 标记`u`的实战

当一个正则表达式设了`u`标记时，它便会切换成字符模式而不是代码单元模式。也就是说，正则表达式不应该再因字符串中的代理对而变得混乱，而应该如预期般运行。比如，以下代码：

```js
var text = "𠮷";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(/^.$/u.test(text));     // true
```

正则表达式`/^.$/`匹配用户输入的任何含有单个字符的字符串。 当不带标识`u`使用时，该正则表达式匹配的是代码单元，因此日本字符（由两个代码单元表示）将不匹配该表达式。当带上`u`标识时，正则表达式将比较字符而非代码单元，因此日本字符也会匹配。

#### 计算字符码

遗憾的是，ES6并没有添加一个可以用来决定一个字符串会有多少个字符码的方法。而带上标识`u`，你可以使用正则表达式来计算。如下：

```js
function codePointLength(text) {
    var result = text.match(/[\s\S]/gu);
    return result ? result.length : 0;
}

console.log(codePointLength("abc"));    // 3
console.log(codePointLength("𠮷bc"));   // 3
```

这个例子通过一个设置了激活了Unicode的全局应用的正则表达式，调用`match()`来检查`text`的空格和非空格字符（使用`[\s\S]`来保证该表达式匹配换行）。`result`包含了一个至少有一个匹配的数组，因此该数组的长度便是该字符串的字符码的个数。在Unicode中，字符串`"abc"`和`"𠮷bc"`都有三个字符，因此数组长度为3。

W> 尽管此方法可行，但它并不快，尤其是在应用长字符串时。你也可以使用字符串迭代器（在第八章中将会讨论）。通常尽可能地减少计算字符码。

#### 决定对标识`u`的支持

既然标识`u`是一个语法变化，在与ES6不兼容的JavaScript引擎中尝试使用它会导致语法错误。决定一个函数是否支持标识`u`的最安全的做法，如下：

```js
function hasRegExpU() {
    try {
        var pattern = new RegExp(".", "u");
        return true;
    } catch (ex) {
        return false;
    }
}
```

这个函数使用`RegExp`构造函数，将标识`u`作为一个参数传入。就是在旧版的JavaScript引擎，这语法也是有效的。但如果不支持`u`，该构造函数将会报错。

I> 如果你的代码仍需在旧版JavaScript引擎中运行，那么当使用标识`u`时请一直`RegExp`构造函数。这可以预防语法错误，并允许你可以不用停止执行而随意检测和使用标识`u`。

## 其它字符串的改动

JavaScript的字符串相比其它语言中相似的功能来说总是要滞后一些的。比如，在ES5中，字符串最后也只是获得了一个`trim()`方法，而在ES6中则继续扩大了JavaScript用新功能解析字符串的能力。

### 鉴别字符串子集的方法

自从JavaScript一开始引入了通过`indexOf()`方法在一个字符串中鉴别另一个字符串的，开发者们经常使用。ES6还包括了以下三个方法，来实现这功能：

* `includes()`方法若在字符串任意地方找到给定的文本便会返回true，否则返回false。
* `startsWith()`方法若在字符串最开始的地方找到给定的文本便返回true，否则返回false。
* `endsWith()`方法若在字符串的最后找到给定的文本便返回true，否则返回false。

以上每一个方法都接收两个参数：用来查找的文本，和可选的用来指定从何处开始查找的索引。当传入第二个参数时，`includes()`和`startsWith()`从索引位置开始匹配，而`endsWith()`则从字符串长度减去第二个参数所得的索引位置开始匹配；当省略第二个参数时，`includes()`和`startsWith()`从字符串最开始的位置开始搜索，而`endsWith()`则从末位位置开始查询。实际上，第二个参数减小了用以搜索的字符串长度。以下是这三个方法的一些实际用法：

```js
var msg = "Hello world!";

console.log(msg.startsWith("Hello"));       // true
console.log(msg.endsWith("!"));             // true
console.log(msg.includes("o"));             // true

console.log(msg.startsWith("o"));           // false
console.log(msg.endsWith("world!"));        // true
console.log(msg.includes("x"));             // false

console.log(msg.startsWith("o", 4));        // true
console.log(msg.endsWith("o", 8));          // true
console.log(msg.includes("o", 8));          // false
```

前六个调用都不包含第二个参数，因此如若需要它们会查找整个字符串。最后三个调用则只会查找字符串的一部分。`msg.startsWith("o", 4)`的调用从`msg`字符串的第4个索引，也就是“Hello”中的“o”开始查找匹配。因为会从字符串长度（12）中减去参数`8`，所以`msg.endsWith("o", 8)`的调用也是从第4个索引开始匹配。`msg.includes("o", 8)`的调用会从第8个索引，即“world”中的“r”开始匹配。

这三个方法都让鉴别字符串子集变得更容易，但每一个都只返回一个布尔值。如果你要找到一个字符串在另一个字符串中的确切位置，你可以使用`indexOf()`或者`lastIndexOf()`方法。

W> 若你传入的是一个正则表达式而非字符串，方法`startsWith()`，`endsWith()`，和`includes()`都会抛出错误。这便和可将正则表达式参数转成一个字符串进而搜索该字符串的`indexOf()`和`lastIndexOf()`方法形成了反差。

### repeat()方法

ES6同样给字符串添加了`repeat()`方法，它接收一个数字参数，用来说明字符串应该重复几次。该方法返回一个包含了被重复了指定次数次的原始字符串。比如：

```js
console.log("x".repeat(3));         // "xxx"
console.log("hello".repeat(2));     // "hellohello"
console.log("abc".repeat(4));       // "abcabcabcabc"
```

该方法是以上其它方法的一个方便函数，特别是在文本操作方面的用处。它在需要创建缩进级别的代码格式工具类中尤其有用。比如这样：

```js
// 用指定数目的空格来缩进
var indent = " ".repeat(4),
    indentLevel = 0;

// 无论何时你想增加缩进
var newIndent = indent.repeat(++indentLevel);
```

第一个`repeat()`的调用创建来了一个4个空格的字符串，变量`indentLevel`保持对缩进级别的追踪。然后，你可以利用增加的`indentLevel`作为参数调用`repeat()`来改变空格的个数。

对于一些正则表达式不适用的类别，ES6对正则表达式功能也作出了一些有用的改动。下面的部分强调了一些。

## 正则表达式的其他改动

正则表达式是JavaScript中处理字符串的一个重要部分，像该语言的很多功能一样，在最近的几个版本里并没有多大改动。然而，为了适应字符串的更新，ES6对正则表达式做出了一些改善。

### 正则表达式的`y`标记

作为正则表达式的一个专用扩展，`y`标记在Firefox实现了之后，在ES6中得到了标准化。`y`标记会影响正则表达式搜索的`sticky`属性，它告诉搜索应该从字符串某个特定的位置开始匹配字符，该位置由正则表达式的`lastIndex`属性指定。如果在那个位置上没有匹配到，正则表达式则会停止匹配 。以下代码展示了这是如何运行的：

```js
var text = "hello1 hello2 hello3",
    pattern = /hello\d\s?/,
    result = pattern.exec(text),
    globalPattern = /hello\d\s?/g,
    globalResult = globalPattern.exec(text),
    stickyPattern = /hello\d\s?/y,
    stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello1 "
console.log(stickyResult[0]);   // "hello1 "

pattern.lastIndex = 1;
globalPattern.lastIndex = 1;
stickyPattern.lastIndex = 1;

result = pattern.exec(text);
globalResult = globalPattern.exec(text);
stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello2 "
console.log(stickyResult[0]);   // 错误! stickyResult为空
```

这个例子有三个正则表达式。`pattern`中的表达式没有标记，`globalPattern`中使用了`g`标记，而`stickyPattern`则使用了`y`标记。在第一组的三个`console.log`调用中，三个正则表达式都应该返回以空格结尾的`"hello1 "`。

在那之后，三个表达式的`lastIndex`属性被改成了1，即所有正则表达式应该从第二个字符开始匹配。没有标记的表达式完全忽略`lastIndex`的变化，仍不带任何缩进地匹配`"hello1 "`。`g`标记的表达式则从第二个字符（`"e"`）开始往前查找，进而匹配`"hello2 "`。而具有粘连性的正则表达式从第二个字符开始匹配不到任何东西，因此`stickyResult`为`null`。

无论何时一个操作被执行，粘连性标记都会在最后一个匹配之后保存`lastIndex`的下一个字符。如果一个操作没有匹配任何结果，那么`lastIndex`便会被设回0。全局标记的表现行为也一样，如下阐述：

```js
var text = "hello1 hello2 hello3",
    pattern = /hello\d\s?/,
    result = pattern.exec(text),
    globalPattern = /hello\d\s?/g,
    globalResult = globalPattern.exec(text),
    stickyPattern = /hello\d\s?/y,
    stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello1 "
console.log(stickyResult[0]);   // "hello1 "

console.log(pattern.lastIndex);         // 0
console.log(globalPattern.lastIndex);   // 7
console.log(stickyPattern.lastIndex);   // 7

result = pattern.exec(text);
globalResult = globalPattern.exec(text);
stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello2 "
console.log(stickyResult[0]);   // "hello2 "

console.log(pattern.lastIndex);         // 0
console.log(globalPattern.lastIndex);   // 14
console.log(stickyPattern.lastIndex);   // 14
```

对于变量`stickyPattern`和`globalPattern`，第一次调用`exec()`后`lastIndex`的值变为了7，而第二次调用后变为了14。

关于粘连性标记还有额外两个微妙的细节需要记住：

1. 只有在调用正则表达式对象里存在的方法时`lastIndex`属性才得以赞善，如`exec()`和`test()`方法。将正则表达式传入一个字符串方法，如`match()`，不会导致粘连行为。
2. 当使用字符`^`来匹配一个字符串的开头，正则表达式只会从该字符串的开头（或者是多行模式下的首行）开始匹配。而当`lastIndex`为0时，`^`使得粘连性正则表达式与非粘连性表达式无异。若`lastIndex`不对应一个字符串在当行模式下的开头或者多行模式下一个行的开头时，粘连性正则表达式永远不会匹配。

一如其它的正则表达式标记，你可以通过一个属性来检测`y`的存在。这种情况下，你可以如下检查`sticky`属性：

```js
var pattern = /hello\d/y;

console.log(pattern.sticky);    // true
```

若粘连性标记存在，`sticky`属性会被设为true，否则为false。基于该标记的出现，`sticky`属性是可读的，因此不能通过代码来改变。

与`u`标记类似，`y`标记是个语法改动，因此在旧版JavaScript引擎中它都会引起语法错误。你可以使用以下方法来检查是否支持该语法：

```js
function hasRegExpY() {
    try {
        var pattern = new RegExp(".", "y");
        return true;
    } catch (ex) {
        return false;
    }
}
```

就像`u`检查，如果不同创建一个带有`y`标记的正则表达式则返回false。与`u`的最后一个相似之处，若你在旧版JavaScript引擎中运行的代码里使用`y`，请确保在定义那些正则表达式时使用`RegExp`构造函数以避免语法错误。

### 复制正则表达式

在ES5中，你可以通过将正则表达式传入到`RegExp`构造函数中来复制正则表达式，就像这样：

```js
var re1 = /ab/i,
    re2 = new RegExp(re1);
```

变量`re2`只是变量`re1`的一个拷贝。但如果你给`RegExp`构造函数传入用以指定正则表达式标记的第二个参数，你的代码将不可行。如此例：

```js
var re1 = /ab/i,

    // ES5中会抛出错误，ES6则不会
    re2 = new RegExp(re1, "g");
```

若在ES5环境中执行这段代码，则会得到一个错误指明当第一个参数是一个正则表达式时，第二个参数不能使用。ES6改变了这种行为，使得第二个参数可以使用并且改写在第一个参数中存在的任意标记。比如：

```js
var re1 = /ab/i,

    // ES5中会抛出错误，ES6则不会
    re2 = new RegExp(re1, "g");


console.log(re1.toString());            // "/ab/i"
console.log(re2.toString());            // "/ab/g"

console.log(re1.test("ab"));            // true
console.log(re2.test("ab"));            // true

console.log(re1.test("AB"));            // true
console.log(re2.test("AB"));            // false
```

这段代码中，`re1`有一个大小写敏感的`i`标记，而`re2`只有全局标记`g`。`RegExp`构造函数从`re1`复制表达式 ，并以`i`标记替换 `g`标记。没有第二个参数的话，`re2`会有着`re1`一样的标记。

### `flags`属性

随着一个新标记的增加和如何使用标记的改变，ES6增加了一个属性来联合使用。在ES5中，你可以通过`source`属性获取一个正则表达式的文本内容，但要或者标记字符串，你需要解析方法`toString()`的输出。如下：

```js
function getFlags(re) {
    var text = re.toString();
    return text.substring(text.lastIndexOf("/") + 1, text.length);
}

// toString()是"/ab/g"
var re = /ab/g;

console.log(getFlags(re));          // "g"
```

此代码将正则表达式转成字符串，然后返回了在最后一个`/`找到的字符。那些字符便是标记。

ES6通过增加`flags`联合`source`属性，使得获取标记变得更容易。这两个属性都是原型访问器属性，它们只被指派了一个getter方法，因此是可读的。`flags`属性也使得在调试和继承上更容易检测正则表达式。

对ES6迟来的补充，`flags`属性返回应用在正则表达式上的任意标记的字符串表示。比如：

```js
var re = /ab/g;

console.log(re.source);     // "ab"
console.log(re.flags);      // "g"
```

这获取了`re`上的所有标记，并用比`toString()`技术更少行的代码将它们打印到了控制台上。`source`和`flags`一起使用允许你直接提取你所需的正则表达式片段，而不需要解析正则表达式字符串。

本章目前所涵盖的关于字符串和正则表达式的变化必然是强大的，然而ES6在更大程度上改善了你在字符串上的能力。它在表格上引入了文字类型，使得字符串更灵活。

##  模板常量

与其他语言的字符串相比，JavaScript的字符串的功能总是有限的。比如，在ES6之前本章目前所提及的那些字符串方法都是没有的，而且字符串拼接也是尽可能地简单。为了开发者们能够解决更复杂的问题，ES6的*模板常量*提供了一个创建领域特定语言（DSL）来处理内容，这比ES5及更早版本中可行的方法更为安全（DSL是一个编程语言，与JavaScript一样的多用途语言相反，它是为一个特定的，狭隘目的而设计的）。ECMAScript的维基百科为[模板常量]提供了以下描述(http://wiki.ecmascript.org/doku.php?id=harmony:quasis)：

> 此方案用语法糖扩展了ECMAScript的语法，使得用以提供DSL的库相比其他语言更容易生产，查询和操作内容，使得它们免疫于或抵抗注入式攻击，如XSS，SQL注入等。

可是事实上，对于在ES5中缺失的以下JavaScript功能，ES6提供了模板常量作为解决方案：

* **多行字符串** 多行字符串的一个正式概念。
* **基本的字符串格式** 为变量中的值取代部分字符串的能力。
* **HTML转义** 转换字符串以使其安全插入HTML的能力。

相比给JavaScript现存的字符串添加更多的功能，模板常量代表用来解决这些问题的一个全新方法。

### 基本语法

最简单的，模板常量的行为就如被反引号（`` ` ``）而不是单双引号分隔的一般字符串一样。比如以下例子：

```js
let message = `Hello world!`;

console.log(message);               // "Hello world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 12
```

这段代码演示了包含了一个普通JavaScript字符串的变量`message`。模板常量语法被用来创建字符串值，该值随后被指定给了变量`message`。

如果你要在你的字符串中使用反引号，只需通过反斜杠（`\`）来转义。一如变量`message`的这个版本：

```js
let message = `\`Hello\` world!`;

console.log(message);               // "`Hello` world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 14
```

模板常量里的单双引号不需要转义。

### 多行字符串

JavaScript开发者们从该语言的第一个版本开始便想要一个可以创建多行字符串的方法。然而使用单双引号时，字符串必须完全被包含在一行里。

#### ES6之前的临时方案

由于长久以来的一个语法错误，JavaScript有一个临时解决方案。你可以在新行前加入一个反斜杠（`\`）来创建多行。比如这个例子：

```js
var message = "Multiline \
string";

console.log(message);       // "Multiline string"
```

当打印到控制台时，因为反斜杠被用作一个连续符而非新行，所以`message`字符串没有新行。为了在输出中显示一个新行，你需要手动包括它：

```js
var message = "Multiline \n\
string";

console.log(message);       // "Multiline
                            //  string"
```

在所有JavaScript的主要引擎中，这应该将`Multiline String`打印成两行。但这种行为被定义为一个错误，且许多开发者都建议避免它。

其他ES6之前的版本通常 依靠数组或者字符串连接来尝试创建多行字符串。比如：

```js
var message = [
    "Multiline ",
    "string"
].join("\n");

let message = "Multiline \n" +
    "string";
```

开发者们提供的所有JavaScript中缺失的多行字符串功能的临时方案都有着一些遗憾，亟需改善。

#### 多行字符串的简单方法

因为没有特殊语法，ES6的模板常量使多行字符串变得简单。只是在你所需的地方包括新行，它在结果上便会显示。比如：

```js
let message = `Multiline
string`;

console.log(message);           // "Multiline
                                //  string"
console.log(message.length);    // 16
```

反引号里的所有空格都是字符串的一部分，因此对缩进要特别注意。比如：

```js
let message = `Multiline
               string`;

console.log(message);           // "Multiline
                                //                 string"
console.log(message.length);    // 31
```

这段代码中，模板常量第二行前面的所有空格都被认作字符串本身的一部分。如果对你来说，文本行需要适当的缩进，这很重要，那么可以考虑在一个多行的模板常量的第一行不留任何东西，然后在那之后缩进。如下：

```js
let html = `
<div>
    <h1>Title</h1>
</div>`.trim();
```

这段代码在第一行开始模板常量，但并没有任何文本，直到第二行。HTML标签被缩进以便看起来是正确的，然后`trim()`方法 被调用来移除最初的空行。

A> 如果你愿意，你也可以在一个模板常量里使用`\n`来指示一个新行应该被插入：
A> {:lang="js"}
A> ~~~~~~~~
A>
A> let message = `Multiline\nstring`;
A>
A> console.log(message);           // "Multiline
A>                                 //  string"
A> console.log(message.length);    // 16
A> ~~~~~~~~

### 替换

此时此刻，模板常量可能看起来像是一般JavaScript字符串的豪华版。两者真正的区别在于模板常量的*替换*。替换允许你在一个模板常量里插入任何有效的JavaScript表达式，并将输出结果作为字符串的一部分。

替换被`${`开头和`$}`结尾限制着，中间可以是任何JavaScript表达式。最简单的替换让你可在一个结果字符串中直接插入本地变量。像这样：

```js
let name = "Nicholas",
    message = `Hello, ${name}.`;

console.log(message);       // "Hello, Nicholas."
```

替换`${name}`访问本地变量`name`，将`name`插入字符串`message`。变量`message`直接持有替换后的结果。

I> 一个模板常量可以访问其定义所在的作用域内的任意变量。在严格和非严格模式下若在模板常量中尝试使用一个未定义的变量将会抛出错误。

因为所有的替换都是JavaScript表达式，不只是简单的变量名字才能替换。你可以简单地插入计算，函数调用和更多。比如：

```js
let count = 10,
    price = 0.25,
    message = `${count} items cost $${(count * price).toFixed(2)}.`;

console.log(message);       // "10 items cost $2.50."
```

这代码运行了一个计算作为模板常量的一部分。变量`count`和`price`相乘得到一个结果，然后使用`.toFixed()`格式化成两个小数点的数。第二个替换前的美元符号因为后面没有跟着左括号，所以会保留输出。

模板常量也是JavaScript表达式，也就是说你可以把一个模板常量传入到另一个模板常量中，如下例：

```js
let name = "Nicholas",
    message = `Hello, ${
        `my name is ${ name }`
    }.`;

console.log(message);        // "Hello, my name is Nicholas."
```

这个例子在第一个模板常量里嵌套了第二个。在第一个`${`之后开始了另一个模板常量。第二个`${`表示内在模板常量里一个插入表达式的开头。那个表达式便是变量`name`，它被插入到了结果中。

### 标记的模板常量

目前你已看见模板常量是如何创建多行字符串和不用拼接而给字符串插入值。然而模板常量的真正力量来自标记的模板常量。一个*模板标记*在模板常量上作一个转换，然后返回最后的字符串值。该标记在模板的开头处指定，就在第一个`` ` ``字符前，如下显示：

```js
let message = tag`Hello world`;
```

在这个例子中，`tag`是一个应用在模板常量```Hello world` ``上的模板标记。

#### 定义标记

*标记*只是一个函数，与被处理过的模板常量数据一起调用。该标记获取关于模板常量的数据作为单独的片段，并且将这些片段组合起来创建结果。第一个参数是包含JavaScript解析过的文字串的一个数组。而后的每一个参数是每一个替换的解析值。

为了更容易地处理数据，一般来说标记函数都是通过剩余参数来定义的，就像下面的例子：

```js
function tag(literals, ...substitutions) {
    // 返回一个字符串
}
```

为了更好的理解什么东西被传入了标记中，我们来看看下面例子：

```js
let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $${(count * price).toFixed(2)}.`;
```
如果一个函数调用了`passthru()`，那么该函数将获取三个参数。首先，它会得到一个`literals`数组，其包含以下元素：

* 第一个替换（`""`）前的空字符串；
* 第一个替换之后与第二个替换之前的字符串（`" items cost $"`）；
* 第二个替换之后的字符串（`"."`）

下一个参数会是`10`，变量`count`的解析值。这成了`substitutions`数组里的第一个元素。最后一个参数是`(count * price).toFixed(2)`的解析值`"2.50"`，也是`substitutions`数组里的第二个元素。

要注意，`literals`的第一个元素是一个空字符串。这就保证了`literals[0]`总是该字符串的开头，就像`literals[literals.length - 1]`总是字符串的末尾。替换总是要比文本少一个的，也就是说表达式`substitutions.length === literals.length - 1`总是返回true的。

使用该模式，`literals`和`substitutions`数组可以混合来创建一个结果字符串。先是`literals`的第一个元素，然后是`substitutions`的第一个元素，以此循环，直到完成整个字符串。作为一个例子，你可以通过交替这两个数组中的值来模拟一个模板常量的默认行为：

```js
function passthru(literals, ...substitutions) {
    let result = "";

    // 只为计算替换执行该循环
    for (let i = 0; i < substitutions.length; i++) {
        result += literals[i];
        result += substitutions[i];
    }

    // 增加最后一个字面量
    result += literals[literals.length - 1];

    return result;
}

let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $${(count * price).toFixed(2)}.`;

console.log(message);       // "10 items cost $2.50."
```

这个例子定义了一个`passthru`标记，来执行与模板常量的默认行为一样的转换。唯一麻烦的是在循环中使用`substitutions.length`而不是`literals.length`，以避免意外地运行到数组`substitutions`的结尾。因在ES6中定义了`literals`和`substitutions`的良好关系，这方法是可行的。

I> 包含在`substitutions`的值并不是必要的字符串。如上个例子，如果一个表达式求值是一个数，那么该数值就会被传入。标记的一部分职责便是决定该如何在一个结果中输出那样的值。

#### 在模板常量中使用原始数值

模板标记同样可以访问原始的字符串信息，即对字符转义在字符转换成与他们等同的字符之前的访问。与原始的字符串值一起使用，最简单的方法是使用自带的`String.raw()`标记。比如：

```js
let message1 = `Multiline\nstring`,
    message2 = String.raw`Multiline\nstring`;

console.log(message1);          // "Multiline
                                //  string"
console.log(message2);          // "Multiline\\nstring"
```

这段代码中，`message1`的`\n`被解析成了一个新行，而`message2`中的`\n`则以其原始格式`"\\n"`来返回（斜杠和字符`n`）。获取像这样的原始字符串信息，必要时有利于更复杂的流程。

原始的字符串信息同样传给了模板标记。标记函数的第一个参数是有着额外属性`raw`的一个数组。`raw`属性是一个包含每一个字面量值的原始等同值。比如，`literals[0]`的值总是有一个包含原始字符串信息的等同值`literals.raw[0]`。知道这一点，你便可以通过以下代码来模拟`String.raw()`：

```js
function raw(literals, ...substitutions) {
    let result = "";

    // 只为计算替换执行该循环
    for (let i = 0; i < substitutions.length; i++) {
        result += literals.raw[i];      // 使用的是原始值
        result += substitutions[i];
    }

    // 增加最后一个字面量
    result += literals.raw[literals.length - 1];

    return result;
}

let message = raw`Multiline\nstring`;

console.log(message);           // "Multiline\\nstring"
console.log(message.length);    // 17
```

这里使用了`literals.raw`而不是`literals`来输出字符串结果。那就意味着，任何字符转义，包括Unicode字符码转义，应该以它们的原始形式返回。当你想要输出一个包含你想要字符的字符串，该字符需要包括字符转义时，原始字符串在这方面是很有用的（比如说，你想要生成关于某些字符的文档，你可能会想要输出它所显示的真正的字符）。

## 总结

完整的Unicode支持使得JavaScript可以通过许多逻辑方法来处理UTF-16字符。通过`codePointAt()`和`String.fromCodePoint()`在字符码和字符之间转移的这种能力对于字符串操作来说是很重要的一步。正则表达式`u`标记的加入使得在字符码而非16位字符的操作变成了可能，而`normalize()`方法提供了更多恰当的字符串比较。

ES6同样增加了新的方法来协助字符串，无论子集在父字符串的哪个位置，这都让你更容易地识别一个字符串子集。正则表达式里也加入了更多的功能。

模板常量是ES6里一个重要的加入，使得你能够创建领域特定语言（DSL）从而更容易创建字符串。在模板常量里直接插入变量的能力意味着开发者们可以有一个比字符串拼接更安全的工具来组成带有变量的长字符串。

对于多行字符串的内置支持也使得模板常量变成了从没有此功能的一般JavaScript字符串的一个有用的升级。尽管模板常量里允许直接新行，你仍可使用`\n`和其他字符转义顺序。

模板标记是创建DSL功能最重要的一部分。标记是用来获取模板常量片段作为参数的功能。而后你可使用那些数据来返回一个适当的字符串值。该数据包括了字面量，它们的原始等同值和任何替换值。这些信息片段随后可以被用来决定该标记的正确输出。
