# What is "This"

- **This**是一个在每个函数作用域中自动定义的特殊标识符关键字
- **Ket Points:**
- **This** 不是函数自身的引用
- **This** 也不是函数词法作用域的引用
- **This** 实际上是在函数被调用时建立的一个绑定，它指向 什么 是完全由函数被调用的调用点来决定的

---

## 为什么使用this

```javascript
function identify() {
	return this.name.toUpperCase();
}

function speak() {
	var greeting = "Hello, I'm " + identify.call( this );
	console.log( greeting );
}

var me = {
	name: "Kyle"
};

var you = {
	name: "Reader"
};

identify.call( me ); 
identify.call( you );

speak.call( me ); 
speak.call( you );
```
---
## Optional 
- 你也可以明确地把环境变量传递给identify() & speak()

```javascript
function identify(context) {
	return context.name.toUpperCase();
}

function speak(context) {
	var greeting = "Hello, I'm " + identify( context );
	console.log( greeting );
}

identify( you ); // READER
speak( me ); // Hello, I'm KYLE
```
---

## 对于This的两种误解
- 认为This指向函数自己
- 认为This指向了函数的作用域
---
### 认为This指向他自己

- 想要在函数内部调用自己的原因:
- 递归
- 一个在第一次被调用时会解除自己绑定的事件处理器

---
#### Consider Below Result
```javascript
function foo(num) {
	console.log( "foo: " + num );

	// 追踪 `foo` 被调用了多少次
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// `foo` 被调用了多少次？
console.log( foo.count );
console.log(count);
```
---
#### What happened?
- foo.count 为什么没有变化
- this.count 对谁进行了操作呢？

---
#### 回避this的解决方案
- 创建另一个对象来持有 count 属性 利用词法作用域来理解

```javascript
function foo(num) {
	console.log( "foo: " + num );

	// 追踪 `foo` 被调用了多少次
	data.count++;
}

var data = {
	count: 0
};

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// `foo` 被调用了多少次？
console.log( data.count ); 
```
---
#### 猜可不是一个方法
- 从函数对象内部引用它自己, 仅仅通过一个this关键字是远远不够的
- 通常需要通过一个指向它的词法标识符（变量）得到函数对象的引用

---
#### 思考这两个function
```javascript
function foo() {
	foo.count = 4; // `foo` 引用它自己
}
foo();
console.log(foo.count);// 4
setTimeout( function(){
	// 匿名函数（没有名字）不能引用它自己
}, 10 );
```
---
#### 老派方案
- **arguments.callee** 引用 也 指向当前正在执行的函数的函数对象
- 这个引用通常是匿名函数在自己内部访问函数对象的唯一方法
- arguments.callee 已经被废弃而且不应该再使用
---
#### another solution
- 完全依赖于foo的词法作用域

```javascript
function foo(num) {
	console.log( "foo: " + num );

	// 追踪 `foo` 被调用了多少次
	foo.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// `foo` 被调用了多少次？
console.log( foo.count ); // 4
```
---
#### One more solution
- 强迫 **this** 指向 foo函数对象

```javascript
function foo(num) {
	console.log( "foo: " + num );

	// 追踪 `foo` 被调用了多少次
	// 注意：由于 `foo` 的被调用方式（见下方），`this` 现在确实是 `foo`
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		// 使用 `call(..)`，我们可以保证 `this` 指向函数对象(`foo`)
		foo.call( foo, i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// `foo` 被调用了多少次？
console.log( foo.count ); // 4
```
---
### 认为This指向他的作用域
- **this**不会以任何方式指向函数的词法作用域
- 作用域好像是一个将所有可用标识符作为属性的对象,这从内部来说是对的
- 但是 **JavasScript** 代码不能访问作用域对象,因为它是 引擎 的内部实现
--- 
#### Bad ones
- Bridge never exist

