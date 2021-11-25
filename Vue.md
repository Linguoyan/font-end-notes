# Vue 核心



## 认识 Vue



### 什么是 Vue？

- 一套拥有构建用户界面的渐进式 JavaScript 框架。

- 简单应用：只需一个轻量小巧的核心库。

- 复杂应用：可以引入各式各样的 Vue 插件。




Vue 的特点

1. 采用组件化模式，提高代码复用率、让代码更易于维护。
2. 声明式编码，无需直接操作 dom，提高开发效率。
3. 使用虚拟 dom + diff 算法，尽量复用 dom 节点。



强制刷新 ctrl+shift+R shift+F5



### 创建 Vue 实例

1. 使用 vue 必须创建一个 vue 实例，并传入配置对象
2. root 容器里代码依然符合 html 规范，只是混入了 vue 语法
3. root 容器里的代码被称为 vue 模板
4. Vue 实例和容器是一一对应的
   1. 一个实例多个容器，只操作第一个；
   2. 多个实例一个容器，只有第一个实例起作用；

5. 真实开发中只有一个 vue 实例
6. `{{ expression }}` 双花括号里只能放 js 表达式（表达式即会产生一个值）
7. `data` 中数据有变化，将触发模板中对应的数据
8. 开发版 vue 会提示错误，生产版本 vue 不会，版本要对应环境



## 模板语法（两大类）



- 插值语法
  - 解析标签体（开始标签和结束标签中间放的就是标签体）内容
  - 如语法 `{{ xxx }}`，`xxx` 为 js 表达式
- 指令语法（包含动作）
  - 解析标签属性、解析标签体内容、绑定事件
  - 如 `v-bind:href="xxx"`，`xxx` 为 js 表达式，可简写为 `:href="xxx"`



## 数据绑定（两种方式）



- 单向数据绑定 `v-bind`：数据单向流动，`data` 流向页面；
- 双向数据绑定 `v-model`：数据双向流动
  - 双向绑定一般应用在表单输入类元素
  - `v-model:value` 默认收集的是 `value` 值，可简写为 `v-model`



## el 和 data 的写法



el 的两种写法：

1. `new Vue` 里配置 `el` 属性
2. 先创建 Vue 实例，随后用 `vm.$mount()` 挂载根节点



data 的两种写法：

1. 对象式
2. 函数式：组件化时必须 data 必须使用函数式
3. 注意 **Vue 管理的函数不能用箭头函数，否则 this 的指向问题会出错**



## MVVM 模型



M：模型 Model，对应 `data` 中的数据

V：视图 View，模板

VM：视图模型 ViewModel，`Vue` 实例对象

![](H:\Collation\image\mvvm.png)

发现了什么？

- data 中的所有属性，最后都出现在 vm 身上
- vm 实例上的所有属性以及 Vue 原型上的属性，都能在模板中直接使用



## 数据代理



### Object.defineProperty()

~~~js
Object.defineProperty(person, "age", {
    value: 18,
    enumerable: true, // 属性是否可枚举 默认false 
    writable: true, // 属性是否可修改 默认false
    configurable: true, // 属性是否可删除 默认false
});

Object.defineProperty(person, "age", {
    // 当读取age属性时，getter函数会被调用，函数返回值是age的值
    get() {
        return number;
    },
    // 当有人修改age属性时，setter函数会被调用，返回值是所要修改的结果
    set(value) {
        number = value;
    },
});
~~~



### 什么是数据代理？



通过一个对象代理对另一个数据对象的属性操作（读/写）

~~~js
let obj = { x: 100 };
let obj2 = { y: 12 };
Object.defineProperty(obj2, "x", {
    get() {
        return obj.x;
    },
    set(value) {
        obj.x = value;
    },
});
~~~



### Vue 中的数据代理



- 通过 `vm` 对象来代理 `data` 对象中属性的操作（读/写）
- 优点：更方便操作 `data` 中的数据
- 原理：
  - `Object.defineProperty()` 把 `data` 对象中所有属性添加到 `vm` 上 
  - 为每个添加到 `vm` 上的属性指定 `getter/setter` 操作
  - 通过 `getter/setter` 去操作（读/写）`data` 中对应属性
  - `_data` 做数据劫持，监听数据变化，触发页面更新



## 事件处理



### 基本使用



