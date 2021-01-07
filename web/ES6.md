



## let



类似var，所声明变量只在let所在的==块级作用域==有效	->	适用于for循环， 声明循环变量的部分为父作用域，循环体内部是子作用域

不允许重复声明

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



##### 块级作用域嵌套



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





#### 函数不能在 





















