# interview

最近准备跳槽，就意味着要进行面试，趁着出差比较闲，整理了一些前端面试资料，包含了 react、redux、router、webpack 等前端常用技术。



## react



### React18新特性

1. createRoot 创建根节点

~~~js
const rootElement = document.getElementById("root");
const root = ReactDOM.createRoot(rootElement);
root.render( <StrictMode> <App /> </StrictMode> );
~~~

2. 节点自动批处理，`flushSync` 取消自动批处理

3. 新的 Suspense List，这样可以挂起更多的组件

~~~js
<SuspenseList revealOrder="forwards">
  <Suspense fallback={'Loading...'}>
    <ProfilePicture id={1} />
  </Suspense>
  <Suspense fallback={'Loading...'}>
    <ProfilePicture id={2} />
  </Suspense>
   ...
</SuspenseList>
~~~

4. startTransition 
5. useDeferredValue 允许延迟更新

~~~js
import { useState, useDeferredValue } from "react";
...
let [value, setValue] = useState(0);
  const deferredValue = useDeferredValue(value, { timeoutMs: 2000 });
  return (
    <div className="App">
      <div>{deferredValue}</div>
      <button onClick={()=>{setValue(deferredValue+1)}}>click me</button>
    </div>
  );
...
~~~



### setState 是同步还是异步？

* 合成事件中是异步
* 生命周期函数中是异步
* 原生事件中是同步
  * setTimeout setInterval 是同步（class同步，state马上更新；hook需等到下次渲染才更新）



### React 生命周期

**挂载阶段**

1. constructor
2. getDerivedStateFromProps（静态方法，传入 props、state，返回的数据映射到 state 上）
3. render
4. componentDidMount

**更新阶段**

1. getDerivedStateFromProps
2. shouldComponentUpdate（返回 true/false 决定组件是否渲染）
3. render
4. getSnapshotBeforeUpdate（将返回结果传递给 componentDidUpdate 作为其第三个参数）
5. componentDidUpdate

**卸载阶段**

1. componentWillUnmount



### fiber 是什么？



为什么？

react15 架构分为 stack reconciler 递归更新对比和 renderer 渲染；

缺点出现在**递归更新**（深度遍历），react15 对创建和更新节点是通过递归实现；

递归一旦开始，在遍历完完整棵树之前是不会停止的；

该任务一直占用主线程，导致无法响应优先级更高的任务；

从用户角度看，如果递归栈很深，就容易出现卡死，十分影响用户体验；



react16 怎么做？-- fiber 架构 

1. 将大任务拆分成 N 个小任务，每个小任务执行时间很短；

2. 以帧为单位，如果一帧里没有优先级更高的任务，则开始执行小任务；
3. 如果有其他优先级高的任务，优先执行其他，等到有空闲的时候再执行自己；



### 聊一聊 diff 算法

传统的 diff 算法复杂度是 O(n^3)，放弃最优解后，时间复杂度是 O(n)。

1. tree diff：比较 dom 节点，节点不存在直接删除，无需进入下一步，一次遍历即可。
2. component diff：组件类型不同直接删除，并创建新的节点。
3. element diff：通过唯一 id 区分，如果没有 id，有插入动作会导致位置后的列表重新渲染。



### 什么虚拟DOM？有啥用？

真实 DOM 在内存中的表示，实际上与真实 DOM 同步。

作用：虚拟 DOM 相当于在 JS 和 真实 DOM 之间加了个缓存，这样 diff 算法就可以在虚拟 DOM 上运行了，避免对真实 DOM 的干扰，提升了性能。



### state 和 props 有啥区别？

state 是组件内部管理的状态，可变。

props 是外部组件传入的数据，不可变。



### React 组件的通信方式

1. 父传子：props
2. 子传父：通过调用 props 传过来的函数传递需要的参数到父组件作用域里
3. 跨层级通信：context 上下文，redux 状态管理



### 什么是高阶组件？

高阶函数是接收一个组件参数，经过处理，最终返回一个相对增强的 react 组件。



### React 中的 refs 有啥用？

父组件修改子组件唯一的方式是通过 props，当需要强制修改子组件，而 props 满足不了需求时，可以用 refs 获取子组件的实例

也可以在组件中添加 ref 属性，属性值是一个回调函数，接受第一个参数作为组件的挂载实例



### 类方法为啥要绑定到类实例？

JavaScript 中 this 的值会根据当前上下文变化，开发人员希望通过 this 引用组件的当前实例，所以将方法绑定到实例，这一步通常在构造函数中完成



### 构造函数的 super 干啥的？

在你调用 super() 之前，你无法在构造函数中使用 this，JS 不允许这么做。

执行 `super(props)` 子类就能使用 **this.props** 来获取 props



### React 父组件如何调用子组件的方法

1. 方法组件中调用：useRef 、useImperativeHandle、forwardRef

~~~typescript
// child.tsx
const Child = (props: any, ref: any) => {
  useImperativeHandle(ref, () => ({
    getAlert() {
      alert('getAlert from Child');
    },
  }));
  ...
};
export default forwardRef(Child);

// father.tsx
export default function() {
    ...
	const childRef: MutableRefObject<any> = useRef(null);
	...
	<Child ref={childRef} />
	...
}
~~~

2. 类组件中调用：React.createRef

~~~typescript
// Father.tsx
export default class Father extends React.Component<{}, {}> {
  childRef: React.RefObject<any>;
	...
  <Child ref={this.childRef} />
    ...
}
~~~



### React 有哪些优化手段

**类中的优化手段**

1. 使用 PureComponent 作为基类，减少渲染次数，class 传值如果是地址引用类不会触发
2. 使用 React.memo 包装组件，减少渲染次数
3. 使用 shouldComponentUpdate 自定义渲染逻辑

**函数组件中的优化手段**

1. useMemo 减少子组件渲染次数，避免不必要的渲染
2. memo + useCallback 减少渲染次数

