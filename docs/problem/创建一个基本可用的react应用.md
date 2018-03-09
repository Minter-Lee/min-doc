#创建一个基本可用的React应用

## 前言
   一个React项目跑起来非常简单，我们仅需要在一个index.html中引用相应的JS文件，就可以在script标签下书写React代码。但放入生产环境下，这种形式是远远不够的，不说PWA应用的构建。我们至少要构建一个SPA应用。构建一个基本可用的应用需要考虑的不仅仅是主干技术（如：React），还需要对项目结构、打包、插件、辅助包、开发规范、业务特性等等内容进行深入研究和取舍。所以这里仅仅讨论一个基本可用的React应用，而不是一个万金油应用。

## 1 技术选型
   首先明确的是，我们构建应用的主要类库是React。之后还有如下技术，需要取舍。
- **平台**： 原来前端项目基本上都是依托于后端项目上，或者就是单纯的前端项目，但这对于React来说是不合适，它与Node平台结合的更好。所以平台必然是Node。

- **包管理**： 在这方面我们基本上是选用Node自带的NPM。当然也可以选用Yarn，不过从项目角度考虑，我们对安装速度的一致性和是否允许安装时执行等问题，不是特别关注。或者说就目前的场景来讲，可以不过多考虑这个问题。

- **打包工具**：打包是现代前端项目非常重要的一个环节，从过去的grunt、gulp到如今的webpack、rollup等。这些工具不仅可以将分散的代码聚拢，合并，压缩。更是可以在打包过程中，进行代码补充、转义、拆分、统一处理等内容，让我们的开发更舒适，更规范，更高效。就目前来讲webpack相对大众一些，插件形式也非常灵活，何况还有非常适合小项目调试使用的webpack-dev-server插件。

- **语法**：本来这是一个唯一选项的答案，奈何ES6的出现，因为ES5与ES6甚至是ES7在语法上天差地别，而且ES6语法上有很多非常好的改善。解决了许多ES5使用时的尴尬和繁琐。所以这里我们至少使用ES2015(ES6第一个版本)的规范。

- **插件**：因为ES6语法，更新的很快，很多现代浏览器对它的支持并不统一，所以我们需要将他转为ES5规范，来兼容所有的浏览器，值得庆幸的是babel在这方面做得非常好，配置简单，支持的很全面，就算我们使用ES7语法，babel也能很好的转义，并保留这所有语法的特性（如箭头函数作用域不上升等）。babel可以让我们放心大胆的徜徉在ES6的新语法规范中。

- **样式**：随着JS的发展，CSS也在进步，为了提高CSS的开发效能，解决命名空间等问题。sass、less等css规范相应给出，这里以less的编程能力最变态。但说实话，如果没有什么特别需要，sass、less的选择还是随心所欲的。我们很难或者很少有精力对CSS的构建写非常复杂的函数。还有一种是CSS Modules。这也是一种css的规范，但是如果使用了其他UI，CSS Modules就无能为力了。如果想有更多了解，可以自行谷歌。Post CSS，我认为更像是一种CSS插件，而不是开发的规范。至于CSS In JS，恕精力有限，确实没有深入研究过。暂时不讨论了。

- **路由**：路由是一个项目的入口，而针对于React来讲，肯定是React-Router了。但我们这里面使用的是react-router 4.x。它与之前版本在设计理念上就已经不一样了。之前版本是写入一个Route文件中，层层嵌套。而4.x更推荐分散的来写。而且路由的使用方式也有很大变化。配置时再详细说明

- **状态管理**：我们都知道依托react的状态流转是非常难用的。很容易state泛滥，而且组件间通信，尤其是跨组件通信都非常不方便。所以出现了flux，reflux，redux和mbox等专门管理状态的辅助包。目前主流的是redux和Mbox。针对状态管理这方面，其实两者的最开始的思路是一个，为了跨组件通信，就像涉及的状态提取到两个组件共同父类。不同的时候Redux将所有状态融为一颗树。成为所有组件的数据来源，从而达到所有组件都能访问完整的state树。而Mbox则更为零散，看起来更像是将setState隐藏起来了，通过对数据精确的监控来高效更新组件。而跨组件通信是通过使用一个父类store来完成的。网上有很多，关于二者的对比和应用场景分析。有兴趣可以看看，这里面我选择redux，以及配套的react-redux， react-redux-router。

