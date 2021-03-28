

# 基础类型



boolean	数字	字符串	数组





**元组 Tuple**	允许表示已知元素数量和类型的数组，各元素的类型不必相同 

```typescript
let x: [string, number];
x = [10, 'hello'];	//此时类型不匹配,会报错
```



**枚举enum**	

```typescript
enum Color {Red, Green, Blue}
let c: Color = Color.Green;

//枚举的赋值
enum Color {Red = 1, Green = 2, Blue = 4}
```



**any**		可以在编译时可选择地包含或移除类型检查,但不同于Object类型,`Object`类型的变量只是允许你给它赋任意值 , 但是却不能够在它上面调用任意的方法，即便它真的有这些方法



**void**		表示没有任何类型

一般不用于声明变量,因为只能赋予`undefined`和`null`

```typescript
function warnUser(): void {
    console.log("This is my warning message");
}
```



**Null 和 Undefined**	是所有类型的子类型,一般不用于声明变量



**never**	表示永不存在的值的类型

抛出异常 或 void的函数  或箭头函数表达式的返回值类型； **变量也可能是 `never`类型，当它们被永不为真的类型保护所约束时**

`never`类型是任何类型的子类型，也可以赋值给任何类型；然而，*没有*类型是`never`的子类型或可以赋值给`never`类型（除了`never`本身之外）。 即使 `any`也不可以赋值给`never`

```typescript
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
		//抛出异常			或							永不返回
    throw new Error(message);		// while (true) {}
}
```



**Symbols**	ECMAScript 2015起新的原生类型,值是通过 Symbol构造函数创建

每个symbol常量都是不可变且唯一的

```typescript
let sym2 = Symbol("key");
let sym3 = Symbol("key");

sym2 === sym3; // false, symbols是唯一的
```





**Object**	表示非原始类型











## 类型断言 as



类似于强转,但不进行特殊的数据检查和解构。 没有运行时的影响，**只在编译阶段起作用**

```typescript
let someValue: any = "this is a string";
-->
let strLength: number = (<string>someValue).length;
==		//尖括号与as等价,但在TypeScript里使用JSX时，只能用as
let strLength: number = (someValue as string).length;
```











## 箭头函数



通过=>定义；属于匿名函数，即没有函数名称；

不能用作构造函数/Generator函数

```typescript
//定义fun函数，箭头左侧为箭头函数的参数，右侧是函数；一个参数时，小括号省略,无参数时小括号不省略
let fun = (a,b)=>a+b
```





## 类型兼容性

允许忽略参数

当`x`的每个参数能在`y`里找到对应类型的参数,则允许类型兼容

```typescript
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error
```











# 变量声明



## var



var可以在包含它的函数，模块，命名空间或全局作用域内部任何位置被访问



**作用域不明确**

```javascript
function f(shouldInitialize: boolean) {
    if (shouldInitialize) {
        var x = 10;
    }
    return x;
}

f(true);  // returns '10'
f(false); // returns 'undefined'
```



**能重复定义**

```
var x;
var x;	//指向同一个x
```







## let



类似var，所声明变量只在let所在的==块级作用域==有效	->	适用于for循环， 声明循环变量的部分为父作用域，循环体内部是子作用域

不允许重复声明,变量同名时报错

不允许 “变量提升”：**var变量在声明之前可以被使用**，为 undefined 	而let只能在声明后才能被使用，否则报错



**暂时性死区 (temporal dead zone，简称 TDZ) **：同时声明var和let时，块级作用域优先生效

 一进入当前作用域，所要使用的变量就已存在，但不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量

```javascript
var tmp = 123;

if (true) {
  tmp = 'abc'; // 存在let和var重复声明，let生效，但不支持未声明之前使用，此处赋值报错
  let tmp;
}
```



“暂时性死区”也意味着`typeof`不再是一个百分之百安全的操作

```javascript
typeof x; // 在声明之前都属于 x 的“死区”，此处报错
let x;


let x = x;	//此处也会报错
```



```javascript
function bar(x = y, y = 2) {	//x=y,但y在x赋值前还未声明
  return [x, y];
}

bar(); // 报错
```



