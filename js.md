### 1.手写promise

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

### 2.手写promise.all  promise.race  promise.or

```javascript
function promiseAll(promises) {
  return new Promise(function(resolve, reject) {
    if(!Array.isArray(promises)){
        throw new TypeError(`argument must be a array`)
    }
    var resolvedCounter = 0;
    var promiseNum = promises.length;
    var resolvedResult = [];
    for (let i = 0; i < promiseNum; i++) {
      Promise.resolve(promises[i]).then(value=>{
        resolvedCounter++;
        resolvedResult[i] = value;
        if (resolvedCounter == promiseNum) {
            return resolve(resolvedResult)
          }
      },error=>{
        return reject(error)
      })
    }
  })
}

// test
let p1 = new Promise(function (resolve, reject) {
    setTimeout(function () {
        resolve(1)
    }, 1000)
})
let p2 = new Promise(function (resolve, reject) {
    setTimeout(function () {
        resolve(2)
    }, 2000)
})
let p3 = new Promise(function (resolve, reject) {
    setTimeout(function () {
        resolve(3)
    }, 3000)
})
promiseAll([p3, p1, p2]).then(res => {
    console.log(res) // [3, 1, 2]
})
// promise.all
function myPromiseAll(promiseAll) {
    return new Promise((resolve, reject) => {
        if (!Array.isArray(promiseAll)) {
            throw 'promise must be an Array'
        }
        let resArr = [];
        let len = promiseAll.length;
        let count = 0;
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
// 这么简单得益于promise的状态只能改变一次，即resolve和reject都只被能执行一次
Promise.prototype.myRace = function (promises) {

    return new Promise(function (resolve, reject) {
        for (let i = 0; i < promises.length; i++) {
            primises[i].then(resolve, reject)
        }
    })

}
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

### 手写Ajax

``` javascript
function ajax(options){
   //设置默认对象 
	var defaults = {
        type: 'get',
        url: '',
        data: {},
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
        },
        success: function(){},
        error: function(){}
    };
    //将传入参数对象与默认对象合并
    Object.assign(defaults, options);
    var params = '';
    for(var attr in defaults.data){
        params += attr + '=' + defaults.data[attr] + '&';
    }
    params = params.substr(0, params.length);
    if(defaults.type == 'get'){
        default.url = default.url + '?' + params;
    }
    var xhr = new XMLHTTPRequest();
    xhr.open(defaults.type, defaults.url);
    if(defaults.type == 'post'){
        var contentType = defaults.headers['Content-Type'];
        xhr.setResquestHeader('Content-Type', contentType);
        if(contentType == 'application/json'){
            xhr.send(JSON.stringify(defaults.data));
        } else {
            xhr.send(params);
        }    
    } else {
        xhr.send();
    }
    xhr.onload = function(){
        var contentType = xhr.getResquestHeader('Content-Type');
        var responseText = xhr.responseText;
        if(contentType.includes('application/json')){
            responseText = JSON.parse(responseText);
        }
        if(xhr.status == 200){
            defaults.success(responseText, xhr);
        } else {
            defaults.error(responseText, xhr);
        }
    }
}


//promise 实现Ajax
function ajax(url, method, data) {
    const p = new Promise((res, rej) => {
        const xhr = new XMLHttpRequest();
        xhr.open(method, url, true);
        xhr.onreadystatechange = function() {
            if (xhr.readyState == 4) {
                if (xhr.status == 200) {
                    res(
                        JSON.parse(xhr.responseText)
                    )
                } else if (xhr.status === 404 || xhr.status === 500) {
                    rej(
                        new Errow('404 not found')
                    )
                }
            }
        }
        xhr.send(data);
    })
    return p;
}

const url = './data/test.json'
ajax(url, 'GET')
    .then(res => console.log(res))
    .catch(err => console.error(err))
