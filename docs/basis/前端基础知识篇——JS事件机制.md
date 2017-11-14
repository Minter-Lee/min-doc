# 前端基础知识篇（三）

<span style="position: absolute; left: 160px; font-size:20px;">—— JS事件机制</span><br />
## 1 JS事件机制

### 1.1 事件流
事件流分为两种，事件冒泡和事件捕获。两者传递流程完全相反，冒泡是从触发向上至监听节点，捕获则是从监听节点到触发节点。就目前来讲事件捕获的形式很少有人用了。我们监听事件的时候的第三个参数就是设置使用哪个模式的，下面用一个经典例子去说明区别：

     // DOM结构
    <div id="s1">
        <div id="s2"></div>
    </div>
    
    // 监听方法
    s1.addEventListener("click", function(e) {
        console.info("s1 捕获模式 ")
    }, true);

    s2.addEventListener("click", function(e) {
        console.info("s2 捕获模式 ")
    }, true);

    s1.addEventListener("click", function(e) {
        console.info("s1 冒泡模式 ")
    }, false);

    s2.addEventListener("click", function(e) {
        console.info("s2 冒泡模式 ")
    }, false);

    // 输出结果
    s1 捕获模式
    s2 捕获模式
    s2 冒泡模式
    s1 冒泡模式

### 1.2 DOM2级事件流
如图：

![DOM2级事件流模型](https://i.imgur.com/XzWbTv9.png)

可以看出，它主要分为捕获过程，到达事件位置，事件冒泡过程单个过程。阻止事件进一步传播的方法时stopPropagation()（W3C规定的，preventDefault本意不是组织冒泡）。这里阻止的是图中一个节点向上冒泡的过程，不会阻止同节点下的2个监听事件，若需要阻止本方法后的方法（同一节点下，事件是按照监听顺序执行的）使用stopImmediatePropagation()。

DOM2级事件流主要通过addEventListener监听事件有removeEventListener配套删除事件。这是大部分浏览器支持的，IE总是奇葩的，它没有上述两个方法，它用attachEvent()和detachEvent()来实现监听和删除，所以一般情况下，我们常常直接使用的on/off方法都是做了这方面兼容的。这里需要注意，2个方法是有区别的：

    addEventListener(eventName, function(e){
    }, false)

    attachEvent(onEventName, function(){
        var e = window.event;
    })

可以看出

- 标准的方法中传入的事件名（如：click）而IE是on+事件名（onclick）;
- IE中不传入e参数，它直接挂在window.event上；
- IE压根不支持捕获，所以不用设置第三个参数，默认就是冒泡流（false）;

虽然我们在逐步淘汰IE8以下的浏览器，但是还是要把标准的与IE的区别说明一下

- target：即事件绑定的元素，IE中叫srcElement
- currentTarget：即事件触发的元素，IE9以下没有
- preventDedfault：通知浏览器停止执行与事件关联的默认动作（如：继续冒泡）IE中为e.returnValue=false
- stopPropagation：停止事件传播，IE中为e.cancelBubble=true;

还有一个比较古老却很实用的事件流，那就是DOM0级事件流，它简单且跨浏览器兼容性好。与DOM2级事件流的主要区别在于，它的事件监听是直接使用onclick等方法，可以说是元素本身的方法，所以不涉及捕获冒泡等流程。有且仅有这一个方法做监听，不能叠加，否则后者覆盖前者。

接下来，是一个面试过程中经常提到的题目，能够手写一个原生JS的事件监听方法。这是我的实现过程，不一定是准确的。

    let on =>(el, eventName, eventHandler, flowType = false) {
        if(el.addEventListener) {
            el.addEventListener(eventName, eventHandler, flowType);
        }else if(el.attachEvent) {
            if(flowType == true) {
                throw "浏览器版本不只是事件捕获型";
            }
            el.attachEvent("on"+eventName, eventHandler.apply(null, window.event));
        }
    }
