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



## 函数参数和展开运算符

函数参数类型

- 动态参数
- 剩余参数

1，动态参数

- 特点：arguments 是伪数组，仅存在函数中，可以获取传给函数的所有实参

```javascript
function getSum() {
    // arguments 伪数组，仅存在函数中，可以获取传给函数的所有实参
    console.log(arguments);
    let sum = 0;
    for (let i = 0; i < arguments.length; i++) {
        sum += arguments[i];
    }
    console.log("sum: ", sum); // sum: 15
}
getSum(1, 2, 3, 4, 5); 
```

2,剩余参数

- 特点：rest 是真数组，可接受所有剩余的参数。剩余参数可以随意命名。

```javascript
function getSum(a, ...rest) {
    // rest 是真数组，可接受所有剩余的参数
    console.log("rest: ", rest);
    let sum = 0;
    for (let index = 0; index < rest.length; index++) {
        sum += rest[index];

    }
    console.log("sum: ", sum); // sum: 15
}
getSum(1, 2, 3); // 剩余此参数是 [2,3]
getSum(1, 2, 3, 4, 5); // 剩余此参数是 [2,3,4,5]
```

3，扩展：展开运算符

特点：不会修改原有数组。

展开运算符经常用于 求数组元素 **最大值/最小值** 或 **合并数组** 的需求中，在形参数中使用展开运算符。

```javascript
const arr = [1, 2, 3];
console.log(...arr); // 1 2 3
// 1. 求数组元素最大值,最小值
const maxValue = Math.max(...arr);
const minValue = Math.min(...arr);
console.log("maxValue: ", maxValue); // maxValue:  3
console.log("minValue: ", minValue); // minValue:  1
// 2. 合并数组
const arr2 = [3, 4, 5];
const arr3 = [...arr, ...arr2];
console.log("arr3: ", arr3);  //  [1, 2, 3, 3, 4, 5]
```

---



## ES6箭头函数的使用

目的：引入箭头函数的目的是更简短的函数写法并且不绑定`this`，箭头函数的语法比函数表达式更简洁
使用场景：箭头函数更适用于那些**本来需要匿名函数**的地方

### 箭头函数的基本使用

```javascript
// 普通函数
const fn = function () { console.log("普通函数"); }
// 1. 箭头函数：一般形式
// 箭头函数属于表达式函数，因此不存在 函数提升
const fn2 = (a, b, c) => { console.log("箭头函数"); }
// 3. 箭头函数：形参只有一个，则可省略参数 '()'
const fn3 = a => { console.log("a:", a); }
// 4. 箭头函数：函数体只有一个语句，则可省略 '{}'
const fn4 = b => console.log("b:", b);
// 5. 箭头函数：函数体只有一个语句，且是 return 返回语句，则可以省略 return ,省略 '{}'
const fn5 = c => c++;
// 6. 箭头函数：返回值可以是一个对象
const fn6 = (name) => ({ username: name });
```

### 箭头函数的参数

箭头函数中没有 `arguments `，只有 剩余参数` ...rest`

```javascript
// 利用箭头函数求和
// 箭头函数中没有 arguments ，只有 剩余参数 ...rest
const getSum = (...rest) => {
    let sum = 0;
    for (let index = 0; index < rest.length; index++) {
        sum += rest[index];
    }
    console.log("sum: ", sum); // sum:  87
}
getSum(1, 1, 2, 3, 5, 8, 13, 21, 33)
```

### 箭头函数this

箭头函数**不会创建自己的this**,它只会从**自己的作用域链的上一层**沿用thiS。

在开发中 **使用箭头函数前需要考虑函数中this的值** ，事件回调函数使用箭头函数时，`this`为全局的`window`,因此 `DOM`事件回调函数为了简便，**还是不太推荐使用箭头函数**