### 块级作用域嵌套



```javascript
{{{{
  {let insane = 'Hello World'}
  console.log(insane); // 报错 第4层作用域无法读取第5层变量
}}}};

{{{{
  let insane = 'Hello World';
  {let insane = 'Hello World'}		//内层作用域变量可以与外层同名,但不相互影响
}}}};
```



## const



相当于只读的let



## 解构



**数组解构**

```typescript
let input = [1, 2];
let [first, second] = input;
console.log(first); // outputs 1
console.log(second); // outputs 2

let [first, ...rest] = [1, 2, 3, 4];
console.log(first); // outputs 1
console.log(rest); // outputs [ 2, 3, 4 ]

let [, second, , fourth] = [1, 2, 3, 4];
```



**对象解构**

```typescript
let o = {
    a: "foo",
    b: 12,
    c: "bar"
};
let { a, b } = o;

let { a, ...passthrough } = o;

//重命名	':'在此处不代表指定类型
let { a: newName1, b: newName2 } = o;
```





### 解构用于函数声明



```javascript
type C = { a: string, b?: number }
function f({ a, b }: C): void {
    // ...
}

//解构用于赋初始值
function keepWholeObject(wholeObject: { a: string, b?: number }) {
  	//b赋默认值,此时即使 wholeObject.b 为 undefined ,b 仍会有值
    let { a, b = 1001 } = wholeObject;
}
```





## 展开



展开为创建==浅拷贝==

```javascript
let first = [1, 2];
let second = [3, 4];
let bothPlus = [0, ...first, ...second, 5];		//[0, 1, 2, 3, 4, 5]


//对象的展开存在覆盖值的问题
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { ...defaults, food: "rich" };		//后面的food将覆盖前面

const [first, ...rest] = [];
first // undefined
rest  // []

const [first, ...rest] = ["foo"];
first  // "foo"
rest   // []

//扩展运算符必须放在最后一位
const [...butLast, last] = [1, 2, 3, 4, 5];	// 报错
```





字符串->数组

==必须转成数组才能得到正确的字符串的长度,否则JavaScript 会将四个字节的 Unicode 字符，识别为 2 个字符==

```javascript
[...'hello'] -> [ "h", "e", "l", "l", "o" ]	

let str = 'x\uD83D\uDE80y';
str.split('').reverse().join('')	//错误 'y\uDE80\uD83Dx'
[...str].reverse().join('')	// 'y\uD83D\uDE80x'
```



## 对象的拓展



允许在大括号里面，直接写入变量和函数，作为对象的属性和方法

```javascript
const foo = 'bar';

//等价
const baz = {foo};	// {foo: "bar"}
const baz = {foo: foo};

//等价
function f(x, y) {  return {x, y};	}
function f(x, y) {  return {x: x, y: y};	}
```





简洁写法在打印对象时也很有用。

```javascript
let user = {  name: 'test'	};
let foo = {  bar: 'baz'	};

console.log(user, foo)	//	两组键值对 {name: "test"} {bar: "baz"}
console.log({user, foo})	//	对象的简洁表示,不能调用构造函数 {user: {name: "test"}, foo: {bar: "baz"}}
```







# 接口





```typescript
function printLabel(labelledObj: { label: string }) {
  console.log(labelledObj.label);
}

let myObj = { size: 10, label: "Size 10 Object" };	//尽管有多余的属性,但编译器只会检查必需的属性是否存在，并且其类型是否匹配
printLabel(myObj);
```



**只读**	添加readonly / ReadonlyArray<T> 前缀

```typescript
interface Point {
    readonly x: number;
    readonly y: number;
}
```





## 函数类型



为了使用接口表示函数类型,需要给接口定义调用签名。 它就像是只有参数列表和返回值类型的函数定义。参数列表里的每个参数都需要名字和类型



函数的参数名不需要与接口里定义的名字相匹配,会根据接口定义的参数顺序进行匹配

**函数的返回值类型通过返回值自动推算**





## 索引



2种索引签名: string number		可以同时支持2种类型的索引