**其他手段**

1. map 渲染列表时使用唯一 id
2. 必要时通过 css 隐藏组件，而不是通过条件判断
3. Suspense + lazy 进行懒加载

~~~typescript
if () {
    ComponentA = lazy(() => import('./component'))
}
	...
	<Suspense fallback={<div>Loading...</div>}>
        <ComponentA />
    </Suspense>
~~~



### PureComponent 和 Component 的区别

PureComponent 中会自动执行 shouldComponentUpdate，对更新的 props 和 state 进行浅对比，相同的话不会重新 render，反之触发 render；如果 props 和 state 涉及到对象，浅对比是相等的，将不会触发 render。

所以，使用 PureComponent 需满足一下条件：

1. props 和 state 是不可变对象
2. props 和 state 不能有层级嵌套对象



### React 元素为啥有 $$typeof 属性

为了防止 xss （*指通过网页开发的漏洞，注入一些恶意的代码指令*）攻击。react 通过 $$typeof 判断该元素是否来自 react，如果不是，react 会拒绝。



### React 如何区分 class 组件和 funciton 组件

~~~javascript
// instanceof 可以用来判断在继承关系中一个实例是否属于它的父类型
components.prototype instanceof React.Component 
true // class 组件
false // function 组件
~~~



### React 和 HTML 在事件处理上有啥区别？

1. 事件命名：HTML 中事件必须小写，React 中遵循驼峰式命名
2. 阻止事件默认行为：HTML 使用 return false，react 中使用 event.preventDefault()



### 什么是 Suspense 组件

Suspense 让组件「等待」某个异步操作，操作结束即可渲染。主要用于懒加载。

~~~javascript
// 等待某个异步操作
const resource = fetchProfileData();
function ProfileDetails() {
  // 尝试读取用户信息，尽管该数据可能尚未加载
  const user = resource.user.read();
  return <h1>{user.name}</h1>;
}
function ProfilePage() {
  return (
    <Suspense fallback={<h1>Loading profile...</h1>}>
      <ProfileDetails />
    </Suspense>
  );
}

// 懒加载
	...
~~~



### 为啥 JSX 的组件名要首字母大写？

react 要知道当前渲染的是组件还是 HTML 元素。



### React 中代码是如何模块化和组件化？

使用 export 和 import 属性来模块化代码，有助于在单独文件中编写组件。

可以按逻辑不同、业务不同将代码放到不同的 js 文件里，比如写一个公共的工具函数模块（业务）。

组件化是将经常复用的组件代码单独抽离出来，封装成组件，方便其他页面引用，减少代码量。



### this.setState 的原理是啥？

setState 是通过一个队列机制（先进先出）实现 state 更新，当执行 setState 时，会将需要更新的 state 合并后放入状态队列中，而不是立即更新 state，队列机制可以批量更新 state；

如果使用 this.state 改变 state，那么该 state 将不会放入状态队列机制中；



### this.setState 应该注意什么？

1. `setState` 是大部分情况下是浏览器事件/异步的，批处理；setTimeout 同步；
2. 每次调用都会造成重新渲染，即使 state 值不变，需要通过 `shouldComponentUpdate` 控制；hook 解决；
4. 使用 Hook 的 useState 如果值不变（基本数据类型）是不会触发重新渲染的；



### 受控组件和非受控组件

**受控组件**

* 没有自己的状态
* 数据由父组件传入

**非受控组件**

* 有自己的状态
* 数据由 DOM 操作控制



### 什么是 React Hooks？

允许开发者在不用编写类的情况下使用 state 和其他 react 的特性，且代码量更少，更容易实现复用



### useEffect 和 componentDidMount

1. 执行时机

   useEffect 在挂载后运行，但执行时间更加往后，如果此时在里面改变 state 状态，会出现闪屏

   componentDidMount 也是在挂载后运行，如果改变 state，React 将会额外出发一次 render，并将第二次的 state 作为初始的 UI，所以不会发生闪屏

2. props 和 state 的获取

   componentDidMount 每次读取的都是最新的 state 和 props

   useEffect 创建时已经获取了 state/props 的值，并存在缓存中



### hooks 为啥不能放在条件判断里？

useState 初始化的时候，会创建两个数组：state（存放 state 值）和 setters（存放 setState 函数），并把光标 cursor 设为 0；

第一次：首次渲染 useState 时，会将 state 初始值放入 state 数组中，将 setState 函数放入 setters 数组中；光标 cursor++；

后续渲染：每一次重新渲染，光标重新设为 0，hook 根据光标去读取数组中对应的 state 和 setters；

事件调用：每次调用 set 函数，set 函数会去修改 state 数组中对应的值，这种关系是通过 cursor 对应的，如果因为条件判断没有读取到本应该出现的 state 和 setters 那么取值就会出现偏移。



### useEffect 依赖项的说明

当依赖项是基本数据类型，依赖项值变化（改后的值与原值不等）则触发 useEffect 更新

当依赖项是对象/数组，只要依赖项有更新，即使值不变，也会触发 useEffect 的更新

当依赖项是函数，要看函数的依赖项（通常需要 useCallback 包裹）是否有触发函数的更新，函数有更新则触发useEffect 的更新 



### Hook 相对于 class 有什么优点和缺点？

优点

1. 更容易复用代码（class 高阶：祖父=>父=>子）
2. 代码量更少更容易理解
3. 不需要考虑 this 的指向问题

缺点：

1. 状态不同步：定时器获取 state 的例子，先点击 3s 后获取 state，再改变 state，state 没有同步（useEffect+useRef 解决）
2. useEffect 如果依赖太多会容易触发（减少依赖）



------



## redux



### redux 是啥？

独立于 UI 框架之外的状态管理容器，提供状态管理，兼容性强，体积小



### Redux三大原则

**1. 单一数据源**

所用应用 state 都存储在 object tree 中，这棵树存在于唯一的 store。

