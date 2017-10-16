# ReactTodo范例详解

## 1.讲在前面的话

我需要整理一份这次重写reactTodo的文档。
我将原有的积累通过这个小项目表现了出来，以为自己可以记住一辈子，但是今天偶尔翻过，却发现对项目许多细节变得越来越模糊，甚至对整体内容的设计也说不太清楚了，这非常恐怖，Backbone.js时期就是缺乏将积累变为文字的思维，导致我现在不敢说如何精通Backbone.js的开发工作。所以趁着还没全还回去，我将用心一步步记录这个reactTodo的每一个设计细节和结构特性。

## 2.开始

开始，从哪里开始呢？先说说为什么重写这个reactTodo吧。在以往的时间里，我写过ES5版的todo，以为那是充分体现react思维的最好模板程序，但如今看我错了。在这个大生态圈下，它在不断的变好，而我不能止步不前。以我现在的能力可能无法对React做出什么实质性的贡献，但我不能因此就放弃追随。我需要以此为契机，打开我狭隘的眼界，真正去探索，去融入这个生态圈。为人类进步做基石，实现我最大的价值观。

## 3. 结构
我深受MVC思想的影响，所以结构主要以此为依据，并根据功能和逻辑抽象若干组件。todos视图分为四部分，标题（TodoTitleView），输入框（TodoInputView）、Todo列表（TodoListView）以及列表底部操作工具栏（TodoFootbarView）。其中后三者之间有状态交互，为了避免不必要的diff和渲染，将其合并至Todo主体视图统一维护管理（TodoMainView），此处即为react常用优化手段之一，逻辑再抽象，避免不必要的渲染和diff操作。详细结构如下：

├TodoTitleView（标题）<br />
└TodoMainView（主体）<br />
&nbsp;&nbsp;&nbsp;&nbsp;├TodoInputView（输入框）<br />
&nbsp;&nbsp;&nbsp;&nbsp;├TodoListView（列表）<br />
&nbsp;&nbsp;&nbsp;&nbsp; |&nbsp;&nbsp;&nbsp;&nbsp;└TodoItemView（列表内容）<br />
&nbsp;&nbsp;&nbsp;&nbsp; |&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├TodoCompleteBtn（完成按钮）<br />
&nbsp;&nbsp;&nbsp;&nbsp; |&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├TodoTextInputView（Todo内容及修改）<br />
&nbsp;&nbsp;&nbsp;&nbsp; |&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└TodoDeleteBtn（删除按钮）<br />
&nbsp;&nbsp;&nbsp;&nbsp;└TodoFootbarView（底部工具栏）<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├TodoCountView（完成数视图）<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├TodoFilterView（查询）<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└TodoClearBtn（清空所有完成项按钮）<br />

接下来针对有特点的视图一个个分析出来，里面包含优化手段，结构特性，高级用法等等内容。所以并不会逐个将优化手段而是根据代码碰到那个说那个，这适合有一定基础的同学学习观看，关于各个优化手段详细的原理可以单独去翻阅文档，这里注重实践。

## 4.代码分析
### 4.1 标题组件
我看过无数todo的实现方案和方法，一般情况下很少有将标题单独提取组件的，在这里可以理解为抽象原则的刻板规范。通过这么细致的抽象原则，简化技术栈学习内容，更容易理解其应用形式和最根本的设计思想。在正式的项目中，可以根据业务需要或者组件逻辑抽象原则来进行更符合项目本身设计需要的抽象组件。颗粒化越细，确实灵活性越高，复用率越高，但这会极大增加开发和使用元组件的复杂度，增加很多为了抽象而不断复杂的兼容逻辑和组合代码。得不偿失，过犹不及。所以这里这么拆分完全是为了理解方便，切不可照猫画虎。

标题的代码非常少，主要内容如下：

    import styles from './TodoTitle.css';
    import PropTypes from 'prop-types';
    
    const TodoTitleView = (props) => (
        <h1 className = { styles.headTitle } > {props.todoTitle} </h1>
    );
    
    TodoTitleView.propTypes = {
        todoTitle: PropTypes.string.isRequired
    }
    
    export default TodoTitleView;

