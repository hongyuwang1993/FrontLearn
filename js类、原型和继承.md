# js类,原型和继承

## 简单的类
#### 简单类的创建
```javascript
/**
 * 简单类的实例
 * @param name
 * @constructor
 */
function Person(name) {
    this.name = name;

    this.getName = function () {
        //这个this，指向Person作用域
        return this.name;
    };
    function setName(name) {
        //这个this指向调用函数的作用域
        this.name = name;
    }
}
//这里构造两个person实例
var p1 = new Person("person1");
var p2 = new Person("person2");
//把实例打印出来
console.log(p1, p2);
```
- 类的构造函数就是一个第一个字母大写的普通函数
- 通过new操作符创建一个类。没有返回值
- 通过new创建类，this指向这个类。如果直接调用就是普通的函数，this指向调用这个函数的宿主

```javascript
//直接调用
Person("直接调用");
console.log(window.name);
```

#### 调用类的属性
```javascript
console.log(p1.name);
console.log(p1.getName());
```
- 没有和java一样的访问权限控制，相当于所有的都是public
- setName这个方法再外面没法调用。作用域分为函数作用域和全局作用域（es6新增了块级作用域）
- Person是一个闭包。Person内部可以调用Person中和全局作用域中的属性和方法，在全局作用域中不能调用Person中的方法

```javascript
//调用这个方法会报错
p1.setName("setPerson");
```
#### 判断是否类的实例
```javascript
//判断是否类的实例
console.log(p1 instanceof Person);
```
- 用instanceof来判断一个实例是否是某个类的实例
- 所有类都是Object对象的实例

```javascript
//是否是Object的实例？
console.log(p1 instanceof Object);
```

## 原型
#### 原型对象
- 每个函数（包括构造函数和普通函数）都有一个prototype属性，这个属性引用了一个对象，叫做原型对象
- （原型的作用）让所有的对象实例都共用它的属性和方法

```javascript
/**
 * 简单类的实例
 * @param name
 * @constructor
 */
function Person(name) {
    this.name = name;
}
//在原型上声明一个方法
Person.prototype.getName = function () {
    return this.name;
};
var p1 = new Person("person1");
var p2 = new Person("person2");
//调用getName方法，也会成功
console.log(p1.getName(), p2.getName());

//打印Person函数的原型对象
console.log("Person.prototype", Person.prototype);
``` 
- 原型对象内部有一个不可枚举的属性constructor，指向Person自身，创建一个类时，用来初始化
- 每个对象都有一个__proto__属性，包括原型对象

```javascript
//创建一个对象
var obj = new Person("obj");
//打印这个对象的__proto__属性
console.log("obj.__proto__:", obj.__proto__);

//验证obj.__proto__和Person.prototype是指向同一个对象
console.log(obj.__proto__ === Person.prototype);
```
- obj创建的时候，会让obj的__proto__指向Person的prototype属性
- obj创建时，把obj作为this调用constructor进行初始化
- new的内部过程：
    - var o = new Object()
    - o.__proto__ = Person.prototype
    - Person.call(o)


#### 原型链查找
- 多个原型拼接起来的链叫原型链

```flow
st=>start: 开始
e=>end: 结束
op1=>operation: obj
op2=>operation: Person.prototype
op3=>operation: Object.prototype

st->op1->op2->op3->e
```

```
1. obj中是否有'getName'方法，有则停止，没有继续
2. 从Person的原型对象中查找，有则停止，没有继续
3. 从Object的原型中查找，有则停止，没有就是没有
```

```javascript
function Chart() {
    //构造函数内申明一个方法
    this.run = function () {
        console.log("chart run");
    }
}

Chart.prototype.run = function () {
    //通过原型对象添加一个方法
    console.log("Chart.prototype run");
};

Chart.prototype.log = function () {
    //通过原型对象添加一个方法
    console.log("Chart.prototype log");
};

Object.prototype.eat = function () {
    //通过顶层的原型对象添加一个方法
    console.log("Object.prototype eat");
};

var chart = new Chart();
//这里会调用到构造函数中申明的方法
chart.run();
//先从构造函数查找，找不到，会到构造函数的原型对象中找
chart.log();
//都找不到，会从顶层的原型对象中找
chart.eat();
```

## 类的继承
#### 借用原型链

```javascript
//基本的类
function Animal() {
    this.name = "animal";
    this.eat = function () {
        console.log(this.name + " is eating");
    }
}
//基本方法
Animal.prototype.getName = function () {
    return this.name;
};

var animal = new Animal();
//查看Animal的方法
console.log(animal.getName());
animal.eat();

//继承的类
function Dog() {
    this.name = "dog";
}

Dog.prototype = new Animal();
var dog = new Dog();
//查看dog对象
console.log(dog);
//通过原型链调用Animal的方法
console.log(dog.getName());
dog.eat();
```
- 原理
    - 本质是重写原型对象，代之以一个新类型的实例
    - dog通过原型链就可以调用到Animal中的属性和方法
