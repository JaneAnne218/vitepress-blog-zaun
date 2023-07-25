---
title: JS
description: JS基础
date: 2023-6-10
tags:
  - JS
---
> JS
> JS是一种面向对象语言，简称"OOP"（Object-Oriented Programming）
## 对象与类的区别

> 类是ES6引入的概念

1. 对象的创建和属性访问
```js
// 使用字面量创建对象
const person = {
  name: 'John',
  age: 30,
  sayHello() {
    console.log(`Hello, my name is ${this.name}.`);
  }
};

console.log(person.name); // 输出：John
person.sayHello(); // 输出：Hello, my name is John.

```
2. 类的创建和实例化，这里有构造函数。
```js
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  sayHello() {
    console.log(`Hello, my name is ${this.name}.`);
  }
}

// 创建 Person 类的实例
const person1 = new Person('John', 30);
const person2 = new Person('Jane', 25);

console.log(person1.name); // 输出：John
person2.sayHello(); // 输出：Hello, my name is Jane.

```
3. 类的继承
```js
class Animal {
  constructor(name) {
    this.name = name;
  }

  sayHello() {
    console.log(`Hello, I'm ${this.name}.`);
  }
}
//共享sayHello()方法，独有bark()方法
class Dog extends Animal {
  bark() {
    console.log('Woof woof!');
  }
}

const dog = new Dog('Buddy');
dog.sayHello(); // 输出：Hello, I'm Buddy.
dog.bark(); // 输出：Woof woof!
```
## modified
- [ ] 如何给文档之间添加跳转  
比如，这里我想添加，ES6与common Js的区别 
> 希望可以看懂比教专业的文档
## querySelector
querySelector是一个方法，它可以接受一个CSS选择器作为参数，然后返回与该模式匹配的第一个元素，如果没有找到匹配的元素，它将返回null。
css选择器是一种语法，它允许您通过元素的属性，类型，ID，类等来选择元素。
元素的属性是指元素的属性，例如id，class，type等。
元素的类型是指元素的类型，例如div，p，h1等。
```js
const element = document.querySelector('p');


```
## css
css是一种语言，它描述了HTML文档的外观和格式。
从那几个方面学习css：
- [ ] 选择器
- [ ] 盒模型
- [ ] 布局
- [ ] 响应式设计
- [ ] css动画
- [ ] css预处理器
- [ ] css框架
- [ ] css网格
- [ ] css变量
1. 选择器
选择器是一种模式，它允许您选择要样式化的元素。
```css
/* 选择所有段落 */
p {
  color: red;
}
/* 选择所有类为“my-class”的元素 */
.my-class {
  color: red;
}
/* 选择所有ID为“my-id”的元素 */
#my-id {
  color: red;
}
/* 选择所有类型为“div”的元素 */
div {
  color: red;
}
```
2. 盒模型
盒模型是一种布局模型，它描述了元素如何在网页上占用空间。
盒模型由四个部分组成：内容，填充，边框和边距。
```css
/* 设置元素的宽度和高度 */
div {
  width: 100px;
  height: 100px;
  /* 设置元素的填充 */
  padding: 10px;
  /* 设置元素的边框 */
  border: 1px solid black;
  /* 设置元素的边距 */
  margin: 10px;
}
```
3. 布局
布局是一种技术，它描述了如何在网页上放置元素。
css布局有三种类型：定位，浮动和弹性。
```css
/* 定位 */
/* 设置元素的位置 */
div {
  position: absolute;
  top: 10px;
  left: 10px;
}
/* 浮动 */
/* 设置元素的浮动 */
div {
  float: left;
}
/* 弹性 */
/* 设置元素的弹性 */
div {
  display: flex;
}
```
