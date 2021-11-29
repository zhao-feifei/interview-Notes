# 1.Promise相关

## 手写promise



```javascript

//简洁版  无法处理setTimeout异步任务
function myPromise(constructor){
  let self=this;
  self.status="pending" //定义状态改变前的初始状态 
  self.value=undefined;//定义状态为resolved的时候的状态 
  self.reason=undefined;//定义状态为rejected的时候的状态 
  function resolve(value){
    //两个==="pending"，保证了了状态的改变是不不可逆的 
    if(self.status==="pending"){
      self.value=value;
      self.status="resolved"; 
    }
  }
  function reject(reason){
     //两个==="pending"，保证了了状态的改变是不不可逆的
     if(self.status==="pending"){
        self.reason=reason;
        self.status="rejected"; 
      }
  }
  //捕获构造异常 
  try{
      constructor(resolve,reject);
  }catch(e){
    reject(e);
    } 
}
myPromise.prototype.then=function(onFullfilled,onRejected){ 
  let self=this;
  switch(self.status){
    case "resolved": onFullfilled(self.value); break;
    case "rejected": onRejected(self.reason); break;
    default: 
  }
}

// 测试
var p=new myPromise(function(resolve,reject){resolve(1)}); 
p.then(function(x){console.log(x)})
//输出1




//另一种写法
function myPromise(fn){
    var self = this;
    this.state = 'penging';
    this.value = null;
    this.resolvedCallback = [];
    this.rejectedCallback = [];
    function resolve(value){
        if(value instanceof myPromise){
            return value.then(resolved, rejected);
        }
        setTimeout(() => {
            if(self.state == 'pending'){
                self.value = value;
                self.state = 'resolved';
                self.resolvedCallback.forEach(callback => {
                    callback(value);
                })
            }
        }, 0)
    }
    function reject(value){
        setTimeout(() => {
            if(self.state == 'pending'){
                self.value = value;
                self.state = 'rejected';
                self.rejectedCallback.forEach(callback => {
                    callback(value);
                })
            }
        }, 0)
    }
    //将两个方法传入函数执行
    try{
        fn(resolve, reject);
    } catch(e){
        rejecte(e);
    };
}
myPromise.prototype.then = function(onResolved, onRejected){
    //首先判断这两个状态是否为函数，因为这两个参数是可选的
    onResolved = 
        typeof onResolved == 'function' 
        ? onResolved 
    	: function(value){
        return value;
    }
    onRejected = 
        typeof onRejected == 'function'
    	? onRejected
    	: function(error){
        throw error;
    }
    //如果是等待状态
    if(this.state == 'pending'){
        this.resolvedCallback.push(onResolved);
        this.rejectedCallback.push(onRejected);
    }
    //如果状态已经凝固 则直接进行对应的状态
    if(this.state == 'resolved'){
        onResolved(this.value);
    }
    if(this.state == 'rejected'){
        onRejected(this.value);
    }
}
```

## 手写promise.all  .race   .or    .finally  .then  

## 

```javascript


### 简洁版


function myPromiseAll(promiseAll) {
    return new Promise((resolve, reject) => {
        if (!Array.isArray(promiseAll)) {
            throw 'promise must be an Array'
        }
        let resArr = [],
            len = promiseAll.length,
            count = 0;
        for (let i = 0; i < len; i++) {
            Promise.resolve(promiseAll[i]).then((value) => {
                resArr[i] = value;
                count++;
                if (count == len) {
                    resolve(resArr);
                }
            }).catch((err) => {
                reject(err);
            })
        }
    })
}

let p1 = Promise.resolve(1);
let p2 = Promise.resolve(2);
let p3 = Promise.resolve(3);
myPromiseAll([p1, p3, p2]).then((res) => {
    console.log(res);

})



// promise.race();
function myPromiseRace(promiseRace) {
    return new Promise((resolve, reject) => {
        if (!Array.isArray(promiseRace)) {
            throw 'promiseRace must be a array'
        }
        for (let i = 0; i < promiseRace.length; i++) {
            Promise.resolve(promiseRace[i]).then((res) => {
                resolve(res)
            }).catch((err) => {
                reject(err)
            })
        }
    })
}

let p1 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject(1)
    }, 50)
})
let p2 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject(22)
    }, 10)
})
let p3 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(3)
    }, 20)
})
myPromiseRace([p2, p3, p1])
    .then(res => console.log(res))
    .catch(err => console.log(err)) // 1


//promise.or
let p1 = Promise.reject(1);
        let p2 = Promise.reject(2);
        let p3 = Promise.reject(3);
        let p4 = Promise.resolve(4);

        Promise.or = function(promises) {
            let len = promises.length;
            let i = 0
            return new Promise((resolve, reject) => {
                help(resolve, reject);
            })

            function help(resolve, reject) {
                promises[i].then((res) => {
                    resolve(res);
                }).catch((err) => {
                    if (i == len - 1) return reject(err);
                    else {
                        i++;
                        help(resolve, reject)
                    }
                })
            }
        }
        Promise.or([p1, p2, p3, p4]).then((res) => {
            console.log(res);
        }, (err) => {
            console.log('cuowule');
        })
```

