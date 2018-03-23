# 周报2018.2.7



## 1.快捷获取函数参数

```
var args = [].slice.call(arguments,0);
```

函数的参数看着像是数组，但是它不是真正的数组，所以他们没有数组的方法，我们用call方法赋予它

数组的slice方法，这样就可以截取想要的参数。

这样我们就可以看做是：

```
var args = arguments.slice(0); //获取所有参数
```



## 2.Promise,Generator和sync 异步编程方案

之前为了写一些简单动画有简单使用过`promise`的，这周因为计划学习ES6,发现`generator`和`sync`(ES2017)比`promise`更好学和使用。



### promise对象

**Promise**是一个对象，它存储着每个未来结束的事件（通常是一个异步操作）的结果。Promise提供统一的API,各种异步操作都可以用同样的方法进行处理。

Promise的两大特点：

1. 对象的状态不受外界影响。它有三种状态：Pending,fulfilled和rejected.只有异步操作的结果可以决定当前是哪一种状态，任何其它操作都无法改变这个状态。这也应了它的名字“Promise”,承诺，不被外界因素影响说好的结果。
2. 状态一旦改变，就不会再变。Promise对象的状态改变，只有两种：从`pending`到`fulfilled`和从`pending`到`rejected`。只要这两种情况发生，状态就凝固了，不会再变变了，这时称为resolved(已定型)。这时再对Promise添加回调函数，也会立即得到这个结果。

Promise的缺点

1. 一旦建立就会立即执行，无法中途取消。
2. 如果不设置回调函数，promise内部抛出错误，不会反应到外部。
3. 当处于pending状态时，无法得知目前进展到哪一个阶段。

Promise的基本用法：

```
const promise = new Promise(function(resolve,reject){
  // ...some code
  if(/* 条件成立 */){
    resolve(data);
  }else{
    reject(error);
  }
})
```

Promose构造函数接受一个函数作为参数，该函数有两个函数参数分别是`resolve`和`reject`;resolve的作用是将Promise对象的状态从`pending`变为`fulfilled`（成功）。reject是将`pending`变为`rejected`（失败）。resolve和reject都可以通过参数把异步操作信息传递出去。

Promise实例生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数。

```
promise.then(function(data){
  //成功回调函数
},function(error){
  //失败回调函数
})
```



### Generator函数：

**Generator**函数是ES6提供的另外一种异步编程解决方案。它的语法行为和以往的传统函数完全不同。

Generator函数有多种理解角度：

1. 语法上，Generator函数是一个状态封装机，封装了多个内部状态。

2. 执行Generantor函数会返回一个遍历器对象，也就是说Generator函数还是一个遍历器对象生成函数。返回的遍历器对象可以一次遍历Generator函数内部的每一个状态。

3. 形式上，Generator函数是一个普通函数，但是有两个特征：1、function关键字与函数名之间有一个*。2、函数体内使用  yield* 表达式来定义不同的内部状态。

   ```
   function* hellowWorldGenerator(){
     yield 'hello';
     yield 'world';
     return 'ending';
   }
   var hw = helloWorldGenerator();
   ```

    以上定义的Generator函数helloWorldGenerator,内部有两个`yield`表达式，即该函数有三个状态：hello,world 和return语句（结束执行）。

   需要注意的是Generator函数被调用后，该函数并不执行，而是返回指向内部状态的指针对象，也就是遍历器对象（Iterator Object）。

   ```
   hw.next()
   // { value: 'hello', done: false }

   hw.next()
   // { value: 'world', done: false }

   hw.next()
   // { value: 'ending', done: true }

   hw.next()
   // { value: undefined, done: true }
   ```

   遍历器对象使用`next`方法，使得指针移向下一个状态。也就是说，每次调用next方法，内部指针就从函数的头部或者上一次停下来的地方开始执行，知道遇到下一个`yield`表达式（或`return`）为止。换言之，Generator函数是分段执行的，`yield`表达式是暂停执行的标记，而`next`方法可以恢复执行。

   调用 Generator 函数，返回一个遍历器对象，代表 Generator 函数的内部指针。每次调用遍历器对象的`next`方法，就会返回一个有着`value`和`done`两个属性的对象。`value`属性表示当前的内部状态的值，是`yield`表达式后面那个表达式的值；`done`属性是一个布尔值，表示是否遍历结束。



### async函数：

`async`函数其实就是`Generator`函数的语法糖。

```
function timeout(ms) {
  return new Promise((resolve) => {
    setTimeout(function(){console.log(ms);reslove();}, ms);
  });
}

async function asyncPrint(value, ms1,ms2) {
  await timeout(ms);
  await timeout(ms);
  console.log(value);
}

asyncPrint('hello world', 5000,2000);

```

以上代码指定5秒后先输出2000，再过2秒再输出5000，最后输出hello world,因为它要等`timeout`函数执行完毕才会执行。

`async`函数就是将 Generator 函数的星号（`*`）替换成`async`，将`yield`替换成`await`

asyn函数对Generator函数的改进，体现在一下四点：

1. 内置执行器。async函数不像Generator，调用async函数就会自动执行输出结果。
2. 更好的语义。sync和await,比起*和yield，语义更清楚了。
3. 更广的适用性。`co`模块约定，`yield`命令后面只能是 Thunk 函数或 Promise 对象，而`async`函数的`await`命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）。
4. 返回值是Promise。可以使用then方法指定下一步的操作。

进一步说，`async`函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而`await`命令就是内部`then`命令的语法糖。



`sync`函数返回一个 Promise 对象。

`async`函数内部`return`语句返回的值，会成为`then`方法回调函数的参数。

```
async function f() {
  return 'hello world';
}

f().then(v => console.log(v))
// "hello world"
```



相比于Promise和Generator，我觉得async是在前两个的加强版，更加强大和便捷。

以上我目前对三个异步编程方案的简单理解，日后再进一步了解后再做修改。



