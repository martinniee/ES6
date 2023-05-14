# ES6 视频学习笔记

> 学习计划：一集视频一个提交，如果可能的话。
> 学习周长：7 天内完成（2023 年 5 月 13 日-???），每天 6-7 集。

知识要点

1. 作用域
   - 局部作用域
   - 全局作用域
   - 作用域链
   - JS 垃圾回收机制
   - 闭包
   - 变量提升
2. 函数进阶
   - 函数提升
   - 函数参数
   - 箭头函数
3. 解构赋值
4. 综合案例

## 作用域和作用域链

---

回顾作用域

- 作用域(scope)规定了变量能够被访问的“范围”，离开了这个“范围”变量便不能被访问。
- 作用域基本分为：（1）局部作用域；（2）全局作用域。

### 局部作用域

局部作用域又分为：（1）函数作用域；（2）块作用域（block）

1. 函数作用域

在函数内部声明的变量只能在函数内部被访问，**外部**无法直接访问。

```javascript
<script>
    function getSum(){
            // num 在函数内部，而函数内部是函数作用域，所以 sum是局部变量
            const num = 10;
            return sum;
    }
    //  因此 在外部无法直接访问 sum，会报错
    console.log("sum为",sum); // ReferenceError: sum is not defined
</script>
```

小结：

- **函数内**声明的变量以及**函数的参数**都是函数内部的局部变量，在函数外部无法被访问。
- 不同函数内部的声明的变量互相也无法访问。
- 函数执行完，函数内的变量被清空。

2. 块作用域

在 JavaScript 中使用`{}`包裹的代码称为代码块（如 `for`,`if`,...），代码块内部声明的变量外部将【有可能】无法被访问。

```javascript
for (let i = 0; i < 10; i++) {
	// {} 内部成为 块作用域, i 是局部变量，外部不能访问
	// console.log(i);
}
console.log(i); //  Uncaught ReferenceError: i is not defined
```

小结：

- 使用`let`,`const`关键字声明的变量会产生块作用域，而`var`不会。（推荐使用`const`或`let`）
- 不同代码块之间变量无法直接访问。

### 全局作用域

`<script>`标签和`.js`文件的【最外层】就是所谓的全局作用域，在此声明的变量在函数内部也可以被访问。
全局作用域中声明的变量，任何其它作用域都可以被访问

注意：

- 给 `window` 对象添加的属性默认是全局的，函数中未使用 关键字声明的变量也是全局的，都不建议使用！
- 尽可能减少使用全局变量，防止全局变量被污染。

### 作用域链

作用域链本质上是底层的**变量查找机制**。

- 在函数被执行时，会**优先查找当前**函数作用域中查找变量
- 如果当前作用域查找不到则会**依次逐级查找父级作用域**直到全局作用域

小结：

1. 嵌套关系的作用域串联起来形成了作用域链
2. 相同作用域链中按着从小到大的规则查找变量
3. 子作用域能够访问父作用域，**但父级作用域无法访问子级作用域！**

---

## JS 垃圾回收机制及算法

### 内存生命周期

JS 环境中分配的内存，一般有如下生命周期：

1. 内存分配：当我们声明变量、函数、对象的时候，系统会自动为他们分配内存
2. 内存使用：即读写内存，也就是使用变量、函数等
3. 内存回收：使用完毕，由**垃圾回收器**自动回收不再使用的内存

说明：

- 全局变量一般不会回收（关闭页面回收）
- 一般情况下**局部变量**的值，不用了，会被**自动回收**掉

> 内存泄漏：程序中分配的内存由于某种原因程序**未释放**或**无法释放**叫做**内存泄漏**

### 堆栈空间分配说明

1.  栈（操作系统）：由**操作系统自动分配释放**函数的参数值、局部变量等，基本数据类型放到栈里面。
2.  堆（操作系统）：一般由程序员分配释放，若程序员不释放，由**垃圾回收机制回收**。**复杂数据类型**放到堆里面。

### 垃圾回收算法

#### 引用计数法

IE 采用的引用计数算法，定义“**内存不再使用**”，就是看一个**对象**是否有指向它的引用，没有引用了就回收对象

算法：

1. 跟踪记录被引用的次数
2. 如果被引用了一次，那么就记录次数 1，多次引用会**累加++**
3. 如果减少一个引用就减`1--`
4. 如果引用次数是 0，则释放内存

引用计数法存在问题：循环引用可能导致内存泄露！

```javascript
function fun() {
	let o1 = {};
	let o2 = {};
	// 下面的 o1,o2循环引用，可能导致内存泄露！
	o1.a = o2;
	o2.a = o1;
	return "引用计数法无法回收";
}
fun(); // 调用
```

