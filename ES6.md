# ES Module



## Es Module 认识



`Nodejs` 借鉴了 `Commonjs` 实现了模块化 ，从 `ES6` 开始， `JavaScript` 才真正意义上有自己的模块化规范。



### **Es Module 有什么优势**

- 借助`Es Module` 的静态导入导出的优势，实现了 `tree shaking`。
- `Es Module` 可通过 `import()` 懒加载方式实现代码分割。



### ES6 module 特性

**1.静态语法**

ES6 module 的引入和导出是静态的，`import` 会自动提升到代码的顶层 ，**`import` , `export` 不能放在块级作用域或条件语句中**。

**2.执行特性**

ES6 module 和 Common.js 一样，对于相同的 js 文件，会保存静态属性。

但是不同的是 ，`CommonJS` 模块同步加载并执行模块文件，ES6 模块**提前加载并执行模块文件**，ES6 模块在预处理阶段分析模块依赖，在执行阶段执行模块，两个阶段都采用深度优先遍历，执行顺序是子 -> 父。

**3.导出绑定**

**不能修改import导入的属性**，修改会产生错误。



## export 导出和 import 导入



所有通过 export 导出的属性，在 import 中可以通过结构的方式，解构出来。



### **export 正常导出，import 导入**

~~~javascript
// 导出
const name = 'ming' 
const age = '12'
export { name, author }
export const say = function (){
    console.log('hello , world')
}
// 导入
import { name , age , say } from './a.js'
~~~

- export { }， 与变量名绑定，命名导出；
- import { } from 'module'， 导入 `module` 的命名导出 ；
- 这种情况下 import { } 内部的变量名称，要与 export { } 完全匹配。



### **默认导出 export default**

~~~javascript
// 导出
const name = 'ming'
const age = '12'
const say = function (){
    console.log('hello , world')
}
export default { name, author, say } 
// 导入
import mes from './a.js'
console.log(mes) // { name: 'ming',age: '12', say:Function }
~~~

- `export default anything` 导入 module 的默认导出。`anything` 可以是函数，属性方法，或者对象。
- 对于引入默认导出的模块，`import anyName from 'module'`， anyName 可以是自定义名称。



### **混合导入｜导出**

~~~javascript
// 导入
export const name = 'ming'
export const author = 'aa'
export default  function say (){
    console.log('hello , world')
}
// 导出方式1
import theSay , { name, author as  bookAuthor } from './a.js'
console.log(
    theSay,     // ƒ say() {console.log('hello , world') }
    name,       // "ming"
    bookAuthor  // "aa"
)
// 导出方式2
import theSay, * as mes from './a'
console.log(
    theSay, // ƒ say() { console.log('hello , world') }
    mes // { name:'ming' , author: "aa" ...}
)
~~~



### **重命名导入**

~~~JavaScript
import { bookName as name, say, bookAuthor as author } from 'module'
console.log( name , bookAuthor , author )
~~~



### 重定向导出

把当前模块作为一个中转站，一方面引入 module 内的属性，然后把属性再给导出去。

~~~javascript
export * from 'module' // 第一种方式
export { name, author, ..., say } from 'module' // 第二种方式
export { bookName as name, bookAuthor as author, ..., say } from 'module' //第三种方式
~~~

- 第一种方式：重定向导出 module 中的所有导出属性， 但是**不包括 `module` 内的 `default` 属性**。
- 第二种方式：从 module 中导入 name ，author ，say 再以相同的属性名导出。
- 第三种方式：从 module 中导入 name ，对部分属性重命名后导出 。



### **无需导入模块，只运行模块**

~~~javascript
import 'module' 
~~~

执行 module 不导出值  多次调用 `module` 只运行一次



### **动态导入**

~~~javascript
const promise = import('module')
~~~

`import('module')`，动态导入返回一个 `Promise`。为了支持这种方式，需要在 webpack 中做相应的配置处理。



### import 使用场景

**1.动态加载**

首先 `import()` 动态加载一些内容，可以放在条件语句或者函数执行上下文中。

~~~javascript
if(isRequire){
    const result  = import('./b')
}
~~~

**2.懒加载**，如路由懒加载

**3.React 中的动态加载**

~~~JavaScript
const LazyComponent =  React.lazy(()=>import('./text'))
class index extends React.Component{   
    render(){
        return <React.Suspense fallback={ <div className="icon"><SyncOutlinespin/></div> } >
               <LazyComponent />
           </React.Suspense>
    }
~~~



### **tree shaking 实现**

Tree Shaking 在 Webpack 中的实现，是用来尽可能的删除没有被使用过的代码，一些被 import 了但其实没有被使用的代码。



## ES Module 总结



- ES6 Module 静态的，不能放在块级作用域内，代码发生在编译时。
- ES6 Module 的值是动态绑定的，可以通过导出方法修改，可以直接访问修改结果。
- ES6 Module 可以导出多个属性和方法，可以单个导入导出，混合导入导出。
- ES6 模块提前加载并执行模块文件，
- ES6 Module 导入模块在严格模式下。
- ES6 Module 的特性可以很容易实现 Tree Shaking 和 Code Splitting。