## Promise 封装异步上传图片



```javascript
function loadImageAsync(url) {
    return new Promise((resolve, reject)=>{
        var image = new Image();
        image.onload = function() {
            resolve(image) 
        };
        image.onerror = function() {
            reject(new Error('Could not load image at' + url));
        };
        image.src = url;
     });
}   
```

![image-20211129180738993](C:/Users/t1/AppData/Roaming/Typora/typora-user-images/image-20211129180738993.png)

​     

## promise 封装ajax

```javascript
// promise 封装实现：
function getJSON(url) {
  // 创建一个 promise 对象
  let promise = new Promise(function(resolve, reject) {
    let xhr = new XMLHttpRequest();
    // 新建一个 http 请求
    xhr.open("GET", url, true);
    // 设置状态的监听函数
    xhr.onreadystatechange = function() {
      if (this.readyState !== 4) return;
      // 当请求成功或失败时，改变 promise 的状态
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
    // 设置错误监听函数
    xhr.onerror = function() {
      reject(new Error(this.statusText));
    };
    // 设置响应的数据类型
    xhr.responseType = "json";
    // 设置请求头信息
    xhr.setRequestHeader("Accept", "application/json");
    // 发送 http 请求
    xhr.send(null);
  });
  return promise;
}

```

​                                        

## 





### 排序相关

