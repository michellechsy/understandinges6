# Sets and Maps

在JavaScript的历史中，它只有一种集合类型，由`Array`类型来表示（尽管有些人会争论说所有的非数组对象一开始的预期用途都只是键值对的集合，最初和数组非常的不同）。在JavaScript里一开始数组的用法就跟其他语言的数组一样，但是其他集合选项的缺失就意味着数组也要经常被用作队列和栈。由于数组只能使用数字索引，当需要一个非数字索引时开发者们使用非数组对象就显得很有必要了。该技巧就曾导致了使用非数组对象实现自定义的Set和Map。

一个*set*是指一个不包含重复值的列。通常你不会像访问数组元素那样去访问一个Set里的每一个元素；然而检查一个值是否存在一个Set里却是很常见的。一个*map*是指与某些值相对应的键的一个集合。同样地，Map里的每一个元素保存数据的两个部分，且通过指定键来读取某个值。为了存储数据以便之后可以更快地读取，Map频繁被用作缓存。然而，ES5并没有正式地引入Set和Map，这个局限性使得开发者们同样地使用非数组对象来作为临时方案。

ES6为JavaScript添加了Set和Map，本章将为你介绍你所需的关于这两种集合类型的内容。

首先，我将介绍在ES6之前开发者们用以实现Set和Map的临时方案，并且解释一下为什么那些实现是有问题的。基于这些重要的背后原因，我将叙述一下ES6中Set和Map的工作原理。

## ES5中的Sets和Maps

在ES5中，开发者们通过对象属性来模拟Set和Map，如下：

```js
let set = Object.create(null);

set.foo = true;

// 检查是否存在
if (set.foo) {

    // do something
}
```

此例中变量`set`是一个包含了`null`属性的对象，这样可以保证对象没有继承的属性。利用对象属性作为检查的唯一值是ES5中一个常用的方法。当在`set`对象中加入一个属性时，它的值为`true`，因此条件语句（如例子中的`if`语句）可以很容易就判断出该值是否存在。

一个对象被用作Set和Map实际上唯一的差别在于存储的值。比如，这个使用一个对象作为Map的例子：

```js
let map = Object.create(null);

map.foo = "bar";

// retrieving a value
let value = map.foo;

console.log(value);         // "bar"
```

这段代码在键`foo`下存储了一个字符串值`"bar"`。跟Set不一样，Map通常是用来获取信息，而不只是检查键是否存在。

## 临时方案带来的问题

在一些简单的情况下，使用对象作为Set和Map是没问题的，但当遇到对象属性的限制时，这种方式会变得越来越复杂。比如说，由于所有的对象属性都必须是字符串，你必须保证不会有两个键是一样的字符串。考虑下面这个例子：

```js
let map = Object.create(null);

map[5] = "foo";

console.log(map["5"]);      // "foo"
```

这个例子给一个数字键`5`赋值了字符串值`"foo"`。由于该数字值在内部被转换成了一个字符串，`map["5"]`和`map[5]`实际上指向同一个属性。当你想要使用数字和字符串作为键时，这种内部转换会导致一些问题。使用对象作为键时也会导致另一些问题，像这样：

```js
let map = Object.create(null),
    key1 = {},
    key2 = {};

map[key1] = "foo";

console.log(map[key2]);     // "foo"
```

此处`map[key2]`和`map[key1]`指向了同一个值。因为对象属性必须是字符串，对象`key1`和`key2`被转换成了字符串。而由于对象的默认字符串表示是`"[object Object]"`,`key1`和`key2`都被转换成了那个字符串。从逻辑上来说，我们会期望不同的对象键会是不一样的，但这个转换会产生一些可能不易发现的错误。

使用默认的字符串表示来做转换的这种行为使得使用对象作为键变得困难（使用对象作为Set也会有同样的问题）。

使用假值（false）作为键的Map同样有它们自己特有的问题。在需要使用布尔值的情况下，比如在`if`条件语句里，一个假值会被自动转换成false。单独使用这种转换其实不是问题——只要你小心使用这些值。比如这段代码：

```js
let map = Object.create(null);

map.count = 1;

// checking for the existence of "count" or a nonzero value?
if (map.count) {
    // ...
}
```