- **异步方案**：ES6语法中Promise是最常见的异步解决方案。一般情况下我们想服务器的请求都不在使用ajax而是改为Promise。而浏览器fetch是一个不错的选择，简单方面，返回的也全是Promise对象。既然说是方案，就是一套完整的解决方法，而不是单纯一个技术。我们在redux中加载一个数据，至少要发送三个Action，分别再请求时，请求成功和请求失败。然而dispatch的参数是对象，而不是一个函数，为了能使用函数最为参数，我们才使用了redux-thunk这个中间件来进行处理。但是多次在回调中不断请求还是感觉不爽，所以我将失败和成功中回调的内容进行封装，使之在请求中自动执行后续的action。降低我们开发成本，于是我们就有了redux-composable-fetch这个插件，内容其实很简单，就是根据传入的一组固定顺序的type（load，success，error），来一次性完成一个请求。

- **效能方案**：react本身是一个高效的组件，但是用不好还是会产生很多不必要的diff过程。为了优化这一部分，我们有了诸多的优化手段，如purerender、immutable.js、无状态组件等等。purerender基本上可以弃用了，因为react已经集成了PureComponent。而immutable非常重，尤其是在转化为Immutable对象和转为JS对象时，效能消耗非常高，所以如果不是数据承载量非常大，不建议使用。划分好组件更直接有效。

从以上的技术选型来看，一个可以在生产环境基本可用的react项目，构建并不简单。接下来是更麻烦的，我们需要开始筹备项目目录，一步步将这些内容安装、串联、调试，再根据业务场景逐步调整架构，使之更健壮。

# 2 项目构建

## 2.1 package.json

创建一个项目最开始接触的文件就是package.json。里面是该项目的各类信息，包含名称，关键字，简介，作者，证书，命令注册（npm run），已经开发依赖列表和运行依赖列表，在依赖表中存在着大量的开发工具包以及对应的版本，在注册这些包时需要非常注意，包的版本号，最好确定准确的版本，因为很多工具包也依赖各种其他工具包，每个工具包所支持的版本是非常庞杂的一个树。好在我们在安装时，NPM基本上都能识别出问题。
下面简介该项目的包内容：

       {
          "name": "",
          "version": "1.0.0",
          "description": "",
          "main": "index.js",
          "scripts": {
            "startPro": "webpack --config webpack.product.config.js ",
            "startDev": "webpack --config webpack.dev.config.js ",
            "startSev": "node webpack-dev-server.js --history-api-fallback"
          },
          "author": "lei.li@fenbeitong.com",
          "license": "ISC",
          "dependencies": {
            "amfe-flexible": "^2.2.1",  //移动端多屏幕适配方案
            "antd": "^3.0.3", 
            "babel-core": "^6.26.0",    //babel基础库
            "babel-loader": "^7.1.2",   //babel加载器
            "babel-plugin-import": "^1.6.3",    //babel插件导入器
            "babel-plugin-react-transform": "^3.0.0", //react语法转换插件
            "babel-plugin-transform-decorators-legacy": "^1.3.4",  //class装饰器转化器
            "babel-plugin-transform-runtime": "^6.23.0", //babel运行时编译器，避免重复编译，改为引用
            "babel-polyfill": "^6.26.0", // 转义不支持的新的ESAPI
            "babel-preset-env": "^1.6.1", //es6预置转义器，根据环境启动转义插件，替代ES2015、ES2016等插件
            "babel-preset-react": "^6.5.0", // react预置转义器
            "babel-preset-stage-0": "^6.5.0",//es7 0序号编译器（包含1、2、3）
            "babel-register": "^6.7.2", //require功能转义
            "classnames": "^2.2.5", //动态样式组合插件
            "css-loader": "^0.28.7",
            "eslint": "^4.14.0",
            "file-loader": "^1.1.6",
            "history": "^4.7.2", // 路由历史
            "html-loader": "^0.5.1",
            "jshint": "^2.9.5",
            "jsx-loader": "^0.13.2",
            "less": "^2.7.3",
            "less-loader": "^4.0.5",
            "less-plugin-functions": "^1.0.0",
            "moment": "^2.20.1", //日期
            "prop-types": "^15.6.0",
            "pure-render-decorator": "^1.2.1",
            "react": "^16.2.0",
            "react-css-modules": "^4.7.1",
            "react-dom": "^16.2.0",
            "react-redux": "^5.0.6",
            "react-router": "^4.2.0",
            "react-router-dom": "^4.2.2",
            "react-router-redux": "^5.0.0-alpha.9",
            "react-transform-catch-errors": "^1.0.2", //热加载错误捕获
            "react-transform-hmr": "^1.0.4",    //热加载插件
            "redbox-react": "^1.5.0",
            "redux": "^3.7.2",
            "redux-composable-fetch": "^1.0.0",
            "redux-thunk": "^2.2.0",
            "style-loader": "^0.13.0",
            "transform-runtime": "0.0.0",
            "uglifyjs-webpack-plugin": "^1.1.5",
            "url-loader": "^0.6.2",
            "webpack": "^3.10.0",
            "webpack-dev-server": "^2.9.7",
            "whatwg-fetch": "^2.0.3"
          }
    }