```javascript
function foo() {
	var a = 2;
	this.bar();
}

function bar() {
	console.log( this.a );
}

foo(); //undefined
```
---
#### How it works?
- 试图通过 this.bar() 来引用 bar() 函数
- 调用 bar() 最自然的方式是省略开头的 **this.**,而仅使用标识符进行词法引用
- 试图用 this 在 foo() 和 bar() 的词法作用域间建立一座桥,使得bar() 可以访问 foo()内部作用域的变量 a
- It is impossible, 你不能使用**this**引用在词法作用域中查找东西
- 每当你感觉自己正在试图使用**this**来进行词法作用域的查询时,提醒自己:这里没有桥
---
### Step conclusion
- **this**不是编写时绑定,而是运行时绑定。它依赖于函数调用的上下文条件
- **this**绑定与函数声明的位置没有任何关系，而与函数被调用的方式紧密相连
- 当一个函数被调用时,会建立一个称为执行环境的活动记录,这个记录包含函数是从何处（调用栈 —— call-stack）被调用的
- 函数是 如何 被调用的，被传递了什么参数等信息,这个记录的属性之一，就是在函数执行期间将被使用的**this**引用,函数的 调用点（call-site）用以来判定它的执行如何绑定**this**
---
## Call Site
- 为了理解this,不得不理解调用点:函数在代码中被调用的位置**不是被声明的位置**
- 什么是寻找调用点: 找到一个函数是在哪里被调用的
- 思考调用栈,使我们到达当前执行位置而被调用的所有方法的堆栈, 我们关心的调用点就位于当前执行中的函数**之前**的调用
---
### Call stack & Call site
```javascript
function baz() {
    // 调用栈是: `baz`
    // 我们的调用点是 global scope（全局作用域）

    console.log( "baz" );
    bar(); // <-- `bar` 的调用点
}

function bar() {
    // 调用栈是: `baz` -> `bar`
    // 我们的调用点位于 `baz`

    console.log( "bar" );
    foo(); // <-- `foo` 的 调用点
}

function foo() {
    // 调用栈是: `baz` -> `bar` -> `foo`
    // 我们的调用点位于 `bar`

    console.log( "foo" );
}

baz(); // <-- `baz` 的调用点
```
---
### 小心
- 在分析代码来寻找（从调用栈中）真正的调用点时要小心，因为它是影响**this**绑定的唯一因素

### Rules
- 调用点如何决定在函数执行期间 this 指向哪里
- There are four rules behind.
---
#### Default Binding
- 最常见的状况: 独立函数调用
- 可以认为这种**this**规则是在没有其他规则适用时的默认规则
---
##### Consider this
```javascript
function foo() {
	console.log( this.a );
}

var a = 2;

foo(); //
```
---
##### Points
- "var a = 2"是全局作用域中声明的变量
- "var a = 2"是全局对象的同名属性a的同义词
- 为何适用于default binding? foo() 是被一个直白的，毫无修饰的函数引用调用的
---
##### strict mode
```javascript
function foo() {
	"use strict";

	console.log( this.a );
}

var a = 2;

foo(); // TypeError: `this` is `undefined`
```
---
##### 一个细节
- 即便所有的**this**绑定规则都是完全基于调用点的，但如果 foo()的内容没有在strict mode下执行
- 对于default binding来说全局对象是唯一合法的;foo() 的调用点的 strict mode 状态与此无关


```javascript
function foo() {
	console.log( this.a );
}

var a = 2;

(function(){
	"use strict";

	foo(); // 2
})();
```
---
#### Implicit Binding
- 调用点是否有一个环境对象（context object
- 也称为拥有者（owning）或容器（containing）对象
---
##### example
```javascript
function foo() {
	console.log( this.a );
}

var obj = { // obj 就是foo的环境变量
	a: 2,
	foo: foo
};

obj.foo(); // 2
```
##### explanation
- foo()被声明然后作为引用属性添加到 obj 上的方式
- 无论foo()是否一开始就在obj上被声明,还是后来作为引用添加（如上面代码所示,这个 函数 都不被obj所真正“拥有”或“包含”
- 然而, 调用点使用obj环境来引用函数, 所以你 可以说obj对象在函数被调用的时间点上“拥有”或“包含”这个函数引用
- 在foo()被调用的位置上,它被冠以一个指向obj的对象引用
- 当一个方法引用存在一个环境对象时,是这个对象应当被用于这个函数调用的this绑定
---
##### one more example
- 只有对象属性引用链的最后一层是影响调用点的