- 使用 `v-on:xxx` 或 `@xxx` 绑定事件，`xxx` 是事件名
- 事件的回调函数配置在 `methods` 中，最终挂载到 `vm` 上
- `methods` 中配置的函数不可使用箭头函数，否则 `this` 就不会指向 `vm` 了
- `methods` 中配置的函数是被 `Vue` 管理的函数，`this` 指向 `vm` 或实例对象
- `@click="demo"` 和 `@click="demo(num, $event)"` 效果一样，后者可以传参



### 事件的修饰符



- `prevent`：阻止默认事件（常用）
- `stop`：阻止事件冒泡（常用）
- `once`：事件只触发一次（常用）
- `capture`：使用事件的捕获模式
- `self`：当 `event.target` 是当前 self 指定元素时才触发
- `passive`：事件默认行为立即执行，无需等待回调函数执行完成

~~~html
<!-- 事件捕获：默认先处理冒泡事件后捕获(info2->info2) capture 即先捕获后冒泡(info->info2) -->
<div class="capture1" @click.capture="showInfo">
    div1
	<div class="capture2" @click="showInfo2">div2</div>
</div>

<!-- 修饰符可以连着写：先阻止事件冒泡，后阻止默认事件触发 -->
<div @click="showInfo">
    <a href="..." @click.stop.prevent>标签</a>
</div>
~~~



`scroll` 事件和 `wheel` 事件的区别：

- 两者都是监听页面滚动的事件
- `scroll` 事件在拖动滑块或鼠标滚轮滚动时都会被触发 `wheel` 事件仅在滚轮滚动时触发
- 拖动到底部继续滚动，`scroll` 事件不再触发，`wheel` 事件依然被触发
- `scroll` 事件无需等待回调函数执行完成，立即执行
- `wheel` 事件需等待回调函数执行完成后才响应，容易造成页面卡顿，可结合 `passive` 修饰符解决



### 键盘事件



常用按键名：

- 回车：`enter`
- 删除：`delete` （捕获删除和退格键）
- 退出：`esc`
- 空格：`space`
- 换行：`tab` （注意：配合 `keydown` 使用）
- 上：`up`
- 下：`down`
- 左：`left`
- 右：`right`



注意：

1. Vue 没有提供其他按键的别名，可以通过 `e.key` 获取，单词组合需转换为 `caps-lock` (短线小写命名)
2. 特殊修饰键：`ctrl` `alt` `shift` `meta(window)`，需配合 `keydown` 才能正常触发
3. 也可使用 `keyCode` 指定具体按键，官方不推荐
4. 可通过 `Vue.config.keyCodes.自定义键名 = 键码` 指定按键名
5. 可以连写，如 `@keyup.ctrl.y` 表示同时按下 `crtl` 和 `y` 键触发



## 属性计算-computed



**是什么？**

需要的属性不存在，需通过已有的属性计算得到结果。

~~~js
computed: {
    fullName: {
        get() {
            console.log("get is called");
            return this.firstName + "-" + this.lastName;
        },
        set(value) {
            console.log("set is called");
            const arr = value.split("-");
            this.firstName = arr[0];
            this.lastName = arr[1];
        },
    },
    ...
    // 简写形式：只考虑读取，不考虑修改
    fullName() {
        return this.firstName + "-" + this.lastName;
    },
},
~~~



**为啥用它？**

使用函数回调的方法计算属性，如果多次使用将触发多次更新，对性能不友好。

`computed` 内部有缓存机制，对性能更加友好，调试方便。



**原理是啥？**

底层使用 `Object.defineProperty` 提供的 `setter` 和 `getter`。



**getter 什么时候执行？**

初次读取只执行一次，后续直接读取缓存。

当依赖的数据更新时将再次被调用。



**注意事项**

计算属性最终会放入 `vm` 中，直接读取使用即可，无需添加括号。

如果想修改计算属性，需通过 `setter` 函数去修改，且 `setter` 中要引起计算时依赖的数据发生改变。



## 监视属性-watch



### **是什么**

- 当监视的属性（必须存在）发生变化时，回调函数自动调用，并进行相关处理。

  

属性监视的两种写法

~~~js
// 1. vue实例内配置
new Vue {
    ...
    watch: {
        isHot: {
            immediate: true, // 吃初始化是否调用一次handler 默认false
            handler(newVal, oldVal) {
                //监听回调函数
                ...
            }
        }
    }
}

// 2. vm.$watch 配置
const vm = new Vue({ ... })
vm.$watch("isHot", {
    immediate: true, // 初始化调用一次handler
    handler(newVal, oldVal) {
      ...
    },
});
~~~