**2. state 是只读的**

唯一改变 state 的方法是触发 action，action 用来描述已发生的事件。

**3. 使用纯函数进行修改**

编写 reducer 来描述如何改变 tree，reducer 是一个纯函数，接受旧 state，返回新的 state。



### Redux的四个组件

**1. Action**

是一个普通的 js 对象，将数据传输到 store，type 表示动作，value 表示传参。

**2. State**

存储在 store 中的数据：server-data、ui-data、app-data

**3. Reducer**

本质是一个函数，响应发送过来的 actions，返回更新后的 state。

**4. Store**

数据中心，将 action 和 reducer 绑定到一起的对象。

绑定在 store 上的方法有：

* store.dispatch() 发送 action 给 store
* store.subscribe()  监听 store 的数据变化 
* store.getState() 获取 store 上的 state 数据
* store.replaceReducer()



### 什么是 MVC、MVP、MVVM？

**MVC** 可分成三个部分：

1. 视图 View：用户界面
2. 控制器 Controller：业务逻辑
3. 模型 Model：数据保存

![img](https://www.ruanyifeng.com/blogimg/asset/2015/bg2015020105.png)

**MVP**：View、Model、Presenter

Presenter 可以理解为中间人，负责 View 和 Model 之间的数据流动，防止其之间的直接交流

![img](https://www.ruanyifeng.com/blogimg/asset/2015/bg2015020109.png)

**MVVM**：View、Model、VM（ViewModel）

和 MVP 模式基本保持一致，唯一区别就是采用双向绑定，View 的变动反映在 ViewModel，反之亦然

![img](https://www.ruanyifeng.com/blogimg/asset/2015/bg2015020110.png)

### Redux 存什么数据？

1. 配置信息，如用户信息、token，尽量单独记录
2. 组件之间可能被共享的数据



### 什么是 Flux？

是一种架构思想，和 MVC 架构是同一类东西，最大的特点是数据的「**单向流动**」。和 redux 的模式比较接近。

分成 4 个部分：

1. View: 视图层

2. Action: 视图层的发出的动作

3. Dispatcher: 用来接收 Actions，执行回调函数

4. Store: 存放应用的状态 state，发生变化将触发 View 更新页面



### redux 工作流程

1. 用户（View）以 dispatch 的方式发出 Action：store.dispatch
2. reducer 接收 action，返回新的 state
3. state 发生变化，调用监听函数，更新 View

整个流程中数据保持单向流动，易于追踪错误，数据逻辑变得更加清晰



### 组件如何与 redux 链接

通过 connect，connect 是一个高阶组件，它将 state 和 action creators 绑定到组件的 props 上，这样组件就能直接访问 store 的数据了，store 数据的更新也会触发组件的刷新



------



## react-router



### react-router 核心组件？

路由：<BrowerRouter> <HashRouter>

路由匹配：<Route> <Switch>

导航： <Link> <NavLink> <Redirect>



### 怎么实现代码分割？

1. 动态加载模块：webpack 已经提供内置的动态模块加载的功能，但如果你使用的是 babel，就需要用到 @babel/plugin-syntax-dynamic-import 插件
2. 动态加载组件：使用 loadable-components 库。

~~~javascript
import loadable from '@loadble/component'
const RenderComponent = loadable(() => import('./com'), {
    fallback: <Loading />
})
~~~



------



# Vue



### MVC 和 MVVM 的区别

MVC：View 传递事件触发 Controller，Controller 触发 Model 层事件，Model 层更新完数据之后触发 View 更新页面。具有单向数据流动的特征。

MVVM：View 和 ViewModel 相互绑定，自动同步，简化了业务和页面之间的依赖。

Vue 并没有完全遵循 MVVM 的思想，因为 Vue 提供了 `$refs` 属性，让 Model 可以直接操作 View，违反了 MVVM 的规定。



### 为什么 data 是一个函数

组件中的 data 必须写成一个函数，数据以返回值形式定义，这样每复用一次组件就会返回一个新的 data，相当于给每个组件实例创建一个私有的数据空间，让各个组件维护各自的数据。如果 data 写成对象形式，所有组件会共用一份 data，会导致数据混乱



### Vue 组件通信的几种形式

1. 父传子：`props` 
2. 子传父：`$emit `
3. `$parent` `$children` 获取当前组件的父组件、子组件
4. `$attrs` 和 `$listeners`
5. `$refs` 获取组件实例
6. eventBus 事件总线
7. vuex 状态管理



### Vue 生命周期

beforeCreate：初始实例化之前，此时 data/methods/computed 都不可访问

created：实例创建完成可访问 data/methods

beforeMount：对 DOM 操作无效，此时页面呈现的是未经 Vue 编译的 DOM 结构

mounted：页面呈现的是经 Vue 编译后的 DOM，对 DOM 操作均有效。一般在此阶段做一些初始化操作

beforeUpdate：数据是新的，页面是旧的

updated：数据是新的，页面是新的

beforeDestory：data/methods/computed 都是可用状态，马上要执行销毁了。一般在此阶段关闭定时器、取消订阅、解绑自定义事件等

destoryed：销毁



### v-if 和 v-show 的区别

`v-if`：编译过程中会被转化为三元表达式，条件不满足不渲染此节点，适用于运行时无需频繁切换的场景

`v-show`：被编译成命令，条件不满足被隐藏，适用于频繁切换的场景



### Vue 的内置指令

`v-once` 所定义的元素/组件只渲染一次，包括元素/组件下的所有子节点

`v-cloak` 配合 CSS，解决初始化慢导致页面闪动的问题

`v-bind` 动态绑定属性

`v-on` 事件绑定

`v-html` 插入 html 代码，注意防止 xss 攻击

`v-text` 更新元素的文本内容

`v-model` 双向绑定数据

`v-if/v-else/v-else-if` 

`v-show` 最终通过 display 控制显隐

`v-for` 循环

`v-pre` 跳过该元素的编译过程



### Vue 中单向数据流怎么理解

数据总是从父组件流向子组件，子组件不可修改父组件传递的 props，子组件只能通过 `$emit` 通知父组件去修改。数据不可逆向传递。



### computed 和 watch 区别和使用场景

computed：

- 计算属性，支持缓存。只有当依赖的值发生变化才会重新计算
- 不支持异步，异步操作无效
- 可以设置 `getter` 和 `setter`
- 如果一个属性是依赖其他属性计算得来，多对一或一对多，一般使用 computed

watch：

- 不支持缓存，监听到值的变化就会立即触发相应操作
- 支持异步
- 监听函数接受两个参数，最新状态和上一个状态的值
- 当一个属性发生变化，需执行多个对应操作，即一对多，一般使用 watch



### v-if 和 v-for 为啥不建议一起使用

解析时 `v-for` 比  `v-if` 优先级更高，每次 `v-for` 都会执行 `v-if`，造成不必要的计算，影响性能。

如果遇到需同时使用的场景，可以写成计算属性的方式。或将 `v-if` 置于外层。





# ES6



### var let const 区别

1. let 和 const 声明形成块级作用域；var 有全局作用域、函数作用域，没有块级作用域的概念
2. var 声明变量存在变量提升，let const 没有。即使用 var 声明就初始化为 undefined，let 和 const 要到执行阶段才初始化
4. 同一作用域下，let const 不能声明同名变量，var 可以
5. const 声明后不能修改，为复合类型数据可以修改属性



### 箭头函数 this 的固化

用箭头函数定义时，this 指向是定义时所在的对象，而不是使用时



### JS 深拷贝和浅拷贝

什么是浅拷贝：简单说就是复制引用，当你修改了赋值后的对象/数组，源对象/数组也会相对应的修改

深拷贝的几种方法：

1. JSON 方法

~~~javascript
JSON.parse(JSON.stringify(object))
~~~

有些情况并不适用，如对象某个属性是函数

2. 手写 deepCopy

~~~javascript
const deepCopy = (obj) => {
    // 如果不是数组/对象，返回原始值
    if (typeof obj !== 'object') {
      return obj;
    }
    const newObj = obj.constructor === Array ? [] : {};
    // 如果是数组
    if (obj.constructor === Array) {
      for (const key of obj.keys()) {
        if (typeof obj[key] === 'object') {
          newObj[key] = deepCopy(obj[key]);
        } else {
          newObj[key] = obj[key]
        }
      }
    }
    // 如果是对象
    if (obj.constructor === Object) {
      for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
          if (typeof obj[key] === 'object') {
            newObj[key] = deepCopy(obj[key]); 
          } else {
            newObj[key] = obj[key]; 
          }
        }
      }
    }
    return newObj
  }
~~~



### 解构赋值可以深拷贝吗？

解构赋值是有局限性的深拷贝。

1. 如果数组或对象内部的都是基本数据类型的话，可以实现深拷贝
2. 出现引用类型或嵌套类型的话将不可作为深拷贝



### 说说 async/await ？

async 返回的是一个 promise 对象，不论有没有使用 await

await 后面如果是一个 promise 对象，则返回对象结果；否则，直接返回对应的值

当 await 后面的 promise 变为 reject 状态，整个 async 函数都会中断执行，解决方法：`try catch`



### Promise 并发控制

Promise.race()：将多个 Promise 实例以数组形式包装成新的 Promise 实例，race 是赛跑的意思，即数组里哪个 Promise 实例获取最快就返回该结果，不论结果是 resolve 还是 reject。可用来作并发控制。

Promise.all()：将多个 Promise 实例以数组形式包装成新的 Promise 实例，当数组里所有 Promise （逐个调用）都返回 resolve 状态则成功。其中有一个为 reject 状态结果为失败。如果 Promise 实例参数中自己定义了 catch 方法，则不会触发 Promise.all() 的 catch 方法。



------



# HTTP





### 什么是跨域？为什么？怎么解决？

是什么？即跨域访问，如果两个服务协议（http/https）不同，或域名不同，或端口不同，此时访问另外一个服务器就会产生跨域。如果域名和端口相同，只是请求路径不同，就不属于跨域。

为什么？浏览器自身对 http 请求的一种安全限制，若没有该限制，任意网站的 js 脚本都可以对其他任意网站进行恶意攻击。

CROS 简单请求需满足条件：

* 使用以下方法之一：GET/POST/HEAD
* 请求 Header 包含：
  * Accept
  * Accept-Language
  * Content-Language
  * Content-Type（Application/x-www-form-urlencoded、multipart/form-data、text/plain）

解决方案？

1. nginx 反向代理，将原本跨域的请求代理为不跨域，需在 nginx 服务器上额外配置
2. CROS，主流跨域解决方案，分简单请求（head/get/post）和预检请求（非简单请求）
   1. 简单请求（HEAD/GET/POST），Access-Control-Allow-Origin 要么设为 *，要么设为请求时 Origin 的值
   2. 预检请求（DELETE/PUT/OPTIONS），Access-Control-Request-Method 列出 CROS 需要用到的 Http 方法



### HTTP 缓存

HTTP 缓存指的是，当客户端向服务器请求资源时，会先抵达浏览器缓存，如果浏览器有请求资源的副本，则直接在浏览器缓存中提取，而不是请求服务器获取资源。

根据是否需要重新向服务器发起请求来分类，可分为**强制缓存**和**协商缓存**，由 HTTP 头部字段控制；

强制缓存是在缓存数据未失效的情况下，直接使用浏览器的缓存，不再向服务器发送任何请求，强制生效时，http 状态为 200，这种方式是最快的，性能也很好；

协商缓存，在浏览器发起第二次请求时会与服务器协商，与服务端对比资源是否更新，如果服务端资源没有更新，则返回 304 状态码，告诉浏览器可以使用缓存中的数据。否则，服务器直接返回数据。

强制缓存生效就无需和服务端交互，协商缓存不论是否生效，都必须和服务端交互。



### HTTP 有哪些方法？

* HTTP1.0
  * GET：通常用于请求服务器获取某些资源
  * POST：发送数据给服务器，可用来更新或插入操作
  * HEAD：获取请求资源的头部信息，比如下载一个大文件之前先获取其大小再决定是否获取，达到节约带宽资源

* HTTP2.0：
  * PUT：用来替换目标资源，更新请求必须提供唯一标识
  * DELETE：用于删除指定资源
  * OPTIONS：用于获取目标资源所支持的通信选项
  * TRACE：回显服务器收到的请求，主要用于诊断或测试
  * CONNECT：预留给能够将连接改为管道方式的代理服务器



### GET 和 POST 有什么区别？

1. 数据传输方式不同：GET 通过 URL 传输数据；POST 数据通过请求体传输
2. 安全性不同：GET 数据在请求体内暴露在 URL 中；POST 数据在请求体内，有一定的安全性保证
3. 是否幂等：GET 只读特性是安全的，它刷新、后退等浏览器操作是无害的，即使用该方法不会引起服务器状态改变，而且幂等（同个请求执行一次和执行多次效果完全相同）；POST 可能会重复提交表单，它是非安全非幂等
4. 缓存问题：GET 是读取资源，可对请求的资源做缓存，这个缓存可以添加到浏览器上；POST 因为不可随意多次执行，不幂等，所以不能被缓存；



### PUT 和 POST 都是更新服务器资源，有啥区别？

PUT 方法是幂等的，即调用一次和调用多次的结果是相同的；而 POST 是非幂等的

PUT 方法通常用于更新资源，调用多次不影响；POST 通常用于创建资源；



### HTTP 的状态码有哪些？

2XX：成功

* 200 OK，表示从客户端发送的请求在服务器被正确处理

3XX：重定向

* 302 found，临时重定向，表示资源临时被分配了新的 URL
* 304 not modified，未改变，说明无需再次传输请求内容

4XX：客户端错误

* 400 bad request，请求报文存在语法错误
* 403 forbidden, 表示服务器上没有找到请求的资源
* 404 not found，表示服务器上没有找到请求的资源

5XX 服务端错误

* 500 internal server error，表示服务器在执行请求时发生了错误
* 503 server unavaiable，表示服务器暂时处于超负载或正停机维护
* 505 http version not supported，服务器不支持



### HTTP 中的 keep-alive 是干啥的？

早期的 HTTP1.0 中，每次 HTTP 请求都要创建一个连接，创建连接的过程是需要消耗资源和时间的。为了减少资源消耗，缩短请求时间，开始引入**重用连接**的机制，实现长连接，即在 HTTP 请求头中加入 `Connection: keep-alive` 来告诉对方不要关闭，下一次咱们还用这个请求继续交流。



### 为啥有了 HTTP 还要 HTTPS？

HTTPS 是安全版的 HTTP，因为 HTTP 协议的数据是明文传输的，涉及到信息安全；HTTPS 就是为了解决 HTTP 的不安全而产生的



### HTTPS 是如何保证安全的？

对称加密：通信双方使用同一个密钥进行解密，虽然简单高效，但无法解决把首次密钥发送给对方的问题，容易被拦截；

非对称加密：私钥 + 公钥 = 密钥对；通信双方有各自的密钥对，通信前先把各自的公钥发送给对方，对方拿这个公钥来加密数据后发送给对方，对方再用自己的私钥进行节目。安全性高，但性能慢；

最优加密：结合以上两种方式。将对称加密的密钥使用非对称的公钥进行加密，发送给接收方，接收方使用私钥进行解密得到对称加密后的密钥，然后双方开始使用对称加密进行通信



------



# 浏览器





### HTML 在浏览器渲染过程？



整个渲染过程大概是利用浏览器渲染引擎，去解析 URL 对应的各种资源，最终输出可视化图像

渲染模块有：HTML 解析器、Javascript 引擎、CSS 解析器、布局

基本过程：

1. 解析 HTML，生成 HTML 标签
2. 构建 DOM 树，解析 JavaScript 动态生成的标签，生成 DOM 树
3. DOM 树结合 CSS 样式文件，构建呈现树
4. 布局，结合呈现树，把 DOM 节点大小定位计算出来
5. 绘制，把 CSS 文件中有关颜色的设置呈现在页面上



### 什么是 EventLoop？

Event Loop 是指计算机系统的一种运行机制，JavaScript 就是采用这种机制，用来解决单线程带来的一些问题

JavaScript 本身是单线程语言，这就会遇到一些问题：如果某个任务涉及多个 I/O 操作很耗时，则整个线程大部分时间是在等待 I/O 的返回结果，这种方式称为「同步模式」或「堵塞模式」

Event Loop 解决了这个问题，它在程序中设置两个线程，一个负责程序本身，即主线程；另一个负责和其他线程通信，即 Event Loop 线程；主线程每当遇到 I/O 就让 Event Loop 去通知相应的 I/O 程序然后继续执行，等 I/O 完成操作后，Event Loop 把结果返回给主线程，这样主线程就有更多空闲时间，可以运行更多任务，这种模式称为 「异步模式」或「非堵塞模式」





# NodeJS



### 前后端同构



前后端同构是指前后端共同使用一套代码，构建 DOM 节点，这套代码能在服务端和客户端运行。



**背景：根本原因是白屏问题**。

~~~js
<html>
  <head><title /></head>
  <body>
  	<div id="root"></div>
    <script src="render.js"></script>
  </body>
</html>
~~~

如上代码，浏览器在渲染 HTML 页面的时候，执行到 `body#root` 这个节点时并不是直接绘制，页面一开始是空的；等到浏览器加载完 `render.js` 之后 `body#root` 节点才开始绘制。`render.js` 是通过 `http` 请求获取的，那么在获取 `render.js` 期间，必然会触发白屏问题。



**问题解决**



使用前后端同构，在服务端初次渲染时添加后端同构代码，返回完整的 HTML 文件。在后续交互中使用前端的同构代码，这样就解决了白屏问题，实现无缝衔接，提高用户体验。



### BFF



全称是 Backends For Frontends(服务于前端的后端)，简单说就是浏览器和服务器之间的一个中间件，负责接收服务器返回的数据，并组装成前端需要的数据，最后返回给浏览器。

这样一来，客户端不是直接访问服务接口，而是调用 BFF 层提供的接口，BFF 层再调用后台的服务。不同的客户端拥有不同的 BFF 层。



两个职责：

1. 对用户端提供 HTTP 服务；
2. 使用后端的 RPC 服务



### RPC 



全称为「远程过程调用」。是基于服务端和服务端之间的通信。



**特点**

RPC 是服务端和服务端之间的通信

RPC 一般再内网中相互通信，使用特有的服务进行寻址

RPC 不使用 HTTP 协议，采用基于二进制协议的 TCP 或 UDP



通信方式有三种：单工通信、半双工通信、全双工通信 



# webpack



### webpack配置过程

1. package.json .gitignore .npmrc
2. 代码风格：prettier eslint stylelint commitlint
3. 基本配置：
   1. 入口文件、输出文件
   2. 区分环境 针对性文件处理
   3. 开发环境配置 让项目跑起来 devtool：eval-source-map
   4. css less 样式文件处理、图片处理、文件/字体处理
   5. 支持 react，支持 ts，babel 配置，tsconfig 配置
4. 优化：
   1. 拷贝公共静态资源
   2. 显示编译进度、ts 类型检查 fork-ts-checker-webpack-plugin
   3. 加快二次编译（hard-source...）、抽离公共代码（splitChunks）
   4. 抽离 CSS 样式、js/css 代码压缩
5. 热更新



### 有哪些常见的 loader？你用过哪些？

* babel-loader 处理 js/jsx/ts/tsx 文件
* awesome-typescript-loader 将 typescript 转换成 JavaScript 
* css-loader 加载 css 文件，支持模块化
* style-loader 将 css 文件注入到 JavaScript 中
* postcss-loader 拓展 css 语法，搭配 autoprefixer 自动补齐 css3 前缀
* less-loader 将 less 代码转换成 css
* sass-loader 将 sass 代码转换成 css
* file-loader 处理字体、音频、图片等文件
* url-loader 处理图片，可设置阈值，小于阈值转换成 base64 编码
* json-loader 处理 json 文件
* eslint-loader 使用 eslint 检测 JavaScript 代码
* tslint-loader 使用 tslint 检测 typescript 代码
* xml-loader 解析 xml 文件



### babel 可以用来配置什么？

1. @babel/core 是 babel 的核心，必须引入
2. @babel/preset-react 处理 jsx 语法
3. @babel/preset-typescript 处理 typescript 语言
4. @babel/preset-env 处理 es6 语法
5. @babel/plugin-transform-runtime 将 es6 新语言特性（promise）注入打包后的文件中，支持按需加载



### 有哪些常见的 plugin？你用过哪些？

* html-webpack-plugin 简化创建 html
* terser-webpack-plugin 压缩 js
* fork-ts-checker-webpack-plugin 打包/启动项目时提示 ts 的代码错误
* mini-css-extract-plugin 将 css 提取为单独文件
* css-minimizer-webpack-plugin 对 css 压缩
* webpack-bundle-analyzer 可视化 webpack 输出文件体积
* copy-webpack-plugin 复制 public 下的文件到指定的目录中
* clean-webpack-plugin 打包生成的目录清理



### 有没有自己写过 Plugin？



### Loader 和 Plugin 的区别

**配置形式**

* Loader 在 module.rules 中配置，类型是数组，作为模块的解析规则。每一项 object 内部包含 test、loader、options
* Plugin 在 plugins 下单独配置，类型为数组，每一项是一个 Plugin 的实例，参数通过构造函数传入

**功能区别**

* Loader 本质上是一个函数，对接收到的内容进行转换，返回转换结果。因为 webpack 只认识 js，loader 就像一个翻译官，对其他类型的资源进行转换工作
* Plugin 是插件，在 webpack 运行时会广播出许多事件，Plugin 可以监听这些事件，在特定的时机改变事件的输出结果



### 简单说一些 webpack 的构建流程

1. 初始化：开始 webpack 构建，读取与合并配置参数，加载 plugin，实例化 compiler
2. 编译：从文件入口出发，调用 module 下的 loader 去解析对应的资源文件，找到模块依赖的相应 module，递归编译处理
3. 输出：将编译后的 module 组合成 chunk，将 chunk 转换成文件，输出到文件系统中



### 说说 webpack 的热更新原理

webpack 热更新又称热模块替换(Hot Module Replacement)，简称 HMR。这个机制能做到不用刷新浏览器就能将新的模块替换旧的模块

HMR 核心是客户端向服务端拉取更新的文件资源。

1. 当 webpack 监听到本地文件资源发生变化后，对模块重新编译打包存储在内存中。
2. webpack-dev-server(后面简称为 wds)和浏览器之间维护一个 websocket，当本地资源发生变化，wds 向浏览器发送更新内容并附带上 hash
3. 客户端接收到更新内容后与上一次资源进行对比，对比差异后向 wds 发起 ajax 请求获取更改内容
4. 这样客户端就能借助这些信息继续向 wds 发送 jsonp 请求获取 chunk 增量更新



### 文件指纹是什么？怎么用？

* hash：和整个项目的构建相关，只要项目文件有修改，整个项目构建的 hash 值也会修改，常用于图片文件的指纹设置
* chunkhash：和 webpack 打包的 chunk 有关，不同 entry 生成不同的 chunkhash 值，常用于入口 js 文件指纹设置
* contenthash：根据内容定义 hash，文件内容不变，contenthash 不变，常用于 css 指纹设置



### 如何优化 webpack 的构建速度？

1. 使用高版本的 webpack 和 node.js
2. 多进程并行压缩：terser-webpack-plugin 开启 parallel 参数
3. 图片压缩：image-webpack-loader
4. 预编译资源模块：webpack.DllPlugin hard-source-webpack-plugin
5. 缩小打包作用域：
   1. exclude/include
   2. resolve.extensions 减少后缀尝试
6. 提取页面公共文件：
   1. 将基础包通过 CDN 引入，不打入 bundle 中
   2. 通过 SplitChunksPlugin 进行公共脚本分离
7. Tree shaking 
   1. 尽量使用 es6 module 的模块，提高 shaking 效率
   2. 使用 PurifyCSS 擦除无用的 css 



### 谈谈 webpack5 有啥不同的地方？

1. webpack5 内置 js 压缩，内置 tarser-webpack-plugin 插件，生产环境自动开启
2. webpack5 内置资源模块，加载资源不需要第三方 loader，assets/(resource/inline/source) -- (file-loader/url-loader/raw-loader)
3. 热更新配置，webpack5（HotModuleReplacementPlugin）
4. 内部的构建优化，更好的 tree-shaking



### 首屏加载速度如何优化？

1. 第三方 JS 库分离打包
   1. 如果是生产环境是内网，那就通过静态资源引入
   2. 如果成产环境是外网，可以通过 CDN 方式引入
2. 单页面应用 react router 使用懒加载
3. 图片资源的压缩 tinypng.com 网站压缩图片，图标可合成一张图片通过 css 定位去显示需要的部分
4. webpack 的相关优化
   1. 使用版本更高的 webpack 和 nodejs
   2. splitchunksplugin 提取公共模块
   3. js 压缩（tarser-webpack-plugin）
   4. 提取 css 到单独的文件，使用 css-minimizer-webpack-plugin 压缩 css
   5. 按需引入



------



# Javascript



### JS 数据类型有哪些

值类型：Number String Boolean Null Undefined Symbol

引用类型：Object Array Function

原始数据类型直接存储在栈中，引用数据类型在栈中存储了指针，该指针指向该实体的地址。



### JS 数据类型转换

转换为布尔值 Boolean()

转换为数字 Number() parseInt() parseFloat()

转换为字符串 toString() String()



### JS 中数据类型的判断

typeof：除了 null、数组判断为 object，其他都能显示正确的类型

instanceof：能显示正确的类型，undefined 和 null 会报错

constructor：能正确显示类型，undefined 和 null 会报错，如 `(123).constructor === Number`



### null 和 undefined 的区别

null 和 undefined 都是基本数据类型

undefined 表示未定义，变量声明了但没有定义会返回 undefined

null 表示空对象，主要用于赋值给一些会返回对象的变量作为初始化，`typeof null === 'object'`



### {} 和 [] 的 valueOf 和 toString 结果是什么

`{}.valueOf()` 结果为 {} 对象本身，toString 结果为 "[object object]"

`[].valueOf() `结果为 [] 数组本身，toString 结果为 ""



### 你对 this、apply、call、bind 的理解

全局范围内 this 指向 window 对象

apply、call、bind 的 this 绑定在指定的对象上（第一个参数）

普通函数中，this 指向函数的调用者（看函数前面有没有点）

箭头函数中的 this 指向是静态的（上面是动态的），即在指向定义时所处在的作用域。箭头函数没有自己的 this，只会从作用链上一层继承 this 



### apply、call、bind三者区别

* 三者都可以改变 this 的指向
* 三者第一个参数为 this 所指向的对象，如果没有该参数或为 undefined 或 null，则默认指向全局 window
* 三者皆可传参，apply 传入的是数组，call 和 bind 传入的是参数列表，apply 和 call 是一次性传入，而 bind 可分多次传入
* bind 返回的是一个拷贝后的新函数，而 call 和 apply 是立即执行



### slice splice split 区别



slice(start, end) 在数组中选中元素以数组形式返回，不改变原数组

~~~js
const array1 = ['a', 'b', 'c', 'd'];
array1.slice(0,2); // ['a', 'b']
~~~



splice(index, num, item1, item2.....) 在数组中添加、删除元素；以数组形式返回删除的元素，改变原数组

~~~js
// 删除
const array1 = ['a', 'b', 'c', 'd', 'e'];
const dels = array1.splice(2,2); // ['a', 'b']
array1 // ['a', 'b', 'e']

// 添加
const array1 = ['a', 'b', 'c', 'd', 'e'];
const dels = array1.splice(2, 1, 'm', 'n'); // ['c']
array1 // ['a', 'b', 'm', 'n', 'e']
~~~



split() 把一个字符串分割成一个数组



### 数组扁平化

将一个多维数组变为一个一维数组

~~~javascript
const arr = [1,2,3,[4,5,[6,7]]];
arr.flat()

JSON.stringfy(arr).replace(/\[|\]/g, '').split(',');
~~~



### 数组去重

~~~javascript
const arr = [1,2,2,2,2,2,3,3,4,4,5];
Array.from(new Set(arr)); // [1,2,3,4,5]
~~~



### == 和 ===

原始值是值的比较，他们的值相等，== 结果就为 true

对象的比较不是值的比较，而是引用的比较，所以任意两个数组或对象都是不相等的



### 用 JS 创建包含 0-99 的整数

for 循环

ES6：`Array.from(new Array(100).keys())`

拓展运算符：`[...Array(100).keys()]`



### 闭包

**什么是 JS 执行上下文？**

* 全局作用域：第一次执行代码的环境，即全局执行上下文

* 函数作用域：当执行进入函数体时，本地（函数）执行上下文

* 本地上下文销毁：当函数遇到 return，或结束括号 } ，函数结束，执行上下文销毁