```javascript
function foo() {
	console.log( this.a );
}

var obj2 = {
	a: 42,
	foo: foo
};

var obj1 = {
	a: 2,
	obj2: obj2
};

obj1.obj2.foo(); // 42
```
---
#### Implicitly Lost 隐含丢失
- this binding 最沮丧的事情就是当一个**隐含绑定**丢失了它的绑定
- 通常意味着它会退回到**默认绑定**
- 根据strict mode的状态,其结果不是全局对象就是 undefined

##### example
```javascript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var bar = obj.foo; // 函数引用！

var a = "oops, global"; // `a` 也是一个全局对象的属性

bar(); // "oops, global"
```
---
##### explanation
- bar似乎是**obj.foo**的引用
- 但实际上只是另一个**foo**本身的引用而已
- 起作用的调用点是 bar(), 一个直白，毫无修饰的调用，因此**默认绑定**适用于这里

---
#### accident way
- 当考虑传递一个回调函数时

```javascript
function foo() {
	console.log( this.a );
}

function doFoo(fn) {
	// `fn` 只不过 `foo` 的另一个引用

	fn(); // <-- 调用点!
}

var obj = {
	a: 2,
	foo: foo
};

var a = "oops, global"; // `a` 也是一个全局对象的属性

doFoo( obj.foo ); // "oops, global"
```
---
##### explanation
- 参数传递仅仅是一种隐含的赋值
- 而且因为我们在传递一个函数它,是一个隐含的引用赋值,所以最终结果和我们前一个代码段一样
---
##### 如果接收你所传递回调的函数不是你创建的，而是语言内建呢
```javascript
function setTimeout(fn,delay) {
  // （通过某种方法）等待 `delay` 毫秒
	fn(); // <-- 调用点!
}
```
- 回调函数丢掉他们的 this 绑定是十分常见的事情
- 接收我们回调的函数故意改变调用的**this**, 这很糟糕
- 不管哪一种意外改变**this**的方式, 都不能真正地控制你的回调函数引用将如何被执行
- 没有办法控制调用点给你一个固定的绑定
---
### Explicit Binding 明确绑定
- 当使用Implicitly binding, 不得不改变目标对象使它自身包含一个对函数的引用
- 而后使用这个函数引用属性来间接地（隐含地）将 this 绑定到这个对象上
- 但是如果想强制一个函数调用使用某个特定对象作为 this 绑定，而不在这个对象上放置一个函数引用属性？
---
#### solution
- call(..)
- apply(...)
- 接收的第一个参数都是一个用于 this 的对象, 之后使用这个指定的 this 来调用函数

---
#### example
- foo.call(..)使用明确绑定来调用foo,允许我们强制函数的 this 指向 obj

```javascript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```
---
#### tips
- 如果你传递一个简单基本类型值（string，boolean，或 number 类型）作为 this 绑定
- 那么这个基本类型值会被包装在它的对象类型中（分别是 new String(..)，new Boolean(..)，或 new Number(..)）
- 就 this 绑定的角度讲,call(..) 和 apply(..) 是完全一样的
---
#### But
- 单独依靠**Explicit Binding**仍然不能为我们先前提到的问题提供解决方案
- 函数“丢失”自己原本的 this 绑定
- 被第三方框架覆盖
- ...
---
### Hard Binding
- Explicit Binding 的变种

```javascript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

var bar = function() {
	foo.call( obj );
};

bar(); // 2
setTimeout( bar, 100 ); // 2

// `bar` 将 `foo` 的 `this` 硬绑定到 `obj`
// 所以它不可以被覆盖
bar.call( window ); // 2
```
---
#### explanation
- 创建了一个函数 bar()
- 它的内部手动调用 foo.call(obj)
- 强制 this 绑定到 obj 并调用 foo
- 无论怎样调用函数 bar，它总是手动使用 obj 调用 foo
---
#### example-1
- 用**Hard Binding**将一个函数包装起来的最典型的方法,是为所有传入的参数和传出的返回值创建一个通道

```javascript
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = function() {
	return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```
---
#### example-2
- 创建一个可复用的帮助函数

```javascript
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

// 简单的 `bind` 帮助函数
function bind(fn, obj) {
	return function() {
		return fn.apply( obj, arguments );
	};
}

var obj = {
	a: 2
};

var bar = bind( foo, obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```
---
#### In ES5
- **Hard Binding**已作为 ES5 的内建工具提供：Function.prototype.bind
- bind(..)返回一个硬编码的新函数,它使用你指定的**this**环境来调用原本的函数

