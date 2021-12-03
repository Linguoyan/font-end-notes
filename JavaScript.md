## 数组



### 数组方法



#### reduce

一、语法

~~~js
arr.reduce(function(prev,cur,index,arr){
}, init);
~~~

参数说明：

- arr：原数组
- prev：上一次回调返回值，或初始值 init
- cur：当前处理的元素
- index：当前处理数组元素的索引，有 init 则索引为 0，否则为 1
- init：pre 初始值

二、使用场景

1. 求数组之和

~~~js
var sum = arr.reduce((prev, cur) => {
    return prev + cur;
}, 0);
// 有条件的求和
var sum = arr.reduce((prev, cur) => {
    return prev + (cur.isNum ? cur : 0);
}, 0);
~~~

2. 求数组最大值

~~~js
var max = arr.reduce((prev, cur) => {
    return Math.max(prev,cur);
});
~~~

3. 数组去重

~~~js
var newArr = arr.reduce((prev, cur) => {
    prev.indexOf(cur) === -1 && prev.push(cur);
    return prev;
}, []);
~~~



