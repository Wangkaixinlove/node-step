# Step.js
### 1.	介绍Introduction  
它一个简单的node.JS控制流库，可以实现并行执行，串行执行和错误处理。
首先，在讲解之前我们需要先来了解一些知识。Step.js是控制流程工具，
解决回调嵌套层次过多等问题。适用于读文件等回调函数相互依赖，或者分别获取内容最后组合数据返回等应用情景。异步执行简单地可以分为“串行执行”和“并行”执行，下面我们分别去看看。
### 2.	安装 install  
将lib/step.js复制到我的$Home/node-demo/18-bigwork/lib中。  
### 3.	使用 using  
#### 串行执行：
这个库只有一个方法 Step(fn,fn,fn…)。Step 方法其参数允许是多个回调函数，这些函数被依次执行。
```
Step(  
    function readSelf() {  
      fs.readFile(__filename, this); // 把 this 送入 readFile 的异步参数中。此时 this 其类型为 function 相当于调用next()                                                          // 将读文件的结果传给results传给text
                                     // 注意这里无须 return 任何值  
     },  
    function capitalize(err, text) { // err 为错误信息，如果有则抛出异常  
      if (err) throw err;  
      return text.toUpperCase();     // text 为 上个步骤 readFile 的值，也就是文件内容。注意此处有返回值。  
     },  
    function showIt(err, newText) {  
      if (err) throw err;  
      console.log(newText);  
    }  
);  
```  
*需要注意的点：*
  回调函数顺序依赖、依次执行，下一个回调函数依赖于上一个函数执行的结果，例如当文件读取完成时，step将把结果作为参数发送给链中的下一个函数。
Step 的回调函数的第一个参数总是 err，第二个才是值。如果上一个步骤发生异常，那么异常对象将被送入到下一个步骤中。因此我们应该检查 err 是否有意义，如果 err 存在值，应抛出异常用于处理错误，中止下面的执行；如果 err 为 null 或者 undefined，则表示第二个参数才有意义。
为什么要大家注意有 return 和无 return 的区别呢？ Step 的一个思想：只要是同步的工作像上例中的第二个fn，我们只需直接 return 即可。如果需要异步的话像第一个fn则调用对象的this，此时 this 是一个函数类型的对象。
像上例一个 fn 接着一个 fn 执行，称为“串行执行”。  

#### 并行执行：
所谓并行执行，就是等待所有回调产生的结果，并把所有结果results按调用次序作为参数传给最后一个函数处理。此方法不同于上面的串行执行，所有的回调是并行执行的， 只不过是在最后一个回调返回结果时，才调用最终处理函数。
#### 确定数目的任务
```
	Step(  
	  // 同时执行两项任务
	  function loadStuff() {  
	    fs.readFile(__filename, this.parallel());  
	    fs.readFile("/etc/passwd", this.parallel());  
	  },  
	  // 获取结果
	  function showStuff(err, code, users) {  
	    if (err) throw err;  
	    console.log(code);  
	    console.log(users);  
	  }  
	)  
```
我们用 Step.js API 提供的 this.parallel() 方法代替了 上一例的 this。this.parallel() 可以帮助解决我们上一例中只能产生一次异步函数的问题，适合多个异步步骤，
由最后一个回调收集结果数据。调用了 n 次的 this.parallel()，就调用了 n 次的收集 results 的函数。如果任何一个异步分支发生异常，那个异常都会被收集到 results 中，
并且特别地声明到 err 参数中，表示任务失败，最后一个回调不执行。  
#### 不确定数目的任务
	Step(  
	  function readDir() {  //读取目录
	    fs.readdir(__dirname, this); 
	  },  
	  function readFiles(err, results) {  //读取目录下的文件
	    if (err) throw err;  
	    // 创建 group，其实内部创建一个 results 数组，保存结果  
	    var group = this.group();  
	    results.forEach(function (filename) {  //遍历resulets读取*.js文件到group()中
	      if (/\.js$/.test(filename)) {  
	        fs.readFile(__dirname + "/" + filename, 'utf8', group()); //将读取结果存到files中
	      }  
	    });  
	  },  
	  function showAll(err , files) {  //显示文件名,files是一个数组
	    if (err) throw err;  
	    console.dir(files);  
	  }  
	);  
this.group 与 this.parallel都是用于异步场景，区别在于 this.parallel 会把结果作为一个个单独的参数来提供，而 this.group 会将结果合并为数组 Array。两者的真正区别是 thsi.group 用于不太确定异步任务数量的场景如上例读取文件并不知道文件数目。
由此可见，Step.js 可以很好地结合同步与异步的任务组织技术。  
*总结：*
- 既不使用 this，又无 return 任何值，任务中断
- 直接 return 值，此时无任何异步过程发生，同步
- 调用 this : Function，产生一个异步过程，此异步过程完成后，进入下一个 step
- 用 this.parallel() : Function，产生多个异步过程，等待这些异步过程通通完成之后，才进入下一个 step
-	如果不确定多个异步任务的数量，这时结果参数数目不确定，可以使用 this.group()。