这个例子有歧义的地方在于我们不知道`map.count`应该怎么用。`if`条件是想要检查`map.count`是否存在还是检查它的值不为零？因为值1是true的，`if`语句里的代码会被执行。然而，如果`map.count`是0，或者`map.count`不存在，`if`语句里的代码则不会被执行。

如果这问题出现在大型应用中，这些问题会很难追踪和调试。这也是ES6增加Set和Map的一个主要原因。

I> JavaScript里有一个`in`操作，如果一个属性存在于一个对象中，不需读取对象的值，这个操作也会返回`true`。然而，`in`操作还会搜索对象的原型，这就意味着只有当对象有一个`null`原型时它才是安全的。就算是这样，很多开发者仍然错误地使用像上述例子那样的代码而不是用`in`。

## ES6中的Set

ES6添加了一个`Set`类型，它是一个没有重复值的有序数列。通过增加一个追踪离散值更有效的方法，Set允许快速访问它们的数据。

### 创建Set及添加元素

使用`new Set()`可以创建一个Set，而调用`add()`方法则可以将元素添加到一个Set里边。你可以检查`size`属性来判断一个Set有多少个元素：

```js
let set = new Set();
set.add(5);
set.add("5");

console.log(set.size);    // 2
```

Set并不强迫值来判断它们是否相同。那就意味着一个Set既可以有数字`5`也可以有字符串`"5"`作为两个单独的元素（唯一的例外是-0和+0会被认为是一样的）。你也可以往Set里添加多个对象，而那些对象将保持不同：

```js
let set = new Set(),
    key1 = {},
    key2 = {};

set.add(key1);
set.add(key2);

console.log(set.size);    // 2
```

因为`key1`和`key2`不被转换成字符串，它们被当作Set里的两个独立元素（记住，如果它们被转换成字符串，它们都等于`"[Object object]"`）。

`add()`方法以同样地值被调用了多次，实际上第一次之后的所有调用都会被忽略：

```js
let set = new Set();
set.add(5);
set.add("5");
set.add(5);     // 重复——会被忽略

console.log(set.size);    // 2
```

你可以用一个数组来初始化一个Set，而且`Set`构造器将保证只有唯一的值会被使用。比如：

```js
let set = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
console.log(set.size);    // 5
```

这个例子中，一个包含了重复值的数组被用来初始化Set。虽然数字`5`在数组中出现了四次，它在这个Set中只会出现一次。这功能使得在现有代码或者JSON结构中使用Set更容易。

I> `Set`构造器常接收任意可迭代的对象作为参数。数组是可以的，因为它们默认是可迭代的，Set和Map也是。`Set`构造器使用一个迭代器从参数中提取值（可迭代和迭代器的详情将在第八章讨论）。

你可以使用`has()`方法来测试Set中有哪些值，如下：

```js
let set = new Set();
set.add(5);
set.add("5");

console.log(set.has(5));    // true
console.log(set.has(6));    // false
```

这里因为Set中并没有值6，`set.has(6)`会返回false。

### 移除值

从一个Set中移除值也是可能的。你可以使用`delete()`方法移除一个值，或者通过调用`clear()`方法移除所有的值：

```js
let set = new Set();
set.add(5);
set.add("5");

console.log(set.has(5));    // true

set.delete(5);

console.log(set.has(5));    // false
console.log(set.size);      // 1

set.clear();

console.log(set.has("5"));  // false
console.log(set.size);      // 0
```

`delete()`调用后，只有`5`消失了；而在`clear()`方法执行后，`set`空了。

所有的这些相当于追踪一个唯一有序值的非常简单的机制。然而，如果你想要在一个Set里添加元素，然后对每一个元素进行一些操作会怎样呢？这就是`forEach()`的作用了。

### Set的forEach()方法

如果你习惯使用数组，那么你可能已经对`forEach()`方法很熟悉了。ES5为数组添加了`forEach()`方法，它使得操作数组的每一个元素变得更简单，并且不需要设立`for`循环。这方法在开发者中颇受欢迎，因此这方法在Set中同样可用而且用法是一样的。

`forEach()`方法被传入一个有三个参数的回调函数：