**作用域？**

作用域是查找变量的地方，在函数中找到变量可以说是在函数作用域中找到该变量，在全局找到对应全局作用域。查找变量的时候是一个从内往外查找的过程，顺着一链条从下往上查找，该链条称为原型链。

函数作用域：一个函数可以访问它的调用上下文中所定义的变量

**返回函数的函数**

* 函数定义可以存储在变量中
* 函数定义在被调用之前是不可见的
* 每次调用函数时会临时创建一个本地执行上下文，函数完成，执行上下文销毁
* 函数遇到 `return` 或 `}` 时执行结束

**何为闭包？**

从技术上来说，所有的 JavaScript 函数都是闭包，因为他们都关联到作用域。

当函数包含一个连接到外部上下文的变量时，就产生了闭包。

优点：隐藏变量，避免全局污染，模块化；可以读取到函数内部变量。

缺点：变量不会被回收机制回收，造成内存消耗。



### JS原型和原型链

简单来说， `prototype` 指原型，`__proto__` 指原型链（往上1级），原型链可以说是一个过程。

`anyType.prototype.__proto__ === Object.prototype`，所以说在 JavaScript 里，万物皆对象。

四个规则

1. 引用类型 array/object/function，都具有对象属性，可自由拓展属性
2. 引用类型，都有一个隐式原型 `__proto__` 属性（Number/String/Boolean也有），属性值是一个对象
3. 引用类型的隐式原型 `__proto__` 属性值指向它的构造函数显式原型 `prototype` 的属性值

