# 前端基础之引用类型

JS常用的变量类型分为基本类型和引用类型两种。

* **基本类型** Number、String、Boolean、Undefined、Null、Symbol，存于栈内存中，独立使用，互不影响

* **引用类型** Object、Function，存于堆内存中，无法直接更改堆内存中的值，只能由栈内存中的指针处理（指向堆内存中地址），相互影响

## 赋值运算

```javascript
let a = {
    name: "advanced",
    age: 18
}
let b = a
```

a,b通过指针指向堆内存中的地址，a、b共享这块地址，互相影响

```javascript
b.name = "change"; // a也变啦
```

## 浅拷贝

针对引用类型的操作，拷贝结果指向同一块堆内存，比如引用类型的赋值操作，互相影响

## 深拷贝

针对引用类型的操作，开辟了全新一块区域

```javascript
//常用的深拷贝方法
//lodash
_.clonedeep(sources)
//JSON.parse
JSON.parse(JSON.stringify(sources))
//jQuery
jQuery.extend( [deep ], target, object1 [, objectN ] )

//immutableJS
import { Map } from 'immutable';
let a = Map({
  select: 'users',
  filter: Map({ name: 'Cam' })
})
let b = a.set('select', 'people');
a === b; // false
```

## Object.assign

Object.assign(target, ...sources)：将所有可枚举属性的值从一个或多个源对象复制到目标对象。它将返回目标对象

如果对象的属性值为简单类型（string， number），通过Object.assign({},srcObj);得到的新对象为‘深拷贝’；如果属性值为对象或其它引用类型，那对于这个对象而言其实是浅拷贝的

```javascript
let a = {
    name: "target",
    age: 18
}
let b = {
    name: "source",
    book: {
        title: "have a try",
        price: "45"
    }
}
let c = Object.assign(a, b);//c === a --> true, a != b ---> true
```

## 函数传参

值传递 or 引用传递？这是个问题

举个例子

```javascript
function setName(obj) {
    obj.name = "Nicholas";
}

var person = new Object();
setName(person);   // obj = person
alert(person.name);   // "Nicholas" 看起来是按引用传递，但不是按引用传递~~~
```

迷惑的例子

```javascript
function setName(obj) {
    obj.name = "Nicholas";
    obj = new Object(); //改变obj的指向，此时obj指向一个新的内存地址，不再和person指向同一个
    obj.name = "Greg";
}

var person = new Object();
setName(person);  //你看看下面，相信我也是按值传递的了吧
alert(person.name);  //"Nicholas"
```

## 如何避免

如果你不确定赋值操作会带来什么负面影响，则深拷贝就是最佳选择。比如在Vue中，给data中的属性赋值，或在React中，给state某个属性赋值。
