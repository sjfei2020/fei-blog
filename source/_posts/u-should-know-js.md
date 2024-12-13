---
title: 你应该知道的JS
date: 2023-12-25
updated: 2024-05-22
---

### 使用const 或者 let 替代 var

使用const 声明你不打算改变的值，比如配置，比如对象的引用。

使用let 声明打算改变的值

### 缩小变量的作用域

不用在开头列出所有的变量，现在推荐使用let 或者 const 就近声明，提高代码的可维护性

### 用块状作用域代替匿名函数

````js
for(var n = 0; n < 10; n++){
    (function(value){
        setTimeout(
        	function(){
                console.log(value);
            },10
        );
    })(n)
}
````

使用块状作用率代替匿名函数

```js
for(let n = 0; n < 10; n++){
	setTimeout(function(){
        console.log(n);
    })
}
```

### 使用箭头函数代替各种访问this值的变通方式

为了回调中访问上下文的this，使用了这些 

旧习惯

- 使用变量，var self = this
- 使用 bind
- 在支持的函数中，使用 thisArg 形参

新习惯

使用箭头函数

在不涉及this,或者大多数情况下，箭头函数都可以代替普通函数，更简洁

### 使用参数默认值，而不是代码实现

旧习惯

```javascript
function do(参数){
	if(参数 === undefined){	
		参数 = 默认值
	}
    // XXX
}
```

新的习惯

```javascript
function do(参数 = 默认值){
	// XXX
}
```

### 使用 rest 参数替代 arguments 关键字

旧习惯

```javascript
function func1(a, b, c) {
  console.log(arguments[0]);
  // expected output: 1

  console.log(arguments[1]);
  // expected output: 2

  console.log(arguments[2]);
  // expected output: 3
}

func1(1, 2, 3);
```

新习惯

```javascript
function func1(...rest) {
  console.log(rest[0]);
  // expected output: 1

  console.log(rest[1]);
  // expected output: 2

  console.log(rest[2]);
  // expected output: 3
}

func1(1, 2, 3);
```

### 考虑尾后逗号

旧习惯

```js
const temp = {
	a:1,
	b:2
}
```

新习惯

```js
const temp = {
	a:1,
	b:2,
}
```

好处在哪尼，问题在修改的时候可以少敲一个逗号

### 使用类创建构造函数

旧习惯

```js
// 使用传统的函数语法
var Bird = function Bird(name){
   	this.name = name;
}
Bird.prototype.talk = function talk(string){
    return this.name + 'talk:' + string;
}

var bird = new Bird("小红");

bird.talk("Hi!");
// '小红talk:Hi!
```

新习惯

```javascript
class Bird{
	constructor(name){
        this.name = name;
    }
    talk(string){
        return this.name + 'talk:' + string;
    }
}
var bird = new Bird("小红");

bird.talk("Hi!");
// '小红talk:Hi!
```

### 创建对象时使用可计算属性名

旧习惯

- 之前，如果属性名是个变量或者需要动态计算，则只能通过 对象.[变量名] 的方式去访问,且只能运行时才能确定

```js
var name = 'answer';   
var obj = { };
obj[name]=42;
console.log(obj[name]) // 42
```

新习惯

```javascript
var name = 'answer';   
var obj = { 
[name]:42
};
console.log(obj[name]) // 42
```

### 同名变量初始化对象时，使用简写语法

旧习惯：哪怕变量是局部变量，也要写完整key与value

```javascript
function getParams(){
	let name;
	// 其他操作
	return {name:name}
}
```

新习惯：使用简写

```javascript
function getParams(){
	let name;
	// 其他操作
	return {name}
}
```

### 使用 *Object.assign()*代替自定义的扩展方法或者显式复制所有属性

旧习惯 使用遍历key然后复制给新的对象

```javascript
function mergeObjects() {
    var resObj = {};
    for(var i=0; i < arguments.length; i += 1) {
         var obj = arguments[i],
             keys = Object.keys(obj);
         for(var j=0; j < keys.length; j += 1) {
             resObj[keys[j]] = obj[keys[j]];
         }
    }
    return resObj;
}
```

新习惯 使用Object.assign代替

```javascript
const target = { a: 1, b: 2 };

const returnedTarget = Object.assign({}, target);

console.log(returnedTarget); // {a:1,b:2}
```

### 使用属性扩展语法

旧习惯 基于已有对象创建新对象时，使用 Object.assign

```javascript
const target = { a: 1, b: 2 };
const source = { b: 3, c: 4 };

const returnedTarget = Object.assign({},target, source);

console.log(returnedTarget); // {a:1,b:3,c:4}
```

新习惯 使用属性扩展语法

```javascript
const a = { a: 1, b: 2 };
const b = { b: 3, c: 4 };

const returnedTarget = {...a,...b}; // {a: 1, b: 3, c: 4}
```

### 使用 Symbol 避免属性名冲突

旧习惯 使用可读性很差的又很长的字符串来用作属性名来避免冲突