```javascript
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```
---
#### In ES6
- bind(..) 生成的硬绑定函数有一个名为 .name 的属性
- name源自于原始的 目标函数
- "bar = foo.bind(..)" 有一个name属性,值为 "bound foo"
- 这个值应当会显示在调用栈轨迹的函数调用名称中
---
#### API调用环境
- 许多库中的函数,和许多在 JavaScript 语言以及宿主环境中的内建函数都提供一个可选参数，通常称为环境(context)
- 这种设计作为一种替代方案来确保你的回调函数使用特定的 this 而不必非得使用 bind(..)

```javascript
function foo(el) {
	console.log( el, this.id );
}

var obj = {
	id: "awesome"
};

// 使用 `obj` 作为 `this` 来调用 `foo(..)`
[1, 2, 3].forEach( foo, obj ); // 1 awesome  2 awesome  3 awesome
```
---
### New Binding
- 面向对象语言中的构造器是什么:“构造器”是附着在类上的一种特殊方法当使用 new 操作符来初始化一个类时,这个类的构造器就会被调用
```javascript
something = new MyClass(..);
```
- JavaScript 的“构造器”是什么?
- 在 JS 中，构造器仅仅是一个函数
- 它们偶然地与前置的 new 操作符一起调用。
- 不依附于类 它们也不初始化一个类。
- 不是一种特殊的函数类型 本质上只是一般的函数 在被使用 new 来调用时改变了行为
---
#### More
- 任何函数，包括像 Number(..)这样的内建对象函数都可以在前面加上 new 来被调用
- 这使函数调用成为一个 构造器调用（constructor call）
- 实际上不存在“构造器函数”这样的东西 而只有函数的构造器调用
---
#### when new
- 当在函数前面被加入 new 调用时 也就是构造器调用时
- 一个全新的对象会凭空创建
- 这个新构建的对象会被接入原形链（[[Prototype]]-linked）
- 这个新构建的对象被设置为函数调用的 this 绑定
- 除非函数返回一个它自己的其他 对象 否则这个被 new 调用的函数将自动返回这个新构建的对象
---
#### Consider this
- 通过在前面使用 new 来调用 foo(..) 
- 构建了一个新的对象并把这个新对象作为 foo(..) 调用的 this

```javascript
function foo(a) {
	this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```
---
### Everything has a order
```javascript
function foo() {
	console.log( this.a );
}

var obj1 = {
	a: 2,
	foo: foo
};

var obj2 = {
	a: 3,
	foo: foo
};

obj1.foo(); // 2
obj2.foo(); // 3

obj1.foo.call( obj2 ); // 3
obj2.foo.call( obj1 ); // 2
```
---
#### conclusion-1
-  默认绑定优先级最低
-  明确绑定 的优先权要高于 隐含绑定
-  检查隐含绑定 之前检查 明确绑定 是否适用
-  **New Binding**会如何呢
---
#### example
- new 绑定 的优先级要高于 隐含绑定

```javascript
function foo(something) {
	this.a = something;
}

var obj1 = {
	foo: foo
};

var obj2 = {};

obj1.foo( 2 );//隐式绑定
console.log( obj1.a ); // 2

obj1.foo.call( obj2, 3 );//现式绑定
console.log( obj2.a ); // 3

var bar = new obj1.foo( 4 );//new绑定
console.log( obj1.a ); // 2
console.log( bar.a ); // 4 bar等于是 obj1.foo(4)的绑定
```
---
#### Attention
- new 和 call/apply 不能同时使用

```javascript
function foo(something) {
	this.a = something;
}

var obj1 = {};

var bar = foo.bind( obj1 );//bar 是硬绑定到 obj1 的  new bar(3) 并没有像我们期待的那样将 obj1.a 变为 3
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar( 3 );// 硬绑定（到 obj1）的 bar(..) 调用 可以 被 new 所覆盖
console.log( obj1.a ); // 2
console.log( baz.a ); // 3 new 被实施 我们得到一个名为 baz 的新创建的对象 确实看到 baz.a 的值为 3
```
---
#### 结论
- 硬绑定函数如果是被new调用的 将会导致一个新构建的对象作为它的 this 
- 就用那个新构建的 this 而非先前为 this 指定的 硬绑定
- 为什么 new 可以覆盖 硬绑定 这件事很有用？
- 这种行为的主要原因是，创建一个实质上忽略 this 的 硬绑定 而预先设置一部分或所有的参数的函数 这个函数可以与 new 一起使用来构建对象
---
#### 技巧
- bind(..) 的一个能力是 任何在第一个 this 绑定参数之后被传入的参数 
- 默认地作为当前函数的标准参数 技术上这称为"局部应用(partial application)
- 这种方式是一种 柯里化(currying) 的实现

