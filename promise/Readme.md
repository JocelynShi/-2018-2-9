# 周报2018.3.14

*二月简单学习了`promise`，`generator`和`sync`的简单用法，现在有时间，先再深入的学习一下`Promise`，之后有时间再深入学习`generator`和`sync`*


### promise对象

**Promise**是一个对象，它存储着每个未来结束的事件（通常是一个异步操作）的结果。Promise提供统一的API,各种异步操作都可以用同样的方法进行处理。

## 1、Promise的两大特点：

1. 对象的状态不受外界影响。它有三种状态：Pending,fulfilled和rejected.只有异步操作的结果可以决定当前是哪一种状态，任何其它操作都无法改变这个状态。这也应了它的名字“Promise”,承诺，不被外界因素影响说好的结果。
2. 状态一旦改变，就不会再变。Promise对象的状态改变，只有两种：从`pending`到`fulfilled`和从`pending`到`rejected`。只要这两种情况发生，状态就凝固了，不会再变变了，这时称为resolved(已定型)。这时再对Promise添加回调函数，也会立即得到这个结果。

### Promise的缺点

1. 一旦建立就会立即执行，无法中途取消。
2. 如果不设置回调函数，promise内部抛出错误，不会反应到外部。
3. 当处于pending状态时，无法得知目前进展到哪一个阶段。

## 2、Promise的基本用法：

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


### 2.1 [Promise 新建之后就会立即执行](./promise.html)。
```
    let promise = new Promise(function(resolve, reject) {
      console.log('Promise');
      resolve();
    });

    promise.then(function() {
      console.log('resolved.');
    });

    console.log('Hi!');

    // Promise  //Promise 新建后就立即执行
    // Hi!
    // resolved    //Promise.resolve()在本轮"事件循环"结束时最后执行，所以在console.log("Hi")之后执行。

 ```
### 2.2 [Promise包装一个图片异步加载：](./loadImage.html)
```
function loadImageAsync(url) {
  return new Promise(function(resolve, reject) {
    const image = new Image();

    image.onload = function() {
      resolve(image);
    };

    image.onerror = function() {
      reject(new Error('Could not load image at ' + url));
    };

    image.src = url;
  });
}

```
如果图片加载成功，执行resolve，如果失败则执行reject.

### 2.3 [Promise 对象实现Ajax操作](./promiseAjax.html)：


	 const getJSON = function (url) {
            const promise = new Promise(function (resolve,reject) {
                const handler = function () {
                    if(this.readyState !==4){
                        return;
                    }
                    if(this.status ===200){
                        resolve(this.response);
                    }else{
                        reject(new Error(this.statusText))
                    }
                };

                const client = new XMLHttpRequest();
                client.open("GET",url);
                client.onreadystatechange=handler;
                client.responseType="json";
                client.setRequestHeader("Accept","application/json");
                client.send();

            });
            return promise;
        };
        getJSON("js/posts.json").then(function (json) {
            console.log('content:'+ json);
            let p = new Promise(function (resolve,reject) {
                setTimeout(function () {
                    resolve("第二个Promise完成")
                },5000)
            })
            return p;
        },function (error) {
                console.error("出错了",error);
                let p = new Promise(function (resolve,reject) {
                    setTimeout(function () {
                        resolve("第二个Promise完成")
                    },5000)
                })
            return p;
        }).finally(function () {
            console.log("completed")
        })

面代码中，getJSON是对 XMLHttpRequest 对象的封装，用于发出一个针对 JSON 数据的 HTTP 请求，并且返回一个Promise对象。需要注意的是，在getJSON内部，resolve函数和reject函数调用时，都带有参数。

如果调用resolve函数和reject函数时带有参数，那么它们的参数会被传递给回调函数。reject函数的参数通常是Error对象的实例，表示抛出的错误；
注意：如果then里不return Promise对象实例，后面的then或者finally将会按照同步顺序执行，如果有返回Promise状态，则会等待状态变化后再执行。   
  
### 2.4 [resolve函数的参数除了正常的值外，还可能是另外一个Promise实例](./resolve(promise).html)：

    const p1 = new Promise(function (resolve, reject) {
        console.log("p1")
        setTimeout(() => reject(new Error('failed')), 3000)
    })

    const p2 = new Promise(function (resolve, reject) {
        console.log('p2')
        setTimeout(() => resolve(p1), 1000) //由于p2返回的是另一个Promise，
        //导致p2自己的状态无效了，由p1的状态决定p2的态度。所以，后面的then
        //语句都变成针对后者(p1)。又过了2秒，p1变为reject,导致触发catch方法指定的回调函数、
    })

    p2
        .then(result => console.log(' 成功了'))
        .catch(error => console.log(error))
      

#### 注意，调用resolve或reject并不会终结 Promise 的参数函数的执行。

	new Promise((resolve, reject) => {
	resolve(1);
	console.log(2);
	}).then(r => {
	console.log(r);
	});
	// 2
	// 1

