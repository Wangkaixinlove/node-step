## 问题清单

- 成果仓库是否创建，仓库名称是什么？  
step。
- README.md 文档是否翻译？  
是。
- 是否撰写 MarkDown 文档来记录自己代码阅读的收获？
是。  
- 通过代码阅读，自己学到了什么？  
学到了一些js高级语法，断言测试的巧妙之处，工厂化的便利。
- 项目的功能是什么？  
Step.js是控制流程工具，解决回调嵌套层次过多等问题。适用于读文件等回调函数相互依赖，或者分别获取内容最后组合数据返回等应用情景。
- 请展示项目功能或者运行项目（不同项目类型，运行方式不同）。  
- 程序中的注释是否翻译？  
已翻译。
- 是否在程序中补充注释？补充注释的理由是什么？不补充注释的理由是什么？  
是，一些必要的地方补充了注释。
- 项目类型是什么？（命令行程序、web 网站、第三方库、其他，如果是其他项目类型，请给出项目类型的具体名称）  
第三方库。
- 项目的入口文件是哪个？  
step.js。
- 项目的依赖项有哪些，各个依赖项都是做什么，有什么功能？  
断言 用来测试最终结果与预期结果是否相等  
fs 用来读取文件，读取目录等对文件的操作
- 项目有哪些代码模块？各个代码模块之间有什么关联性？  
Healper模块，断言模块
Healper模块中引入了assert断言模块,step模块,fs模块，其他文件引入了healper模块
Healper模块还包括一个错误数组用于创建错误。
- 代码模块中有哪些函数？各个函数都是做什么的？    
 1.function next()主函数  
 2.function parallel()确定数目的多个异步
 3.function group()组，不确定数目的多个异步
 4.function fn()创建工厂
- 项目中的数据结构有哪些种类？功能作用是什么？  
数组,错误数组:用来保存错误和抛出错误，  
结果数组:用来存储结果。    
- 项目中的设计模式有哪些种类？功能作用是什么？   
工厂方法模式，减少代码的编写，更方便使用，例如fn函数相当于一个
包装步骤列表的工厂 ,读取多个文件的时候，可以将公用的函数包装成工
厂，不用每次都传参（函数书写麻烦） 
- 项目中使用了哪些高级的 JavaScript 语法？  
1.	Process.nextTick() http://blog.csdn.net/hkh_1012/article/details/53453138
2.	Array.forEach() Array.map()
3.	next.apply(null, results);
4.	assert.deepEqual(dirListing, results);
- 代码中哪一个或几个函数的代码比较难于理解？你搞明白了吗？你认为难点是什么？  
1.	多次用到返回值是函数，例如fn工厂
2.	对于Process.nextTick()的理解
3.	This.group() this.parallel()区别
4.	this.parallel()的结果会按顺序传入最后一个函数中
- 项目中用到自动化测试吗？用到自动化测试框架了吗？用的是哪个自动化测试框架？  
断言模块
- 项目中所有的模块都有单元测试吗？哪些有？哪些没有？这样安排的理由是什么？  
对每个函数都进行了测试
- 仓库根目录都有哪些文件？每个文件的作用都是什么？  
Test文件，包含测试代码
Lib文件，包含step.js源码
Readme文件说明该项目的使用
pakage.json配置文件，包括name,version,url等属性
LICENSE 文件是开源协议
- 代码中是否有 bug？  
1. 注释不全面。  
2. 断言时无比对信息比较不方便。
3. 异步与同步比较时时没有用utf8编码。
- 代码中是否有可以改进的地方？
添加函数注释。 
- 项目是如何划分模块、划分函数的，划分的好吗？如果是你，你会怎么做
按照功能划分函数，按代码的重用性和功能划分模块，较好。
- 代码的可读性如何？结构清晰吗？编码风格如何？  
结构较清晰，代码简洁。  
- 你是否调试运行过项目？通过调试运行，你搞明白了哪些问题？如果没有调试运行，说出不调试运行项目的理由。   
是，通过调试明白了一些变量的实际代表的意义，比如args,steps,toRun等 
  