```javascript

/* 
冒泡排序的方法
*/
function bubbleSort (array) {
  // 1.获取数组的长度
  var length = array.length;

  // 2.反向循环, 因此次数越来越少
  for (var i = length - 1; i >= 0; i--) {
    // 3.根据i的次数, 比较循环到i位置
    for (var j = 0; j < i; j++) {
      // 4.如果j位置比j+1位置的数据大, 那么就交换
      if (array[j] > array[j + 1]) {
        // 交换
        // const temp = array[j+1]
        // array[j+1] = array[j]
        // array[j] = temp
        [array[j + 1], array[j]] = [array[j], array[j + 1]];
      }
    }
  }

  return arr;
}

/* 
选择排序的方法
*/
function selectSort (array) {
  // 1.获取数组的长度
  var length = array.length

  // 2.外层循环: 从0位置开始取出数据, 直到length-2位置
  for (var i = 0; i < length - 1; i++) {
    // 3.内层循环: 从i+1位置开始, 和后面的内容比较
    var min = i
    for (var j = min + 1; j < length; j++) {
      // 4.如果i位置的数据大于j位置的数据, 记录最小的位置
      if (array[min] > array[j]) {
        min = j
      }
    }
    if (min !== i) {
      // 交换
      [array[min], array[i]] = [array[i], array[min]];
    }
  }

  return arr;
}

/* 
插入排序的方法
*/
function insertSort (array) {
  // 1.获取数组的长度
  var length = array.length

  // 2.外层循环: 外层循环是从1位置开始, 依次遍历到最后
  for (var i = 1; i < length; i++) {
    // 3.记录选出的元素, 放在变量temp中
    var j = i
    var temp = array[i]

    // 4.内层循环: 内层循环不确定循环的次数, 最好使用while循环
    while (j > 0 && array[j - 1] > temp) {
      array[j] = array[j - 1]
      j--
    }

    // 5.将选出的j位置, 放入temp元素
    array[j] = temp
  }

  return array
}

//堆排序


function heapSort(array) {
  // 初始化大顶堆，从第一个非叶子结点开始
  for (let i = Math.floor(array.length / 2 - 1); i >= 0; i--) {
    adjustHeap(array, i, array.length);
  }
  // 排序，每一次for循环找出一个当前最大值，数组长度减一
  for (let j = array.length - 1; j > 0; j--) {
    // 根节点与最后一个节点交换
    const temp = array[0];
    array[0] = array[j];
    array[j] = temp;
    adjustHeap(array, 0, j);
  }
  return array;
}

//插入排序
function insertionSort(array) {
  for (let i = 1; i < array.length; i++) {
    let j = i - 1;
    const temp = array[i];
    while (j >= 0 && array[j] > temp) {
      array[j + 1] = array[j];
      j--;
    }
    array[j + 1] = temp;
  }
  return array;
}


//归并排序
function merge(left, right) {
  let res = [];
  while(left.length > 0 && right.length > 0) {
    if (left[0] < right[0]) {
      res.push(left.shift());
    } else {
      res.push(right.shift());
    }
  }
  return res.concat(left).concat(right);
}

function mergeSort(arr) {
  if (arr.length == 1) return arr;
  var middle = Math.floor(arr.length / 2);
  var left = arr.slice(0, middle);
  var right = arr.slice(middle);
  return merge(mergeSort(left), mergeSort(right));
}

//快速排序另一版本
function quicksort(arr) {
  if (arr.length <= 1) return arr;
  let pivotIndex = arr.length >> 1;
  let pivot = arr.splice(pivotIndex, 1)[0];
  let left = [];
  let right = [];
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] <= pivot)  {
      left.push(arr[i]);
    } else {
      right.push(arr[i]);
    }
  }
  return quicksort(left).concat(pivot, quicksort(right));

}
//选择排序
function selectionSort(array) {
  for (let i = 0; i < array.length; i++) {
    let minIndex = i;
    for (let j = i + 1; j < array.length; j++) {
      if (array[j] < array[minIndex]) {
        minIndex = j;
      }
    }
    const temp = array[i];
    array[i] = array[minIndex];
    array[minIndex] = temp;
  }
  return array;
}

//希尔排序
function shellSort(array) {
  const len = array.length;
  let gap =  Math.floor(len / 2);
  for (gap; gap > 0; gap = Math.floor(gap / 2)) {
    for (let i = gap; i < len; i++) {
      let j = i - gap;
      const temp = array[i];
      while (j >= 0 && array[j] > temp) {
        array[j + gap] = array[j];
        j -= gap;
      }
      array[j + gap] = temp;
    }
  }
  return array;
}



```

## 

# 2.原生API

## 数组API



#### Map

```javascript
Array.prototype.map = function(fn) {
    let newArray = [];
    for (let i = 0; i < this.length; i++) {
        newArray.push(fn(this[i]));
    }
    return newArray;
}
```

#### Filter

```javascript
Array.prototype.myFilter = function (fn, thisArr) {
  if (this === undefined) {
    throw new Error('this is null or not undefined')
  }
  if (Object.prototype.toString.call(fn) !== '[object Function]') {
    throw new Error(fn + ' is not a function')
  }
  let filterArr = this
  let filterRes = []
  for (let i = 0; i < filterArr.length; i++) {
    if (fn.call(thisArr, filterArr[i], i, filterArr)) {
      filterRes.push(filterArr[i])
    }
  }
  return filterRes
}
let arr = [1, 2, 3, 45, 6]
let arr1 = arr.myFilter((item) => item >= 6)
console.log(arr1)
```

#### Reduce

```javascript
Array.prototype.myReduce = function (callback, initialValue) {
  const arr = this
  let acc = typeof initialValue === 'undefined' ? arr[0] : initialValue
  let startIndex = typeof initialValue === 'undefined' ? 1 : 0
  for (let i = startIndex; i < arr.length; i++) {
    let curr = arr[i]
    acc = callback(acc, curr, i, arr)
  }
  return acc
}

let Arr = [1, 2, 3, 4, 5, 6, 7, 8, 9]
const sum = Arr.myReduce((pre, cur) => {
  return pre + cur
}, 10)
console.log(sum)

```

#### Foreach

