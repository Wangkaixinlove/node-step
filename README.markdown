# Step 步

A simple control-flow library for node.JS that makes parallel execution, serial execution, and error handling painless.  
一个简单的node.JS控制流库，可以实现并行执行，串行执行和错误处理.

## How to install 如何安装

Simply copy or link the lib/step.js file into your `$HOME/.node_libraries` folder.  
只需将lib / step.js文件复制或链接到您的$HOME/.node_libraries文件夹即可。

## How to use 如何使用

The step library exports a single function I call `Step`.  It accepts any number of functions as arguments and runs them in serial order using the passed in `this` context as the callback to the next step.  
步骤库导出一个我调用的函数Step。它接受任意数量的函数作为参数，并使用传入的this上下文以串行顺序运行它们作为下一步的回调。
```
    Step(
      function readSelf() {
        fs.readFile(__filename, this);
      },
      function capitalize(err, text) {
        if (err) throw err;
        return text.toUpperCase();
      },
      function showIt(err, newText) {
        if (err) throw err;
        console.log(newText);
      }
    );
```
Notice that we pass in `this` as the callback to `fs.readFile`.  When the file read completes, step will send the result as the arguments to the next function in the chain.  Then in the `capitalize` function we're doing synchronous work so we can simple return the new value and Step will route it as if we called the callback.   
  
注意我们this作为回调传入fs.readFile。当文件读取完成时，step将把结果作为参数发送给链中的下一个函数。然后在capitalize函数中，我们正在做同步工作，所以我们可以简单的返回新的值，Step会调用回调。  
  
The first parameter is reserved for errors since this is the node standard.  Also any exceptions thrown are caught and passed as the first argument to the next function.  As long as you don't nest callback functions inline your main functions this prevents there from ever being any uncaught exceptions.  This is very important for long running node.JS servers since a single uncaught exception can bring the whole server down.   
  
第一个参数是为错误保留的，因为这是节点标准。此外，任何抛出的异常都会作为下一个函数的第一个参数被捕获并传递。这防止从来没有任何未捕获的异常。这对长时间运行的node.JS服务器非常重要，因为单个未捕获的异常可能会导致整个服务器停机。
Also there is support for parallel actions:  
也支持并行操作：
```
    Step(
      // Loads two files in parallel
      function loadStuff() {
        fs.readFile(__filename, this.parallel());
        fs.readFile("/etc/passwd", this.parallel());
      },
      // Show the result when done
      function showStuff(err, code, users) {
        if (err) throw err;
        console.log(code);
        console.log(users);
      }
    )
```
Here we pass `this.parallel()` instead of `this` as the callback.  It internally keeps track of the number of callbacks issued and preserves their order then giving the result to the next step after all have finished.  If there is an error in any of the parallel actions, it will be passed as the first argument to the next step.  
  
在这里，我们通过this.parallel()而不是this作为回调。它在内部跟踪发出的回调数量并保留它们的顺序，然后在结束之后将结果提供给下一步。如果在任何并行操作中有错误，它将作为第一个参数传递给下一步。  
  
Also you can use group with a dynamic number of common tasks.  
您也可以使用动态数量的常见任务组。
```
    Step(
      function readDir() {
        fs.readdir(__dirname, this);
      },
      function readFiles(err, results) {
        if (err) throw err;
        // Create a new group 
        var group = this.group();
        results.forEach(function (filename) {
          if (/\.js$/.test(filename)) {
            fs.readFile(__dirname + "/" + filename, 'utf8', group());
          }
        });
      },
      function showAll(err , files) {
        if (err) throw err;
        console.dir(files);
      }
    );
```
*Note* that we both call `this.group()` and `group()`.  The first reserves a slot in the parameters of the next step, then calling `group()` generates the individual callbacks and increments the internal counter.   
*注意* `this.group()`和`group()`。第一个在下一步的参数中保留一个数组，然后调用group()生成单独的回调并递增内部计数器，将结果依次保存。