~~~js
bj.__proto__ == Object.prototype // true
arr.__proto__ === Array.prototype // true
fn.__proto__ == Function.prototype // true
~~~

4. 每个 js 对象（除null）创建时都会关联一个对象（prototype原型），每个对象都会继承原型中的属性。通过它的隐式原型 `__proto__` 查找

~~~js
// obj 对象并没有 toString 属性，但从构造函数 Object.prototype 中继承了
const obj = { a:1 }
obj.toString
// ƒ toString() { [native code] }
~~~



# HTML



### HTML 语义化标签



**什么是语义化**

顾名思义，语义化标签就是 HTML 和开发者能够容易理解的一些标签。如：

~~~html
header footer body nav article aside a ul li h p 
~~~



**为什么语义化**

div+css 的结构比较泛滥，不利于 SEO 优化。





# CSS



### Less 有什么优点

1. 设置变量，可用来设置一系列通用的样式
2. 混合：可以带参数去调用，像使用函数那么简单，先写成类后面直接引用
3. 嵌套：在选择器中嵌套另一个选择器实现继承
4. 函数和运算：运算提供加、减、乘、除操作，函数操作



### CSS 选择器优先级

1. 标记选择器（body, div, p）
2. id 选择器（#id）
3. 类选择器（.class）
4. 后代选择器（#head .nav）
5. 子元素选择器（div>p）
6. 伪类选择器（link, active, hover）