```javascript
const shapeType = {
  triangle: 'Triangle'
};

function getArea(shape, options) {
  let area = 0;
  switch (shape) {
    case shapeType.triangle:
      area = .5 * options.width * options.height;
      break;
  }
  return area;
}

getArea(shapeType.triangle, { width: 100, height: 100 });
```

新习惯

```javascript
const shapeType = {
  triangle: Symbol()
};

function getArea(shape, options) {
  let area = 0;
  switch (shape) {
    case shapeType.triangle:
      area = .5 * options.width * options.height;
      break;
  }
  return area;
}

getArea(shapeType.triangle, { width: 100, height: 100 });
```

### 使用    *Object.getPrototypeOf/setPrototypeOf*  来代替 *__Proto__*

旧习惯 使用非标准的 __Proto__ 来获取或者设置原型

新习惯 使用标准方法 Object.getPrototypeOf/setPrototypeOf 来获取或者设置原型

具体查看 https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/proto

### 使用对象方法的简写语法来定义对象中的方法

旧习惯 key 与 value 的形式

```javascript
const obj = {
    fun : function(){
        // ...
    }
}
```

新习惯

```javascript
const obj = {
    fun(){
        // ...
    }
}
```

### 遍历的姿势-可迭代对象

旧习惯 使用for循环或者 forEach 方法 来循环

```javascript
// for
for(let i = 0; i < array.length; ++i){
    console.log(array[i]);
}
// forEach
array.forEach(entry => console.log(entry));
```

新习惯 当不需要下标的时候，考虑for-of

```javascript
let arr = [1, 2, 3, 4]
for(let item of arr) {
  console.log(item) // 1, 2, 3, 4 
}

需要下标可以考虑
let arr = [1, 2, 3, 4]
for(let [index, value] of arr.entries()) {
  arr[index] = value * 10
}
console.log(arr) // [ 10, 20, 30, 40 ]

```

### 遍历的姿势-DOM

旧习惯 为了遍历DOM集合，将DOM集合转为数组或者使用*Array.prototype.forEach.call* 方法

```javascript
Array.prototype.slice.call(document.querySelectorAll("div")).forEach(
    (div)=>//....
)
    
Array.prototype.forEach.call(document.querySelectorAll("div"),div=>{
    // ...
})
```

新习惯　确保DOM集合是可迭代的，然后使用for-of

```javascript
for(const div of document.querySelectorAll("div"){
    // ...
}
```

### 在使用  Function.prototype.apply() 的大部分场景就可以使用可迭代对象的展开语法

旧习惯

```javascript
const numbers = [5, 6, 2, 3, 7];

const max = Math.max.apply(null, numbers);

console.log(max);
// expected output: 7

const min = Math.min.apply(null, numbers);

console.log(min);
// expected output: 2
```

新习惯　使用可迭代对象的展开语法

```javascript
const numbers = [5, 6, 2, 3, 7];

const max = Math.max([...numbers]);

console.log(max);
// expected output: 7

const min = Math.min([...numbers]);

console.log(min);
// expected output: 2
```



### 该用解构用解构

旧习惯　使用对象中的少量属性就把整个对象保存

```javascript
const bird = getBird();
console.log(bird.age);
console.log(bird.name);
```

新习惯　使用解构赋值，不需要整个对象的时候

```javascript
let {age,name} = getBird();
console.log(age);
console.log(name);
```


### 解构重命名

旧习惯　使用解构赋值，key 不一致

```javascript
let {age,name} = getBird();
let birdAge = age;
let birdName = name;

console.log(birdAge); // 输出年龄，
console.log(birdName); // 输出名字
```

// 新习惯：使用解构赋值并重命名变量

```javascript
const { age: birdAge, name: birdName } = bird;
console.log(birdAge); // 输出年龄，变量名改为 birdAge
console.log(birdName); // 输出名字，变量名改为 birdName
```



### 对可选项使用解构赋值

旧习惯　使用代码处理默认值

```javascript
function check(options){
    let options = Object.assign({},options,{
        a:1,
        b:2,
    })
    if(options.c === undefined){
        options.c = options.a + options.b
    }
    console.log(options.c)
    // ...
}
```

新习惯　将默认值提出来，通过解构来直接处理

```javascript
function check(
{	
    a=1,
    b=2,
    c=a+b
} = {}){
    console.log(c)
    // ...
}

check({a:2}) // 4
```

### 用 async / await 代替 Promise

旧习惯 使用 Promise

```javascript
function fetchJSON(url){
    return fetch(url).then(
    	(response)=>return response.json();
    )
}
```

新习惯 使用 async/await 

```javascript
async function fetchJSON(url){
    const response = await fetch(url);
    return response.json();
}
```

### 使用模板字面量代替字符串连接

旧习惯

```javascript
const getInfo = bird => bird.name + ' ' + bird.age;
```

新习惯

```javascript
const getInfo = bird => `${bird.name} ${bird.age}`;
```

### 使用字符串迭代

旧习惯 使用下标访问字符串中的字符

```javascript
const str = "abcde";
for (let i = 0; i < str.length; ++i){
    // str[i]
}
```