这里讲述代码中第一个要点——**无状态组件**。
无状态组件即过程中没有state参与，仅接收父组件的props传递的内容渲染结果。这不仅仅是写起来很简单，省去很多代码，更是从根本上省去了React组件生命周期方法。没有状态的更迭，就没有自行触发生命周期的过程，它需要的仅仅是props改变时render。请注意，我觉得这里需要深入关注一个问题：省略组件生命周期方法究竟意味着什么呢？意味着从MOUTING（渲染）到RECEIVE_PROPS（更新）再到UNMOUTING（回收）之间绝大部分的生命周期方法都不执行了，单纯的接收数据渲染。如果你看过这部分源码，你会发现这绝对不是一小部分的逻辑代码，它包含了数据对比，渲染判定，状态队列管理等等渲染、更新、回收的代码。这些源码我也仅仅是了解了个大概。我觉得在不需要精确优化生命周期的情况下，目前这些够用了。

*题外话：getDefaultProps为何只执行一次？*

*因为它不在上述所有生命周期过程中。它是创建组件时由构造方法管理的，这些重复的过程中调不到它。*

这里面还有一些其他的细节，首先样式我并没有直接使用名称而是通过调用的方式使用的。样式使用CSSModule的格式书写，关于CSSModule的相关内容详见第5部分。其次代码中特意追加了针对propTypes的验证部分，否则视图整体连个displayName都不用定义直接返回一个匿名函数即可。propTypes验证是保障视图顺利渲染，快速纠正数据错误的关键关卡。这简简单单的数据验证，看着没什么，但是单项数据流的架构中，视图渲染的必要条件是数据，如果数据不正确或者缺失，轻则渲染失败或错乱，重则后续关联数据全部错误，别忘了这些数据是一个个状态树，一个父节点完蛋，后面全完蛋。所以添加它是必须的，同时还有一个好处，在（...）这种ES6语法糖泛滥的情况下（详见ES6扩展运算符），能够知道该组件究竟传入些什么数据，这非常有利于许多数据型BUG的追踪。当然数据太多的时候，如何合理的合并或者复用一些验证，我还没有想好，希望今后这部分能够补全，给一个合理而简单的应对方案！

### 4.2主体容器
介绍结构的时候阐述过这个容器组件的来历和用途，这里需要说明，在没有类似Redux、Flux这些统一管理状态树的情况下，这是一种优化状态树及渲染逻辑的方式。如果将他们与TodoTitle平级使用，那么顶层容器在接收这三个组件之间的数据转换时，会一直携带者永远不会变的Title组件，虽然它不会真正的重新渲染一遍，但是至少每次都会diff一遍，目前看省略diff的部分很小可是如果涉及的内容很大呢。这是很多性能问题的主要成因之一，不能仅仅根据视图结构去划分组件，有时候需要冗余的容器型组件去解耦数据。那么混入Redux的情况下呢，其实原理类似，只不过状态树是一个。至于每改变一次state都会diff一次，那另有对应的解决方案，后续会讲到。

已经提及好几次diff这个词了，这是React高效渲染最重要的一环。diff这种做法早有，react把它升华了一下，将原来O(n^3)的计算量变为O(n)，它主要遵循三个策略：

1. 针对WebUI中DOM节点跨层级的移动特别少，选择忽略不计。
2. 拥有相同类的两个组件将会生成相似的树形结构，用后不同类的两个组件将会生成不同的树形结构。
3. 对于同一层级的一组子节点，可以用唯一ID区分。

用上面的策略进行diff算法的优化可以极大的降低计算的复杂度。diff算法可不是一个，从状态树的tree diff，到组件的component diff，再到元素的element diff，都会按照这三个原则去优化。 

- tree diff：使用策略1优化后，仅对树进行同级层次的对比，而不是一层层循环。那如果是有跨层级的移动呢，官方建议尽量不要这么做，因为针对这种情况React的处理方法是删除原有节点并重新建立被移动的位置后的节点，这会带来性能问题，这也是对于技术的取舍问题，DOM树的稳定有助于性能的提升，所以可以原则隐藏或显示节点来做这些事情，而不是真正的删除或添加节点（两者结合即移动）。