```javascript
// -----------------1. 普通函数的 this-----------------
console.log(this); // window
function fn() {
    console.log(this); // window
}
windows.fn();
const obj = {
    name: 'foo',
    sayHi: function () {
        console.log(this); // obj
    }
}
obj.sayHi();
// -----------------2. 箭头函数的 this-----------------
const fn1 = () => {
    // 箭头函数的 this 指向上一层作用域的 this
    console.log(this); // window
}
fn1();
// 对象方法的箭头函数
const obj2 = {
    name: 'foo',
    sayHi: () => {
        // 箭头函数的 this 指向上一层作用域的 this
        console.log(this); // window
    }
}
obj2.sayHi();
// 对象方法的箭头函数
const obj3 = {
    name: 'foo',
    sayHi: function () {
        let i = 10;
        // 当前作用域的 this 是 obj3
        const cont = () => {
            // 箭头函数的 this 指向上一层作用域的 this
            console.log(this); // obj3
        }
    }
}
obj3.sayHi();
```



## 解构赋值

### 数组解构

数组解构是将数组的单元值**快速批量赋值**给一系列变量的**简洁语法**

基本语法：

1. 赋值运算符`=`左侧的`[]`用于批量声明变量，右侧数组的单元值将被赋值给左侧的变量
2. 变量的顺序**对应数组单元值的位置**依次进行赋值操作

1，基本使用

```javascript
// 1. 数组解构：基本使用
const arr = [100, 200, 300];
const [max, min, avg] = arr;
console.log("max: ", max); // max:  100
console.log("min: ", min); // min:  200
console.log("avg: ", avg); // avg:  300
// 2. 数组解构：交换变量的值
let a = 1;
let b = 2;
[a, b] = [b, a];
console.log(`a是${a},b是${b}`); // a是2,b是1
```

2，⚠️ 必须加分号的两种情况

```javascript
// 1. 立即执行函数需要添加 ';' 
(function () { })();

// 2. 使用数组的时候
// const arr = [1, 2, 3];
const str = 'foo'
// const str = 'foo'; // 使用封号
[1, 2, 3].map(function (item) {
	console.log("item:", item); // Uncaught TypeError: Cannot read properties of undefined (reading 'map')
})
/*
    如果上述代码 const str = 'foo' 后 无 ';'，则代码解析会成为
    const str = foo[1,2,3],map...  这样的结构，而没有 foo[1,2,3] 这样的数组，因此没有
    map()方法，就会出错
*/
```

3，数组解构细节

```javascript
// 1. 赋值元素数量 大于 解构元素数量，则未赋值元素值为 undefine
const [a, b, c, d] = [1, 2, 3];
console.log(a, b, c, d); // 1 2 3 undefined

// 2. 赋值元素数量 小于 解构元素数量，则有多余的
// 3. 使用剩余参数接收剩余的值
const [x, y, ...rest] = [4, 5, 6, 7];
console.log("x, y: ", x, y);    // x, y:  4 5
console.log("rest: ", ...rest); // rest:  6 7

// 4. 为了防止因 出现（1）的情况，可以给赋值元素设置 默认值
const [m, n, p = 30, q = 40] = [10, 20];
console.log("m,n: ", m, n); // m,n:  10 20
console.log("p ", p); // p  30
console.log("q ", q); // q  40

// 5. 按需导入（赋值)解构元素
// 比如下面的 l 不是 '3' 而是 '4' ，因为空出了一个元素
const [h, j, , l] = [1, 2, 3, 4];
console.log('h,j: ', h, j); // h,j:  1 2
console.log('l: ', l); // l:  4

// 6. 支持 多维数组 的数组解构
const arr = [1, 2, [3, 4]];
const [k, r, s] = arr;
console.log("k,r: ", k, r); //  k,r:  1 2
console.log("s: ", s); // s:  [3, 4]
const [u, v] = s;
console.log("u,v: ", u, v); // u,v:  3 4
/**
    上述 s 表示 [3,4] 即原数组第三个元素。然后在对 s 解构
    即 [u,v] = s，u为3,v为4
    也可以一步到位
    const [k,r,[s,t]] = arr 
    k为1,r为2,s为3,t为4
*/

```





### 对象解构
