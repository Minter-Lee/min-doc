# 前端基础知识篇（四）
<span style="position: absolute; left: 160px; font-size:20px;">—— apply与call</span><br />

&emsp;&emsp;经常看到许多JS代码中使用的call、apply等方法，由于入行时的浅显无知，对于许多JS基础方法一直没有过具体了解，以前看到这些也就是翻翻网页，了解一下就好了，一直以为以后有使用的时候就能明确了，但是因为不清楚，所以在设计代码结构时，完全忽略这些方法，以至于可能没有很准确的结构和逻辑。以后类似问题，一概用此类形式记录下来，就算是记不清了，也可以回头看看。

## 1. call

### 语法结构：

    fun.call(thisObj[, arg1[, arg2[, ...]]])；

### 方法描述：
&emsp;&emsp;call方法可以用来代替另一个对象调用一个方法。call方法可将一个函数的对象上下文从初始的上下文改变为由thisObj，指定的新对象。在非严格模式下，如果没有提供thisObj参数，那么默认指向全局对象（在浏览器中即指向window对象）

### 应用示例

	function A(){
		this.name = "A";
		this.showName = function(){
			alert(this.name);
		}
	}

	function C(){
		this.name = "C";
	}
	var a = new A();
	var c = new C();
	// 输出结果为C
	a.showName.call(c,[]);

上述方法中 通过call方法，将原本属于a的showName方法交给c来使用，所以c拥有了showName方法，同时c.name为“C”，打印结果就不言而喻了。

因为可以转接其他对象的方法，所以还可以以此实现继承，代码如下：

	function A(name){
		this.name = name;
		this.showName = function(){
			console.info(this.name)			
		}
	}

    function C(name){
    	// 此处使用call方法让C继承A
    	A.call(this, name);
    }
	var c = new C();
	c.showName();

当然C中能继承A也可以同时继承B，D，E等等其他类，这里不一一举例，但是建议少用这类多重继承，会覆盖其中相同的方法，当被继承的类越来越复杂时，有太多不确定因素导致代码或逻辑出现问题。而且极难追溯问题根源。

## 2. apply

### 语法定义

	fun.apply(thisObj,[argsArray])

### 方法描述

&emsp;&emsp;除了参数形式不一致外，其余参数限制和默认条件等都与call一致。第二个参数为传入参数的数组集合。

### 应用示例

	function getMax(arr){
	    return Math.max.apply(null,arr)
	}

除了上述外，apply方法也可以实现继承，形式同call几乎一致，就是第二个参数所要求的数据格式不同，此处不赘述。

## 3. call和apply区别
&emsp;&emsp;其实在介绍apply时，其区别就已经说了，下面是MDN的原文：
>while the syntax of this function is almost identical to that of call(), the fundamental difference is that call() accepts an argument list, while apply() accepts a single array of arguments.
    
&emsp;&emsp;记忆技巧：A（apply）for Array and C(call) for comma(意指逗号分隔的list)
下面是一个经典的例子：

	function showContent(title, author){
        var content = "标题是" + title +"，作者是" + author;
        console.info(content);
    }
    //文章标题为hello，作者为Tom</span>
    showContent.apply(undefined,['hello','Tom']);
    //文章标题为hello，作者为Jim</span>
    showContent.call(undefined,'hello','Jim');

## 4. 扩展阅读callee和caller
### caller
&emsp;&emsp;caller 名如其意，其返回的是一个对函数的引用，用于查找那个函数调用了当前函数。对于函数来讲，caller属性只有在函数执行时才有定义，如果函数是由JavaScript程序顶层调用的，那么caller返回的内容就是null。
### callee
&emsp;&emsp;callee返回的是正被执行的Function对象，也就是所指定的Function对象的正文。
示例如下：

	// 验证参数</span>
	function calleeLength(arg1, arg2){
	    if(arguments.length == arguments.callee.length){
	        alert("形参和实参长度匹配")；
	    }else{
	        alert("形参长度："arguments.callee.length);
	        alert("实参长度："arguments.length);
	    }
	}

## 小结

这是在翻阅redux相关书籍时，无意间看到柯里化这个问题后，发现这部分的短板。在翻阅柯里化相关内容时，就有几个关键的方法模模糊糊，误导了好几次思路，本来简单易懂的概念，愣是让我看了一个小时，遂决定记录下这些原来模糊的概念，或许明天我就记得不那么清楚了，但是不会再盲目的从网络上获取内容了，以上的内容，示例主要来自于几篇网文，那个网站的都有，由于作者层次和笔法的问题，有的越看越糊涂（张鑫旭咱能有话就说，不来回来去的绕吗！）有的过于撩草粗浅（其实我也是），所以综合起来摘抄了我觉得还过得去的东西，留了下来，以备后用，记住大部分不是我写的，是我抄的，抄的，抄的，重要的事情说三遍！