```javascript
function foo(p1,p2) {
	this.val = p1 + p2;
}

// 在这里使用 `null` 是因为在这种场景下我们不关心 `this` 的硬绑定
// 而且反正它将会被 `new` 调用覆盖掉！
// 但是初始的参数已经被传递进去了 这是一种很棒的技巧
var bar = foo.bind( null, "p1" );

var baz = new bar( "p2" );

baz.val; // p1p2
```
---
### 判定this
- 函数是通过 new 被调用的吗 如果是this就是新构建的对象
```javascript
var bar = new foo()
```
- 函数是通过 call 或 apply 被调用(明确绑定)甚至是隐藏在bind硬绑定之中吗? 如果是,this 就是那个被明确指定的对象
```javascript
var bar = foo.call(obj2)
```
- 函数是通过环境对象(也称为拥有者或容器对象)被调用的吗(隐含绑定)?如果是this就是那个环境对象
```javascript
var bar = obj1.foo()
```
- 使用默认的 this(默认绑定) 如果在 strict mode下就是 undefined 否则是global对象
```javascript
var bar = foo()
```
---
### 凡事总有例外
---
#### 被忽略的this
- 如果你传递 null 或 undefined 作为 call apply 或 bind 的 this 绑定参数 
- 那么这些值会被忽略掉 取而代之的是 默认绑定 规则将适用于这个调用

```javascript
function foo() {
	console.log( this.a );
}

var a = 2;

foo.call( null ); // 2
```
---
##### why
- 为什么会向 this 绑定故意传递像 null 这样的值
- 使用 apply(..) 来将一个数组散开 从而作为函数调用的参数
- bind(..) 可以柯里化参数（预设值）也可能非常有用
- 两种工具都要求第一个参数是 this 绑定 但是目标函数不关心this null就成为了一个合理的占位
---
### 
```javascript
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// 将数组散开作为参数
foo.apply( null, [2, 3] ); // a:2, b:3

// 用 `bind(..)` 进行柯里化
var bar = foo.bind( null, 2 );
bar( 3 ); // a:2, b:3
```
---
#### ES6相关
- ES6 中有一个扩散操作符:...
- 它让你无需使用 apply(..) 而在语法上将一个数组“散开”作为参数
- 比如 foo(...[1,2]) 表示 foo(1,2)
- currying是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数 并且返回接受余下的参数而且返回结果的新函数的技术
- 柯里化在 ES6 中没有语法上的替代品 所以 bind(..) 调用的 this 参数依然需要注意
---
#### Alert！
- 在你不关心 this 绑定而一直使用 null 的时候有些潜在的危机
- 处理一些函数调用-第三方包 而且那些函数确实使用了 this 引用 默认绑定 规则意味着它可能会不经意间引用global 对象(在浏览器中是 window)
---
#### 更安全的this
- 为了 this 而传递一个特殊创建好的对象 这个对象保证不会对你的程序产生副作用
- DMZ对象 —— 只不过是一个完全为空，没有委托的对象
- 为了忽略不用关心的 this 绑定而总是传递一个 DMZ 对象 那么就可以确定任何对 this 的隐藏或意外的使用将会被限制在这个空对象中
- 也就是说这个对象将 global 对象和副作用隔离开来
- 创建 完全为空的对象 的最简单方法就是 Object.create(null)
---
### code
```javascript
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// 我们的 DMZ 空对象
var ø = Object.create( null );

// 将数组散开作为参数
foo.apply( ø, [2, 3] ); // a:2, b:3

// 用 `bind(..)` 进行 currying
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```
---
### 间接
- 当有意或无意地创建对函数的“间接引用（indirect reference）”
- 在那样的情况下，当那个函数引用被调用时，默认绑定 会使用

