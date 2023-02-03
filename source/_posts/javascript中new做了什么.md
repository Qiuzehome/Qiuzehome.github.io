---
title: javascript中new做了什么
date: 2023-02-01 11:55:00
tags: JavaScript
description: 在JavaScript中，new的时候，它都做了什么
---
# 1.new是什么
在JavaScript中，new操作符用于创建一个给定构造函数的实例对象
````javascript
function Person(name){
	this.name = name
}

Person.prototype.getName = function () {
    console.log(this.name)
}
const person1 = new Person('A')
console.log(person1)  // Person {name: "A"}
person1.getName() // 'A'
````
从以上代码可以推测到，new一个实例对象的时候，它实现了这些功能
1. new出来的实例对象能访问到构造函数中的属性
2. new出来的实例对象能访问到构造函数原型链中的属性

我们对构造函数做一些改造，让它返回一些东西
````javascript
function Person(name){
	this.name = name
	return 1
}
const person1 = new Person('A')
console.log(person1) //Person {name:"A"}
````
由此可发现，虽然构造函数返回了一个1，但是new出来的实例并没受此影响，返回值无生效

假如将构造函数的返回值换成一个对象
````javascript
function Person(name){
	this.name = name
	return {age:1}
}
const person1 = new Person('A')
console.log(person1) //Person {age:1}
````
这里可以发现，构造函数返回了一个对象，那么new出来的实例会使用的是这个对象，而不是构造函数的this。
# 2.new做了什么
在new一个实例对象的时候，其实它做了以下操作
1. 创建一个空对象
2. 将空对象的原型链指向构造函数的原型链
3. 执行构造函数，将this绑定到空对象上
4. 根据构造函数的返回值做判断，若是原始数据则忽略，若是对象的话则正常返回处理
# 3.手撕new
```javascript
function myNew(fn,...args){
	//创建一个空对象
	let obj = {}
	//将obj的原型指向构造函数的原型
	obj.__proto__ = fn.prototype;
	//将构造函数的this指向obj，并且拿到构造函数返回值
	let res = Fn.apply(obj,args)
	//判断构造函数返回值类型，决定new构造出来的实例对象的值
	return res instanceof Object ? res:obj
}
```