## 2.2 .babelrc

babel应用了大量的插件，其中大部分是觉有针对性的转义使用，这些东西如何使用呢？，首先第一步就是在其固定的配置文件.babelrc中。下面是一个配置完整的babelrc内容

    {
      "presets": ["env", "react", "stage-0"],
      "plugins": [
        "transform-decorators-legacy"
      ],
      "env": {
        "development": {
          "plugins": [
            ["react-transform", {
                "transforms": [
                  {
                    "transform": "react-transform-catch-errors",
                    "imports": ["react", "redbox-react"]
                  },
                  {
                    "transform": "react-transform-hmr",
                    "imports": ["react"],
                    "locals": ["module"]
                  }
                ]
              }
            ],
            ["transform-runtime"],
            ["import",{
                "libraryName": "antd",
                "libraryDirectory": 'es',
                "style": true
              }
            ]
          ]
        }
      }
    }

## 2.3 webpack.config.js
webpack是我们使用的打包工具，其中主要规定了打包文件的目录和输出文件，其余的都是打包过程中的一些中间操作配置，比如默认共同引用一些文件，针对各类文件的各种处理加载器，尤其是推崇的将css in js方案。还有就是基本的babel编译等内容。

    /**
     * webpack配置文件
     * author : lei.li@fenbeitong.com
     * date : 2017.12.26
    **/
    var webpack = require('webpack');
    var path = require('path');
    
    let theme = {
      "primary-color": "#f09a37"
      // "highlight-color": "#f09a37"
    }
    
    var WebpackCfg = {
      context: path.join(__dirname, "app"),
      //页面入口文件
      entry: {
        "index": [
          "./scripts/index.js"
        ]
      },
      //出口文件输出配置
      output: {
        filename: '[name]Bundle.js',
        path: path.join(__dirname, 'app/bundles'), //真实路径
        publicPath: "/"  //访问路径
      },
      //插件部分
      plugins: [
        new webpack.ProvidePlugin({
          'React': 'react'
        })
      ],
      //模块
      module: {
        //加载器配置
        rules: [{
            test: /\.css$/,
            // 使用CSSmodule对CSS进行处理
            // loader: 'style-loader!css-loader?modules&localIdentName=[name]__[local]-[hash:base64:5] '
            use:['style-loader','css-loader']
        },
        {
          test: /\.js$/,
          use: ['babel-loader'],
          exclude: /node_modules/
        }, 
        {
          test: /\.(png|jpg)$/,
          use: 'url-loader?limit=8192' //图片文件小于8K，直接转码base64
        }, {
          test: /\.html$/,
          use: 'html-loader' //将html转为String
        }, {
          test: /\.less$/,
          use:['style-loader','css-loader',`less-loader?{"modifyVars":${JSON.stringify(theme)}}`]
        }]
      }
    }
    
    module.exports = WebpackCfg;

