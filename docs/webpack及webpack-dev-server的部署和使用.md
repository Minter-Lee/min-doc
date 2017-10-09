# webpack及webpack-dev-server的部署和使用

网络上关于webpack的部署很详细，这里面就一一说明了，关于webpack-dev-server的部署也有很详细的资料，也不说，就说一件事儿。这件事儿在大量的说明文档中都没有体现，就是一带而过，这也就说明很多文章在理解或者不理解的情况下，并没有说清楚webpack-dev-server是怎样实现热加载的，这里面有几个关键参数，都没有给出详细说明。当你需要按照自己的路径配置时，你就会发现许多路径匹配不上了。OK，下面我一个个说，首先是webpack.config.js中output的配置
    
    //出口文件配置
    output: {
        filename: '[name]Bundle.js',
        path: path.join(__dirname, 'app/bundles'),
        publicPath: "/"
    }

这里面还有配置项，但是一般情况下这三个就够了，那么这三个参数究竟意味着什么，配置后产生了什么影响，我想光看字面是不够的，下面我一个一个的说：

1. filename

    你可能会说这就是配置输出文件的名称，错！是配置webpack输出文件的名称！为什么这么说，后面说完webpack-dev-server的配置你就知道了。

2. path

     path 字面解释就是路径，是webpack输出打包好的文件的路径，后面的参数详见webpack的解释，这里面就不详述了，它配置后，与filename共同决定了由webpack打包处理后的bundle文件真实路径。

3. publicPath

    上面不是有一个path了吗，这个干什么用？，GitHub上原始描述是加载路径，什么加载路径？加载路径不是由path决定了吗？不，path是打包路径，打包路径可能有局限，所以需要一个加载路径，应该说这是一个虚拟的加载路径！配置后index.html中加载由webpack打包的bundle文件，就用这个路径。

OK 说道这里，你或许会觉得没啥问题啊，对如果仅仅是使用webpack，这就够了。但是我们需要webpack-dev-server的配合，我们需要自动打包，自动刷新。那么webpack-dev-server，里面的配置就很奇妙了。

    var server = new WebpackDevServer(compiler, {
        // 访问根目录路径（index.html所在目录）
        contentBase: 'app',
        filename: 'indexBundle.js',
        publicPath: '/',
        hot: true,
        stats: {
            colors: true
        }
    });

你会一眼看到这里面还有filename，还有publicPath。这两个所起到的作用与webpack中的几乎一致，差别在于这是针对webpack-dev-server根据webpack.config的规则所打包出的具有热替换功能的bundle文件，请一口气读完。这个文件存于内存中，不会真实存在在文件夹内，但是如果你想使用热替换，你想自动刷新页面，请使用webpack-dev-server打包出来的bundle文件。那些在bundles文件夹下（webpack.config中output的path配置）产生的update小文件，不是针对webpack打包出来的bundle文件进行更新的，而是针对webpack-dev-server打包出来的。这也就是为啥我配置了好几种方式，却始终不知道为什么看着webpack-dev-server明明重新打包了却不刷新的原因，究其根本还是因为不知道webpack-dev-server究竟做了什么实现的热部署。许多讲述的文章中，要么说这么配就好，要么就说字面意思，弄得云里雾里的就是行不通。若能知道其内部运行原理和过程产物，我觉得这一切都不是问题，还是那句话理解的太浅了，就别瞎哔哔。用一样东西，或许不需要看懂所有的源码，但至少要了解一些，了解一些中间过程，才能体悟到设计者的用意，才能真的用好这个函数，这个中间件，以至整个项目！