### CSS 哪些属性可以继承

1. 字体系列属性
2. 文本系列属性
3. 表格布局属性
4. 列表属性
5. 页面样式属性
6. 声音样式属性



### CSS 水平居中的几种方式？

~~~css
// flex
.wrap {
    display: flex;
    justify-content: center;
}
// margin 
.son {
    margin: 0 auto;
}
// 行内元素居中
text-align: center;
~~~



### CSS 垂直居中的几种方式？

~~~css
// flex 
.wrap {
    display: flex;
    align-items: center;
}
// padding
.wrap {
    width: 300px
    height: 400px;
}
.box {
    width: 100px;
    height: 100px;
    padding: 150px 0;
}
// 行内元素 line-height
~~~





### CSS 水平垂直居中的几种方式

1.flex 居中

~~~css
.wrap {
    display: flex;
    justify-content: center; // 主轴对齐方式
    align-items: center; // 交叉轴对齐方式
}
~~~

2.position+transform 

~~~css
.transform { 
    transform: translate(); // 平移
    transform: scale(); // 缩放
    transform: rotate(); // 旋转
    transform: skew(); // 倾斜
}
.wrap {
  position: relative;
    .son {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%); // 表示二维上按照指定x轴和y轴进行50%平移
    }
}
~~~

3.position+margin