### 深度监视



- Vue 中的 `watch` 默认使用浅对比监测对象内部属性的变化（一层）
- 设置配置项 `deep: true` 可深度对比监测对象内部属性的变化（多层级）
- 监测单个属性可通过 `"obj.xxx"` 

~~~js
watch {
    "number.b": {
        handler() {
            console.log("b is updated");
        },
    },
    number: {
        deep: true, // 监视对象所有属性的变化
        ...
    }
}
~~~





**注意事项**

- Vue 自身可以监测对象内部属性的改变，但 Vue 提供的 watch 默认不可以
- 使用 watch 监测对象时，根据数据结构决定是否采用深度监视



**简写形式**



当只需要用到 `handler` 函数，无需用到其他配置项，就能使用简写：

~~~js
watch {
    isHot(newVal, oldVal) {
        ...
    }
}
...
vm.$watch('isHot', function(newVal, oldVal) {
    ...
})
~~~



### computed vs watch



- `computed` 能实现的功能 `watch` 也能实现
- `watch` 能实现的功能 `computed` 不一定能，比如异步操作



**tips**

- 所有被 Vue 管理的函数，最好写成普通函数，这样 `this` 指向才是 `vm` 或组件实例对象
- 所有不被 Vue 管理的函数（setTimeout/setInterval/Promise...），应写成箭头函数。因为异步回调是浏览器引擎执行的，使用普通函数将指向 `Windows`，用箭头函数才能往外寻找到 vm 对应的实例对象。



## 绑定样式



### 绑定 class

~~~html
<!-- 字符串写法 -->
<div class="basic" :class="a" @click="changeMood">hello vue.</div>

<!-- 数组写法 -->
<div class="basic" :class="classArr">hello vue.</div>

<!-- 对象写法 -->
<div class="basic" :class="classObj">hello vue.</div>

...

data: {
    classArr: ["bold", "sizeBigger", "underline"],
    classObj: {
        bold: false,
        sizeBigger: false,
        underline: true,
	},
},
~~~



三种写法适用场景：

- 字符串写法：-样式类名不确定，需要动态指定
- 数组写法：绑定样式的个数不确定，名字不确定
- 对象写法：绑定样式的个数、名字确定，要动态决定是否展示



### 内联 style

~~~html
 <!-- style内联写法 -->
<div class="basic" :style="{fontSize: fsize + 'px'}">style内联写法.</div>

<!-- style 对象写法 -->
<div class="basic" :style="styleObj1">style对象写法.</div>

<!-- style 数组写法 -->
<div class="basic" :style="[styleObj1, styleObj2]">style对象写法.</div>

...
data {
 	styleObj1: {
        color: "red",
        fontSize: "32px",
    },
}
~~~



## 条件渲染



`v-if`：判断结果为 `flase` 直接删除 dom 节点，适用于切换频率比较低的场景，影响性能。可以配合 `v-else-if` `v-else` 使用，提高效率。但前提是不被其他节点打断。

~~~html
<div v-if="n === 1">Angular</div>
<div v-if="n === 2">React</div>
<div v-if="n === 3">Vue</div>

 <!-- 使用v-if和v-else-if做条件渲染 提高性能 避免重复计算-->
<div v-if="n === 1">Angular</div>
<div v-else-if="n === 2">React</div>
<div v-else-if="n === 3">Vue</div>
~~~



`v-show`：通过样式控制节点是否展示，适用于切换频率较高的场景。

~~~html
<div v-show="false">{{text}}.</div>
<div v-show="!isShow">{{text}}.</div>
~~~



`template`，不破坏原有 dom 结构，类似 React 中的 `Fragment`，可配合 `v-if` 使用，不能配合 `v-show`

~~~html
<template v-if="n === 1">
    <h2>list1</h2>
    <h2>list2</h2>
    <h2>list3</h2>
</template>
~~~



## 列表渲染



### v-for 

1. 通常用于展示列表数据
2. 语法：`v-for="item in items :key="item.id"` 其中 `items` 是源数组，`item` 是数组元素的别名
3. 可遍历：数组、对象、字符串（用得少）、指定次数（用得少）

~~~html
<!-- 数组遍历 -->
<li v-for="(p, index) id persons" :key="p.id">
    {{p.name}}-{{p.age}}
</li>

<!-- 遍历对象 -->
<li v-for="(value, k) in car" :key="k">
    {{k}}-{{value}}
</li>