上面代码中，调用resolve(1)以后，后面的console.log(2)还是会执行，并且会首先打印出来。这是因为立即 resolved 的 Promise 是在本轮事件循环的末尾执行，总是晚于本轮循环的同步任务。

一般来说，调用resolve或reject以后，Promise 的使命就完成了，后继操作应该放到then方法里面，而不应该直接写在resolve或reject的后面。所以，最好在它们前面加上return语句，这样就不会有意外。   

	new Promise((resolve, reject) => {
	  return resolve(1);
	  // 后面的语句不会执行
	  console.log(2);
	})


## 3、 [Promise.prototype.then()](./promise.then.html)
Promise实例具有`then`方法，也就是说，`then`方法定义在原型对象Promise.prototype上的。它的作用是为Promise实例添加状态改变时的回调函数。前面说过，`then`方法的第一个参数是	`resolved`状态的回调函数，第二个参数（可选）是`rejected`状态的回调函数。   
`then`方法返回的是一个新的`Promise`实例（不是原来那个Promise实例）。因此可以采用链式写法，即`then`方法后面再调用一个`then`方法。


	let p= new Promise(function (resolve,reject) {
	    if(1===1){
	        resolve("ok");
	    }else{
	        reject(new Error("wrong"));
	    }
	});
	p.then(function (data) {
	   console.log("第一个then函数参数:",data);
	   return "第一个then函数执行完毕";
	}).then(function (data) {
	   console.log("第二个then函数参数:",data);
	    return new Promise(function (reslove,reject) {
	        setTimeout(function () {
	           if(2===2){
	               reslove("第二个then函数执行成功");
	
	           }else{
	                reject(new Error("出错了"))
	           }
	       },5000);
	    })
	
	}).then(function (data) {
	   console.log("第三个then函数参数:",data)
	},function (error) {
	   console.log("第三个then函数参数",error)
	})
	//第一个then函数参数: ok
	//第二个then函数参数: 第一个then函数执行完毕
	//第三个then函数参数: 第二个then函数执行成功 (这个大约是5秒后打印的)


以上的代码使用`then`方法，依次指定了3个回调函数。第一个回调函数执行完成后，会讲return返回的结果作为参数，传入第二个回调函数。第二个回调函数又新建了一个Promise，并且resolve/reject传出新的结果,所以第三个then函数要等第二个then回调函数返回的Promise对象的状态发生变化，才会被调用，而且接收resolve/reject传出的结果作为参数。   
  
## 4、[Promise.prototype.catch()](./promise.catch.html)
Promise.prototype.catch方法是,then(null,rejection)的别名，用于指定发生错误时的回调函数。

	const p = new Promise(function (resolve,reject) {
	            if(1 === 2){
	                resolve("ok")
	            }else{
	                reject(new Error("出错了"))
	            }
	        });
	        p.then(function (data) {
	            console.log(data);
	        }).catch(function (err) {
	            console.error(err);
	        })

注意：`promise`函数参数运行中抛出错误和`then`方法指定的回调函数运行中抛出错误，也都会被`catch`方法捕获。

	const promise = new Promise(function(resolve, reject) {
	  throw new Error('test');
	});
	promise.catch(function(error) {
	  console.log(error);
	});
	// Error: test

由于Promise的状态一旦改变，就永久保持改状态，以下抛出的错误就会被无视：

	const promise = new Promise(function(resolve, reject) {
	  resolve('ok');
	  throw new Error('test');
	});
	promise
	  .then(function(value) { console.log(value) })
	  .catch(function(error) { console.log(error) });
	// ok

Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个catch语句捕获。

  
	const p2 = new Promise(function (resolve, reject) {
        if (1 === 1) {
            resolve("ok")
        } else {
            reject(new Error("出错了"))
        }
    });
    p2.then(function (data) {
        let c = x + y;
        console.log(data);
    }).then(function (data) {
        let a = b + 1;
    }).catch(function (err) { //一旦有抛出错误，.catch就会被立即执行
        console.log(err);   //如果这里没有.catch，错误就得不到处理了
    })

所以，一般来说，不要在`then`方法里定义Reject状态的回调函数（即`then`的第二个参数），总是使用`catch`的方法捕获错误。

## 5、[ Promise.prototype.finally()](./promise.finally.html)
`finally`方法用于指定管Promise对象最后的状态如果，都会执行的操作。该方法是ES2018引入标准的。   

	promise
	.then(result => {···})
	.catch(error => {···})
	.finally(() => {···});

## 6、[Promise.all();](./promise.all.html)
Promise.all方法用于将多个Promise实例包装成一个新的Promise实例对象。

	 const p = Promise.all([p1,p2,p3]);
       p.then(function (data) {
          // console.log(data); //当p1,p2,p3的状态太都变成resolved之后才会执行
       },function (err) {
           console.log("error"); //当数组p1,p2,p3任何一个执行reject时，就会把这个状态传给p.但是p1，p2,p3都会执行完毕。
       })

