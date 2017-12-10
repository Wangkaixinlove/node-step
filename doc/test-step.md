### 1.	callbackTest.js
```
require('./helper');
var selfText = fs.readFileSync(__filename, 'utf8');//通过同步方式读取文件
expect('one');
expect('two');
expect('three');
Step(
  function readSelf() {
fulfill("one");//清除错误，如果没有清除就传入下个函数的err
fs.readFile(__filename, 'utf8', this); //异步方式读取文件
  },
  function capitalize(err, text) {
    fulfill("two");
    if (err) throw err;//如果有错误就抛出
    assert.equal(selfText, text, "Text Loaded");//assert.equal(实际值，预期值，错误提示)
    return text.toUpperCase();//返回值直接当参数传导下一个函数中
  },
  function showIt(err, newText) {
fulfill("three");
//console.log(newText);
    if (err) throw err;
    assert.equal(selfText.toUpperCase(), newText, "Text Uppercased");
  }
);
```
执行效果  
 ![image](https://github.com/Wangkaixinlove/node-step/blob/master/doc/imgs/callbackTest.jpg?raw=true)
### 2.	helper.js
```
global.assert = require('assert');//引入断言模块
global.fs = require('fs');//引入文件模块
global.Step = require('../lib/step');//引入step

var expectations = {};//存储错误的对象
global.expect = function expect(message) {//创建错误
  expectations[message] = new Error("Missing expectation: " + message);
  //console.log(expectations);
}
global.fulfill = function fulfill(message) {
  delete expectations[message];//清除错误
  //console.log(expectations);
}
process.addListener('exit', function () {
  Object.keys(expectations).forEach(function (message) {
    throw expectations[message];//程序退出时如果错误对象中还有错误，就抛出
  });
});
```
### 3. errorTest.js
```
require('./helper');

var exception = new Error('Catch me!');//创建一个错误
//创建四个错误到错误对象中
expect('one');
expect('timeout');
expect('two');
expect('three');
Step(
  function () {
    fulfill('one');
    var callback = this;//function，指next
    setTimeout(function () {
      fulfill('timeout');
      callback(exception);//传一个err进去
    }, 0);
  },
  function (err) {
    fulfill('two');
    assert.equal(exception, err, "error should passed through");// exception与捕捉到的 err相同
    throw exception;
  },
  function (err) {
    fulfill('three');
assert.equal(exception, err, "error should be caught");  })；
```
执行效果     

 ![image](https://github.com/Wangkaixinlove/node-step/blob/master/doc/imgs/errorTest2.jpg?raw=true)
 ![image](https://github.com/Wangkaixinlove/node-step/blob/master/doc/imgs/errorTest3.jpg?raw=true)
### 4. fnTest.js 创建工厂
```
require('./helper');
var myfn = Step.fn(//定义myfn函数，它实际上是Step.fn的返回值函数
  function (name) {
    fs.readFile(name, 'utf8', this);
  },
  function capitalize(err, text) {
    if (err) throw err;
    return text.toUpperCase();¬
  }
);//这两个函数参数保存在steps中

var selfText = fs.readFileSync(__filename, 'utf8');

expect('result');
//调用myfn函数，参数保存到args中
myfn(__filename, function (err, result) {
  fulfill('result');
  if (err) throw err;
  assert.equal(selfText.toUpperCase(), result, "It should work");
});
```
 ![image](https://github.com/Wangkaixinlove/node-step/blob/master/doc/imgs/fnTest.jpg?raw=true)
 
 //Args,toRun(包含steps),toRun(包含steps,args的最后一个参数)
相当于把他们组成一个toRun函数数组，依次执行
不需要每次都输入三个函数，更改文件名即可

执行效果  
![image](https://github.com/Wangkaixinlove/node-step/blob/master/doc/imgs/fnTest1.jpg?raw=true)  
### 5. groupTest.js
```
require('./helper');

var dirListing = fs.readdirSync(__dirname),//同步方式读取当前目录
    dirResults = dirListing.map(function (filename) {
      return fs.readFileSync(__dirname + "/" + filename, 'utf8');
    });//map返回一个新数组，新数组成员是当前目录下全部的文件

expect('one');
expect('two');
expect('three');
Step(
  function readDir() {
    fulfill('one');
    fs.readdir(__dirname, this);//异步方式读取目录
  },
  function readFiles(err, results) {
    fulfill('two');
    if (err) throw err;
    assert.deepEqual(dirListing, results);// deepEqual()方法能够比较数组和json等数据，也能比较一般数据，能够进行更为深层次的数据比较。没有message不好
    var group = this.group();// 创建一个组
    results.forEach(function (filename) {//遍历results
      if (/\.js$/.test(filename)) {//判断是否是js文件
        fs.readFile(__dirname + "/" + filename, 'utf8', group());//将读取结果依次放入group的results中
      }
    });
  },
  function showAll(err , files) {//files就是results数组
    fulfill('three');
    if (err) throw err;
    assert.deepEqual(dirResults, files);
  }
);

expect('four');
expect('five');
//创建一个空组 
 Step(
  function start() {
    var group = this.group();
    fulfill('four');
  },
  function readFiles(err, results) {
    if (err) throw err;
    fulfill('five');
    assert.deepEqual(results, []);
  }
);
// 测试具有n个并行调用的组
expect("test3: 1");
expect("test3: 1,2,3");
expect("test3: 2");
Step(
    function() {
        return 1;
    },
    function makeGroup(err, num) {
        if(err) throw err;
        fulfill("test3: " + num);
        var group = this.group();
        
        setTimeout((function(callback) { return function() { callback(null, 1); } })(group()), 100);
        group()(null, 2);
        setTimeout((function(callback) { return function() { callback(null, 3); } })(group()), 0);
    },
    function groupResults(err, results) {
        if(err) throw err;
        fulfill("test3: " + results);
        return 2
    },
    function terminate(err, num) {
        if(err) throw err;
        fulfill("test3: " + num);
    }
);

// 测试0个大小的组
expect("test4: 1");
expect("test4: empty array");
expect("test4: group of zero terminated");
expect("test4: 2");
Step(
    function() {
        return 1;
    },
    function makeGroup(err, num) {
        if(err) throw err;
        fulfill("test4: " + num);
        this.group();//创建空组
    },
    function groupResults(err, results) {
        if(err) throw err;
        if(results.length === 0) { fulfill("test4: empty array"); }
        fulfill('test4: group of zero terminated');
        return 2
    },
    function terminate(err, num) {
        if(err) throw err;
        fulfill("test4: " + num);
    }
);
// 测试立即返回的并行调用的功能

expect("test5: 1,2");
expect("test5 t1: 666");
expect("test5 t2: 333");
setTimeout(function() {
Step(
  function parallelCalls() {
    var group = this.group();
    var p1 = group(), p2 = group();
    p1(null, 1);//this.group()(null,1);两个括号是因为返回值是函数
    p2(null, 2);//this.group()(null,2);
  },
  function parallelResults(err, results) {
    if(err) throw err;
    fulfill("test5: " + results);
    return 666;
  },
  function terminate1(err, num) {
    if(err) throw err;
    fulfill("test5 t1: " + num);
    var next = this;
    setTimeout(function() { next(null, 333); }, 50);
  },
  function terminate2(err, num) {
    if(err) throw err;
    fulfill("test5 t2: " + num);
    this();
  }
);
}, 1000);
```
执行效果  
 ![image](https://github.com/Wangkaixinlove/node-step/blob/master/doc/imgs/groupTest.jpg?raw=true)  
 ![image](https://github.com/Wangkaixinlove/node-step/blob/master/doc/imgs/errorTest1.jpg?raw=true)
 
###  6. parallelTest.js
```
require('./helper');

var selfText = fs.readFileSync(__filename, 'utf8'),//同步
    etcText = fs.readFileSync('/etc/passwd', 'utf8');

expect('one');
expect('two');
Step(
  //并行读取两个文件
  function loadStuff() {
    fulfill('one');
    fs.readFile(__filename, this.parallel());//异步，第二个参数是回调函数,没有用utf8格式
    fs.readFile("/etc/passwd", this.parallel());
  },
  // 显示结果
  function showStuff(err, code, users) {// this.parallel 会把结果作为一个个单独的参数来提供
    fulfill('two');
    if (err) throw err;
    assert.equal(selfText, code, "Code should come first");
    assert.equal(etcText, users, "Users should come second");
  }
);
//测试具有n个并行调用的功能
expect("test2: 1");
expect("test2: 1,2,3");
expect("test2: 2");
Step(
    function() {
        return 1;
    },
    function makeParallelCalls(err, num) {
        if(err) throw err;
        fulfill("test2: " + num);
        setTimeout((function(callback) { return function() { callback(null, 1); } })(this.parallel()), 1000);      
        this.parallel()(null, 2);
        setTimeout((function(callback) { return function() { callback(null, 3); } })(this.parallel()), 0);//为什么每次一定按顺序返回？
    },//IIFE立即执行表达式自己执行，无需调用，第一个参数null传给err
    function parallelResults(err, one, two, three) {
        if(err) throw err;
        fulfill("test2: " + [one, two, three]);//1,2,3
        return 2
    },
    function terminate(err, num) {
        if(err) throw err;
        fulfill("test2: " + num);
    }
)
//测试具有延迟的并行调用的功能
expect("test3: 1,2");
expect("test3 t1: 666");
expect("test3 t2: 333");
Step(
  function parallelCalls() {
    var p1 = this.parallel(), p2 = this.parallel();
    process.nextTick(function() { p1(null, 1); });//异步，延迟，类比setTimeout setImmediate
    process.nextTick(function() { p2(null, 2); });
  },
  function parallelResults(err, one, two) {
    if(err) throw err;
    fulfill("test3: " + [one, two]);
    return 666;
  },
  function terminate1(err, num) {
    if(err) throw err;
    fulfill("test3 t1: " + num);
    var next = this;
    setTimeout(function() { next(null, 333); }, 50);//调用this送到下一个函数中
  },
  function terminate2(err, num) {
    if(err) throw err;
    fulfill("test3 t2: " + num);
    this();
  }
);
//具有返回的并行调用的测试锁功能
expect("test4: 1,2");
expect("test4 t1: 666");
expect("test4 t2: 333");
Step(
  function parallelCalls() {
    var p1 = this.parallel(), p2 = this.parallel();
    p1(null, 1);
    p2(null, 2);
  },
  function parallelResults(err, one, two) {
    if(err) throw err;
    fulfill("test4: " + [one, two]);
    return 666;
  },
  function terminate1(err, num) {
    if(err) throw err;
    fulfill("test4 t1: " + num);
    var next = this;
    setTimeout(function() { next(null, 333); }, 50);
  },
  function terminate2(err, num) {
    if(err) throw err;
    fulfill("test4 t2: " + num);
    this();
  }
);
```
执行效果  
 ![image](https://github.com/Wangkaixinlove/node-step/blob/master/doc/imgs/parallelTest.jpg?raw=true)  
 ![image](https://github.com/Wangkaixinlove/node-step/blob/master/doc/imgs/parallelTest1.jpg?raw=true)