1. set中来自下一个位置的值
2. 同样的值作为第一个参数
3. 值被读取的set

Set版本和数组版本的`forEach()`最奇怪的一个差异在于回调函数的第一第二个参数是一样的。虽然这个看起来像是一个错误，但这里还是有一个很好的原因。

其它那些有`forEach()`方法（数组和Map）的对象的回调函数有三个参数。数组和Map的头两个参数是值和键（数组的是数字索引）。

然而Set没有键。ES6标准背后的团队可能曾让Set版本的`forEach()`接收两个参数，但这就使得它和其它两个不一样。因而他们找到了一个方法使得回调函数可以保持一致，接收三个参数：Set里的每一个值都会被当做键和值。这样，Set的`forEach()`方法的头两个参数总是一样的，从而使得这个功能与数组和Map的`forEach()`方法保持一致。

除了参数上的差异，Set和数组的`forEach()`用法基本上是一样的。比如：

```js
let set = new Set([1, 2]);

set.forEach(function(value, key, ownerSet) {
    console.log(key + " " + value);
    console.log(ownerSet === set);
});
```

这段代码迭代了Set中的每一个元素并输出了传入到回调函数`forEach()`的值。回调函数每一次执行，`key`和`value`都是一样的，而`ownerSet`总是`set`。这段代码会输出以下内容：

```
1 1
true
2 2
true
```

数组也是一样的，如果你需要在回调函数中使用`this`，你可以把`this`作为第二个参数传入到`forEach()`中：

```js
let set = new Set([1, 2]);

let processor = {
    output(value) {
        console.log(value);
    },
    process(dataSet) {
        dataSet.forEach(function(value) {
            this.output(value);
        }, this);
    }
};

processor.process(set);
```

此例中`processor.process()`方法调用了set中的`forEach()`并且把`this`作为回调的`this`值。这样`this.output()`才能正确地调用到`processor.output()`方法。`forEach()`回调函数只用到了第一个参数`value`，所以省略了其它参数。你也可以通过箭头函数来达到同样的效果，而且不需要传入第二参数。像这样：

```js
let set = new Set([1, 2]);

let processor = {
    output(value) {
        console.log(value);
    },
    process(dataSet) {
        dataSet.forEach((value) => this.output(value));
    }
};

processor.process(set);
```

这个例子中的箭头函数从内嵌的`process()`函数中读取`this`，所以它应该正确地将`this.output()`传给`processor.output()`调用中。

要记住，Set适用于追踪值而且`forEach()`可以让你按顺序地读取每一个值，但你不能像数组那样通过索引来直接访问值。如果你需要这样做，那最好的做法是将Set转换成一个数组。

### 将Set转换成数组

将数组转成一个Set很容易，只需将数组传给`Set`构造器。通过展开操作符将一个Set转回成一个数组也容易。第三章介绍过展开操作符 （`...`）可用来将一个数组里的元素拆分成独立的函数参数。你也可以使用展开操作符来操作可迭代对象，如Set，将它们转换成数组。比如：

```js
let set = new Set([1, 2, 3, 3, 3, 4, 5]),
    array = [...set];

console.log(array);             // [1,2,3,4,5]
```

Here, a set is initially loaded with an array that contains duplicates. The set removes the duplicates, and then the items are placed into a new array using the spread operator. The set itself still contains the same items (`1`, `2`, `3`, `4`, and `5`) it received when it was created. They've just been copied to a new array.

This approach is useful when you already have an array and want to create an array without duplicates. For example:

```js
function eliminateDuplicates(items) {
    return [...new Set(items)];
}

let numbers = [1, 2, 3, 3, 3, 4, 5],
    noDuplicates = eliminateDuplicates(numbers);

console.log(noDuplicates);      // [1,2,3,4,5]
```

In the `eliminateDuplicates()` function, the set is just a temporary intermediary used to filter out duplicate values before creating a new array that has no duplicates.

### Weak Sets

The `Set` type could alternately be called a strong set, because of the way it stores object references. An object stored in an instance of `Set` is effectively the same as storing that object inside a variable. As long as a reference to that `Set` instance exists, the object cannot be garbage collected to free memory. For example:

```js
let set = new Set(),
    key = {};

set.add(key);
console.log(set.size);      // 1

// eliminate original reference
key = null;

console.log(set.size);      // 1

// get the original reference back
key = [...set][0];
```

In this example, setting `key` to `null` clears one reference of the `key` object, but another remains inside `set`. You can still retrieve `key` by converting the set to an array with the spread operator and accessing the first item. That result is fine for most programs, but sometimes, it's better for references in a set to disappear when all other references disappear. For instance, if your JavaScript code is running in a web page and wants to keep track of DOM elements that might be removed by another script, you don't want your code holding onto the last reference to a DOM element. (That situation is called a *memory leak*.)

To alleviate such issues, ECMAScript 6 also includes *weak sets*, which only store weak object references and cannot store primitive values. A *weak reference* to an object does not prevent garbage collection if it is the only remaining reference.

#### Creating a Weak Set

Weak sets are created using the `WeakSet` constructor and have an `add()` method, a `has()` method, and a `delete()` method. Here's an example that uses all three:

```js
let set = new WeakSet(),
    key = {};

// add the object to the set
set.add(key);

console.log(set.has(key));      // true

set.delete(key);

console.log(set.has(key));      // false
```

Using a weak set is a lot like using a regular set. You can add, remove, and check for references in the weak set. You can also seed a weak set with values by passing an iterable to the constructor:

```js
let key1 = {},
    key2 = {},
    set = new WeakSet([key1, key2]);

console.log(set.has(key1));     // true
console.log(set.has(key2));     // true
```

In this example, an array is passed to the `WeakSet` constructor. Since this array contains two objects, those objects are added into the weak set. Keep in mind that an error will be thrown if the array contains any non-object values, since `WeakSet` can't accept primitive values.

#### Key Differences Between Set Types

The biggest difference between weak sets and regular sets is the weak reference held to the object value. Here's an example that demonstrates that difference:

```js
let set = new WeakSet(),
    key = {};

// add the object to the set
set.add(key);

console.log(set.has(key));      // true

// remove the last strong reference to key, also removes from weak set
key = null;
```

After this code executes, the reference to `key` in the weak set is no longer accessible. It is not possible to verify its removal because you would need one reference to that object to pass to the `has()` method. This can make testing weak sets a little confusing, but you can trust that the reference has been properly removed by the JavaScript engine.

These examples show that weak sets share some characteristics with regular sets, but there are some key differences. Those are:

1. In a `WeakSet` instance, the `add()` method throws an error when passed a non-object (`has()` and `delete()` always return `false` for non-object arguments).
2. Weak sets are not iterables and therefore cannot be used in a `for-of` loop.
3. Weak sets do not expose any iterators (such as the `keys()` and `values()` methods), so there is no way to programmatically determine the contents of a weak set.
4. Weak sets do not have a `forEach()` method.
5. Weak sets do not have a `size` property.

The seemingly limited functionality of weak sets is necessary in order to properly handle memory. In general, if you only need to track object references, then you should use a weak set instead of a regular set.

Sets give you a new way to handle lists of values, but they aren't useful when you need to associate additional information with those values. That's why ECMAScript 6 also adds maps.

## Maps in ECMAScript 6

The ECMAScript 6 `Map` type is an ordered list of key-value pairs, where both the key and the value can have any type. Keys equivalence is determined by using the same approach as `Set` objects, so you can have both a key of `5` and a key of `"5"` because they are different types. This is quite different from using object properties as keys, as object properties always coerce values into strings.

You can add items to maps by calling the `set()` method and passing it a key and the value to associate with the key. You can later retrieve a value by passing the key to the `get()` method. For example:

```js
let map = new Map();
map.set("title", "Understanding ES6");
map.set("year", 2016);

console.log(map.get("title"));      // "Understanding ES6"
console.log(map.get("year"));       // 2016
```

In this example, two key-value pairs are stored. The `"title"` key stores a string while the `"year"` key stores a number. The `get()` method is called later to retrieve the values for both keys. If either key didn't exist in the map, then `get()` would have returned the special value `undefined` instead of a value.

You can also use objects as keys, which isn't possible when using object properties to create a map in the old workaround approach. Here's an example:

```js
let map = new Map(),
    key1 = {},
    key2 = {};

map.set(key1, 5);
map.set(key2, 42);

console.log(map.get(key1));         // 5
console.log(map.get(key2));         // 42
```

This code uses the objects `key1` and `key2` as keys in the map to store two different values. Because these keys are not coerced into another form, each object is considered unique. This allows you to associate additional data with an object without modifying the object itself.

### Map Methods

Maps share several methods with sets. That is intentional, and it allows you to interact with maps and sets in similar ways. These three methods are available on both maps and sets:

* `has(key)` - Determines if the given key exists in the map
* `delete(key)` - Removes the key and its associated value from the map
* `clear()` - Removes all keys and values from the map

Maps also have a `size` property that indicates how many key-value pairs it contains. This code uses all three methods and `size` in different ways:

```js
let map = new Map();
map.set("name", "Nicholas");
map.set("age", 25);

console.log(map.size);          // 2

console.log(map.has("name"));   // true
console.log(map.get("name"));   // "Nicholas"

console.log(map.has("age"));    // true
console.log(map.get("age"));    // 25

map.delete("name");
console.log(map.has("name"));   // false
console.log(map.get("name"));   // undefined
console.log(map.size);          // 1

map.clear();
console.log(map.has("name"));   // false
console.log(map.get("name"));   // undefined
console.log(map.has("age"));    // false
console.log(map.get("age"));    // undefined
console.log(map.size);          // 0

```

As with sets, the `size` property always contains the number of key-value pairs in the map. The `Map` instance in this example starts with the `"name"` and `"age"` keys, so `has()` returns `true` when passed either key. After the `"name"` key is removed by the `delete()` method, the `has()` method returns `false` when passed `"name"` and the `size` property indicates one less item. The `clear()` method then removes the remaining key, as indicated by `has()` returning `false` for both keys and `size` being 0.

The `clear()` method is a fast way to remove a lot of data from a map, but there's also a way to add a lot of data to a map at one time.

### Map Initialization

Also similar to sets, you can initialize a map with data by passing an array to the `Map` constructor. Each item in the array must itself be an array where the first item is the key and the second is that key's corresponding value. The entire map, therefore, is an array of these two-item arrays, for example:

```js
let map = new Map([["name", "Nicholas"], ["age", 25]]);

console.log(map.has("name"));   // true
console.log(map.get("name"));   // "Nicholas"
console.log(map.has("age"));    // true
console.log(map.get("age"));    // 25
console.log(map.size);          // 2
```

The keys `"name"` and `"age"` are added into `map` through initialization in the constructor. While the array of arrays may look a bit strange, it's necessary to accurately represent keys, as keys can be any data type. Storing the keys in an array is the only way to ensure they aren't coerced into another data type before being stored in the map.

### The forEach Method on Maps

The `forEach()` method for maps is similar to `forEach()` for sets and arrays, in that it accepts a callback function that receives three arguments:

1. The value from the next position in the map
1. The key for that value
1. The map from which the value is read

These callback arguments more closely match the `forEach()` behavior in arrays, where the first argument is the value and the second is the key (corresponding to a numeric index in arrays). Here's an example:

```js
let map = new Map([ ["name", "Nicholas"], ["age", 25]]);

map.forEach(function(value, key, ownerMap) {
    console.log(key + " " + value);
    console.log(ownerMap === map);
});
```

The `forEach()` callback function outputs the information that is passed to it. The `value` and `key` are output directly, and `ownerMap` is compared to `map` to show that the values are equivalent. This outputs:

```
name Nicholas
true
age 25
true
```

The callback passed to `forEach()` receives each key-value pair in the order in which the pairs were inserted into the map. This behavior differs slightly from calling `forEach()` on arrays, where the callback receives each item in order of numeric index.

I> You can also provide a second argument to `forEach()` to specify the `this` value inside the callback function. A call like that behaves the same as the set version of the `forEach()` method.

### Weak Maps

Weak maps are to maps what weak sets are to sets: they're a way to store weak object references. In *weak maps*, every key must be an object (an error is thrown if you try to use a non-object key), and those object references are held weakly so they don't interfere with garbage collection. When there are no references to a weak map key outside a weak map, the key-value pair is removed from the weak map.