```

### 节流与防抖

​     在页面中如果持续触发一个事件会对性能不利，例如页面滚动、鼠标移动等若持续触发会造成事件冗余，也为页面加载带来负担。节流指的是函数在触发过了规定时间后再执行，若在规定时间内再次触发会重新计时， 再过规定时间后再执行。

``` javascript
function jieliu(fn, wait){
    var timer;
    if(timer){
        clearTimeout(timer)
    }
    return function(){
        timer = setTimeout(fn, wait)
    }
}
function fn(){
    console.log('防抖')
}
window.addEventListen('scroll', jieliu(fn, 1000))
```

防抖是在规定时间内只会触发一次，可以分为有时间戳和定时器两种。

``` javascript
function shijianchuo(fn, wait){
	var pre = new Date();
    return function(){
        var that = this;
        var args = arguments;
        var now = new Date();
        if(now - pre >= wait){
            fn.apply(that, args);
        }
    }
}
function fn(){
    console.log('时间戳')
}
window.addEventListener('scroll', shijianchuo(fn, 1000));
```

``` javascript
function dingshiqi(fn, wait){
    var timer = null;
    return function(){
        var that = this;
        var args = arguments;
        if(!timer){
            timer = setTimerout(function(){
                fn.apply(that, args);
                timer = null;
            }, wait)
        }
    }
}
function fn(){
    console.log('定时器')
}
window.addEventListener('scroll', dingshiqi(fn, 1000));
```

### 深浅拷贝

``` javascript
//浅拷贝的实现 只拷贝对象
function shallowCope(object){
    if(!object || typeof object !== 'object') return;
    //根据object的类型进行判断新建一个数组还是对象
    let newObject = Array.isArray(object) ? [] : {};
    for(let k in object){
        if(object.hasOwnProperty(k)){
            newObject[k] = object[k];
        }
    }
    return newObject;
}
//深拷贝的实现
function deepCope(object){
    if(!object || typeof object !== 'object') return;
    let newObject = Array.isArray(object) ? [] : {};
    for(let k in object){
        if(object.hasOwnProperty(k)){
            newObject[k] = 
                typeof object[k] === 'object' ? deepCope(object[k]) : object[k];
        }
    }
    return newObject;
}
```

### 手写call apply bind

``` javascript
//手写call
Function.prototype.myCall = function(context){
    if(typeof this !== 'function'){
        console.error('type error')
    };
    //获取参数
    let args = [...arguments].slice(1),
        result = null;
    //判断context是否传入 若没传入则设为window
    context = context || window;
    //将调用函数设为对象的方法
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
    context = context || window;
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
Function.prototype.myBind = function(newObject){
    if( typeof this !== 'function'){
        console.error('type error');
    }
    var args = [...arguments].slice(1);
    var that = this;
    return function(){
       return that.apply(newObject, args.concat([...arguments]))
    }
}
```

### 函数柯里化

函数柯里化是指将一种使用多个参数的函数转换为一系列单个参数函数的调用。

``` javascript
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
```

### setTimeout 实现 setInterval

``` javascript
function myInterval(fn, wait){
    var timer = {
        flag: true
    };
    function interval(){
        if(timer.flag){
            fn();
            setTimeout(interval, wait);
        }
    }
    setTimeout(interval, wait);
    return timer;
}
```

### 封装一个 javascript 的类型判断函数

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

### 闭包实现每隔一秒打印 1,2,3,4

``` javascript
for(let i = 1; i <= 4; i++){
    setTimeout(function(){
        console.log(i)
    }, i*1000)
}

for(var i = 1; i <=4; i++){
    (function(j){
        setTimeout(function(){
            console.log(j)
        }, j*1000)
    })(i)
}
```

### 手写jsonp



``` javascript
function jsonp(url, params, callback){
    //首先判断url本身是否带有参数 即是否带有？
    queryString = url.indexOf('?') === '-1' ? '?' : '&';
    for(var k in params){
        if(params.hanOwnProperty(k)){
            queryString += k + '=' + params[k];
        }
    }
    //定义一个随机的函数名 添加到参数中
    var randomName = Math.random().toString().replace('.', '');
    var myFunction = 'myFunction' + randomName;
    queryString += 'callback' + '=' + myFunction;
    url += queryString;
    //动态创建script标签
    var myScript = document.createElement('script');
    myScript.src = url;
    window[myFunction] = function(){
        callback(...arguments);
        //删除这个动态脚本
        doucment.getElementsByTagName('head')[0].removeChild(myScript);
    };
    //将脚本插入到head中
    document.getElementsByTagName('head')[0].appendChild(myScript);
}
```

### 手写观察者模式

``` javascript
class Subject{
  constructor(name){
    this.name = name;
    this.state = '开心的';
    this.observers = []; // 存储所有的观察者
  }
  // 收集所有的观察者
  attach(observer){
    this.observers.push(observer);
  }
  // 更新被观察者的状态
  setState(newState){
    this.state = newState; 
    this.observers.forEach(observer=>observer.update(this)); // 通知观察值，更新状态
  }
}
// 观察者 例如老师、家长
class Oberver{
  constructor(name){
    this.name = name;
  }
  update(student){
    console.log('当前' + student.name + '学生的状态是' + student.state + ',已经通知了' + this.name);
  }
}

let student = new Subject('小羊');
let parent = new Oberver('家长');
let teacher = new Oberver('老师');

student.attach(parent);
student.attach(teacher);

student.setState('成绩优秀！');
// 当前小羊学生的状态是成绩优秀！,已经通知了家长
// 当前小羊学生的状态是成绩优秀！,已经通知了老师
```

###



### EventEmitter 实现

``` javascript
class eventEmitter {
    constructor(){
        this.events = {}
    }
    on(event, callback){
        let callbacks = this.events[event] || [];
        callbacks.push(callback);
        this.events[event] = callbacks;
        return this;
    }
    emit(event, ...args){
        let callbacks = this.events[event];
        callbacks.forEach(fn => {
            fn(...args);
        })
        return this;
    }
    off(event, callback){
        let callbacks = this.events[event];
        callbacks = callbacks.filter(fn => return fn !== callback);
        this.events[event] = callbacks;
        return this;
    }
    once(event, callback){
        let wrapFun = function(...args){
            callback(...args);
            this.off(wrapFun);
        }
        this.on(event, wrapFun);
        return this;
    }
}
```

### 懒加载

(1).将需要懒加载的img标签的src设置缩略图或者不设置src，这里的占位图可以是缺省图，loading图；
(2).判断该img标签是否在浏览器可视区域，如果在可视区域，则将真实的图片url设置到img标签的src属性；
(3).用户滚动浏览器，遍历需要懒加载的标签，根据步骤2判断并执行；

``` JavaScript
let imgs = document.getElementTageName('img')
let num = imgs.length
let n = 0; // 存储图片加载到的位置，避免每次都从第一张图片开始遍历

function lazyload() {

    let seeHeight = document.documentElement.clientHeight // 可见区域高度
    let scrollTop = document.documentElement.scrollTop || document.body.scrollTop //滚动条距离顶部高度
    for (let i = n; i < num; i++) {

        if (imgs[i].offsetTop < seeHeight + scrollTop) {
            if (imgs[i].getAttribute('src') === 'default.jpg') {
                imgs[i].src = img[i].getAttribute('data-src')
            }
            n = i+1
        }

    }

}
```

### 模拟实现new

``` JavaScript
function create() {
    // 创建一个空的对象
    console.log(arguments);
    var obj = new Object(),
        // 删除并拿到arguments的第一项就是我们的构造函数
        Con = [].shift.apply(arguments);
    console.log(Con);
    // 链接到原型，obj 可以访问到构造函数原型中的属性
    obj.__proto__ = Con.prototype;
    // 绑定 this 实现继承，obj 可以访问到构造函数中的属性
    var ret = Con.apply(obj, arguments);
    // 优先返回构造函数返回的对象
    return typeof ret === 'object' ? ret : obj;
};

function Car(color) {
    this.color = color;
}
Car.prototype.start = function() {
    console.log(this.color + " car start");
}
var car = create(Car, "black");
car.color;
// black
car.start();
// black car start
```

### 数组的三种方法

``` JavaScript
// filter
Array.prototype.filter1 = function(fn) {
    let newArray = [];
    for (let i = 0; i < this.length; i++) {
        if (fn(this[i])) {
            newArray.push(this[i]);
        }
    }
    return newArray
}
let arr = [1, 2, 3, 45, 6]
let arr1 = arr.filter1((item) =>
    item >= 6
)
console.log(arr1);


// map
Array.prototype.map1 = function(fn) {
    let newArray = [];
    for (let i = 0; i < this.length; i++) {
        let key = fn(this[i]);
        newArray.push(key);
    }
    return newArray;
}



let arr = [1, 2, 3, 4];
let arr1 = arr.map1((item) => {
    return item * 2;
})
console.log(arr1);

// reduce 无初始值
Array.prototype.reduce1 = function(fn) {
    let newVal = this[0];
    for (let i = 1; i < this.length; i++) {
        newVal = fn(newVal, this[i], i, arr);
    }
    return newVal
}
let arr = [1, 2, 3, 4, 5]
let resMult = arr.reduce1((value, item, i, arr) => { //value为累计值，item当前值，i当前值索引，arr当前操作的数组
    // console.log(value, item, i, arr)
    return value * item
})
let resAdd = arr.reduce1((value, item, i, arr) => {
    // console.log(value, item, i, arr)
    return value + item
})
console.log(resMult, resAdd) // 120 15

//reduce 有初始值

Array.prototype.reduce2 = function(fn, ini) {
    let key = ini ? ini : this[0];
    for (let i = ini ? 0 : 1; i < this.length; i++) {
        key = fn(key, this[i], i, arr);
    }
    return key;
}


let arr = [10, 2, 3, 4, 5];
let arr2 = arr.reduce2((val, item, i, arr) => {
    return val + item;
}, 10)
let arr3 = arr.reduce2((val, item, i, arr) => {
    return val * item;
})
console.log(arr2, arr3);
```

### 数组降维

``` JavaScript
//数组降维  去掉null 【】 和 undefined（字节笔试题）
let arr = [
    [
        [
            [
                [0]
            ],
            [1]
        ],
        [
            [
                [2],
                [3]
            ]
        ],
        [
            [4],
            [5]
        ]
    ]
];
let arr2 = [
    ['a', 'b'],
    [0, ['a']],
    [false],
    []
];
let res = [1, 2, [3, 4, [10, 20, [100, 200]]], 5];
Array.prototype.myFlatten = function() {
    return this.reduce((pre, cur, index) => {
        if (cur == undefined) return pre;
        else {
            return Array.isArray(cur) ? pre.concat(cur.myFlatten()) : pre.concat(cur);
        }
    }, [])
}

let array = arr.myFlatten();
let array2 = arr2.myFlatten();
let array3 = res.myFlatten();
console.log(array);
console.log(array2);
console.log(array3);
```

### 实现千位分隔符

``` JavaScript
// 方法一：
// function format(num) {
//     let reg = /(\d{1,3})(?=(\d{3})+$)/g
//     return (num + '').replace(reg, '$&,');
// }

console.log(format(98765321));

//正则表达式 \d{1,3}(?=(\d{3})+$)  表示前面有1~3个数字，后面的至少由一组3个数字结尾。

//?=表示正向引用，可以作为匹配的条件，但匹配到的内容不获取，并且作为下一次查询的开始。

//$& 表示与正则表达式相匹配的内容，具体的使用可以查看字符串replace()方法的

//replace是字符串的方法 需要把num转成字符串 (num + '')


// 方法二：
function format(num) {
    var num = num + '';
    var str = '';
    for (let i = num.length - 1, j = 1; i >= 0; i--, j++) {
        if (j % 3 == 0 && i != 0) {
            str += num[i] + ','
            continue;
        }
        str += num[i];
    }
    console.log(str);

    return str.split('').reverse().join('');
}
```

### 数组去重

``` JavaScript
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

### 实现sleep

``` JavaScript
function sleep(time) {
  return new Promise(reslove => setTimeout(reslove, time));
}
function say(name) {
  console.log(`hello ${name}`);
}

async function task() {
  await sleep(3000);
  say('Chocolate');
  say('Yang');
}

task();		
```

### 手写发布订阅

```JavaScript
class EventEmitter {
  constructor() {
    this.events = {};
  }
  // 订阅事件
  on(eventName, callback) {
    if (!this.events[eventName]) {
      this.events[eventName] = [callback];
    } else {
      this.events[eventName].push(callback);
    }
  }
  // 触发事件
  emit(eventName) {
    this.events[eventName] && this.events[eventName].forEach(cb => cb());
  }
}

let em = new EventEmitter();

function workDay() {
  console.log("每天工作");
}
function makeMoney() {
  console.log("赚100万");
}
function sayLove() {
  console.log("找到另一伴");
}
em.on("task", workDay);
em.on("task", makeMoney);
em.on("task", sayLove);

em.emit("task");
```

