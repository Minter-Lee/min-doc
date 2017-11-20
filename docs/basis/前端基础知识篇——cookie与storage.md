# 前端基础知识篇（六）
<span style="position: absolute; left: 160px; font-size:20px;">——cookie与storage</span><br />

## 前言
 起因很简单，面试经常被问到，虽然之前多多少少了解过，但是一直比较模糊，尤其是对其操作方法和详细特性。秉承着一贯的态度，整理一下，以文档形式记录。（PS：其实我已经偷懒好几天了......）

## 1.Cookie

## 1.1简介
Cookie，小甜饼的意思，它的出现也正如其名——小甜饼，首先它解决了浏览器端迫切需要的缓存问题，够甜。其次它很小，这也是它主要的特点，最大长度仅有4KB。Cookie的全称为HTTP Cookie，其初衷是为了在浏览器端存储会话信息，也正因如此，Cookie有个缺点，它每次都需要携带入HTTP响应和请求中。形式如下：
    
    // 报头响应信息
    HTTP/1.1 200 OK
    Content-type: text/html
    Set-Cookie: name:value
    Other-header: other-header-value
    
    //请求报头信息
    GET /index.html HTTP/1.1
    Cookie: name/value
    Other-header: other-header-value

由于需要每次携带，无疑增加了请求和响应的负担，所以每个Cookie限定了大小，但为了防范Cookie泛滥，每种浏览器都对同源下Cookie的数量进行了限制，一般为30~50个，当然chrome/Safari并没有限制。

## 1.2 Cookie格式
我们在上述报头信息中看到了Cookie最基本的结构形式，但是一个Cookie的信息，不仅仅是内容，还有名称，所在域，路径，失效时间以及安全标志。

- **名称**：一个唯一确定cookie的名称。不区分大小写，但是默写服务器会区分，所以不要将大小写混用，这也是一个良好的编码规范。名称内容为URL编码。
- **值**：就是我们最常看到的name:value的形式。内容为URL编码
- **域**：cookie是遵循同源策略的，即不支持跨域访问，所以这个值就是确定域所在的。所有向该域发送的请求中都会包含这个cookie信息。如果没有明确规定，则默认认为来自cookie设置的所在域。
- **路径**：指定域中的那个路径，应该向服务器发送cookie信息，设定后，路径不同，即使在同一域中也不能访问到cookie信息。
- **失效时间**：即cookie何时回收时间，标准的格林威治格式。默认为浏览器结束会话（如：关闭浏览器），也可以手动设定，可以保证浏览器关闭后cookie也一直存在。若设定为以前的时间，则cookie将立即被删除。
- **安全标志**：制定后，cookie只有在使用SSL链接的时候才发送到服务器。例如。cookie的信息只能发送给 https://www.abc.com，而 http://www.abc.com 的请求则不能发送cookie。

我们来看一段完整的响应头

    Set-Cookie: name:value; expires=Mon, 22-Jan-07 07:10 :20 GMT; domain=.abc.com; path=/; secure

这里面有两点需要注意：第一，secure就是安全标识，这是cookie信息中唯一不是键值对形式的标识，就是一个单词；第二，这是服务器给浏览器的提示信息，所以在发送请求中不会有这么多信息，仅有值得键值对。

## 1.3 Cookie的操作
令人恶心的cookie的API——document.cookie。非常奇特的设计，使用的方式不同document.cookie所代表的行为迥异。我们在页面中获取document.cookie的时候，其返回的内容为当前域，当前路径，符合安全标识下，可用的所有cookie的字符串，表现形式是一个字符串。例如：
    
    name1=value1;name2=value2;name3=value3

需要注意的是这里的字符全是URL编码，所以我们获取的时候还需要进行解码decodeURIComponent，同理注入时需要转码encodeURIComponent()

当我们注入时呢？给这个字符串替换内容？并不是，使用document.cookie赋值时，它不会覆盖原有cookie的内容（除非名称相同），它会将内容添加到原有的cookie中，设置时可以注入域、路径、失效时间等内容。这里面有个非常关键的标识就是cookie的name。这是辨别cookie的唯一标识。最简单的赋值如下：

    document.cookie = encodeURIComponent("name") + "=" 
       + encodeURIComponent(“value”) + "; domain=.abc.com; path=/"

API几乎就是没有，而且比较麻烦，所以一般情况下都是会自行封装get、set方法。

    var util = {
        get: function(name){
            var cookieName = encodeURIComponent(name) + "=",
                cookieStart = document.cookie.indexof(cookieName),
                cookieValue = null;
            if(cookieStart > -1) {
                // 通过indexof设定fromIndex确定该cookie的具体结束点
                var cookieEnd = document.cookie.indexof(";", cookieStart);
                if(cookieEnd == -1) {
                    cookieEnd = document.cookie.length;
                }
                cookieValue = decodeURIComponent(document.cookie.substring(cookieStart+cookieName.length, cookieEnd))
            }
            return cookieValue;  
        },
        
        set: function(name, value,expires, path, domain, secure) {
            var cookieText = encodeURIComponent("name") + "=" + encodeURIComponent(value);
            // 根据参数一一注入内容
            if(expires instaceof Date ) {
                cookieText += "; expires=" + expirse.toGMTString();
            }
            
            if(path) {
                cookieText += "; path=" + path;
            }
            
            if(domain) {
                cookieText += "; domain=" + domain;
            }
            
            if(secure) {
                cookieText += "; secure=" + secure;
            }
            
            document.cookie = cookieText;
        },
        
        // 利用过期时间手动清除cookie
        unset: function (name, path, domain, secure) {
            this.set(name, new Date(0), path, domain, secure);
        }
    }