```javascript
Array.prototype.myForeach = function (fn, thisArg) {
  let arr = this
  for (let i = 0; i < arr.length; i++) {
    // arr[i] = fn.call(thisArg, arr[i], i, arr)
    fn.call(thisArg, arr[i], i, arr)
  }
  return arr
}
let arr = [1, 2, 3, 45, 6]
let arr1 = arr.myForeach((item) => {
  console.log(item)
})
console.log(arr)
console.log(arr1)
```

#### Some && Every

```javascript

Array.prototype.mySome = function (fn, thisArg) {
  if (this === undefined) {
    throw new Error('this is null or not undefined')
  }
  if (Object.prototype.toString.call(fn) !== '[object Function]') {
    throw new Error(fn + ' is not a function')
  }
  let someArr = this
  for (let i = 0; i < someArr.length; i++) {
    if (fn.call(thisArg, someArr[i], i, someArr)) {
      return true
    }
  }
  return false
}
let arr = [1, 2, 3, 45, 6]
let res = arr.mySome((item) => {
  return item >= 49
})
console.log(res)



//多了一个取反   并且true和false互换位置
Array.prototype.myEvery = function (fn, thisArg) {
  if (this === undefined) {
    throw new Error('this is null or not undefined')
  }
  if (Object.prototype.toString.call(fn) !== '[object Function]') {
    throw new Error(fn + ' is not a function')
  }
  let everyArr = this
  for (let i = 0; i < everyArr.length; i++) {
    if (!fn.call(thisArg, everyArr[i], i, everyArr)) {
      return false
    }
  }
  return true
}
let arr = [1, 2, 3, 45, 6]
let res = arr.myEvery((item) => {
  return item > 0
})
console.log(res)


```

#### Includes && IndexOf

```javascript
Array.prototype.myIncludes = function (searchElement, formIndex = 0) {
  let includesEle = this
  let len = includesEle.length
  if (searchElement.length >= len || !len) return false
  for (let i = formIndex; i < len; i++) {
    if (includesEle[i] === searchElement) {
      return true
    }
  }
  return false
}
var str = 'Hello world, welcome to the Runoob。'
console.log(str.includes('world'))

```



## 对象API

#### instanceof

```javascript
/* 
自定义instanceof工具函数: 
  语法: myInstanceOf(obj, Type)
  功能: 判断obj是否是Type类型的实例
  实现: Type的原型对象是否是obj的原型链上的某个对象, 如果是返回true, 否则返回false
*/
function myInstanceOf(obj, Type) {
  // 得到原型对象
  let protoObj = obj.__proto__

  // 只要原型对象存在
  while(protoObj) {
    // 如果原型对象是Type的原型对象, 返回true
    if (protoObj === Type.prototype) {
      return true
    }
    // 指定原型对象的原型对象
    protoObj = protoObj.__proto__
  }

  return false
}
```

#### Object.create

```javascript
Object.create() = function create(prototype) {
  // 排除传入的对象是 null 和 非object的情况
	if (prototype === null || typeof prototype !== 'object') {
    throw new TypeError(`Object prototype may only be an Object: ${prototype}`);
	}
  // 让空对象的 __proto__指向 传进来的 对象(prototype)
  // 目标 {}.__proto__ = prototype
  function Temp() {};
  Temp.prototype = prototype;
  return new Temp;
}


//写法2
function create(proto) {
    function Fn() {};
    Fn.prototype = proto;
    Fn.prototype.constructor = Fn;
    return new Fn();
}
let demo = {
    c : '123'
}
let cc = Object.create(demo)

```

#### 实现New

```javascript
/* 
自定义new工具函数
  语法: newInstance(Fn, ...args)
  功能: 创建Fn构造函数的实例对象
  实现: 创建空对象obj, 调用Fn指定this为obj, 返回obj
*/
function newInstance(Fn, ...args) {
  // 创建一个新的对象
  const obj = {}
  // 执行构造函数
  const result = Fn.apply(obj, args) // 相当于: obj.Fn()
  // 如果构造函数执行的结果是对象, 返回这个对象
  if (result instanceof Object) {
    return result
  }
  // 如果不是, 返回新创建的对象
  obj.__proto__.constructor = Fn // 让原型对象的构造器属性指向Fn
  
  return obj
}
```

#### typeof