注意，如果作为参数的Promise实例，自己定义了catch方法，那么它一旦被`rejected`,并不会触发Promise.all()的catch方法。
   

	const p1 = new Promise((resolve, reject) => {
	  resolve('hello');
	})
	.then(result => result)
	.catch(e => e);
	
	const p2 = new Promise((resolve, reject) => {
	  throw new Error('报错了');
	})
	.then(result => result)
	.catch(e => e);
	
	Promise.all([p1, p2])
	.then(result => console.log(result))
	.catch(e => console.log(e));
	// ["hello", Error: 报错了]

在上面的代码中，p1会`resolved`，p2首先会`rejected`,但是p2有自己的`catch`方法，该方法返回的是一个新的`Promise`实例，p2指向的实际上是这个实例。该实例执行完`catch`方法后，也会变成`resolved`,导致`Promise.all()`方法参数里面的两个实例都会`resolved`,因此会调用`then`方法指定的回调函数，而不会调用`catch`方法指定的回调函数。  
当然，如果p2没有自己的`catch`方法，就会调用`Promise.all()`的`catch`方法。
  
  
## 7、 [Promise.race()](./promise.race.html)
Promise.race方法同样是将多个Promise实例包装成一个新的Promise实例。
和Promise.all的区别是，只要数组里面的一个实例的状态发生改变，新Promise实例的状态就会跟着改变。那个率先改变的Promise实例返回的值，就传递给新实例的回调函数。

## 8、Promise.resolve()
有时需要将现有对象转为Promise对象，Promise.resolve方法就起到这个作用。

	const jsPromise = Promise.resolve($.ajax('/whatever.json'));
上面代码将jQuery 生成的deferred对象转为一个新的Promise对象。   


Promise.resolve等价于下面的写法：  
```  

	Promise.resolve('foo');
	//等价于
	new Promise(resolve => resolve('foo'));  

```   

Promise.resolve方法的参数分成四种情况。
####（1）参数一个Promise实例：
如果参数是Promise实例，那么Promise.resolve将不做任何修改，原封不动的返回这个实例。
#### （2） 参数是一个thenable对象
thenable 对象指的是具有then方法的对象，比如下面这个对象。  

		let thenable = {
			 then: function(resolve, reject) {
	    		resolve(42);
	 		 }
			}
Promise.resolve方法会将这个对象转为 Promise 对象，然后就立即执行thenable对象的then方法。    


		let thenable = {
			  then: function(resolve, reject) {
			    resolve(42);
			  }
			};
			
			let p1 = Promise.resolve(thenable);
			p1.then(function(value) {
			  console.log(value);  // 42
			});

在上面代码中，thenable对象的then方法执行后，对象p1的状态就变为resolved，从而立即执行最后那个then方法指定的回调函数，输出42.  
#### (3) 参数不具有then方法的对象，或者根本不是对象  
如果参数是一个原始值，或者是一个不具有then方法的对象，则Promise.resolve方法返回一个新的 Promise 对象，状态为resolved。  

	const p = Promise.resolve('Hello');

		p.then(function (s){
		  console.log(s)
		});
		// Hello

上面代码生成一个新的 Promise 对象的实例p。由于字符串Hello不属于异步操作（判断方法是字符串对象不具有 then 方法），返回 Promise 实例的状态从一生成就是resolved，所以回调函数会立即执行。Promise.resolve方法的参数，会同时传给回调函数。  
#### (4) 不带有任何参数
Promise.resolve方法允许调用时不带参数，直接返回一个resolved状态的 Promise 对象。  

	const p = Promise.resolve();
	p.then(function(){})

需要注意的是，立即resolve的 Promise 对象，是在本轮“事件循环”（event loop）的结束时，而不是在下一轮“事件循环”的开始时。  
```

	 setTimeout(function () {
	  console.log('three');
	}, 0);
	
	Promise.resolve().then(function () {
	  console.log('two');
	});
	
	console.log('one');
	
	// one
	// two
	// three
```
上面代码中，setTimeout(fn, 0)在下一轮“事件循环”开始时执行，Promise.resolve()在本轮“事件循环”结束时执行，console.log('one')则是立即执行，因此最先输出。
*我觉得好像是因为有then，才回在本轮“时间循环”的最后才执行的。*



## 9、[Generator 函数与 Promise 的结合](./promise.generator.html)


使用 Generator 函数管理流程，遇到异步操作的时候，通常返回一个Promise对象。  
  
```

	function getFoo () {
	  return new Promise(function (resolve, reject){
	    resolve('foo');
	  });
	}
	
	const g = function* () {
	  try {
	    const foo = yield getFoo();
	    console.log(foo);
	  } catch (e) {
	    console.log(e);
	  }
	};
	
	function run (generator) {
	  const it = generator();
	
	  function go(result) {
	    if (result.done) return result.value;
	
	    return result.value.then(function (value) {
	      return go(it.next(value));
	    }, function (error) {
	      return go(it.throw(error));
	    });
	  }
	
	  go(it.next());
	}
	
	run(g);

```

上面代码的`Generator`函数g之中，有一个一步操作`getFoo`，它返回的就是一个`Promise`对象。函数run用来处理这个`Promise`对象，并调用下一个`next`方法。