- component diff：即对比Virtual DOM，哪里有问题就更新哪里（包含其子组件在内），由于Virtual DOM数据结构也很繁琐，提早的知道哪些不用更新就是提升React性能的最大要点，那么在哪里控制呢？就是每个component都有的shouldComponentUpdate方法。这里可以拦截diff，拦截不必要的渲染，至于如何高效的判定是否需要拦截，后续会讲解。

- element diff：最末级的diff算法，这里需要说明几个逻辑：
 1. INSERT_MARKUP： 新组件类型不在旧集合里，即全新的节点，需要对节点进行插入操作<br />
 2. MOVE_EXISTING ： 旧集合中有新组件类型，且element为可更新类型，generateComponentChildren 已经调用receiveComponent，prevChild=nextChild，做移动操作，复用以前的节点。（比较拗口，可以参看相关解析文章再行了解学习）
 3. REMOVE_NODE ：旧组件类型，在新组件类型中存在，但对应的element不同则不能直接复用和更新，需要删除。或者根本不在新组件类型中，同样执行删除操作。

这些都是逻辑，实际应用我就不在这里背书了，以上这些确实很难一下子理解，建议参看《深入React技术栈》第3.5章diff算法部分，后面的实际举例会非常详细的说明上述三个逻辑的运转过程。 

这里还要指出一个重要的优化点，就是给同级节点编写唯一的id，这样几个节点间位置移动时，不会用删除+添加的方式完成，而仅仅的改变顺序。

