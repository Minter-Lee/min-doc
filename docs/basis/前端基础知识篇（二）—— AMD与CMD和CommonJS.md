#前端基础知识篇（二）

<span style="position: absolute; left: 160px; font-size:20px;">—— AMD与CMD和CommonJS</span><br />

## 1.AMD（异步模块定义）与CMD（通用模块定义）
通用的来说AMD和CMD都是加载机制规范，而且都是异步加载机制，AMD推崇依赖前置，用不用都先加载进来，如requireJS。CMD推崇依赖就近，即使用时再加载然后使用。他们都是推广模块化开发的产生的规范。
    
AMD基本格式如下：

    define('hello',['JQuery'],function(require, exports, module){
        JQuery.doSomething()...
    });

CMD基本格式如下：

    define(function(require, exports, module){
        var $ = require('JQuery');
        $.doSomething()...
    });

就目前来讲最明显的差别就是这些了，但是请注意，这两者都是规范和概念，具体的实现时RequireJS和seaJS。这些概念下还有一个神器：**CommonJS** 

CommonJS是什么，应该说是一个宏伟的目标，JS只能构建基于浏览器的应用程序，主要的原因是没有其他环境下的API，而CommonJS就是来填补这个空白的。最好的代表作是什么？NodeJS，它开启了JS模块化编程。即支持用JS编写服务端程序（服务端的复杂性必须要模块化）。他具体的学习这里先不说了。我们用的webpack也用的是CommonJS规范来书写。

随着JS发展，ES6提出了module概念，它与CommonJS存在差异，ES6模块的设计思想是尽量静态化，编译时就能确定模块的依赖关系。而CommonJS则是在运行时确定模块依赖关系即运行时加载。例如：

    let { stat, exists, readFile } = require('fs');

事实上，fs模块全部加载了，因为CommonJS的模块就是对象，输入时必须查找对象属性（stat等）。而ES6的模块不是对象，而是通过exports命令指定输入的代码

    import { stat, exists, readFile } from('fs');

代码实际上只加入了这三个方法，而不是整个fs，这就是编译时加载，即ES6可以在编译时就完成模块的编译工作，这里效能要比CommonJS高。
ES6的模块还有其他好处，我尚未读明白，我觉得需要单独的文档来学习记录。