```javascript
function typeof(value){
    if(value === null){
        return null + '';
    }
    if(typeof value === 'object'){
        //如果为引用数据类型
        let valueClass = Object.prototype.toString.call(value).split(' ')[1],
            type = valueClass.split('');
        type.pop();
        return type.join('').toLowerCase();
    } else {
        return value;
    }
}
```



## 函数API

#### call apply bind

```javascript
//手写call
Function.prototype.myCall = function(context){
    if(typeof this !== 'function'){
        console.error('type error')
    };
    //获取参数
    let args = [...arguments].slice(1),
        result = null;
    //判断context是否传入 若没传入则设为window
    let  context = context || window;
    //将函数的调用者设为对象的方法  
    context.fn = this;
    result = context.fn(...args);
    delete context.fn;
    return result;
}
//手写apply
Function.prototype.myApply = function(context){
    if(typeof this !== 'function') {
        console.error('type error');
    }
    let result = null;
    let context = context || window;
    context.fn = this;
    if(arguments[1]){
        result = context.fn(...arguments[1]);
    } else {
        result = context.fn()
    }
    delete context.fn;
    return result;
}
//手写bind
Function.prototype.bind = function(context, ...args) {
    // context为要改变的执行上下文
    // ...args为传入bind函数的其余参数
    return (...newArgs) => {
        // 这里返回一个新的函数
        // 通过调用call方法改变this指向并且把老参和新参一并传入
        return this.call(context, ...args, ...newArgs);
    }
};
```



## 其他API

# 3.数组,对象,函数操作

### 数组乱序

```javascript
function shuffle(a) {
    for (let i = a.length; i; i--) {
        let j = Math.floor(Math.random() * i);
        [a[i - 1], a[j]] = [a[j], a[i - 1]];
    }
    return a;
}
```

### 数组去重

```javascript
/* 用不同的三种思想实现数组去重 */

let arr = [12, 23, 12, 15, 25, 23, 16, 25, 16];

/* 思想一：数组最后一项元素替换掉当前项元素，并删除最后一项元素 */

let setToArr = (arr) => {
  for (let i = 0; i < arr.length - 1; i++) {
    let cur = arr[i]; // 获取当前项
    let remainArr = arr.slice(i + 1); // 从i+1开始取（包括i+1），取数组后所有元素
    let idx = remainArr.indexOf(cur); // 看从i+1开始是否含有与cur相同的元素
    if (idx !== -1) {
      arr[i] = arr[arr.length - 1]; // 把最后一项替换当前项
      arr.length--; // 数组长度减1
      i--; // 从当前项再次判断
    }
  }
  return arr;
}

console.log(setToArr(arr)); // [ 16, 23, 12, 15, 25 ]

/* 思想二：利用map键值对来进行去重操作，替换操作与思想一一致 */

let setToArr2 = (arr) => {
  let map = new Map();
  for (let i = 0; i < arr.length; i++) {
    let cur = arr[i];
    if (map.has(cur)) {
      arr[i] = arr[arr.length - 1];
      arr.length--;
      i--;
    }
    map.set(cur, true);
  }
  return arr;
}
console.log(setToArr2(arr)); // [ 16, 23, 12, 15, 25 ]

/* 思想三：直接食用内置的 Set 去重 */

let setToArr3 = (arr) => {
  return [...new Set(arr)];
}
console.log(setToArr3(arr)); // [ 16, 23, 12, 15, 25 ]	
```

### 数组扁平化

```javascript
//另一个版本  无误
var flatten = function(arr) {
  let res = [];
  for (let i = 0; i < arr.length; i++) {
    if (Array.isArray(arr[i])) {
      res = res.concat(flatten(arr[i]))
    } else {
      res.push(arr[i])
    }
  }
  return res;
}
console.log(flatten([1,[1,2,[2,4]],3,5]));  // [1, 1, 2, 2, 4, 3, 5]

//扩展运算符(...)
function flattenDeep(arr){
	while(arr.some(item=>Array.isArray(item))){
		arr=[].concat(...arr);
	}
	return arr;
}
//reduce() 
function flattenDeep(arr){
	return arr.reduce((prev, next)=>{
		return prev.concat(Array.isArray(next) ? flattenDeep(next) : next);
	},[])
}
```

### 类数组转换为数组