The most useful place to employ weak maps is when creating an object related to a particular DOM element in a web page. For example, some JavaScript libraries for web pages maintain one custom object for every DOM element referenced in the library, and that mapping is stored in a cache of objects internally.

The difficult part of this approach is determining when a DOM element no longer exists in the web page, so that the library can remove its associated object. Otherwise, the library would hold onto the DOM element reference past the reference's usefulness and cause a memory leak. Tracking the DOM elements with a weak map would still allow the library to associate a custom object with every DOM element, and it could automatically destroy any object in the map when that object's DOM element no longer exists.

I> It's important to note that only weak map keys, and not weak map values, are weak references. An object stored as a weak map value will prevent garbage collection if all other references are removed.

#### Using Weak Maps

The ECMAScript 6 `WeakMap` type is an unordered list of key-value pairs, where a key must be a non-null object and a value can be of any type. The interface for `WeakMap` is very similar to that of `Map` in that `set()` and `get()` are used to add and retrieve data, respectively:

```js
let map = new WeakMap(),
    element = document.querySelector(".element");

map.set(element, "Original");

let value = map.get(element);
console.log(value);             // "Original"

// remove the element
element.parentNode.removeChild(element);
element = null;

// the weak map is empty at this point
```

In this example, one key-value pair is stored. The `element` key is a DOM element used to store a corresponding string value. That value is then retrieved by passing in the DOM element to the `get()` method. When the DOM element is later removed from the document and the variable referencing it is set to `null`, the data is also removed from the weak map.

Similar to weak sets, there is no way to verify that a weak map is empty, because it doesn't have a `size` property. Because there are no remaining references to the key, you can't retrieve the value by calling the `get()` method, either. The weak map has cut off access to the value for that key, and when the garbage collector runs, the memory occupied by the value will be freed.

#### Weak Map Initialization

To initialize a weak map, pass an array of arrays to the `WeakMap` constructor. Just like initializing a regular map, each array inside the containing array should have two items, where the first item is the non-null object key and the second item is the value (any data type). For example:

```js
let key1 = {},
    key2 = {},
    map = new WeakMap([[key1, "Hello"], [key2, 42]]);

console.log(map.has(key1));     // true
console.log(map.get(key1));     // "Hello"
console.log(map.has(key2));     // true
console.log(map.get(key2));     // 42
```

The objects `key1` and `key2` are used as keys in the weak map, and the `get()` and `has()` methods can access them. An error is thrown if the `WeakMap` constructor receives a non-object key in any of the key-value pairs.

#### Weak Map Methods

Weak maps have only two additional methods available to interact with key-value pairs. There is a `has()` method to determine if a given key exists in the map and a `delete()` method to remove a specific key-value pair. There is no `clear()` method because that would require enumerating keys, and like weak sets, that isn't possible with weak maps. This example uses both the `has()` and `delete()` methods:

```js
let map = new WeakMap(),
    element = document.querySelector(".element");

map.set(element, "Original");

console.log(map.has(element));   // true
console.log(map.get(element));   // "Original"

map.delete(element);
console.log(map.has(element));   // false
console.log(map.get(element));   // undefined
```

Here, a DOM element is once again used as the key in a weak map. The `has()` method is useful for checking to see if a reference is currently being used as a key in the weak map. Keep in mind that this only works when you have a non-null reference to a key. The key is forcibly removed from the weak map by the `delete()` method, at which point `has()` returns `false` and `get()` returns `undefined`.

#### Private Object Data

While most developers consider the main use case of weak maps to be associated data with DOM elements, there are many other possible uses (and no doubt, some that have yet to be discovered). One practical use of weak maps is to store data that is private to object instances. All object properties are public in ECMAScript 6, and so you need to use some creativity to make data accessible to objects, but not accessible to everything. Consider the following example:

```js
function Person(name) {
    this._name = name;
}

Person.prototype.getName = function() {
    return this._name;
};
```

This code uses the common convention of a leading underscore to indicate that a property is considered private and should not be modified outside the object instance. The intent is to use `getName()` to read `this._name` and not allow the `_name` value to change. However, there is nothing standing in the way of someone writing to the `_name` property, so it can be overwritten either intentionally or accidentally.

