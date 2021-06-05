@[TOC](目录)
## 前言
exports和module.exports有这么难理解吗？有些教育机构用地址介绍这两个东西，还介绍得特别乱，把整个学习成本都提上去了。

来看看某教育机构的介绍：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210605105635701.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2VmX2Vm,size_16,color_FFFFFF,t_70)
是不是特别乱？反正我不想看，也不想通过这个地址来看懂。

这篇文章目的不再揭露node的行为，而是为了让大家便于理解。请不要抱着理解node的思想去看这篇文章（因为连我现在都还没接触到node本质性的知识），而是抱着理解exports、module、require这些东西的本质而去阅读。

## 提前说总结
由于有些人悟性比较好，或者javascript基础较扎实，我觉得有必要提前说下总结，避免看到文章后面说一句——“就这？废话了这么多？”。当然，理解能力差的同学还是建议你跳过这一段，直接看证明过程。

其实，exports、module、require无非就是三个变量而已，什么地址的，无非就是javascript顺带的东西，只要你javascript够熟悉，何须去理解它这些地址的变换。

这三个变量，是node执行你的代码前，默默帮你声明并赋值的三个变量，node是怎么帮你声明并赋值的呢，我推导了一下，大概是这样的：
```javascript
// 声明module
var module = {
  exports: this // module.exports指向的是全局的this对象，不信你自己打印一下是否完全相等
}
// 声明exports
var exports = module.exports
var require = function(path) {
	// 1. 获取path对应的文件并解析成程序a（假设是a）
	// 2. 提取程序a的this对象并返回（既然module,exports指向的是this，那么require就得提取出目标程序的this）
}
```
require没法写了，因为node是用C++写的，文件获取并解析等等操作肯定也依赖于C++。所以我只能用注释写个大概意思。

如果上面这些听得有点懵的话，可以试试执行下面这段代码，对你的理解会有很大的帮助：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210605113609422.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2VmX2Vm,size_16,color_FFFFFF,t_70)


## 开始证明
### 第一步，node对module.exports的实现
有些人肯定会奇怪，为啥说它们都是变量？

其实不止他们是变量，来看node下运行的这段代码：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210605110746808.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2VmX2Vm,size_16,color_FFFFFF,t_70)
可以看到，控制台输出require已经被定义，你就算把require换成exports、module，也是同样的——已经被定义。

这说明什么，node在执行程序前已经帮我们声明了这三个变量，可以参考undefined，它也是变量，只不过在全局下undefined是只读的，而这三个变量是可改的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210605110934878.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2VmX2Vm,size_16,color_FFFFFF,t_70)
把let换成var，或者连var都不用写，即证明。

然后我又发现，module.exports居然跟全局的this一样？666
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021060511415372.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2VmX2Vm,size_16,color_FFFFFF,t_70)
很神奇是不是，然后再 ```console.log(this)```一下，发现值是 ```{}```，就可以推出node对module.exports的实现：
```javascript
var module = {
  exports: this
}
```
很好，这样第一步就完成了。

### 第二步，exports的实现
相信学习node的时候都有听过一个点：exports只能添加属性，不能赋值。
用代码来阐述一下：
```javascript
exports = 1 // 这会导致exports无法使用
exports.name = 'juln' // 这才是正确的用法
```
为啥？其实很好理解：
```javascript
var exports = this
exports.name = 'juln' // 正常使用
exports = 1 // 裂开了，因为exports被重新赋值了，它不再是this
```
懂了吧？什么地址什么鬼的，你在学javascript的时候可能有学过，但是也不可不去学，因为这东西从字面上就很好理解。

整合一下之前的代码：
```javascript
var module = {
  exports: this
}
var exports = module.exports
```
这样，第二步也搞定了。

### 第三步，node对require的实现
额，这个就无法用javascript代码来解释了，因为这个涉及到文件的读取和解析代码，完全是node的底层——C++的功劳...

但是还是可以用注释写一下思路的：
```javascript
var require = function(path) {
	// 1. 获取path对应的文件并解析成程序a（假设是a）
	// 2. 提取程序a的this对象并返回（既然module,exports指向的是this，那么require就得提取出目标程序的this）
}
```
如果连这个都看懂的话，恭喜你，我这篇文章对你已经没任何帮助了。

## 最后
另外提一句，esm的那个import和export是语法，不是变量，跟cjs差挺多的，不要去研究了。

希望这篇文章能帮助大家，之后发现些其它神奇的东西我也会写出来的，关注一下我，不吃亏。