```javascript
const arrayLike=document.querySelectorAll('div')

// 1.扩展运算符
[...arrayLike]
// 2.Array.from
Array.from(arrayLike)
// 3.Array.prototype.slice
Array.prototype.slice.call(arrayLike)
// 4.Array.apply
Array.apply(null, arrayLike)
// 5.Array.prototype.concat
Array.prototype.concat.apply([], arrayLike)


```



### 函数柯里化

```javascript
//手写函数柯里化
function curry(fn, args){
    //获取函数需要的总参数长度
    let leng = fn.length;
    args = args || [];
    return function(){
        let subargs = args.slice(0);
        for( let i = 0; i < arguments.length; i++){
            subargs.push(arguments[i]);
        }
        //判断此时subargs是否已经满足函数需要的参数长度需求
        if(subargs.length >= leng){
            return fn.apply(this, subargs)
        } else {
            return curry.call(this, fn, subargs);
        }
    }
}
function fn(a,b,c){
    return a+b+c;
}
var newCurry = curry(fn, 1);
newCurry(2);
newCurry(3);


/* 
柯理化函数含义：是给函数分步传递参数，每次传递部分参数，并返回一个更具体的函数接收剩下的参数，
这中间可嵌套多层这样的接收部分参数的函数，直至返回最后结果。
*/

function add(a, b, c, d) {
  return a + b + c + d;
}

function currying(fn, ...args) {
  if (fn.length === args.length) {
    return fn(...args);
  } else {
    return function (...newArgs) {
      return currying(fn, ...args, ...newArgs);
    }
  }
}

let addSum = currying(add)(1,2);
console.log(addSum(3)(4)); // 10

//实现add()方法，使计算结果能够满足如下预期:    
//add(1)(2)(3) = 6;    
//add(1, 2, 3)(4) = 10;    
//add(1)(2)(3)(4)(5) = 15;    
function add() {
	// 第一次执行时，定义一个数组专门用来存储所有的参数
	let _args = Array.prototype.slice.call(arguments);

	// 在内部声明一个函数，利用闭包的特性保存_args并收集所有的参数值
	let _adder = function() {
	    _args.push(...arguments);
	    return _adder;
	};

	// 利用toString隐式转换的特性，当最后执行时隐式转换，并计算最终的值返回
	_adder.toString = function () {
	    return _args.reduce(function (a, b) {
	        return a + b;
	    });
	}
	return _adder;
}


/////////////
function curry(fn, args) {
    length = fn.length;
    args = args || [];
    return function() {
        var _args = args.slice(0),
            arg, i;
        for (i = 0; i < arguments.length; i++) {
            arg = arguments[i];
            _args.push(arg);
        }
        if (_args.length < length) {
            return curry.call(this, fn, _args);
        }
        else {
            return fn.apply(this, _args);
        }
    }
}


var fn = curry(function(a, b, c) {
    console.log([a, b, c]);
});

fn("a", "b", "c") // ["a", "b", "c"]
fn("a", "b")("c") // ["a", "b", "c"]
fn("a")("b")("c") // ["a", "b", "c"]
fn("a")("b", "c") // ["a", "b", "c"]
```

### 对象深拷贝

```javascript
// 深拷贝的实现
function deepCopy(object) {
  if (!object || typeof object !== "object") return;

  let newObject = Array.isArray(object) ? [] : {};

  for (let key in object) {
    if (object.hasOwnProperty(key)) {
      newObject[key] =
        typeof object[key] === "object" ? deepCopy(object[key]) : object[key];
    }
  }

  return newObject;
}

```



# 4.场景题

### 节流防抖

```javascript
function throttle(fn, wait) {
  let  pre = new Date();
  return function() {
    let context = this;
    let args = arguments;
    let now = new  Date();
    if (now - pre >= wait) {
      fn.apply(context, args);
      pre = now;
    }
  }
}

//防抖
function debounce(fn, wait) {
  let timeout = null;
  return function() {
    if (timeout) clearTimeout(timeout);
    timeout = setTimeout(() => {
      fn.call(this, arguments);
    }, wait);
  }
}
```

### 字符串翻转

```javascript
String.prototype._reverse = function(a){
    return a.split("").reverse().join("");
}
var obj = new String();
var res = obj._reverse ('hello');
console.log(res);    // olleh

```