基本上来讲cookie的内容就是以上这些了，至于子cookie的内容，我想因为有了storage的存在，应该没什么需求需要使用了，值得注意的是，cookie可以在浏览器端任意获取，挺不安全的，所以不建议存放敏感的信息。而且cookie的大小时4K左右（4096B+-1），我们如果注入过量了，cookie是删除之前的，还是删除多余的，还是随机删除一个，各个浏览器都不一致，十分危险的操作。

## 2. storage

### 2.1 简介
全称 Web Storage，它是对cookie的补充和发展，打破了一些cookie的限制，当数据需要被严格的控制在客户端时，无须持续的将数据发回服务器。但它只能存储字符串类型数据，其他类型都会转成字符串进行存储。主要目标有两个。
- 提供一种cookie之外的会话数据存储的途径。
- 提供一种存储大量可以跨会话存在的数据的机制（5M左右）。

web storage 分为两类，即sessionStorage 和 localStorage。这两个都可以通过window对象直接调用。并且它又非常优化的API，基本上不需要进行二次封装。
- **clear()** ：删除所有值 Firefox没有实现
- **getItem(name)** ：获取制定name对应的值
- **key(index)** ：获取index位置处值得名称
- **removeItem(name)** ：删除指定name的数据（键值对）
- **setItem( name, value)** ：为指定name设定对应的值

虽然作为storage对象可以直接通过属性名获取内容，但是建议还是使用规范的API方法对storage进行操作。以免属性覆盖等隐藏问题的出现。

### 2.2 SessionStorage

   sessionStorage用于存储某个特定回话的数据，也就是说该数据仅保存到浏览器关闭。如cookie一样，浏览器关闭就清除。但是该数据可以跨页面刷新仍旧存在，如果浏览器支持，浏览器崩溃后重启，该数据依旧存在。
   因为sessionStorage对象绑定于某个服务器会话，所以当文件在本地运行的时候是不可用的。其中的数据仅能在最初建立sessionStorage的页面进行访问，所以多页面应用可能有局限。注入数据的方法可以用属性赋值和setItem方法，但这里需要注意一点，IE的赋值过程是异步的，数据使用衔接上可能有问题，其他内核是同步的，数据量大时，有延迟效应。IE8可以强制将数据存储至磁盘上，有兴趣可以查一下，这里不赘述了。对sessionStorage的获取和遍历分别通过getItem和length配合key来实现，也可以使用for-in来进行迭代，这里就不赘述了。还有就是删除操作，由于delete操作符WebKit不支持，我们就老老实实的使用removeItem来删除就好了。

### 2.3 GlobalStorage

   虽然被H5规范抛弃了，还是简单说一下吧，最初由Firefox2实现了这个storage，主要用于跨会话使用，但可以限定访问，可以指定域。通过如下形式：

    globalStorage["abc.com"].name = "myStorage"

   获取也是需要携带对应域的，但是这里也有问题，就是如果这个域的规则过于简单甚至是空字符串，那就意味着所有打开的页面都能访问修改，这十分危险的行为。

### 2.4 LocalStorage
    
   在H5的规范中LocalStorage取代了globalStorage作为了持久保存客户数据的方案。它封闭了访问规则的设定，而是提前设定好规则了，这样更安全。要访问同一个localStorage对象，页面必须来自同一个域名，而且子域名无效，同时必须使用同一种协议，在同一个端口上，这相当于globalStorage[location.host]。虽然一定程度上降低了其灵活性，但却保证了安全，有舍才有得嘛。至于其增删改查的操作就同sessionStorage一致了。同样的它的回收也同sessionstorage一样，浏览器缓存被清除，JS手动删除，数据就回收。

### 2.5 Storage事件

   这正是storage强大的地方，cookie的增删改是不可监听的，但是storage有事件，这意味着很多事情我们有了可以监听的节点，但同时也意味着操作不再是纯粹，中间过程会有意外，增加不可控的bug几率。事件触发的节点包括clear()、removeItem（delete操作也算）和setItem。事件属性如下：
    
   - domain：发生变化存储空间的域名
   - key：设置或删除的键名
   - newValue ：新值（新增）或者为null（删除）
   - oldValue ：原始值

兴奋不兴奋，但是这类事件的触发还有些问题，我自己还没有完整使用过storage内容，后面会补充自己的实践经历，先来说说网上的问题，下面是摘抄的问题描述：
   
>  在firefox和chrome中存储和读取都是正常的, 但是对storage事件的触发似乎有点问题, 自身页面进行setItem后没有触发window的storage事件, 但是同时访问A.html和B.html, 在A页面中进行 setItem能触发B页面中window的storage事件, 同样的在B页面中进行setItem能触发A页面中window的storage事件. 在IE9中, 页面自身的设值能触发当前页面的storage事件,同样当前页面的设值能触发同一”起源”下其他页面window的storage事件,这看起来似乎更 让人想的通些.

有兴趣的话可以访问http://www.cnblogs.com/shihao/archive/2011/12/23/2298854.html 作者还提供了两个验证用的页面。效果确实如他所说很诡异。

## 3.小结

本来我想着在这里将cookie和storage的区别说出来，但是我突然觉得这些面试题真是坑，明明就是两个东西，原理和性质有很大的不同，而且很不容易混淆，而很多面试者再被问到后都以为这些是容易混淆的内容，就按照这个方向不断挖掘两者的区别，而忽略了它们解决的问题和设计的初衷。我也是被坑了，在这次整理中发现了我对cookie很多误解，所以我在这里偏不说cookie和storage那些表面上的区别，如果你想知道，那就慢慢看吧。