OK，回过头来我们接着说TodoMainView，它做的事情可就多了，我先贴出一部分代码来

    import { Component } from 'react';
    import TodoInputView from './TodoInputView';
    import TodoListView from './TodoList/TodoListView';
    import TodoFootbarView from './TodoFootbar/TodoFootbarView';
    import { ALL, ACTIVE, COMPLETED } from './Constants';
    import Immutable from 'immutable';
    import {immutableRenderDecorator} from 'react-immutable-render-mixin';
    
    @immutableRenderDecorator
    export default class TodoMainView extends Component {
        constructor ( props, context ) {
            super(props, context);
            this.addTodo = this.addTodo.bind(this);
            this.deleteTodo = this.deleteTodo.bind(this);
            this.updateTodoValue = this.updateTodoValue.bind(this);
            this.completeTodo = this.completeTodo.bind(this);
            this.getCompletedCount = this.getCompletedCount.bind(this);
            this.changeFilterType = this.changeFilterType.bind(this);
            this.getShowTodoList = this.getShowTodoList.bind(this);
            this.clearCompletedTodos = this.clearCompletedTodos.bind(this);
        }
    
        state = {
            data: Immutable.fromJS({
                todoList: [],
                completedCount: 0,
                filterType: ALL
            })
        }
        . 
        .
        .

我们可以看到，关于todo的增删改查全在里面实现的。所以一个状态的改变可以同时出发若干相关组件的更新操作。那么与这个状态无关的组件怎么办呢。虽说经过component diff对比后不会更新，但毕竟浪费了一次diff的计算。有人说，我们可以在shouldComponentUpdate中去对比前后的props和state来拦截不必要的更新。那如果props和state数量很多呢？是的虽然手写可以达到非常准确拦截的效果，但是恐怖的逻辑复杂度，或许还会增加性能问题。这里有一个取舍的办法——purerender，purerender将props和state分别对比value和key的length，但是仅仅是对比第一层不会深入非基本类型结构数据中去逐层对比。它放弃了深层次的对比，降低了对比的逻辑复杂度。有舍有得，是一个可以值得使用的方法。同时针对purerender我们也有一些使用细节要注意，比如不要在JSX中计算或断言了，将数据准备好后一次性传入，否则每次都是重新计算后的结果，就算数据是一致的，但是地址变了，依旧没能被purerender过滤掉。你或许发现了，我说的purerender并没有体现在我写的代码中，为啥？因为有更好用的，它就是Immutable！

Immutable，顾名思义不可变数据。每当对immutable对象进行更新等操作时，它将返回一个全新的immutable对象，它使得旧数据不变，同时避免深拷贝带来的性能风险。返回的对象与就对象重复的节点都会因为结构共享而进行共享，仅改变更新的数据与受影响的父节点，其优势有以下几点：

1. 通过不可变数据降低可变数据带来的未知风险，例如篡改数据等。
2. 解约内存，immutable使用结构共享尽量节省内存，没有被引用即被GC回收。
3. 时间旅行，因为是保留原始数据，所以撤销什么的就非常容易实现了。
4. 函数式编程，我是越来越理解这种编程理念了，输入什么即输出什么，准确安全。

有利就有弊，缺点就是与普通JS对象容易混淆，因为操作对象的API不同所以会怕混淆。

我们来看一下具体应用，我在代码中引用了immutable并使用fromJS方法将state的data数据构建成了一个Immutable对象，为啥不直接将state写成immutable对象？ 因为state本身需要可变（setState）。这样在shouldComponentUpdate中通过length和value比较后，非常高效的完成了拦截判定操作。为什么高效因为immutable采用的是trie数据结构，只要2个对象的hashcode相等，值就是一样的，不用深层遍历比较，所以性能非常好。那么结合purerender的理念和immutable的数据结构，我们通过重写shouldComponentUpdate方法就可以高效进行性能优化工作。关于immutable操作的API可以看View中具体方法的实现，其中最有意思的是setState的方法，我经常写错，这是ES6简化后的方法：

    this.setState(({data}) => ({
        data: data.update('todoList', list => list.setIn([index,'value'], newValue))
    }));

针对上面这段代码，尤其是({data})这块，我给两个字：解构。

如果你注意观察，我的代码里并没有重写而是在Component顶部追加了一个decorator。这是针对ES6Class的一个高阶组件混入写法。他直接覆盖了shouldComponentUpdate方法。那高阶组件又是怎么工作的，除了方便混入方法外，它其他的功效又是啥，接下来一一介绍。

高阶组件、高阶函数、高阶Reducer，其实实现原理是一样的。高阶函数是指接收函数作为输入，或者输出一个函数的函数。高阶组件就是接收一个React组件为参数，输出新的React组件。高阶组件的实现方式主要有以下两种

1. 属性代理

   直接看范例：

    const MyContainer = (WrapComponent) => 
        class extends Component {
            render () {
                return <WrapComponent {...this.props}>
            }
        } 

 传入一个React组件作为参数，然后通过属性代理的方式，控制了props。这样通过高阶组件我们就可以封装一些公共的处理逻辑等，还可以抽象state，让组件变成展示型组件，简化代码，提升性能。这里面还有一个点，我目前尚未理解透彻，就是通过调用refs拿到实例，然后修改实例的props或调用实例的方法。

2. 反向继承

范例

    const MyContainer = (WrapComponent) => 
        class extends WrapComponent {
            render () {
                return super.render();
            }
        } 

你可以看到通过反向继承的方式，高阶组件劫持了render方法，这是非常危险的一种使用方式。反向继承可以控制组件的所有方法参数，包含state，如果使用不当，后果可想而知。所以一边情况下，我们还是老老实实的用属性代理的方式做一些高阶组件用用。

至于高阶组件的实际应用我觉得我以后要在一个完整的项目中多写几个之后，再来补充比较好。介绍了这么多内容，高阶组件从架构层面究竟怎样影响着我们的组件拆解工作呢，下面一张图甚好，图片同样来自《深入React技术栈》这本书，高阶组件带来了组合式组件架构，而不是单一的继承。
![组合式组件架构](https://i.imgur.com/H0mdS9o.png)

说道这里是不是觉得不要轻看任何一行代码。这背后有太多的技术隐藏着，这也使得许多人貌似知晓其实就是个表面。
MainView的主要内容就是这些了，或者说JS部分所有的技术要点就这些了，剩下的就是在这个规则下增加功能，输出代码了。

## 5.JS之外
为什么要提及这个因为我还欠着一个CSSModules以及一个完整的webpack热部署开发方案。我们先说CSSModules。未完待续


##6. 顶层容器