### 数字千分位逗号分割

```javascript
let format = n => {
    let num = n.toString() 
    let len = num.length
    if (len <= 3) {
        return num
    } else {
        let remainder = len % 3
        if (remainder > 0) { // 不是3的整数倍
            return num.slice(0, remainder) + ',' + num.slice(remainder, len).match(/\d{3}/g).join(',') 
        } else { // 是3的整数倍
            return num.slice(0, len).match(/\d{3}/g).join(',') 
        }
    }
}
format(1232323)  // '1,232,323'

```

### 非负大整数相加

```javascript
function sumBigNumber(a, b) {
  let res = '';
  let temp = 0;
  
  a = a.split('');
  b = b.split('');
  
  while (a.length || b.length || temp) {
    temp += ~~a.pop() + ~~b.pop();
    res = (temp % 10) + res;
    temp  = temp > 9
  }
  return res.replace(/^0+/, '');
}
如果想要对一个超大的整数(> Number.MAX_SAFE_INTEGER)进行加法运算，但是又想输出一般形式，那么使用 + 是无法达到的，一旦数字超过 Number.MAX_SAFE_INTEGER 数字会被立即转换为科学计数法，并且数字精度相比以前将会有误差。

其主要的思路如下：

首先用字符串的方式来保存大数，这样数字在数学表示上就不会发生变化
初始化res，temp来保存中间的计算结果，并将两个字符串转化为数组，以便进行每一位的加法运算
将两个数组的对应的位进行相加，两个数相加的结果可能大于10，所以可能要仅为，对10进行取余操作，将结果保存在当前位
判断当前位是否大于9，也就是是否会进位，若是则将temp赋值为true，因为在加法运算中，true会自动隐式转化为1，以便于下一次相加
重复上述操作，直至计算结束


```

### 将数组里面的对象与数形结构相互转换

```javascript
// 转换前：
source = [{
            id: 1,
            pid: 0,
            name: 'body'
          }, {
            id: 2,
            pid: 1,
            name: 'title'
          }, {
            id: 3,
            pid: 2,
            name: 'div'
          }]
// 转换为: 
tree = [{
          id: 1,
          pid: 0,
          name: 'body',
          children: [{
            id: 2,
            pid: 1,
            name: 'title',
            children: [{
              id: 3,
              pid: 1,
              name: 'div'
            }]
          }
        }]
        
        
   function jsonToTree(data) {
  // 初始化结果数组，并判断输入数据的格式
  let result = []
  if(!Array.isArray(data)) {
    return result
  }
  // 使用map，将当前对象的id与当前对象对应存储起来
  let map = {};
  data.forEach(item => {
    map[item.id] = item;
  });
  // 
  data.forEach(item => {
    let parent = map[item.pid];
    if(parent) {
      (parent.children || (parent.children = [])).push(item);
    } else {
      result.push(item);
    }
  });
  return result;
}


//树形转列表
function treeToList(data) {
  let res = [];
  const dfs = (tree) => {
    tree.forEach((item) => {
      if (item.children) {
        dfs(item.children);
        delete item.children;
      }
      res.push(item);
    });
  };
  dfs(data);
  return res;
}


```

### 实现红绿灯

```javascript
function red() {
    console.log('red');
}
function green() {
    console.log('green');
}
function yellow() {
    console.log('yellow');
}

const task = (timer, light) => 
    new Promise((resolve, reject) => {
        setTimeout(() => {
            if (light === 'red') {
                red()
            }
            else if (light === 'green') {
                green()
            }
            else if (light === 'yellow') {
                yellow()
            }
            resolve()
        }, timer)
    })
const step = () => {
    task(3000, 'red')
        .then(() => task(2000, 'green'))
        .then(() => task(2100, 'yellow'))
        .then(step)
}
step()

```

### 每隔一秒打印1234

```javascript
// 使用闭包实现
for (var i = 0; i < 5; i++) {
  (function(i) {
    setTimeout(function() {
      console.log(i);
    }, i * 1000);
  })(i);
}
// 使用 let 块级作用域
for (let i = 0; i < 5; i++) {
  setTimeout(function() {
    console.log(i);
  }, i * 1000);
}

```

### Promise 实现图片异步加载