再生产环境中，一个固定的webpack配置时不够的，因为一般情况下 我们开发测试时，需要打包后的文件依旧能够清晰看到代码结构，而线上环境则需要压缩代码，保证引用大小的问题。所以在真正使用时，上述的config仅仅是基础的，我们还需要根据环境再行配置一些必要的参数或者插件，如下面是生产环境时使用的配置。

    /**
     * webpack生产配置文件
     * author : lei.li@fenbeitong.com
     * date : 2017.12.26
    **/
    var webpack = require('webpack');
    var baseConfig = require('./webpack.config.js');
    var UglifyJSPlugin = require('uglifyjs-webpack-plugin');
    
    // 生产环境追加sourceMap和压缩
    baseConfig.plugins.push(
      new webpack.DefinePlugin({
        'process.env.NODE_ENV': JSON.stringify('production')
      }),
      new UglifyJSPlugin({
        cache: true,
        sourceMap: true
      })
    );
    
    module.exports = baseConfig;

## 2.4项目基础目录

- app
    - images
    - bundles
    - scripts
        - layouts
        - redux
        - route
        - views
        - components
        - index.js
    - styles
    - index.html

1. index.html就是这个单页面应用的入口文件，主要作用是引入基础样式、某些ploy-fill方案的JS，项目打包的bundle文件等内容。
2. scripts文件夹
a. layouts 项目布局方案
b. redux 总redux文件及store文件
c. route 全局路由
d. views 容器视图，都是需要使用connect做链接的视图，主要负责请求数据，调用action处理数据，处理逻辑等内容。
e. 展示型组件，即没有直接参与数据处理，主要用于展示的组件。
f. index.js 全局入口JS文件，主要完成，热部署配置等全局公共配置、react-router并入redux状态管理配置、全局初始化数据加载和异常捕获、页面入口组件渲染等内容，下面看详细文件内容。


    import ReactDom from 'react-dom';
    import { Router, Route, Switch } from 'react-router-dom';
    
    import { Provider } from 'react-redux';
    import { ConnectedRouter } from 'react-router-redux';
    import { store, history} from './redux/configureStore';
    import MainLayout from './layout/MainLayout';
    import { Modal, Icon } from 'antd';
    import moment from 'moment';
    
    // 推荐在入口文件全局设置 locale
    import 'moment/locale/zh-cn';
    moment.locale('zh-cn');
    
    //告知该文件及其所有关联被修正时，进行热替换
    if (module.hot) {
      module.hot.accept();
    }
    
    // 加载系统基础数据，全局登录人员信息，基本企业信息，枚举常量等
    fetch('/api/getInitialData.json', {method: 'GET'})
    .then( response => response.json())
    .then( json => {
        window.initialData = json.data;
        ReactDom.render((
            <Provider store={store}>
                <ConnectedRouter history={history}>
                    <Switch>
                        <Route path='/'component={MainLayout} />
                    {/* <Route exact path='/loginIn'component={} /> */}
                    </Switch>
                </ConnectedRouter>
            </Provider>
        ), document.getElementById('container'));
    })
    .catch( msg => {
        ...
    })

其中ConnectRouter是react-router4.x的history并入redux管理的中间组件，主要是history、match等router使用的方法并入到redux的状态树中，使得页面跳转得以实现。

另一个重要的配置就是redux中configureStore文件，下面看文件内容：

    import { createStore, combineReducers, applyMiddleware } from 'redux';
    import { routerReducer, routerMiddleware } from 'react-router-redux';
    import ThunkMiddleware from 'redux-thunk';
    import rootReducer from './reducers';
    
    import createFetchMiddleware from 'redux-composable-fetch';
    import {createBrowserHistory} from 'history';
    
    // 向中间件中追加处理json的内容，否则payload内容未转换
    const FetchMiddleware = createFetchMiddleware({
        afterFetch({action, result}) {
            return result.json().then(data => {
                return Promise.resolve({
                    action,
                    result: data
                })
            })
        }
    });
    
    const history = createBrowserHistory();
    
    const store = createStore(
        combineReducers(Object.assign({},rootReducer,{
            routing: routerReducer
        })),
        applyMiddleware(
            ThunkMiddleware,
            FetchMiddleware,
            routerMiddleware(history)
        )
    );
    
    export {history};
    
    export {store};


未完待续