#### 标记清除法

**现代的浏览器已经不再使用引用计数算法了**。

**现代浏览器通用的大多是基于标记清除算法的某些改进算法**，总体思想都是一致的。

核心：

1. 标记清除算法将“**不再使用的对象**”定义为“**无法达到的对象**”。
2. 就是从**根部**(在 S 中就是全局对象)出发定时扫描内存中的对象。凡是能从根部到达的对象，都是还需要使用的。
3. 那些无法由根部出发触及到的**对象被标记为不再使用**，稍后进行回收。

> 引用计数法存在的问题以及标记清除法解决的问题；
>
> 1. 函数内部的局部变量应当在函数被调用结束后被销毁（回收），但假如在函数中存在循环引用，使用引用计数法则不会被回收，如此一来可能会造成内存泄露。
> 2. 使用标记清除法可以解决函数内部变量循环引用导致无法垃圾回收造成的内存泄露。目的是函数内部（局部变量）垃圾回收问题。

---

## 闭包

闭包（closure）：一个函数对周围状态的引用捆绑在一起，内层函数中访问到其外层函数的作用域

> 简单理解：闭包=内层函数+外层函数的变量

闭包的使用

- 闭包的基本结构
- 常用闭包的结构
- 闭包的应用

1. 闭包的基本结构

```javascript
function outer(params) {
	const a = 1;
	function inner(params) {
		console.log(a);
	}
	inner();
}
outer();
```

2. 常用闭包的结构

在实际使用中，闭包并不是像前面一样，在外部函数（outer）内部调用内部函数（inner）来访问外部函数的中的变量。而是希望在全局作用域（函数外部）访问。因此常用的闭包结构如下：

```javascript
function outer() {
	const a = 1;
	function inner() {
		console.log(a);
	}
	// 返回外部函数中内部函数 inner
	return inner;
}
const a = outer();
a(); // 外部调用 inner(),访问函数内变量
```

3. 闭包的应用
   闭包应用：**实现数据的私有**
   比如，我们要做个统计函数调用次数，函数调用一次，就++

```javascript title="闭包的应用"
// -----------------1.不使用闭包-----------------
let i = 0; // 全局作用域的变量容易被修改
function counter() {
	i++;
}
// -----------------2.使用闭包-----------------
/**
     注意：此时 i 虽然是 函数内的局部变量，但是但函数执行完后 i 并不会被回收。
    因为 counter返回 fun的引用，f 指向了 fun，fun中的 i 可以被“找到”，不会被回收
    但是这样会导致 i 一直不回收，可能会导致 内存泄露
*/
function counter() {
	let i = 0;
	function fun() {
		i++;
		console.log(`函数被调用了${i}次`);
	}
	return fun;
}
const f = counter();
```

---

## 变量提升和函数提升

### 变量提升

变量提升是 avaScript 中比较“奇怪”的现象，它允许在变量声明之前即被访问(仅存在于 var 声明变量)

- 变量在未声明即被访问时会报语法错误。
- 在使用`var`,`let`,`const`中，只有使用`var`声明的变量存在变量提升。变量在`var`声明之前即被访问，变量的值为`undefined`（相当于只提升变量的声明，不提升赋值）。
- 变量提升出现在相同作用域中。
- 实际开发中推荐先声明，后访问。

```javascript
// 1. 变量提升：把所有var 声明的变量的声明提到 【当前作用域】的最前面
console.log(num, " 件"); // undefined 件
var num = 10;
// 上述代码相当于
/**
        var num;
        console.log(num, "件"); // 因此num声明但位赋值，默认值为 undefined
        num = 10;
        */
function fun() {
	console.log(num, " 件"); // undefined 件
	var num = 10;
}
fun();
```

### 函数提升

两种情况

- 普通函数的函数提升
- 函数表达式的函数变量的提升

普通函数的函数提升

```javascript
// 1. 函数提升：只提升函数声明，不提升调用
fun(); // 在函数声明前调用函数
function fun() {
    console.log("函数被调用了");
}
/**
上述的代码相当于

function fun() {
     console.log("函数被调用了");
}

fun(); // 在函数声明前调用函数
*/
```

函数表达式的函数变量的提升

```javascript
// 函数表达式
fun2();
var fun2 = function () {
	console.log("函数表达式"); //Uncaught TypeError: fun2 is not a function
}
/*
上述的代码由于是 函数表达式，存在 变量的提升，
因此相当于在 fun2赋值之前调用fun2(),此时fun2 为 undifiend，因此 fun2 不是函数，会报错
*/
```