### 4.源代码解读  
```
/ *
Copyright（c）2011 Tim Caswell <tim@creationix.com>
MIT许可证
特此免费授予任何获得副本的人
这个软件和相关的文档文件（“软件”）来处理
在软件中没有限制，包括但不限于权利
使用，复制，修改，合并，发布，分发，再许可和/或销售
该软件的副本，并允许软件所属的人员
提供这样做，但须符合以下条件：
上述版权声明和本许可声明应包含在内
本软件的副本或实质性部分。
本软件按“原样”提供，不附有任何形式的明示或暗示保证
默示的，包括但不限于对适销性的保证，
适用于特定目的和不侵权。在任何情况下，
作者或版权持有者对任何索赔，损坏或其他责任均不负任何责任
责任，无论是在合同，民事侵权行为或其他方面，
与本软件或本软件的使用或其他交易有关或与之有关
软件。
* /

function Step() {  
	  var steps = Array.prototype.slice.call(arguments), // 参数列表  
	      counter, pending, results, lock;               // 全局的四个变量  
	  
	  // 定义主函数
	  function next() {  
	    counter = pending = 0; // counter 和 pendig 是用于并行步骤任务：保存数据的索引和是否执行完毕的计数器  
	  
	    // 看看是否还有剩余的 steps 
	    if (steps.length === 0) {  
	      // 抛出未捕获的异常   
	      if (arguments[0]) {  //有错就抛出
	        throw arguments[0];  
	      }  
	      return;  
	    }  

	  
	    // 得到要执行的步骤
	    var fn = steps.shift(); //执行完的就删除 
            results = [];  
	  
	    // try...catch 捕获异常 
            try {  
	      lock = true;  //开始执行开锁
	      var result = fn.apply(next, arguments); //  将fn执行的结果保存到result,arguments是 next 的参数列表  
	    } catch (e) {  
	      // 如果有异常，把捕捉到的异常送入下一个回调
              next(e);  
	    }  
	  
	    if (counter > 0 && pending == 0) { // couter > 0 表示有并行任务，pending == 0 表示全部运行完毕，如果执行了并行任务（异步的意思）,                                                //而且全部分支都同步执行完毕后，立刻执行下一步  
	      next.apply(null, results); // 注意是 results 数组。此时 results 已经包含了全部的结果  window.next(results)
	    } 
	    else if (result !== undefined) {  // 如发现有 return 执行（串行的意思），将其送入到回调 
	      next(undefined, result);  //第一个参数err，第二个参数是上一个函数的执行结果
	    }  
	    lock = false;  //执行完上锁
	  }  
	  
	  // 用于确定数目并行的生成器
	  next.parallel = function () {  
	    var index = 1 + counter++;  //因为第一个永远给err，所以要从1开始
	    pending++; // 开启了一个新的异步任务  
	    //异步任务完成时返回
	    return function () {  
	      pending--;// 计算器减 1，表示执行完毕  
            if (arguments[0]) {  //err,如果有错误，则保存在结果数组的第一个元素中
	        results[0] = arguments[0];  
	      }  
	      results[index] = arguments[1];   // 按次序保存结果  
	      if (!lock && pending === 0) {// 最后才执行,当所有分支都搞定，执行最后一个回调。
	        next.apply(null, results);  //相当于window.next(results)，执行下一个函数,将results数组传入
	      }  
	    };  
	  };  
	  
	  //用于不确定数目并行的生成器
	  next.group = function () {  
	    var localCallback = next.parallel(); //下一个异步任务 
	    var counter = 0;  //保存数据的索引
	    var pending = 0;  //计数器
	    var result = [];  
	    var error = undefined;  
	  
	    function check() {  
	      if (pending === 0) {  
	        //当组完成时，调用回调函数，将result数组传给下一个任务
	        localCallback(error, result);  
	      }  
	    }  
	    process.nextTick(check); // 确保检查至少被调用一次，效果是将一个函数推迟到代码书写的下一个异步方法的事件回调函数开始执行时；与                                            //setTimeout(fn, 0) 函数的功能类似，但它的效率高多了。 
	    // 返回一个函数
	    return function () {  
	      var index = counter++;  
	      pending++;  
	      return function () {  
	        pending--;  
	        // 如果有错误，则保存在结果数组的第一个元素中 
	        if (arguments[0]) {  
	          error = arguments[0];  
	        }  
	       // 按次序保存结果 
	        result[index] = arguments[1];  
	        if (!lock) { check(); }  //锁住了就检查任务是否全部完成
	      };  
	    };  
	  };  
	  
	  // 开始工作流，调用next   
	  next();  
	}  
	  
	/**
	*方法说明:一个包装步骤列表的工厂 ,读取多个文件的时候，可以将公用的函数包装成工厂，不用每次都传参（函数书写麻烦）
	* @method  StepFn
	* @return {function} 
	*/  
	Step.fn = function StepFn() {  
	  var steps = Array.prototype.slice.call(arguments);  //step.fn的参数数组
	  return function () {  
	    var args = Array.prototype.slice.call(arguments);  //myfn调用时的参数
	  
	    //  将steps连接到toRun数组中,toRun数组所有的元素都是函数
	    var toRun = [function () {  
	      this.apply(null, args);  //相当于调用next()
	    }].concat(steps);  
	  
	    //  如果args最后一个参数是函数，就加到toRun中，并在args中删除
	    if (typeof args[args.length-1] === 'function') {  
	      toRun.push(args.pop());  
	    }  
	  Step.apply(null, toRun);  // call和apply的第一个参数是null/undefined时函数内的的this指向global，全局调用step()
	
	  }  
	}  
	  
  
	//暴露Step
	if (typeof module !== 'undefined' && "exports" in module) {  
	  module.exports = Step;  
	}  
```  
### 5.测试


