number索引的返回值必须是 string的子类型。 这是因为当使用 `number`来索引时，JavaScript会将它转换成`string`然后再去索引对象。 也就是说用 `100`去索引等同于`"100"`去索引，因此两者需要保持一致



```typescript
interface StringArray {
  //表示当用 number去索引StringArray时会得到string的返回值
  readonly [index: number]: string;		//索引签名设置为只读,防止被修改
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```





# 函数



## 函数类型



```typescript
function add(x: number, y: number): number {
    return x + y;
}

let myAdd = function(x: number, y: number): number { return x + y; };
```





**必选/可选/带默认值参数**

```typescript
//a必选	b可选	c带默认值			可选必须在必选后面		默认值在最后面
function test(a:string, b?:string, c:string="jojo"){
  console.log(a);
  console.log(b);
  console.log(c);
}
test("mike")	//调用时无需全部赋值

a:mike
b:undefined
c:jojo
```



# 常用方法





```typescript
setInterval(函数,时间)	//周期性地调用函数
clearInterval( setInterval()时返回的ID )	//取消Interval
```



JSON.parse( Json )







## for..of/in

`for..in`迭代的是对象的 键 ，而`for..of`则迭代对象的值

```ts
let list = [4, 5, 6];
for (let i in list) {}// "0", "1", "2",
for (let i of list) {} // "4", "5", "6"
```













# Promise



表示异步操作的最终结果,使用then()与返回值进行交互



回调函数容易陷入回调地狱,维护较为困难

而Promise是对象,可以保存状态,创建后立即执行,**无法取消**



3种状态:

- `等待态（Pending）`

- `执行态（Fulfilled）`

- `拒绝态（Rejected）`

只有两种状态转变：
pending -> fulfilled    pending -> rejected

**状态只能改变一次**



## resolve



将现有对象转为Promise对象

```js
var jsPromise = Promise.resolve($.ajax('/whatever.json'));//将 jQuery 生成 deferred 对象，转为ES6  Promise 对象
```



如果 resolve 的入参，不是具有 then()的对象（thenable对象），则返回状态为fulfilled的新Promise对象

```js
var p = Promise.resolve('Hello');
p.then(function (s){  console.log(s) }); // Hello
```



上面代码生成一个新的Promise对象的实例p，它的状态为fulfilled，所以回调函数会立即执行，Promise.resolve方法的参数就是回调函数的参数。

如果Promise.resolve方法的参数是一个Promise对象的实例，则会被原封不动地返回。

Promise.reject(reason)方法也会返回一个新的Promise实例，该实例的状态为rejected。Promise.reject方法的参数reason，会被传递给实例的回调函数。

var p = Promise.reject('出错了');  p.then(null, function (s){  console.log(s) }); // 出错了



```typescript
//onFulfilled / onRejected都为可选的函数参数,入参为promise对象,最后返回promise对象
//未捕获异常,等待->执行,调用onFulfilled
//捕获到异常,等待->拒绝,调用onRejected
promise.then(onFulfilled, onRejected);
```



## async函数

返回 Promise 对象，可以用`then()`添加回调函数。当函数执行的时候，一旦遇到`await`就先返回，等异步操作完成，再执行后面的语句



```javascript
async function getStockPriceByName(name) {
  const symbol = await getStockSymbol(name);
  const stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
```



## await 命令



`await`命令后面是一个 Promise 对象，返回该对象的结果



另一种情况是，`await`命令后面是一个`thenable`对象（即定义了`then`方法的对象），那么`await`会将其等同于 Promise 对象。

```javascript
class Sleep {
  constructor(timeout) {
    this.timeout = timeout;
  }
  then(resolve, reject) {
    const startTime = Date.now();
    setTimeout(
      () => resolve(Date.now() - startTime),
      this.timeout
    );
  }
}

(async () => {
  const sleepTime = await new Sleep(1000);
  console.log(sleepTime);
})();
```























function render(callback?:()=>void): string
callback的返回值是函数 返回函数的返回值是void

function render(callback?:void): string
callback的返回值是void

















**展开运算符**

```js
let a = [1,2,3];
let b = [4,5,6];
let c = [...a,...b]; // [1,2,3,4,5,6]
```















