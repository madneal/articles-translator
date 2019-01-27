# javascript中的对象字面量为啥这么酷
>原文：[Why object literals in JavaScript are cool](https://rainsoft.io/why-object-literals-in-javascript-are-cool/)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)


在[ECMAScript 2015](https://rainsoft.io/why-object-literals-in-javascript-are-cool/www.ecma-international.org/ecma-262/6.0/)之前，Javascript中的对象字面量（也称为对象初始化器）是非常基础的。能够定义两种类型的属性：

* 成对出现的名称以及相应的值```{ name1: value1 }```


* [Getters](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/get) `{ get name(){..} }` 以及[setters](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/set) `{ set name(val){..} }` 可以用于动态的属性值。

遗憾的是，这个对象字面量可能会出现下面这样的情况：

```
var myObject = {  
var myObject = {  
  myString: 'value 1',
  get myNumber() {
    return this._myNumber;
  },
  set myNumber(value) {
    this._myNumber = Number(value);
  }
};
myObject.myString; // => 'value 1'  
myObject.myNumber = '15';  
myObject.myNumber; // => 15  
```

Javascript一个基于[原型的语言](https://en.wikipedia.org/wiki/Prototype-based_programming)，所以其中所有的皆是对象。所以必须在创建对象，配置以及访问原型的时候必须提供一个便利的构建方式。

通常都会涉及到对象的定义和对象原型的设置。我经常觉得对于原型的设置应该允许直接在对象字面量进行，用一条语句即可。

不幸的是，对象字面量的限制不允许通过使用一个直接的方法来达到这个目的。你必须通过结合使用`Object.create()`以及对象字面量来设置原型：

```
var myProto = {  
  propertyExists: function(name) {
    return name in this;    
  }
};
var myNumbers = Object.create(myProto);  
myNumbers['array'] = [1, 6, 7];  
myNumbers.propertyExists('array');      // => true  
myNumbers.propertyExists('collection'); // => false  
```

我觉得这是一个让人很不爽的解决方案。Javascript既然是一个基于原型的语言，为什么还要花这么大力气从一个原型中创建对象。

幸运的是，这个语言每天都在变化。Javascript很多令人沮丧的地方也都在一步步地被解决。

这篇文章解释了ES2015是如何解决上述问题并通过以下额外的好处来提高对象字面量：

* 在对象构造函数中设置原型
* 简单函数声明
* 利用`super`来调用
* 动态的属性名称

我们也可展望下未来可以下心的提议在（[第二部分](https://github.com/sebmarkbage/ecmascript-rest-spread#status-of-this-proposal)）：通过使用对象中的rest以及spread属性

![Infographic](http://ac-Myg6wSTV.clouddn.com/825d7c6a95690b5818eb.jpg)

### 1.在对象构造函数中设置原型

你已经知道可以通过使用这个getter 属性`__proto__`来访问一个对象的原型：

```
var myObject = {  
  name: 'Hello World!'
};
myObject.__proto__;                         // => {}  
myObject.__proto__.isPrototypeOf(myObject); // => true  
```

`myObject.__proto__`返回`myObject`的原型对象。

好消息是[ES2015允许使用](http://www.ecma-international.org/ecma-262/6.0/#sec-__proto__-property-names-in-object-initializers)对象字面量`__proto__`作为属性名在对象字面量`{ __proto__: protoObject }`中来设置原型。

让我们来使用`__proto__`来初始化一个对象从而改善上述我们提到的糟糕的情形：

```
var myProto = {  
  propertyExists: function(name) {
    return name in this;    
  }
};
var myNumbers = {  
  __proto__: myProto,
  array: [1, 6, 7]
};
myNumbers.propertyExists('array');      // => true  
myNumbers.propertyExists('collection'); // => false  
```



As you know already, one option to access the prototype of an existing object is using the getter property `__proto__`:


    var myObject = {  
      name: 'Hello World!'
    };
    myObject.__proto__;                         // => {}  
    myObject.__proto__.isPrototypeOf(myObject); // => true  


`myNumbers`通过一个特别的属性名称`__proto__`来使用`myProto`的原型。这个对象通过一句话就可以声明，不需要额外的函数比如`Object.create()`。

很显然，使用`__proto__`十分简单。我往往更喜欢这种简单并且有效的解决方案。

有点偏离主题了。我觉得奇怪的是一个简单并且灵活的解决方案往往需要大量的工作和设计。如果一个解决方案是简单的，你可能觉得它很容易设计。然而恰恰相反：

* 让它简明直接是十分复杂的
* 让它复杂并且难以理解是很简单的

如果有的东西看起来很复杂或者用起来很麻烦，那么它可能在设计的时候考虑的不是很充分。所以你对于简便的观点是什么呢？

#### 2.1`__proto__`使用的一些特别案例

即使`__proto__`看起来十分简单，但是还是又一些特殊的情形需要注意：

![Infographic](http://ac-Myg6wSTV.clouddn.com/e46fa45d4cce81bc3be9.jpg)

只允许在对象字面量中使用一次`__proto__`。一旦重复使用就会出现下面的错误：

    var object = {  
      __proto__: {
        toString: function() {
          return '[object Numbers]'
        }
      },
      numbers: [1, 5, 89],
      __proto__: {
        toString: function() {
          return '[object ArrayOfNumbers]'
        }
      }
    };

在这个例子中，对象中使用了2次`__proto__`属性，这个是不被允许的。在这种情形下出现报错`SyntaxError: Duplicate __proto__ fields are not allowed in object literals`。

Javascript只允许对象或者`null`来使用`__proto__`属性。任何尝试通过基本类型（strings,numbers,booleans)或者`undefined`来使用只会被忽略掉并不会改变对象的原型。

让我们在看一下例子吧：

    var objUndefined = {  
      __proto__: undefined
    };
    Object.getPrototypeOf(objUndefined); // => {}  
    var objNumber = {  
      __proto__: 15
    };
    Object.getPrototypeOf(objNumber);    // => {}  

这个对象字面量通过使用`undefined`以及数字15来设置`__proto__`值。因为只有对象或者`null`才允许成为原型，`objUndefined`以及`objNumber`依然有他们自己默认的原型：普通的Javascript对象。`__proto`属性会被忽略。

当然，通过使用基本类型来设置对象的原型也很奇怪，所以这里的限制也是理所当然的了。



### 2. Shorthand method definition

### 简单函数声明

在对象字面量中可以通过使用一个简短的表达式来声明方法，通过这种方式关键字`function`以及符号`:`可以省略。这个就叫做简单函数声明。

让我们使用新的简短的形式来定义函数：

    var collection = {  
      items: [],
      add(item) {
        this.items.push(item);
      },
      get(index) {
        return this.items[index];
      }
    };
    collection.add(15);  
    collection.add(3);  
    collection.get(0); // => 15  

`add()` and `get()` are methods defined in `collection` using a short form.

在`collection`通过一种简短的形式来定义`add()`和`get()` 方法。

这样做的好处之一是声明的方法就是命名的函数，这对于调试来说很有利。通过执行`collection.add.name`就会返回上述例子中的方法`add`的名称。



### 3.通过`super`来调用

一个有趣的提升是通过关键字`super`就可以访问从原型链中继承的属性，如下所示：

    var calc = {  
      sumArray (items) {
        return items.reduce(function(a, b) {
          return a + b;
        });
      }
    };
    var numbers = {  
      __proto__: calc,
      numbers: [4, 6, 7],
      sumElements() {
        return super.sumArray(this.numbers);
      }
    };
    numbers.sumElements(); // => 17  

`calc`就是`numbers`对象的原型。在`numbers`对象中的`sumElements`方法中，可以通过利用关键字`super`:`super.sumArray()`来访问原型中的方法。

`super`是反问对象原型链中继承属性的快捷方式。

#### 3.1 `super`使用限制

`super`只能被用于对象字面量中简单方法的定义。

如果常识通过一个普通的函数声明来访问`{ name: function() {} }`，Javascript就会抛出错误：

    var calc = {  
      sumArray (items) {
        return items.reduce(function(a, b) {
          return a + b;
        });
      }
    };
    var numbers = {  
      __proto__: calc,
      numbers: [4, 6, 7],
      sumElements: function() {
        return super.sumArray(this.numbers);
      }
    };
    // Throws SyntaxError: 'super' keyword unexpected here
    numbers.sumElements();  

方法`sumElements`是以属性的方式来定义：`sumElements: function() {…}`。因为`super`只能在简单函数中使用，在这种情形调用的话就回抛错`SyntaxError: 'super' keyword unexpected here`。

这一限制并太会影响对象字面量声明的方式。通常在对象字面量中更多的会使用简单函数定义。



### 4.动态的属性名称

在ES2015之间，对象初始化器中的属性名称只是字面上的，大多数都是静态字符串。为了创建一个具有动态名称的属性，你必须使用属性访问器：

    function prefix(prefStr, name) {  
       return prefStr + '_' + name;
    }
    var object = {};  
    object[prefix('number', 'pi')] = 3.14;  
    object[prefix('bool', 'false')] = false;  
    object; // => { number_pi: 3.14, bool_false: false }  

很显然，这种定义属性的方式远远不能让人满意。动态属性命名以优雅的方式解决了这个问题。

当通过一个表达式来获得属性的名称的时候，把代码放在一个中括号里面`{[expression]:value}`。这个表达式最终的结果就会成为这个属性的名称。

我很喜欢这种方式：简短。

我们再升级一下：

    function prefix(prefStr, name) {  
       return prefStr + '_' + name;
    }
    var object = {  
      [prefix('number', 'pi')]: 3.14,
      [prefix('bool', 'false')]: false
    };
    object; // => { number_pi: 3.14, bool_false: false }  

`[prefix('number', 'pi')]`通过计算表达式`prefix('number', 'pi')`的值来获得属性的名称。

相应的 `[prefix('bool', 'false')]`将第二个属性名称命名为`'bool_false'`。

#### 4.1 将`Symbol`作为属性名称

[Symbols](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol)也能够被用于动态属性名称。只要确保在中括号中包含它们：`{ [Symbol('name')]: 'Prop value' }`。

比如，让我们使用一个特别的属性`Symbol.iterator` 来遍历对象中的属性。示例如下：

    var object = {  
       number1: 14,
       number2: 15,
       string1: 'hello',
       string2: 'world',
       [Symbol.iterator]: function *() {
         var own = Object.getOwnPropertyNames(this),
           prop;
         while(prop = own.pop()) {
           yield prop;
         }
       }
    }
    [...object]; // => ['number1', 'number2', 'string1', 'string2']

`[Symbol.iterator]: function *() { }` 定义了一个属性来用于遍历对象中的属性。这个spread操作符`[...object]` 用迭代器访问并返回属性列表。



### 5. 展望未来：rest以及spread属性

对象字面量中[rest以及spread属性](https://github.com/sebmarkbage/ecmascript-rest-spread)是接下来的草案，这将可能作为新版本javascript的新特性。

在ECMAScript 2015中的数组意境存在一个替代物。

[Rest属性](https://github.com/sebmarkbage/ecmascript-rest-spread/blob/master/Rest.md) 允许收集解构赋值后遗留的对象中的属性。

下面这个例子就是在解构赋值`object`之后收集剩余的属性：

    var object = {  
      propA: 1,
      propB: 2,
      propC: 3
    };
    let {propA, ...restObject} = object;  
    propA;      // => 1  
    restObject; // => { propB: 2, propC: 3 }  

[Spread属性](https://github.com/sebmarkbage/ecmascript-rest-spread/blob/master/Spread.md)  允许从源对象中拷贝属性到另一个对象字面量中。在下面的例子中对象字面量中的额外属性来自于源对象。

    var source = {  
      propB: 2,
      propC: 3
    };
    var object = {  
      propA: 1,
      ...source
    }
    object; // => { propA: 1, propB: 2, propC: 3 }  



### 6. 总结

Javascript正在飞速前进。

即使一个相当小的对象字面量的构建在ECMAScript 2015都得到了相当可观的提升。更多的特性在草案议程上。

你可以通过直接设置`__proto__`这个属性名称来直接设置对象的原型。在简单函数声明中可以通过使用关键字`super`来轻松地访问对象原型链中所继承的属性。

如果一个属性的名称是实时获得的，现在你可以通过使用动态属性名称`[expression]`来初始化对象。

诚然，对象字面量真是酷爆了！！！

欢迎评论。

_