In ECMAScript 5, it's possible to get close to having truly private data, by creating an object using a pattern such as this:

```js
var Person = (function() {

    var privateData = {},
        privateId = 0;

    function Person(name) {
        Object.defineProperty(this, "_id", { value: privateId++ });

        privateData[this._id] = {
            name: name
        };
    }

    Person.prototype.getName = function() {
        return privateData[this._id].name;
    };

    return Person;
}());
```

This example wraps the definition of `Person` in an IIFE that contains two private variables, `privateData` and `privateId`. The `privateData` object stores private information for each instance while `privateId` is used to generate a unique ID for each instance. When the `Person` constructor is called, a nonenumerable, nonconfigurable, and nonwritable `_id` property is added.

Then, an entry is made into the `privateData` object that corresponds to the ID for the object instance; that's where the `name` is stored. Later, in the `getName()` function, the name can be retrieved by using `this._id` as the key into `privateData`. Because `privateData` is not accessible outside of the IIFE, the actual data is safe, even though `this._id` is exposed publicly.

The big problem with this approach is that the data in `privateData` never disappears because there is no way to know when an object instance is destroyed; the `privateData` object will always contain extra data. This problem can be solved by using a weak map instead, as follows:

```js
let Person = (function() {

    let privateData = new WeakMap();

    function Person(name) {
        privateData.set(this, { name: name });
    }

    Person.prototype.getName = function() {
        return privateData.get(this).name;
    };

    return Person;
}());
```

This version of the `Person` example uses a weak map for the private data instead of an object. Because the `Person` object instance itself can be used as a key, there's no need to keep track of a separate ID. When the `Person` constructor is called, a new entry is made into the weak map with a key of `this` and a value of an object containing private information. In this case, that value is an object containing only `name`. The `getName()` function retrieves that private information by passing `this` to the `privateData.get()` method, which fetches the value object and accesses the `name` property. This technique keeps the private information private, and destroys that information whenever an object instance associated with it is destroyed.

#### Weak Map Uses and Limitations

When deciding whether to use a weak map or a regular map, the primary decision to consider is whether you want to use only object keys. Anytime you're going to use only object keys, then the best choice is a weak map. That will allow you to optimize memory usage and avoid memory leaks by ensuring that extra data isn't kept around after it's no longer accessible.

Keep in mind that weak maps give you very little visibility into their contents, so you can't use the `forEach()` method, the `size` property, or the `clear()` method to manage the items. If you need some inspection capabilities, then regular maps are a better choice. Just be sure to keep an eye on memory usage.

Of course, if you only want to use non-object keys, then regular maps are your only choice.

## Summary

ECMAScript 6 formally introduces sets and maps into JavaScript. Prior to this, developers frequently used objects to mimic both sets and maps, often running into problems due to the limitations associated with object properties.

Sets are ordered lists of unique values. Values are not coerced to determine equivalence. Sets automatically remove duplicate values, so you can use a set to filter an array for duplicates and return the result. Sets aren't subclasses of arrays, so you cannot randomly access a set's values. Instead, you can use the `has()` method to determine if a value is contained in the set and the `size` property to inspect the number of values in the set. The `Set` type also has a `forEach()` method to process each set value.

Weak sets are special sets that can contain only objects. The objects are stored with weak references, meaning that an item in a weak set will not block garbage collection if that item is the only remaining reference to an object. Weak set contents can't be inspected due to the complexities of memory management, so it's best to use weak sets only for tracking objects that need to be grouped together.

Maps are ordered key-value pairs where the key can be any data type. Similar to sets, keys are not coerced to determine equivalence, which means you can have a numeric key `5` and a string `"5"` as two separate keys. A value of any data type can be associated with a key using the `set()` method, and that value can later be retrieved by using the `get()` method. Maps also have a `size` property and a `forEach()` method to allow for easier item access.

Weak maps are a special type of map that can only have object keys. As with weak sets, an object key reference is weak and doesn't prevent garbage collection when it's the only remaining reference to an object. When a key is garbage collected, the value associated with the key is also removed from the weak map. This memory management aspect makes weak maps uniquely suited for correlating additional information with objects whose lifecycles are managed outside of the code accessing them.