新习惯  视为可迭代对象

```javascript
const str = "abcde";
for (const char of str){
    // char
}
```

### 使用 fing 和 findIndex 方式来代替循环找数组中某个符合条件的

旧习惯 使用for循环 forEach filter取第一项等 

```javascript
let target;
let array = [{id:1},{id:2}];
for (let i = 0; i < array.length; ++i){
    if (array[i].id === 2){
        target = array[i];
        break;
    }
}
```

新习惯 使用 fing 和 findIndex

```javascript
let array = [{id:1},{id:2}];
let target = array.find(i => return i.id === 2);
```

### 使用 Array.fill 代替循环填充数组

旧习惯 使用for循环填充数组

```javascript
// 一维度数组
const len = 5;
const array = new Array(len);
for (let i = 0; i < len; ++i){
    array[i] = 0;
}

// M*N数组
const M = 5;
const N = 5;
const array = new Array(M);
for (let i = 0; i < M; ++i){
    array[i] = new Array(N);
    for (let j = 0; i < N; ++j){
    	array[i][j] = 0;
	}
}
```

新习惯 使用fill

```javascript
// 一维度数组
const len = 5;
const array = new Array(len).fill(0);

// M*N数组
const M = 5;
const N = 5;
// const array = new Array(M).fill(new Array(N).fill(0));
const array = new Array(M).fill(0).map(()=>new Array(N).fill(0)); 
```

### 读取文件时使用 readAsArrayBuffer  取代 readAsBinaryString

旧习惯 使用readAsBinaryString 来处理文件

新习惯 使用readAsArrayBuffer 来处理文件

### 使用Map代替对象

旧习惯 使用对象存储

```javascript
const map = {};
const id = "1";
const obj = {};
map[id] = obj;
```

新习惯 使用Map

```javascript
const store = new Map();
const id = {};
const obj = {};
store.set(id,obj);

// 使用
let target = stroe.get(id);
```

### 使用Set 代替对象做集合

旧习惯 使用对象的key来标记是否存在

```javascript
let birds = {};
birds["xiaohong"] = true;
if(birds["xiaohong"]){
    // 存在
}
```

新习惯 使用Set

```javascript
const birds = new Set();
birds.add("xiaohong");
if(birds.has("xiaohong")){
    // 存在
}
```

### 使用模块代替伪命名空间

旧习惯 导出一整个对象，jquery时代的操作

```javascript
var utils = (function(utils){
    function toString(){
        
    }
    // ...
})(utils || {})
```

新习惯 使用ESM的导入导出操作

```javascript
function toString(){
        
}
const utils = {
    toString:toString
}

export default utils;
export { toString };
```

参考资料：https://es6.ruanyifeng.com/#docs/module-loader

### 将CJS、AMD和其他模块的转为 ESM

旧习惯 打包成CJS AMD 等其他格式，因为没有ESM

新习惯 使用ESM格式，至少新的要支持ESM，拥抱未来

### 使用通用的打包器而不是自研

旧习惯 自研一套打包器

新习惯 拥抱社区，使用通用的有社区支持的打包器

### 使用代理，而不是提供API来get,set 属性

旧习惯 直接提供整个对象，约定不修改

新习惯 改用代理

### 使用代理将部分判断与实现分开

旧习惯 将判断与功能实现写在一起

新习惯 可以考虑将功能函数写对象上，判断或者检测在代理上实现

### 使用二进制字面量

旧习惯 在一些可能更适合二进制的地方，使用十六进制数字

```javascript
const flags = {
    a: 0x01,
    b: 0x02,
    c: 0x04,
    d: 0x08,
}
```

新习惯 在合理的地方采用二进制数字

```javascript
const flags = {
    a: 0b00000001,
    b: 0b00000010,
    c: 0b00000100,
    d: 0b00001000,
}
```

这个多提一句，其实应用挺多的，比如Vue3 里面的 **PatchFlags** 来标记动静节点就是典型的，比如react的车道标记也是

### 使用新的Math函数，而不是各种自己实现的变通方式

旧习惯 使用代码自己实现

新习惯 使用新提供的Math 函数

```javascript
//大部分用不到，这里写几个刷题可能会用到的
** =  Math.pow 
Math.sign(x); // 返回x的符号 【NaN,0,+1,-1】
Math.trunc(x);  // 直接返回整数部分
```

### 使用空值合并

旧习惯  代码判断并赋值

```javascript
const data = res.data === null || res.data === undefined ? 200 : res.data;
```

新习惯 使用空值合并 ??

```javascript
const data = res.data ?? 200;
```

### 使用可选链

旧习惯 先判断是否存在再调用

```javascript
// 属性
let xiaohong = bird && bird.one && bird.one.name;
// 函数
if(xiaohong.talk){
    xiaohong.talk();
}
```

新习惯 使用可选链

注意如果是 Number 或者 布尔数值 仍然会报错

```javascript
// 属性
let xiaohong = bird?.one?.name;
// 函数
xiaohong.talk?.();
```