```javascript
let imageAsync=(url)=>{
            return new Promise((resolve,reject)=>{
                let img = new Image();
                img.src = url;
                img.οnlοad=()=>{
                    console.log(`图片请求成功，此处进行通用操作`);
                    resolve(image);
                }
                img.οnerrοr=(err)=>{
                    console.log(`失败，此处进行失败的通用操作`);
                    reject(err);
                }
            })
        }
        
imageAsync("url").then(()=>{
    console.log("加载成功");
}).catch((error)=>{
    console.log("加载失败");
})

```

### 带委托的通用事件监听函数

```javascript
/* 
绑定事件监听的通用函数(带委托)
*/
function bindEvent2(ele, type, fn, selector) {

  ele.addEventListener(type, event => {
    // 得到发生事件的目标
    const target = event.target
    if (selector) {
      // 如果元素被指定的选择器字符串选择, 返回true; 否则返回false。
      if (target.matches(selector)) {
        // 委托绑定调用
        fn.call(target, event)
      } 
    } else {
      // 普通绑定调用
      fn.call(ele, event)
      // fn(event) // this不对
    }
  })
}


<ul>
   <span>
    <li>
    <li>
</ul>
    
bindEvent2(ul, 'click', (event) => {}, 'li')
bindEvent2(ul, 'click', (event) => {})
```

### 实现发布订阅模式

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }
  // 实现订阅
  on(type, callBack) {
    if (!this.events[type]) {
      this.events[type] = [callBack];
    } else {
      this.events[type].push(callBack);
    }
  }
  // 删除订阅
  off(type, callBack) {
    if (!this.events[type]) return;
    this.events[type] = this.events[type].filter((item) => {
      return item !== callBack;
    });
  }
  // 只执行一次订阅事件
  once(type, callBack) {
    function fn() {
      callBack();
      this.off(type, fn);
    }
    this.on(type, fn);
  }
  // 触发事件
  emit(type, ...rest) {
    this.events[type] &&
      this.events[type].forEach((fn) => fn.apply(this, rest));
  }
}
// 使用如下
// const event = new EventEmitter();

// const handle = (...rest) => {
//   console.log(rest);
// };

// event.on("click", handle);

// event.emit("click", 1, 2, 3, 4);

// event.off("click", handle);

// event.emit("click", 1, 2);

// event.once("dbClick", () => {
//   console.log(123456);
// });
// event.emit("dbClick");
// event.emit("dbClick");
```

实现jsonP

```javascript
// 动态的加载js文件
function addScript(src) {
  const script = document.createElement('script');
  script.src = src;
  script.type = "text/javascript";
  document.body.appendChild(script);
}
addScript("http://xxx.xxx.com/xxx.js?callback=handleRes");
// 设置一个全局的callback函数来接收回调结果
function handleRes(res) {
  console.log(res);
}
// 接口返回的数据格式
handleRes({a: 1, b: 2});

```

### setTimeout 实现 setInterval

```javascript
function myInterval(fn, time) {
  let context = this;
  setTimeout(() => {
    fn.call(context);
    myInterval(fn, time);
  }, time);
}
```

### 分片思想解决大量数据渲染问题

```javascript
let ul = document.getElementById("container");
// 插入十万条数据
let total = 100000;
// 一次插入 20 条
let once = 20;
//总页数
let page = total / once;
//每条记录的索引
let index = 0;
//循环加载数据
function loop(curTotal, curIndex) {
  if (curTotal <= 0) {
    return false;
  }
  //每页多少条
  let pageCount = Math.min(curTotal, once);
  window.requestAnimationFrame(function () {
    for (let i = 0; i < pageCount; i++) {
      let li = document.createElement("li");
      li.innerText = curIndex + i + " : " + ~~(Math.random() * total);
      ul.appendChild(li);
    }
    loop(curTotal - pageCount, curIndex + pageCount);
  });
}
loop(total, index);

```

### LRU

![image-20211129194433731](C:/Users/t1/AppData/Roaming/Typora/typora-user-images/image-20211129194433731.png)

### ul里面生成，反转1000个li

![image-20211129195759456](C:/Users/t1/AppData/Roaming/Typora/typora-user-images/image-20211129195759456.png)

![image-20211129195814849](C:/Users/t1/AppData/Roaming/Typora/typora-user-images/image-20211129195814849.png)