```javascript
function foo() {
	console.log( this.a );
}

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };

o.foo(); // 3
(p.foo = o.foo)(); // 2
```
---
#### explanation
- 赋值表达式 p.foo = o.foo 的 结果值 是一个刚好指向底层函数对象的引用
- 起作用的调用点就是 foo(), 而不是p.foo()或者o.foo()
- 默认绑定启动
---
### Softening Binding
- 硬绑定 是一种通过将函数强制绑定到特定的 this 上来防止函数调用在不经意间退回到 默认绑定 的策略
- 硬绑定 极大地降低了函数的灵活性阻止我们手动使用 隐含绑定 或后续的 明确绑定 来覆盖 this
- 为 默认绑定 提供不同的默认值（不是 global 或 undefined） 同时保持函数可以通过 隐含绑定 或 明确绑定 技术来手动绑定 this
---
#### code
- 一种逻辑将指定的函数包装起来 这个逻辑在函数调用时检查 this 如果它是 global 或 undefined 就使用预先指定的 默认值

```javascript
if (!Function.prototype.softBind) {
	Function.prototype.softBind = function(obj) {
		var fn = this,
			curried = [].slice.call( arguments, 1 ),
			bound = function bound() {
				return fn.apply(
					(!this ||
						(typeof window !== "undefined" &&
							this === window) ||
						(typeof global !== "undefined" &&
							this === global)
					) ? obj : this,
					curried.concat.apply( curried, arguments )
				);
			};
		bound.prototype = Object.create( fn.prototype );
		return bound;
	};
}

```
---
#### example
```javascript
function foo() {
   console.log("name: " + this.name);
}

var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };

var fooOBJ = foo.softBind( obj );

fooOBJ(); // name: obj

obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2   

fooOBJ.call( obj3 ); // name: obj3  

setTimeout( obj2.foo, 10 ); // name: obj   <---- soft binding
``` 
---
### 词法this ES6
- ES6 引入了一种不适用于这些规则特殊的函数:箭头函数（arrow-function）
- 箭头函数不是通过 function 关键字声明的 而是通过所谓的“大箭头”操作符：=>
- 与使用四种标准的 this 规则不同的是 箭头函数从封闭它的（函数或全局）作用域采用 this 绑定
---
#### code

```javascript
function foo() {
  // 返回一个箭头函数
	return (a) => {
    // 这里的 `this` 是词法上从 `foo()` 采用的
		console.log( this.a );
	};
}

var obj1 = {
	a: 2
};

var obj2 = {
	a: 3
};

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, 不是3!
```
---
### explanation
- 在 foo() 中创建的箭头函数在词法上捕获 foo() 被调用时的 this 不管它是什么
- 因为 foo() 被 this 绑定到 obj1 bar（被返回的箭头函数的一个引用）也将会被 this 绑定到 obj1
---
### usage
```javascript
function foo() {
	setTimeout(() => {
		// 这里的 `this` 是词法上从 `foo()` 采用
		console.log( this.a );
	},100);
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```
---
### same with ES5
```javascript
function foo() {
	var self = this; // 词法上捕获 `this`
	setTimeout( function(){
		console.log( self.a );
	}, 100 );
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```
---
### remember
- 仅使用词法作用域并忘掉虚伪的 this 风格代码
- 完全接受 this 风格机制 包括在必要的时候使用 bind(..) 并尝试避开 self = this 和箭头函数的词法 this技巧
- 开发时二选一
--- 
## This 总结-1
- 执行中的函数判定 this 绑定需要找到这个函数的直接调用点 找到之后 四种规则将会以这种优先顺序施用于调用点
- 通过 new 调用 -> 使用新构建的对象
- 通过 call 或 apply（或 bind）调用 ->使用指定的对象
- 通过持有调用的环境对象调用 -> 使用那个环境对象
---
## This 总结-2
- 默认:strict mode 下是 undefined 否则就是全局对象
- 如果你想“安全”地忽略 this 绑定 一个像 ```ø = Object.create(null)``` 这样的“DMZ”对象是一个很好的占位值 以保护 global 对象不受意外的副作用影响
- ES6 的箭头方法使用词法作用域来决定 this 绑定，这意味着它们采用封闭他们的函数调用作为 this 绑定