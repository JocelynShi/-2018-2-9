# 周报2018.3.21

# JavaScript单线程运行机制
*参考资料：*  
*[http://blog.csdn.net/w2765006513/article/details/53743051（浅谈js运行机制(线程））](http://blog.csdn.net/w2765006513/article/details/53743051)*  
*[http://www.ruanyifeng.com/blog/2014/10/event-loop.html JavaScript 运行机制详解：再谈Event Loop 作者：阮一峰](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)*  
*[http://www.ruanyifeng.com/blog/2012/12/asynchronous%EF%BC%BFjavascript.html Javascript异步编程的4种方法 作者：阮一峰](http://www.ruanyifeng.com/blog/2012/12/asynchronous%EF%BC%BFjavascript.html)*

## 1、为什么JavaScript是单线程语言？
JavaScript语言的一大特点就是单线程，也就是说，同一个时间是能做一件事。
那么为什么JavaScript不能有多个线程呢？因为这样能提高效率。
JavaScript的单线程，与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户交互，以及操作DOM。这决定了它只能是单线程，否则会带来很多复杂的同步问题。
比如: 假定JavaScipt同时有两个线程，一个线程把DOM节点添加到内容，另外一个线程删除这个节点，这时浏览器应该以哪个线程为准呢？

所以，为了避免复杂性，从一诞生，JavaScript就是单线程，只已经成了这门语言的核心特征，将来也不会改变。
为了利用多核CPU的计算能力，HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变JavaScript单线程的本质。
## 2、浏览器线程

由于浏览器是多功能的，所以它是多线程的，主要有以下5中线程：  

- UI渲染线程：用于渲染页面
- js线程：用于执行js任务
- 浏览器事件触发线程：用于控制交互，响应用户
- http请求线程：用于请求，ajax是委托给浏览器新开一个http线程
- EventLoop轮循的处理线程：用于轮循消息队列

## 3、JavaScript单线程
- 单线程的含义是js只能在一个线程上运行，也就是说，js同时只能执行js任务，其它的任务则排队等待执行。
- js是单线程的，并不代表js引擎线程只有一个。上文已介绍，HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。
- 多线程之间共享运行资源，浏览器端的js会操作DOM,多个线程必然会带来同步问题，所以js核心选择单线程来避免处理麻烦。js可以操作DOM,影响渲染，所以js引擎线程和UI线程是互斥的。这也就解释了js执行时会阻塞页面的渲染。


## 4、任务队列
单线程意味着所有任务需要排队，前一个任务结束，才会执行后一个任务。如果前一个任务耗时很长，后一个任务就不得不一直等着。   
如果排队是因为计算量大，CPU忙不过来，倒也算了，但是很多CPU是闲着的，因为IO设备(输入输出设备)很慢（比如Ajax操作从网络读取数据），不得不等结算出来，再往下执行。  
JavaScript语言设计者意识到，这时主线程完全可以不管IO设备，挂起处于等待的任务，先运行排在后面的任务。等到IO设备返回了结果，再回头把挂起的任务继续执行下去。  
于是，所有任务可以分成两种：  
 1、 **同步任务（synchronous）**：在主线程执行的任务，只有前一个任务执行完毕，才执行后一个任务；   

 2、 **异步任务（asynchronous）**: 不进入主线程，而进入“任务队列”（task queue）的任务，只有“任务队列”通知主线程，某个异步任务可以执行了，该任务才会进入主线程。  

具体来说，异步执行的运行机制如下：（同步执行也是如此，因为它可以被视为没有异步任务的异步执行。）    


```

	1. 所有同步任务都在主线程上执行，形成一个执行栈（execution context stack）。
	2. 主线程之外，还存在一个“任务队列”（task queue）。只要异步任务有了运行结果，就在“任务队列”之中放置一个事件。
	3. 一旦“执行栈”中的所有同步任务执行完毕，系统就会读取“任务队列”，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。
	4. 主栈程不断重复上面的第三步。
	


```

下图就是主线程和任务队列的示意图：
![](http://www.ruanyifeng.com/blogimg/asset/2014/bg2014100801.jpg)
  
只要主线程的队列空了，就会去读取“任务队列”，这就是JavaScript的运行机制。这和过程会不断重复。

## 5、异步模式
前面4个段落分析了js的单线程的同步任务和异步任务，同步任务可以让我们约定好的任务按照顺序执行，异步任务可以按照预定好的事件或者事件来执行。同步的有点我们自不用分析，因为它简单明了，提高了执行效率，但是容易出现阻塞，假死现象。由此，异步也非常重要，它可以让我们写出更合理、性能更出色、维护更方便的js程序。目前主要有四种“异步模式”变成的方法：  
### [ 5.1 回调函数](./callback.html)
这是异步变成最基本的方法：
 假定有两个函数f1和f2,后者必须等待前者的执行结果。  

```

	f1();
	f2();
```  
如果f1是一个很耗时的任务，可以考虑改写f1,把f2写成f1的回调函数：  
  
```
	
	function f1(callback){  

		//f1主要逻辑代码

		callback();
	
	　　　　setTimeout(function () {
	
	　　　　　　// f1的任务代码
	
	　　　　　　
	
	　　　　}, 1000);
	
	　　}

```
  
执行代码就变成下面的这样：  
  
```  

	f1(f2);
```
  
采用这种方法，我们把同步操作变成了异步操作，f1不会堵塞程序的运行，相当于先执行程序的主要罗，将耗时的操作推迟执行。  
回调函数的有点是简单、容易理解和部署，缺点是不利于代码的阅读和维护，各个部分之间高度耦合（coupling），流程会很混乱，而且每个任务只能指定一个回调函数。  
### [5.2 事件监听](./eventListener.html)
另一种思路是采用事件驱动模式。任务的执行不取决于代码的顺序，而取决于某个事件是否发生。
  
还是以f1和f2为例。首先，f1绑定一个事件，对f1进行改写：  
  
```
 	  
	let Event = {
		//通过on接口监听事件eventName
		//如果事件eventName被触发，则执行callback回调函数
		on:function(eventName,callback){
			//你的代码
                if(!this.handles){
                    //this.handles={};
                    Object.defineProperty(this, "handles", {
                        value: {},
                        enumerable: false,
                        configurable: true,
                        writable: true
                    })
                }

                if(!this.handles[eventName]){
                    this.handles[eventName]=[];
                }
                this.handles[eventName].push(callback);
		},
		//你的代码
        if(this.handles[arguments[0]]){
            for(var i=0;i<this.handles[arguments[0]].length;i++){
                this.handles[arguments[0]][i]();
            }
        }
	}
	  
  
	function f1() {
            console.log("执行f1");
            setTimeout(function () {
                //f1代码
                Event.emit("done");
            },5000)
        }

        function f2() {
            console.log("执行f2")
        }
        f1();
        Event.on("done",f2)
```

 Event.emit("done")表示触发Event对象的done事件，从而开始执行f2.  
这种方法的有点比较容易理解，可以绑定多个事件，每个事件可以指定多个回调函数，而且可以“去耦合”（Decoupling）,有利于实现模块化。缺点是整个程序都要变成事件驱动型，运行流程变得很不清晰。  
### [ 5.3 发布/订阅](./publishSubscribe.html)

上一节的"事件"，完全可以理解成"信号"。  
我们假定存在一个“信号中心”，某个任务完成，就向信号“发布”（publish)一个信号，其它任务可以向信号中心“订阅”（subscribe）这个信号，从而知道什么时候自己可以开始执行。这就叫做“发布/订阅模式”（publish-subsribe pattern）,又称“观察者模式”（observer pattern）。  
首先，f2向信息中心订阅“add”信号。  

```    

	var ob = new Pubsub(); 
	ob.subscribe("add",f2);
```    

通过f1发布这个“add”信号：  
  
```    

	function f1(){
		ob.pushlish("add")
	}
```  

f2执行完成后，也可以取消订阅（unsubscribe）:   
  
```  
  
	ob.unsubscribe("add",f2);
```
  
这种方法的性质与“事件监听”类似，但是明显优于后者。因为我们可以通过“消息中心”了解存在多少信号、每个信号中心有多少订阅者，从而监控程序的运行。  
### 5.4 Promise 对象，Generator 函数，async 函数
  通过Promise对象（构造函数）和Generator函数、async函数可以实现异步链式操作，后面的任务执行取决于前面一个任务的执行结果。   
Promise 示例：  
 
```  
  
	const promise = new Promise(function(resolve, reject) {
	  // ... some code
	
	  if (/* 异步操作成功 */){
	    resolve(value);
	  } else {
	    reject(error);
	  }
	});
	promise.then(function(res){
		//fullfiled状态代码
	}).catch(function(err){
		//rejected 状态代码
	})
```  
Generator函数示例：   
  
```    

	  function* helloWorldGenerator() {
		  yield 'hello';
		  yield 'world';
		  return 'ending';
		}
		
		var hw = helloWorldGenerator();


		hw.next()
		// { value: 'hello', done: false }
		
		hw.next()
		// { value: 'world', done: false }
		
		hw.next()
		// { value: 'ending', done: true }
		
		hw.next()
		// { value: undefined, done: true }

```  
async 函数示例：  
  
```    

	async function main() {
	  try {
	    const val1 = await firstStep();
	    const val2 = await secondStep(val1);
	    const val3 = await thirdStep(val1, val2);
	
	    console.log('Final: ', val3);
	  }
	  catch (err) {
	    console.error(err);
	  }
	}  
```  
## 6、 定时器  
除了放置异步任务的事件，"任务队列"还可以放置定时事件，即指定某些代码在多少时间之后执行。这叫做"定时器"（timer）功能，也就是定时执行的代码。

定时器功能主要由setTimeout()和setInterval()这两个函数来完成，它们的内部运行机制完全一样，区别在于前者指定的代码是一次性执行，后者则为反复执行。以下主要讨论setTimeout()。

setTimeout()接受两个参数，第一个是回调函数，第二个是推迟执行的毫秒数。   
   
```    

  	console.log(1);
	setTimeout(function(){console.log(2);},1000);
	console.log(3);
```  
  
上面代码的执行结果是1，3，2，因为setTimeout()将第二行推迟到1000毫秒之后执行。

如果将setTimeout()的第二个参数设为0，就表示当前代码执行完（执行栈清空）以后，立即执行（0毫秒间隔）指定的回调函数。  

    setTimeout(function(){console.log(1);}, 0);
    console.log(2);

上面代码的执行结果总是2，1，因为只有在执行完第二行以后，系统才会去执行"任务队列"中的回调函数。

总之，setTimeout(fn,0)的含义是，指定某个任务在主线程最早可得的空闲时间执行，也就是说，尽可能早得执行。**它在"任务队列"的尾部添加一个事件，因此要等到同步任务和"任务队列"现有的事件都处理完，才会得到执行。**  
HTML5标准规定了setTimeout()的第二个参数的最小值（最短间隔），不得低于4毫秒，如果低于这个值，就会自动增加。在此之前，老版本的浏览器都将最短间隔设为10毫秒。另外，对于那些DOM的变动（尤其是涉及页面重新渲染的部分），通常不会立即执行，而是每16毫秒执行一次。这时使用requestAnimationFrame()的效果要好于setTimeout()。

需要注意的是，setTimeout()只是将事件插入了"任务队列"，必须等到当前代码（执行栈）执行完，主线程才会去执行它指定的回调函数。要是当前代码耗时很长，有可能要等很久，所以并没有办法保证，回调函数一定会在setTimeout()指定的时间执行。  
## 7、Event Loop
主线程从"任务队列"中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为Event Loop（事件循环）。

为了更好地理解Event Loop，请看下图（转引自Philip Roberts的演讲《Help, I'm stuck in an event-loop》）。

![](http://www.ruanyifeng.com/blogimg/asset/2014/bg2014100802.png)

上图中，主线程运行的时候，[产生堆（heap）和栈（stack）](https://www.cnblogs.com/mydia/p/6713671.html)，栈中的代码调用各种外部API，它们在"任务队列"中加入各种事件（click，load，done）。只要栈中的代码执行完毕，主线程就会去读取"任务队列"，依次执行那些事件所对应的回调函数。

*堆heap与栈stack基本是所有程序语言中都带的，它将数据分配到内存空间汇总来完成各种调用（当然了，内存里除了heap和stack还有常量池）*  
*为什么要有heap和stack?*  
*为什么要有栈内存和堆内存之分？*  
*通常于垃圾回收机制有关。为了程序运行时占用的内存最小化。*   
*当一个方法执行时，每个方法都会建立自己的`内存栈`，在这个方法内定义的变量将会放入这块内存里，随着方法的执行结束，这个方法的内存栈也将自然销毁。因此，所有在方法内定义的变量都是放在栈内存中的。*    
*当我们在程序中建立一个对象时，这个对象被保存到运行时数据区中，以便反复利用（因为对象创建成本 通常较大），这个运行时数据区就是`堆内存`。堆内存中的对象不会随方法的结束而结束，即便方法结束后，这个对象还可能被另一个引用变量所引用（方法参数传递时很常见），则这个对象依然不会被销毁，只有当一个对象没有任何引用变量引用它时，系统的垃圾回收机制才会在核实的时候收回它。*  
*栈中存储的是基础变量已经对象的引用变量，基础变量的值是存储在栈内的，而引用变量存储在占中是指向堆中的数组或者对象的地址，这就是为何修改引用类型总会影响到其它指向这个地址的引用变量*


执行栈中的代码（同步任务），总是在读取"任务队列"（异步任务）之前执行。请看下面这个例子。


    var req = new XMLHttpRequest();
    req.open('GET', url);    
    req.onload = function (){};    
    req.onerror = function (){};    
    req.send();
上面代码中的req.send方法是Ajax操作向服务器发送数据，它是一个异步任务，意味着只有当前脚本的所有代码执行完，系统才会去读取"任务队列"。所以，它与下面的写法等价。


    var req = new XMLHttpRequest();
    req.open('GET', url);
    req.send();
    req.onload = function (){};    
    req.onerror = function (){};   
也就是说，指定回调函数的部分（onload和onerror），在send()方法的前面或后面无关紧要，因为它们属于执行栈的一部分，系统总是执行完它们，才会去读取"任务队列"。     

  
由以上原理可以得出代码执行顺序原则：     

**同步代码 => 异步代码 => 定时器**