<!-- 遍历指定次数 -->
<li v-for="(number, index) of 5" :key="index">
    {{index}}-{{number}}
</li>
~~~



### Vue 中的 Key



**虚拟 Dom 中 key 的作用**

key 是虚拟 DOM 对象的标识，当数据发生变化时，Vue 会根据新数据生成新的虚拟 DOM。然后将新虚拟 DOM 和旧虚拟 DOM 进行差异比较（diff 算法）。



**虚拟节点对比规则**

- 旧虚拟 DOM 中找到了与新虚拟 DOM 相同的 key
  - 若虚拟 DOM 中内容没变, 直接使用之前的真实 DOM（`input` 输入框对比不变，不更新，其实并没有对上）
  - 若虚拟 DOM 中内容变了, 则生成新的真实 DOM，随后替换掉页面中之前的真实 DOM（文本内容）
- 旧虚拟 DOM 没有找到和新虚拟 DOM 相同的 key
  - 创建新的 DOM 节点，随后渲染页面（用唯一表示作为 key）

~~~html
<li v-for="(p,index) of persons" :key="index">
    {{p.name}}-{{p.name}}
    <input type="text">
</li>
~~~





**用 index 作为 key 可能引发的问题**

若对数据进行：逆序添加、逆序删除等破坏顺序操作，index 发生改变会产生没有必要的更新

如果 DOM 结构包含输入类节点，会产生错误的更新



**如何合理选择 key？**

- 最好使用每条数据的唯一标识作为 key, 比如 id、身份证等唯一值
- 如果不存在对数据的逆序添加、逆序删除等破坏顺序操作，仅用于展示，使用 index 作为 key 也是可以的

~~~html
<!-- 出现错误 key需取唯一值，即id -->
<ul>
    <li v-for="(p,index) of persons" :key="index">
        {{p.name}}-{{p.age}}
        <input type="text">
    </li>
</ul>
<script>
new Vue({
    el:'#root',
    data:{
        persons:[
            {id:'001',name:'张三',age:18},
            {id:'002',name:'李四',age:19},
            {id:'003',name:'王五',age:20}
        ]
    },
    methods: {
        add(){
            const p = {id:'004',name:'老刘',age:40}
            this.persons.unshift(p)
        }
    },
})
</script>
~~~



### 列表排序



可用 `watch` 或 `computed` 实现

~~~js
// 使用 watch 监听 keyword，修改原数组
watch:{
    keyWord:{
        immediate:true,
        handler(val){
          this.filPerons = this.persons.filter((p)=>{
              return p.name.indexOf(val) !== -1
          })
        }
    }
}

// 使用 computed 不修改原数组，直接返回结果，更加简洁
new Vue({
    el:'#root',
    data:{
        keyWord:'',
        persons:[
            {id:'001',name:'马冬梅',age:19,sex:'女'},
            {id:'002',name:'周冬雨',age:20,sex:'女'},
            {id:'003',name:'周杰伦',age:21,sex:'男'},
            {id:'004',name:'温兆伦',age:22,sex:'男'}
        ]
    },
    computed:{
        filPerons(){
            return this.persons.filter((p)=>{
                return p.name.indexOf(this.keyWord) !== -1
            })
        }
    }
})
~~~



### 列表过滤



使用 `computed` 不直接修改原数组

~~~js
computed: {
    filPerons(){
        const arr = this.persons.filter((p)=>{
            return p.name.indexOf(this.keyWord) !== -1
        })
        //判断一下是否需要排序 0原序 1升序 2降序
        if(this.sortType){
            arr.sort((p1,p2)=>{
                return this.sortType === 1 ? p2.age-p1.age : p1.age-p2.age
            })
        }
        return arr
    }
}
~~~





## 表单提交



### 输入框的不同类型

`<input type="text"/>`，`v-model` 收集的是 value 值，用户输入的就是 value 值；

`<input type="radio"/>`，`v-model` 收集的是 value 值，且要给标签配置 value 值。

`<input type="checkbox"/>`：

- 没有配置 value 属性，则收集的是 checked（勾选 or 未勾选，是布尔值）
-   配置 value 属性:
  -  `v-model` 的初始值是非数组，则收集的是 checked（勾选 or 未勾选，是布尔值）
  -  `v-model` 的初始值是数组，则收集的的是 value 组成的数组

​    

 备注：`v-model` 的三个修饰符

​         lazy：失去焦点再收集数据

​         number：输入字符串转为有效的数字

​         trim：输入首尾空格清楚
