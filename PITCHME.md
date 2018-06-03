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