~~~css
.wrap {
    position: relative;
    .son {
        width:200px;
        height:100px;
        top: 50%;
        left: 50%;
        margin-top: -50px;
        margin-left: -100px;
    }
}
~~~



### CSS3 的 box-sizing 属性有什么作用？

默认情况下，元素的宽高是这么计算的：width/height + padding + border = 实际宽高，这会导致真实的页面效果宽高会变大。

设置 `box-sizing: border-box` 解决了这个问题，将 padding 和 border 包含在了 width 和 height 中



### 说说 flex 布局

flex 英文为 flexible box，意为弹性布局，由父级容器、子容器构成，通过设置主轴和交叉轴来控制子元素的排列方式。

父级容器属性：

* `flex-direction` 主轴方向
* `flex-wrap` 如果一条轴线放不下如何换行？
* `justify-content` 在主轴的对齐方式
* `align-items` 在交叉轴上的对齐方式
* `align-content` 多根轴线的对齐方式

子容器属性：

* `order` 子元素的排列顺序，数值越小越靠前，默认0
* `flex-grow` 子元素放大比例，默认 0（即如果存在剩余空间也不放大）
* `flex-shrink` 定义项目缩小比例，默认 1（即空间不足将缩小）
* `flex-basis` 定义了在分配多余空间之前，项目占据的主轴空间，默认 auto（即项目原本大小）
* <u>`flex`</u> 是 flex-grow flex-shrink flex-basis 的简写，默认 `0 1 auto`
* `align-self` 允许单个项目和其他项目不一样的对齐方式，可覆盖 `align-items` 默认设为 auto 表示继承父元素