- 缺陷
    - 所有对象实例都会共享原型对象。导致一个对象实例对原型对象的变更会引起所有该类型实例的变更
    - 没办法在不影响所有对象实例的情况下，给超类型构造函数传参数

```javascript
//基本的类
function Animal() {
    this.common = ["common"];
}

//继承的类
function Dog() {
    this.name = "dog";
}

Dog.prototype = new Animal();
var dog = new Dog();
//查看dog的common属性
console.log(dog.common);

var dog1 = new Dog();
//查看dog1的common属性
console.log(dog1.common);
//更改dog1的common属性
dog1.common.push("dog1");
//导致所有实例对象的common属性都被更改
console.log("dog1", dog1.common);
console.log("dog", dog.common);
```
#### 借用构造函数

```javascript
//基本的类
function Animal(name) {
    this.name = name;
    this.eat = function () {
        console.log(this.name + " is eating");
    }
}
//基本方法
Animal.prototype.getName = function () {
    return this.name;
};
//*********************借用构造函数*****************
function Cat() {
    //调用Animal构造函数
    Animal.call(this, "cat");
}

var cat = new Cat();

//调用eat方法
cat.eat();
//这个方法会调用失败，因为Cat不会去查找到Animal的原型对象
console.log(cat.getName());
```
- 原理
    - 通过call方法改变this对象，把基类的属性赋给子类
- 缺点
    - 函数无法复用，基类中的函数没法调用

#### 组合继承

```javascript
//基本的类
function Animal() {
    this.common = ["common"];
    this.name = "animal";
    this.eat = function () {
        console.log(this.name + " is eating");
    }
}
//基本方法
Animal.prototype.getName = function () {
    return this.name;
};

//**********************组合继承*******************
function Panda(age) {
    //调用Animal的构造函数
    Animal.call(this);
    //自己额外定义age属性
    this.age = age;
}
Panda.prototype = new Animal();

var panda1 = new Panda(18);
var panda2 = new Panda(20);
//查看两个panda实例的结构
console.log(panda1, panda2);
//可以调用基类的原型对象中的方法
console.log(panda1.getName());
panda1.common.push("panda1");
//改变panda1中的common属性不影响panda2中的。每个对象实例属性独立
console.log(panda1.common, panda2.common);
```
- 原理
    - 借用构造函数让每个实例拥有自己的属性，借用原型链共享了方法

#### 原型式继承

```javascript
//***********原型式继承*********************
function object(o) {
    //一个空的构造函数
    function F() {

    }
    //原型对象指向传入的o
    F.prototype = o;
    return new F();
}
//所有实例都会共享这些属性
var o = {
    name: "name",
    common: ["test1"],
    getCommon: function () {
        return this.common;
    }
};
var p1 = object(o);
var p2 = object(o);
console.log(p1, p2);
//p1对引用对象的改变会引起所有实例的改变
p1.common.push("test2");
console.log(p2.getCommon());
```
- 原理
    - 和原型链继承原理类似
- 缺点
    - 原型链继承的缺点都有

#### 寄生式继承
   
```javascript
    //*****************寄生式继承*********************
function object(o) {
    //一个空的构造函数
    function F() {

    }
    //原型对象指向传入的o
    F.prototype = o;
    return new F();
}
function createAnother(o) {
    //调用原型式的方法
    var clone = object(o);
    //给这个临时创建出来的实例添加自己的属性
    clone.sayHi = function () {
        console.log("hi");
    };
    return clone;
}
var o = {
    name: "name"
};
var p = createAnother(o);
//实例有自己的属性
p.sayHi();
```
- 原理
    - 在原型式继承上包了一层，让不同的实例有自己的额外的属性

#### 寄生式组合继承

```javascript
//**********************寄生式组合继承******************
function object(o) {
    //一个空的构造函数
    function F() {

    }
    //原型对象指向传入的o
    F.prototype = o;
    return new F();
}
/**
 * 让child继承parent
 * @param parent 父类构造函数
 * @param child 子类的构造函数
 */
function extend(parent, child) {
    var prototype = object(parent.prototype);
    prototype.constructor = child;
    child.prototype = prototype;
}
//父类的构造函数
function Parent(name) {
    this.name = name;
    this.common = ["test"];
}
//父类原型对象定义一个方法
Parent.prototype.getName = function () {
    return this.name;
};
//子类的构造函数
function Child(name, age) {
    Parent.call(this, name);
    this.age = age;
}

//让子类继承父类
extend(Parent, Child);

//子类原型对象定义一个方法.必须在继承后赋值
Child.prototype.getAge = function () {
    return this.age;
};
var child = new Child("child", 18);
console.log(child.getName());
console.log(child.getAge());
```
- 组合继承会调用两次父类的构造函数，而且父类和子类中都会保留一份父类定义的属性
- 寄生组合式继承目前是最完美的继承方式


