> 原文地址：https://github.com/sohamkamani/javascript-design-patterns-for-humans
>
> 原文作者：[Soham Kamani](https://github.com/sohamkamani) 

![Design Patterns For Humans](./cover.png)

<p align="center">
🎉 超简单的设计模式指南！ 🎉
</p>
<p align="center">
设计模式这个话题往往令人感到困惑。在这里我会尝试以最简单的方式来解释它，并且尽量能够让读者熟记于心（也许是我自己）。
<br/>
本文基于 <a href="https://github.com/kamranahmedse/design-patterns-for-humans">"Design patterns for humans"</a>
</p>

🚀 介绍
=================

设计模式是**如何处理某些重复问题**的指南和解决方案。它们并不是类、包或者库等能够集成到你的应用程序中去，更不会在你的应用中产生任何魔法。相反，设计模式是如何在特定情景下解决特定问题的通用指南。

维基百科是这么解释设计模式的：

> 在[软件工程](https://zh.wikipedia.org/wiki/軟體工程)中，设计模式是对[软件设计](https://zh.wikipedia.org/wiki/軟件設計)中普遍存在（反复出现）的各种问题，所提出的解决方案。设计模式并不直接用来完成[代码](https://zh.wikipedia.org/wiki/%E7%A8%8B%E5%BC%8F%E7%A2%BC)的编写，而是描述在各种不同情况下，要怎么解决问题的一种方案。

⚠️ 注意
-----------------
- 设计模式并不是解决问题的银弹。
- 不要刻意地去使用设计模式，否则可能会带来坏的结果。要记住设计模式是问题的解决方案，而不是发现问题的解决方案，因此不要滥用它们。
- 如果在正确的地方以正确的方式使用设计模式，它们就会有所帮助。若不是这样，则会让你的代码变得一团糟。


## 🐢 在开始之前

- 本文所有的设计模式都是基于 JavaScript 中最新的 ES6 语法实现。
- 因为 JavaScript 并没有接口的实现，所以本文中的例子使用的都是隐喻的接口，这意味着，只要一个类具有特定接口应该拥有的属性和方法，那就认为它实现了这个接口。为了更容易地告诉我们正在使用的接口，你可以在每个例子的注释中找到相关的信息。

设计模式类型
-----------------

* [创建型](#creational-design-patterns)
* [结构型](#structural-design-patterns)
* [行为型](#behavioral-design-patterns) 


创建型设计模式
==========================

简单地来说

> 创建型模式专注于如何实例化一个对象或者一组相关的对象。

维基百科上的解释

> 在软件工程中，创建型模式是处理对象创建的设计模式，试图根据实际情况使用合适的方式创建对象。基本的对象创建方式可能会导致设计上的问题，或增加设计的复杂度。创建型模式通过以某种方式控制对象的创建来解决问题。

 * [简单工厂模式](#-simple-factory)
 * [工厂方法模式](#-factory-method)
 * [抽象工厂模式](#-abstract-factory)
 * [建造者模式](#-builder)
 * [原型模式](#-prototype)
 * [单例模式](#-singleton)

🏠 简单工厂模式
--------------
现实生活中的例子

> 考虑，你正在建造一座房子并且你需要一些门。如果每次当你需要一扇门的时候，你就得穿上木匠的衣服，开始在你的房子里做一扇门，那将会变得一团糟。相反，你可以从工厂中得到它。

简单地来说

> 简单工厂模式只是生成一个实例，而不会暴露任何实例化的逻辑给客户端。

维基百科上的解释

> 在面向对象编程 (OOP) 中，工厂是一个用来创建其他对象的对象 - 从形式上来讲工厂是一个函数或者方法，调用它能够返回不同原型或类的对象，而这些对象往往通过 **new** 生成。

**编程示例**

首先我们会创建一个 door 接口并实现这个接口

```js
/*
Door

getWidth()
getHeight()

*/

class WoodenDoor {
  constructor(width, height){
    this.width = width
    this.height = height
  }

  getWidth(){
    return this.width
  }

  getHeight(){
    return this.height
  }
}
```
然后我们会创建一个能够制造门并且返回它的 door 工厂

```js
const DoorFactory = {
  makeDoor : (width, height) => new WoodenDoor(width, height)
}
```
最后它可以这样使用

```js
const door = DoorFactory.makeDoor(100, 200)
console.log('Width:', door.getWidth())
console.log('Height:', door.getHeight())
```

**什么时候使用？**

当创建一个对象不仅仅是一些赋值和一些逻辑时，将其放在一个专用的工厂而不是到处重复相同的代码，这样的做法往往是有意义的。

🏭 工厂方法模式
--------------

现实生活中的例子

> 让我们以公司的招聘经理为例。要知道一个人不可能充当所有职位的面试官。基于现有的开放职位，招聘经理需要决定并将具体的面试流程下发给专门的面试官。

简单地来说

> 它提供了一种将实例化逻辑委托到子类的方法。

维基百科上的解释

> 在基于类的编程中，工厂方法模式是一种创建模式，它使用工厂方法来处理创建对象的问题，而无需指定将要创建的对象的确切类。 这是通过调用工厂方法创建对象来完成的 - 在接口中指定并由子类实现，或者在基类中实现并可选地由派生类覆盖 - 而不是通过调用构造函数。

 **编程示例**

让我们以上面的招聘经理为例。首先我们拥有一些面试官的接口以及相关的实现。

```js
/*
Interviewer interface

askQuestions()
*/

class Developer {
  askQuestions() {
    console.log('Asking about design patterns!')
  }
}

class CommunityExecutive {
  askQuestions() {
    console.log('Asking about community building')
  }
}
```

然后让我们来创建招聘经理类

```js
class HiringManager {
    takeInterview() {
        const interviewer = this.makeInterviewer()
        interviewer.askQuestions()
    }
}
```
现在任何子类都可以继承招聘经理类并提供相应的面试官

```js
class DevelopmentManager extends HiringManager {
    makeInterviewer() {
        return new Developer()
    }
}

class MarketingManager extends HiringManager {
    makeInterviewer() {
        return new CommunityExecutive()
    }
}
```
最后它可以这样使用

```js
const devManager = new DevelopmentManager()
devManager.takeInterview() // Output: Asking about design patterns

const marketingManager = new MarketingManager()
marketingManager.takeInterview() // Output: Asking about community buildng.
```

**什么时候使用？**

当在类中有一些通用逻辑并且所需的子类又是运行时多态的时候。换句话说，当客户端不知道它需要什么样的子类时，就可以使用工厂方法模式。

🔨 抽象工厂模式
----------------

现实生活中的例子

> 让我们从简单工厂模式中建造门的例子延伸开来。基于实际情况你可能会从木门店买到一扇木门，从造铁厂买到一扇铁门，或者从相关的商店买到一扇 PVC 门。另外，你可能需要不同的专业人士来安装不同的门，例如木匠安装木门，焊接工安装铁门等等。如你所见，现在不同的门都有其不同的依赖，木门需要木匠，铁门需要焊接工等等。

简单地来说

> 嵌套着工厂的工厂类能够将独立但又相互依赖的工厂组合在一起，而不会说明其具体的类。

维基百科上的解释

> 抽象工厂模式提供了一种方法来封装一组具有公共主题的独立工厂，而无需指定它们的具体类。

**编程示例** 

让我们改造上文建造门的例子。首先我们有 **`Door`** 接口以及相关的实现。

```js
/*
Door interface :

getDescription()
*/

class WoodenDoor {
    getDescription() {
        console.log('I am a wooden door')
    }
}

class IronDoor {
    getDescription() {
        console.log('I am an iron door')
    }
}
```
然后我们有安装每种门的专家

```js
/*
DoorFittingExpert interface :

getDescription()
*/

class Welder {
    getDescription() {
        console.log('I can only fit iron doors')
    }
}

class Carpenter {
    getDescription() {
        console.log('I can only fit wooden doors')
    }
}
```

现在让我们来实现能够生成一组相关对象的抽象工厂，也就是说，木门工厂能够创建一扇木门还能生成一个木匠用来安装木门，铁门工厂能够创建一扇铁门还能生成一个焊接工来焊接铁门。

```js
/*
DoorFactory interface :

makeDoor()
makeFittingExpert()
*/

// Wooden factory to return carpenter and wooden door
class WoodenDoorFactory {
    makeDoor(){
        return new WoodenDoor()
    }

    makeFittingExpert() {
        return new Carpenter()
    }
}

// Iron door factory to get iron door and the relevant fitting expert
class IronDoorFactory {
    makeDoor(){
        return new IronDoor()
    }

    makeFittingExpert() {
        return new Welder()
    }
}
```
最后它可以这样使用
```js
woodenFactory = new WoodenDoorFactory()

door = woodenFactory.makeDoor()
expert = woodenFactory.makeFittingExpert()

door.getDescription()  // Output: I am a wooden door
expert.getDescription() // Output: I can only fit wooden doors

// Same for Iron Factory
ironFactory = new IronDoorFactory()

door = ironFactory.makeDoor()
expert = ironFactory.makeFittingExpert()

door.getDescription()  // Output: I am an iron door
expert.getDescription() // Output: I can only fit iron doors
```

如你所见木门工厂封装了木匠和木门，铁门工厂封装了铁门和焊接工。这确保了每生成一扇门，都会有对应的人员来安装它，并且不会出错。

**什么时候使用？**

在创建对象时，含有相关的依赖并且有比较复杂的逻辑时可以使用抽象工厂模式。

👷 建造者模式
--------------------------------------------
现实生活中的例子

> 假设你在 Hardee's 汉堡店点餐，当你说 “Big Hardee” 时服务员会毫不犹豫地为你递上本店的招牌汉堡，这也是简单工厂模式的做法。但是在某些情况下，创建对象的逻辑可能会包含更多步骤。例如你想要一个定制的汉堡，这时候你会选择面包的类型，酱料的类型以及芝士的类型。这时候建造者模式就能够排上用场了。

简单地来说

> 建造者模式允许你创建不同类型的对象同时又能避免构造函数污染。当一个对象可能有多种类型或者在创建对象时会经历很多个步骤的时候该模式很有用。

维基百科上的解释

> 建造者模式是关于对象创建的软件设计模式，目的是为可伸缩构造函数反模式找到相应的解决方案。

既然说到这里，就让我们来了解一下什么是可伸缩构造函数反模式。有些时候我们可能会看到一个构造函数像下面这样：

```js
constructor(size, cheese = true, pepperoni = true, tomato = false, lettuce = true) {
    // ... 
}
```

如你所见构造函数中参数的数量很容易变得不可控制，同时也很难去理解参数的排列。更糟糕的是，未来该构造函数中还可能增加更多的参数。这就叫做可伸缩构造函数反模式。

**编程示例**

明智的选择是使用建造者模式。首先我们来实现 Burger 类。

```js
class Burger {
    constructor(builder) {
        this.size = builder.size
        this.cheeze = builder.cheeze || false
        this.pepperoni = builder.pepperoni || false
        this.lettuce = builder.lettuce || false
        this.tomato = builder.tomato || false
    }
}
```

然后我们实现 BurgerBuilder 类

```js
class BurgerBuilder {

    constructor(size) {
        this.size = size
    }

    addPepperoni() {
        this.pepperoni = true
        return this
    }

    addLettuce() {
        this.lettuce = true
        return this
    }

    addCheeze() {
        this.cheeze = true
        return this
    }

    addTomato() {
        this.tomato = true
        return this
    }

    build() {
        return new Burger(this)
    }
}
```
最后它可以这样使用

```js
const burger = (new BurgerBuilder(14))
    .addPepperoni()
    .addLettuce()
    .addTomato()
    .build()
```

**友情提示：**当你发现一个函数或者方法中的参数太多（通常两个以上）时，用一个对象来替换这些参数往往是更好的选择。这样做有两个好处：

1. 能够使你的代码看起来不那么混乱，因为现在只有一个参数。
2. 你不需要担心参数的顺序因为这些参数现在是作为对象的属性传递。

例如：

```js
const burger = new Burger({
    size : 14,
    pepperoni : true,
    cheeze : false,
    lettuce : true,
    tomato : true
})
```

而不是：

```
const burger = new Burger(14, true, false, true, true)
```

**什么时候使用？**

当一个对象上有多种类型并且还要避免构造函数伸缩问题时可以使用建造者模式。它跟工厂模式的主要差别在于工厂模式创建对象时只经历一个步骤而建造者模式则需要经历多个步骤。


💍 单例模式
------------
现实生活中的例子

> 在一个时期一个国家只允许有一位总统。无论什么时候需要履行与国家事务相关的职责，都必须让同一位总统采取行动。在这里总统就相当于单例。

简单地来说

> 确保只创建特定类的一个对象

维基百科上的解释

> 在软件工程中，单例模式是一种软件设计模式，它将类的实例化限制为一个对象。 当需要一个对象来协调整个系统的操作时，这非常有用。

单例模式事实上被看作是一种反模式，应该避免过度使用它。虽然它没有那么糟糕并且有其合适的使用场景，但我们也应该谨慎使用该模式，因为它会给你的应用带来全局的状态，在一处进行修改可能会影响其他的很多地方，因此调试就会变得十分困难。

**编程示例**

在 JavaScript 中，单例模式可以用模块模式实现。私有的变量和函数可以在函数闭包中隐藏，公有的方法则会选择性地暴露出去。

```js
const president = (function(){
    const presidentsPrivateInformation = 'Super private'

    const name = 'Turd Sandwich'

    const getName = () => name

    return {
        getName
    }
}())
```

在这里，`presidentsPrivateInformation` 和 `name` 作为私有变量保存。但只有 `name` 能够通过暴露的 `president.getName` 方法访问。

```js
president.getName() // Outputs 'Turd Sandwich'
president.name // Outputs undefined
president.presidentsPrivateInformation // Outputs undefined
```

结构型设计模式
==========================
简单地来说

> 结构型设计模式专注于对象间的组合，换句话说则是实体间如何相互使用。另外一种解释是，它们是如何构建软件组件的解决方案。

维基百科上的解释

> 在软件工程中，结构型设计模式是通过找寻实现实体间关系的简单方法来简化设计的设计模式。

 * [适配器模式](#-adapter)
 * [桥模式](#-bridge)
 * [组合模式](#-composite)
 * [装饰器模式](#-decorator)
 * [外观模式](#-facade)
 * [享元模式](#-flyweight)
 * [代理模式](#-proxy) 

🔌 适配器模式
-------
现实生活中的例子

> 考虑在你的存储卡上有一些图片，你需要把它们传输到你的电脑上。为了传输图片，你需要某种与计算机端口兼容的适配器，以便可以将存储卡附加到计算机上。在这个例子中读卡器就是一个适配器。
>
> 另一个例子是著名的电源适配器，很明显一个三个腿的插头不能够接到两个腿的插头上。
>
> 再比如翻译官就像是适配器，能够将一个人说的话翻译给另一个人听。

简单地来说

> 适配器模式允许你将不兼容的对象包装在适配器中，使其与另一个类兼容。

维基百科上的解释

> 在软件工程中，适配器模式是一种软件设计模式，它允许将现有类的接口用作于另一个接口。 它通常用于使现有类与其他类一起工作而无需修改其源代码。

**编程示例**

考虑现在我们有一个猎人捕猎狮子的游戏。

首先我们有需要被实现的不同类型狮子的 `Lion` 接口。

```js
/*
Lion interface :

roar()
*/

class AfricanLion  {
    roar() {}
}

class AsianLion  {
    roar() {}
}
```
猎人期望着捕猎任何实现了 `Lion` 接口的狮子。

```js
class Hunter {
    hunt(lion) {
        // ... some code before
        lion.roar()
        //... some code after
    }
}
```

假如现在游戏中加入了 `WildDog` 类，并且猎人也能对其进行捕猎。由于狗拥有不同类型的接口，所以我们不能直接捕猎它。为让它与我们的猎人兼容，我们需要创建一个具有兼容性的适配器。

```js
// This needs to be added to the game
class WildDog {
    bark() {
    }
}

// Adapter around wild dog to make it compatible with our game
class WildDogAdapter {

    constructor(dog) {
        this.dog = dog;
    }
    
    roar() {
        this.dog.bark();
    }
}
```
通过 `WildDogAdapter` 类我们的 `WildDog` 就能够被猎人捕猎。 

```js
wildDog = new WildDog()
wildDogAdapter = new WildDogAdapter(wildDog)

hunter = new Hunter()
hunter.hunt(wildDogAdapter)
```

🚡 桥模式
------
现实生活中的例子

> 假设你有一个包含不同页面的网站，并且能够让用户改变不同的主题。你会怎样实现该功能？是为每个页面都重复创建不同的主题还是将主题抽离开来并根据用户的设置加载不同的主题？桥模式就能够让你实现第二种方案。

![With and without the bridge pattern](https://cloud.githubusercontent.com/assets/11269635/23065293/33b7aea0-f515-11e6-983f-98823c9845ee.png)

简单地来说

> 桥模式更倾向于组合而不是继承。其实现细节从一个层次结构被推到了另一个具有独立层次结构的对象。

维基百科上的解释

> 桥模式是软件工程中使用的设计模式，旨在“将抽象与其实现分离，以便两者可以独立变化“。

**编程示例**

以上面的网站页面为例。下面是我们的 `WebPage` 接口

```js
/*
Webpage interface :

constructor(theme)
getContent()
*/

class About{ 
    constructor(theme) {
        this.theme = theme
    }
    
    getContent() {
        return "About page in " + this.theme.getColor()
    }
}

class Careers{
   constructor(theme) {
       this.theme = theme
   }
   
   getContent() {
       return "Careers page in " + this.theme.getColor()
   } 
}
```
不同主题的接口
```js
/*
Theme interface :

getColor()
*/

class DarkTheme{
    getColor() {
        return 'Dark Black'
    }
}
class LightTheme{
    getColor() {
        return 'Off white'
    }
}
class AquaTheme{
    getColor() {
        return 'Light blue'
    }
}
```
将两个接口组合使用
```js
const darkTheme = new DarkTheme()

const about = new About(darkTheme)
const careers = new Careers(darkTheme)

console.log(about.getContent() )// "About page in Dark Black"
console.log(careers.getContent() )// "Careers page in Dark Black"
```

🌿 组合模式
-----------------

现实生活中的例子

> 每个公司都有很多职员。每个职员都有很多特性比如薪资和工作职责，是否需要向某人汇报，是否有下属等等。

简单地来说

> 组合模式允许客户端能够以统一的方式处理单个对象。

维基百科上的解释

> 在软件工程中，组合模式是一种分区设计模式。 组合模式描述了一组对象的处理方式与对象的单个实例相同。 组合的意图是将对象“组合”成树结构以表示部分整体层次结构。 通过实现组合模式，客户端可以统一处理单个对象以及对象间的组合。

**编程示例**

让我们以上文的职员为例，假设我们有不同类型的职员。

```js
/*
Employee interface :

constructor(name, salary)
getName()
setSalary()
getSalary()
getRoles()
*/

class Developer {

    constructor(name, salary) {
        this.name = name
        this.salary = salary
    }

    getName() {
        return this.name
    }

    setSalary(salary) {
        this.salary = salary
    }

    getSalary() {
        return this.salary
    }

    getRoles() {
        return this.roles
    }

    develop() {
        /* */
    }
}

class Designer {

    constructor(name, salary) {
        this.name = name
        this.salary = salary
    }

    getName() {
        return this.name
    }

    setSalary(salary) {
        this.salary = salary
    }

    getSalary() {
        return this.salary
    }

    getRoles() {
        return this.roles
    }

    design() {
        /* */
    }
}
```

然后我们有一个包含不同类型职员的公司

```js
class Organization {
    constructor(){
        this.employees = []
    }

    addEmployee(employee) {
        this.employees.push(employee)
    }

    getNetSalaries() {
        let netSalary = 0

        this.employees.forEach(employee => {
            netSalary += employee.getSalary()
        })

        return netSalary
    }
}
```

最后它可以这样使用

```js
// Prepare the employees
const john = new Developer('John Doe', 12000)
const jane = new Designer('Jane', 10000)

// Add them to organization
const organization = new Organization()
organization.addEmployee(john)
organization.addEmployee(jane)

console.log("Net salaries: " , organization.getNetSalaries()) // Net Salaries: 22000
```

☕ 装饰器模式
-------------

现实生活中的例子

> 假设你在经营一个提供很多服务的汽车服务商店。你该如何计算相关服务的收取费用？当选择一个服务时就动态地增加相应的费用直到计算出最终的费用。在这里每一项服务就是一个装饰器。

简单地来说

> 装饰器模式允许你在运行时通过将对象包装在装饰器类的对象中动态更改对象的行为。

维基百科上的解释

> 在面向对象编程中，装饰器模式是一种设计模式，它允许将行为静态或动态地添加到单个对象，而不会影响同一类中其他对象的行为。 装饰器模式通常用于遵守单一职责原则，因为它允许在具有独特关注区域的类之间划分功能。

**编程示例**

让我们以制作咖啡为例，首先我们有一个制作简单咖啡的接口。

```js
/*
Coffee interface:
getCost()
getDescription()
*/

class SimpleCoffee{

    getCost() {
        return 10
    }

    getDescription() {
        return 'Simple coffee'
    }
}
```
我们想让代码变得具有扩展性，以便在需要的时候通过选项更改，让我们新增一些装饰器。

```js
class MilkCoffee {


    constructor(coffee) {
        this.coffee = coffee
    }

    getCost() {
        return this.coffee.getCost() + 2
    }

    getDescription() {
        return this.coffee.getDescription() + ', milk'
    }
}

class WhipCoffee {

    constructor(coffee) {
        this.coffee = coffee
    }

    getCost() {
        return this.coffee.getCost() + 5
    }

    getDescription() {
        return this.coffee.getDescription() + ', whip'
    }
}

class VanillaCoffee {

    constructor(coffee) {
        this.coffee = coffee
    }

    getCost() {
        return this.coffee.getCost() + 3
    }

    getDescription() {
        return this.coffee.getDescription() + ', vanilla'
    }
}

```

最后让我们开始制作咖啡

```js
let someCoffee

someCoffee = new SimpleCoffee()
console.log(someCoffee.getCost())// 10
console.log(someCoffee.getDescription())// Simple Coffee

someCoffee = new MilkCoffee(someCoffee)
console.log(someCoffee.getCost())// 12
console.log(someCoffee.getDescription())// Simple Coffee, milk

someCoffee = new WhipCoffee(someCoffee)
console.log(someCoffee.getCost())// 17
console.log(someCoffee.getDescription())// Simple Coffee, milk, whip

someCoffee = new VanillaCoffee(someCoffee)
console.log(someCoffee.getCost())// 20
console.log(someCoffee.getDescription())// Simple Coffee, milk, whip, vanilla
```

📦 外观模式
----------------

现实生活中的例子

> 你是如何打开你的电脑的？当然是“按下电源按钮”，因为你使用的是电脑向外面提供的一个简单接口，而整个开机过程在电脑内部则会进行很多复杂的操作。为复杂的子系统提供简单的接口就是一种外观模式。

简单地来说

> 外观模式为复杂的子系统提供了一个简单的接口以便外部使用。

维基百科上的解释

> 外观是一个对象，它为更大的代码体提供了简化的接口，例如类库。

**编程示例**

让我们以打开电脑为例，下面是我们的电脑类

```js
class Computer {

    getElectricShock() {
        console.log('Ouch!')
    }

    makeSound() {
        console.log('Beep beep!')
    }

    showLoadingScreen() {
        console.log('Loading..')
    }

    bam() {
        console.log('Ready to be used!')
    }

    closeEverything() {
        console.log('Bup bup bup buzzzz!')
    }

    sooth() {
        console.log('Zzzzz')
    }

    pullCurrent() {
        console.log('Haaah!')
    }
}
```
这是我们的外观类
```js
class ComputerFacade
{
    constructor(computer) {
        this.computer = computer
    }

    turnOn() {
        this.computer.getElectricShock()
        this.computer.makeSound()
        this.computer.showLoadingScreen()
        this.computer.bam()
    }

    turnOff() {
        this.computer.closeEverything()
        this.computer.pullCurrent()
        this.computer.sooth()
    }
}
```
最后它可以这样使用
```js
const computer = new ComputerFacade(new Computer())
computer.turnOn() // Ouch! Beep beep! Loading.. Ready to be used!
computer.turnOff() // Bup bup buzzz! Haah! Zzzzz
```

🍃 享元模式
---------

现实生活中的例子

> 你在一些店铺中喝过新鲜的茶吗？他们通常会制作不止一杯你所需要的饮品，然后将剩下的留给其他顾客，这样就可以节省相应的资源比如汽油。享元模式的关键就在于共享。

简单地来说

> 享元模式通过尽可能多地与相似对象共享数据来最小化内存使用或计算开销。

维基百科上的解释

> 在计算机编程中，享元模式是一种软件设计模式。享元模式是通过与其他相似对象共享尽可能多的数据来最小化内存使用的对象，当简单的重复表示将使用不可接受的内存量时，它是一种使用大量对象的方法。

**编程示例**

让我们以上文的喝茶为例，首先我们有不同类型的茶和制作茶的人。

```js
// Anything that will be cached is flyweight. 
// Types of tea here will be flyweights.
class KarakTea {
}

// Acts as a factory and saves the tea
class TeaMaker {
    constructor(){
        this.availableTea = {}
    }

    make(preference) {
        this.availableTea[preference] = this.availableTea[preference] || (new KarakTea())
        return this.availableTea[preference]
    }
}
```

然后我们有 `TeaShop` 类用来让用户点餐和送餐

```js
class TeaShop {
    constructor(teaMaker) {
        this.teaMaker = teaMaker
        this.orders = []
    }

    takeOrder(teaType, table) {
        this.orders[table] = this.teaMaker.make(teaType)
    }

    serve() {
        this.orders.forEach((order, index) => {
            console.log('Serving tea to table#' + index)
        })
    }
}
```
最后它可以这样使用

```js
const teaMaker = new TeaMaker()
const shop = new TeaShop(teaMaker)

shop.takeOrder('less sugar', 1)
shop.takeOrder('more milk', 2)
shop.takeOrder('without sugar', 5)

shop.serve()
// Serving tea to table# 1
// Serving tea to table# 2
// Serving tea to table# 5
```

🎱 代理模式
-------------------
现实生活中的例子

> 你有使用通行证通过一扇门的经历吗？想要打开一扇门可以有很多选择，使用通行证或者输入密码都可以让门打开。门的主要功能是打开，但在它上面有着一层代理，这为门增加了一些其他的功能。

简单地来说

> 使用代理模式，一个类就能够表示另一个类的功能。

维基百科上的解释

> 代理以其最一般的形式，是一个充当其他东西的接口的类。 代理是一个包装器或代理对象，客户端正在调用它来访问幕后的真实服务对象。 使用代理可以简单地转发到真实对象，或者可以提供额外的逻辑。 在代理中，可以提供额外的功能，例如当对真实对象的操作是资源密集时的高速缓存，或者在调用对象的操作之前检查先决条件。

**编程示例**

让我们以安全门为例，首先我们有一个 door 类以及它的相关实现。

```js
/*
Door interface :

open()
close()
*/

class LabDoor {
    open() {
        console.log('Opening lab door')
    }

    close() {
        console.log('Closing the lab door')
    }
}
```
然后我们设置一层代理来保护我们向想要保护的门

```js
class Security {
    constructor(door) {
        this.door = door
    }

    open(password) {
        if (this.authenticate(password)) {
            this.door.open()
        } else {
        	console.log('Big no! It ain\'t possible.')
        }
    }

    authenticate(password) {
        return password === 'ecr@t'
    }

    close() {
        this.door.close()
    }
}
```
最后它可以这样使用
```js
const door = new Security(new LabDoor())
door.open('invalid') // Big no! It ain't possible.

door.open('ecr@t') // Opening lab door
door.close() // Closing lab door
```

行为型设计模式
==========================

简单地来说

> 行为型设计模式与对象间的责任分配有关。与结构型设计模式不同的是，它们不仅指定了结构，同时还概述了对象间消息传递 / 通信的模式。换句话说，它们是如何在软件组件中执行具体操作的解决方案。

维基百科上的解释

> 在软件工程中，行为型设计模式是识别对象之间的共同通信模式并实现这些模式的设计模式。 通过这样做，这些模式增加了执行该通信的灵活性。

* [职责链模式](#-chain-of-responsibility)
* [命令模式](#-command)
* [迭代器模式](#-iterator)
* [中介者模式](#-mediator)
* [备忘录模式](#-memento)
* [观察者模式](#-observer)
* [访问者模式](#-visitor)
* [策略模式](#-strategy)
* [状态模式](#-state)
* [模版方法模式](#-template-method) 

🔗 职责链模式
-----------------------

现实生活中的例子

> 例如，在你的账户中有三种付款方法 ( `A`, `B`, `C` ) ，每种方法都有不同的金额。`A` 中有 100 美元，`B` 中有 300 美元，`C` 中有 1000 美元，但付款的顺序被固定成先 `A` 后 `B` 最后是 `C` 。当你想购买价值为 210 美元的物品时。使用职责链模式，`A` 首先会被检查如果足够用于付款则职责链断开整个过程结束，如果账户余额不足则会将付款请求传递给下一个付款方法直到找到满足要求的方法。在这里 `A`, `B`, `C` 就是链中的环节而整个现象就是一条职责链。

简单地来说

> 职责链模式用于建立一个对象链。请求从一端进入，并从一个对象进入另一个对象，直到找到合适的处理程序。

维基百科上的解释

> 在面向对象的设计中，职责链模式是一种由命令对象源和一系列处理对象组成的设计模式。 每个处理对象都包含一个逻辑，用于定义它可以处理的命令对象的类型，其余的传递给链中的下一个处理对象。

**编程示例**

让我们以上面的账户付款为例。首先，我们有一个基本帐户，它具有将帐户和一些帐户链接在一起的逻辑。

```js
class Account {

    setNext(account) {
        this.successor = account
    }
    
    pay(amountToPay) {
        if (this.canPay(amountToPay)) {
            console.log(`Paid ${amountToPay} using ${this.name}`)
        } else if (this.successor) {
            console.log(`Cannot pay using ${this.name}. Proceeding...`)
            this.successor.pay(amountToPay)
        } else {
            console.log('None of the accounts have enough balance')
        }
    }
    
    canPay(amount) {
        return this.balance >= amount
    }
}

class Bank extends Account {
    constructor(balance) {
        super()
        this.name = 'bank'
        this.balance = balance
    }
}

class Paypal extends Account {
    constructor(balance) {
        super()        
        this.name = 'Paypal'
        this.balance = balance
    }
}

class Bitcoin extends Account {
    constructor(balance) {
        super()        
        this.name = 'bitcoin'
        this.balance = balance
    }
}
```

现在让我们先准备职责链中的链接顺序 ( Bank、PayPal、Bitcoin )

```js
// Let's prepare a chain like below
//      bank.paypal.bitcoin
//
// First priority bank
//      If bank can't pay then paypal
//      If paypal can't pay then bit coin

const bank = new Bank(100)          // Bank with balance 100
const paypal = new Paypal(200)      // Paypal with balance 200
const bitcoin = new Bitcoin(300)    // Bitcoin with balance 300

bank.setNext(paypal)
paypal.setNext(bitcoin)

// Let's try to pay using the first priority i.e. bank
bank.pay(259)

// Output will be
// ==============
// Cannot pay using bank. Proceeding ..
// Cannot pay using paypal. Proceeding ..: 
// Paid 259 using Bitcoin!
```

👮 命令模式
-------

现实生活中的例子

> 一个常见的例子是当你在饭店点餐时，你 ( `Client` ) 让服务员 ( `Invoker` ) 给你一些食物 ( `Command` ) ，服务员会将请求递给会做饭的厨师 ( `Receiver` ) 。
>
> 另一个例子是你 ( `Client` ) 可以通过遥控器 ( `Invoker` ) 切换 ( `Command` ) 电视 ( `Receiver` ) 的频道。

简单地来说

> 命令模式允许您将操作封装在对象中。该模式背后的关键思想是提供将客户端与接收器分离的方法。

维基百科上的解释

> 在面向对象编程中，命令模式是行为型设计模式，其中对象用于封装执行动作或稍后触发事件所需的所有信息。 此信息包括方法名称，拥有该方法的对象以及方法参数的值。

**编程示例** 

首先我们有一个接收者，它实现了所有可以执行的操作。

```js
// Receiver
class Bulb {
    turnOn() {
        console.log('Bulb has been lit')
    }
    
    turnOff() {
        console.log('Darkness!')
    }
}
```
然后我们有一个每个命令都要实现的 Command 接口，这个接口中包含很多命令。

```js
/*
Command interface :

    execute()
    undo()
    redo()
*/

// Command
class TurnOnCommand {
    constructor(bulb) {
        this.bulb = bulb
    }
    
    execute() {
        this.bulb.turnOn()
    }
    
    undo() {
        this.bulb.turnOff()
    }
    
    redo() {
        this.execute()
    }
}

class TurnOffCommand {
    constructor(bulb) {
        this.bulb = bulb
    }
    
    execute() {
        this.bulb.turnOff()
    }
    
    undo() {
        this.bulb.turnOn()
    }
    
    redo() {
        this.execute()
    }
}
```
然后我们有一个调用者 ( `Invoker` ) ，它会接收并执行命令。

```js
// Invoker
class RemoteControl {
    submit(command) {
        command.execute()
    }
}
```
最后让我们来看看它该如何使用

```js
const bulb = new Bulb()

const turnOn = new TurnOnCommand(bulb)
const turnOff = new TurnOffCommand(bulb)

const remote = new RemoteControl()
remote.submit(turnOn) // Bulb has been lit!
remote.submit(turnOff) // Darkness!
```

命令模式还可以用来实现基于事务的系统。你可以保留所有执行命令的操作记录，如果最后的命令被成功执行则没有什么大的问题，否则就需要通过迭代历史记录中命令的 `undo` 方法来撤销所有的操作。

➿ 迭代器模式
--------

Real world example
> An old radio set will be a good example of iterator, where user could start at some channel and then use next or previous buttons to go through the respective channels. Or take an example of MP3 player or a TV set where you could press the next and previous buttons to go through the consecutive channels or in other words they all provide an interface to iterate through the respective channels, songs or radio stations.  

In plain words
> It presents a way to access the elements of an object without exposing the underlying presentation.

Wikipedia says
> In object-oriented programming, the iterator pattern is a design pattern in which an iterator is used to traverse a container and access the container's elements. The iterator pattern decouples algorithms from containers in some cases, algorithms are necessarily container-specific and thus cannot be decoupled.

**Programmatic example**
 Translating our radio stations example from above. First of all we have `RadioStation`

```js
class RadioStation {
    constructor(frequency) {
        this.frequency = frequency    
    }
    
    getFrequency() {
        return this.frequency
    }
}
```
Then we have our iterator

```js
class StationList {
    constructor(){
        this.stations = []
    }

    addStation(station) {
        this.stations.push(station)
    }
    
    removeStation(toRemove) {
        const toRemoveFrequency = toRemove.getFrequency()
        this.stations = this.stations.filter(station => {
            return station.getFrequency() !== toRemoveFrequency
        })
    }
}
```
And then it can be used as
```js
const stationList = new StationList()

stationList.addStation(new RadioStation(89))
stationList.addStation(new RadioStation(101))
stationList.addStation(new RadioStation(102))
stationList.addStation(new RadioStation(103.2))

stationList.stations.forEach(station => console.log(station.getFrequency()))

stationList.removeStation(new RadioStation(89)) // Will remove station 89
```

👽 Mediator
========

Real world example
> A general example would be when you talk to someone on your mobile phone, there is a network provider sitting between you and them and your conversation goes through it instead of being directly sent. In this case network provider is mediator. 

In plain words
> Mediator pattern adds a third party object (called mediator) to control the interaction between two objects (called colleagues). It helps reduce the coupling between the classes communicating with each other. Because now they don't need to have the knowledge of each other's implementation. 

Wikipedia says
> In software engineering, the mediator pattern defines an object that encapsulates how a set of objects interact. This pattern is considered to be a behavioral pattern due to the way it can alter the program's running behavior.

**Programmatic Example**

Here is the simplest example of a chat room (i.e. mediator) with users (i.e. colleagues) sending messages to each other. 

First of all, we have the mediator i.e. the chat room 

```js
// Mediator
class ChatRoom {
    showMessage(user, message) {
        const time = new Date()
        const sender = user.getName()

        console.log(time + '[' + sender + ']:' + message)
    }
}
```

Then we have our users i.e. colleagues
```js
class User {
    constructor(name, chatMediator) {
        this.name = name
        this.chatMediator = chatMediator
    }
    
    getName() {
        return this.name
    }
    
    send(message) {
        this.chatMediator.showMessage(this, message)
    }
}
```
And the usage
```js
const mediator = new ChatRoom()

const john = new User('John Doe', mediator)
const jane = new User('Jane Doe', mediator)

john.send('Hi there!')
jane.send('Hey!')

// Output will be
// Feb 14, 10:58 [John]: Hi there!
// Feb 14, 10:58 [Jane]: Hey!
```

💾 Memento
-------
Real world example
> Take the example of calculator (i.e. originator), where whenever you perform some calculation the last calculation is saved in memory (i.e. memento) so that you can get back to it and maybe get it restored using some action buttons (i.e. caretaker). 

In plain words
> Memento pattern is about capturing and storing the current state of an object in a manner that it can be restored later on in a smooth manner.

Wikipedia says
> The memento pattern is a software design pattern that provides the ability to restore an object to its previous state (undo via rollback).

Usually useful when you need to provide some sort of undo functionality.

**Programmatic Example**

Lets take an example of text editor which keeps saving the state from time to time and that you can restore if you want.

First of all we have our memento object that will be able to hold the editor state

```js
class EditorMemento {
    constructor(content) {
        this._content = content
    }
    
    getContent() {
        return this._content
    }
}
```

Then we have our editor i.e. originator that is going to use memento object

```js
class Editor {
    constructor(){
        this._content = ''
    }
    
    type(words) {
        this._content = this._content + ' ' + words
    }
    
    getContent() {
        return this._content
    }
    
    save() {
        return new EditorMemento(this._content)
    }
    
    restore(memento) {
        this._content = memento.getContent()
    }
}
```

And then it can be used as 

```js
const editor = new Editor()

// Type some stuff
editor.type('This is the first sentence.')
editor.type('This is second.')

// Save the state to restore to : This is the first sentence. This is second.
const saved = editor.save()

// Type some more
editor.type('And this is third.')

// Output: Content before Saving
console.log(editor.getContent())// This is the first sentence. This is second. And this is third.

// Restoring to last saved state
editor.restore(saved)

console.log(editor.getContent()) // This is the first sentence. This is second.
```

😎 Observer
--------

(Otherwise known as _"pub-sub"_)

Real world example
> A good example would be the job seekers where they subscribe to some job posting site and they are notified whenever there is a matching job opportunity.   

In plain words
> Defines a dependency between objects so that whenever an object changes its state, all its dependents are notified.

Wikipedia says
> The observer pattern is a software design pattern in which an object, called the subject, maintains a list of its dependents, called observers, and notifies them automatically of any state changes, usually by calling one of their methods.

**Programmatic example**

Translating our example from above. First of all we have job seekers that need to be notified for a job posting
```js
const JobPost = title => ({
    title: title
})

class JobSeeker {
    constructor(name) {
        this._name = name
    }

    notify(jobPost) {
        console.log(this._name, 'has been notified of a new posting :', jobPost.title)
    }
}
```
Then we have our job postings to which the job seekers will subscribe
```js
class JobBoard {
    constructor() {
        this._subscribers = []
    }

    subscribe(jobSeeker) {
        this._subscribers.push(jobSeeker)
    }

    addJob(jobPosting) {
        this._subscribers.forEach(subscriber => {
            subscriber.notify(jobPosting)
        })
    }
}
```
Then it can be used as
```js
// Create subscribers
const jonDoe = new JobSeeker('John Doe')
const janeDoe = new JobSeeker('Jane Doe')
const kaneDoe = new JobSeeker('Kane Doe')

// Create publisher and attach subscribers
const jobBoard = new JobBoard()
jobBoard.subscribe(jonDoe)
jobBoard.subscribe(janeDoe)

// Add a new job and see if subscribers get notified
jobBoard.addJob(JobPost('Software Engineer'))

// Output
// John Doe has been notified of a new posting : Software Engineer
// Jane Doe has been notified of a new posting : Software Engineer
```

🏃 Visitor
-------
Real world example
> Consider someone visiting Dubai. They just need a way (i.e. visa) to enter Dubai. After arrival, they can come and visit any place in Dubai on their own without having to ask for permission or to do some leg work in order to visit any place here just let them know of a place and they can visit it. Visitor pattern let's you do just that, it helps you add places to visit so that they can visit as much as they can without having to do any legwork.

In plain words
> Visitor pattern let's you add further operations to objects without having to modify them.

Wikipedia says
> In object-oriented programming and software engineering, the visitor design pattern is a way of separating an algorithm from an object structure on which it operates. A practical result of this separation is the ability to add new operations to existing object structures without modifying those structures. It is one way to follow the open/closed principle.

**Programmatic example**

Let's take an example of a zoo simulation where we have several different kinds of animals and we have to make them Sound. Let's translate this using visitor pattern 

We have our implementations for the animals
```js
class Monkey {
    shout() {
        console.log('Ooh oo aa aa!')
    }

    accept(operation) {
        operation.visitMonkey(this)
    }
}

class Lion {
    roar() {
        console.log('Roaaar!')
    }
    
    accept(operation) {
        operation.visitLion(this)
    }
}

class Dolphin {
    speak() {
        console.log('Tuut tuttu tuutt!')
    }
    
    accept(operation) {
        operation.visitDolphin(this)
    }
}
```
Let's implement our visitor
```js
const speak = {
    visitMonkey(monkey){
        monkey.shout()
    },
    visitLion(lion){
        lion.roar()
    },
    visitDolphin(dolphin){
        dolphin.speak()
    }
}
```

And then it can be used as
```js
const monkey = new Monkey()
const lion = new Lion()
const dolphin = new Dolphin()

monkey.accept(speak)    // Ooh oo aa aa!    
lion.accept(speak)      // Roaaar!
dolphin.accept(speak)   // Tuut tutt tuutt!
```
We could have done this simply by having a inheritance hierarchy for the animals but then we would have to modify the animals whenever we would have to add new actions to animals. But now we will not have to change them. For example, let's say we are asked to add the jump behavior to the animals, we can simply add that by creating a new visitor i.e.

```js
const jump = {
    visitMonkey(monkey) {
        console.log('Jumped 20 feet high! on to the tree!')
    },
    visitLion(lion) {
        console.log('Jumped 7 feet! Back on the ground!')
    },
    visitDolphin(dolphin) {
        console.log('Walked on water a little and disappeared')
    }
}
```
And for the usage
```js
monkey.accept(speak)   // Ooh oo aa aa!
monkey.accept(jump)    // Jumped 20 feet high! on to the tree!

lion.accept(speak)     // Roaaar!
lion.accept(jump)      // Jumped 7 feet! Back on the ground! 

dolphin.accept(speak)  // Tuut tutt tuutt! 
dolphin.accept(jump)   // Walked on water a little and disappeared
```

💡 Strategy
--------

Real world example
> Consider the example of sorting, we implemented bubble sort but the data started to grow and bubble sort started getting very slow. In order to tackle this we implemented Quick sort. But now although the quick sort algorithm was doing better for large datasets, it was very slow for smaller datasets. In order to handle this we implemented a strategy where for small datasets, bubble sort will be used and for larger, quick sort.

In plain words
> Strategy pattern allows you to switch the algorithm or strategy based upon the situation.

Wikipedia says
> In computer programming, the strategy pattern (also known as the policy pattern) is a behavioural software design pattern that enables an algorithm's behavior to be selected at runtime.

**Programmatic example**

Translating our example from above, we can easily implement this strategy in javascript using its feature of first class functions.

```js
const bubbleSort = dataset => {
    console.log('Sorting with bubble sort')
    // ...
    // ...
    return dataset
}

const quickSort = dataset => {
    console.log('Sorting with quick sort')
    // ...
    // ...
    return dataset
}
```

And then we have our client that is going to use any strategy
```js
const sorter = dataset => {
    if(dataset.length > 5){
        return quickSort
    } else {
        return bubbleSort
    }
}
```
And it can be used as
```js
const longDataSet = [1, 5, 4, 3, 2, 8]
const shortDataSet = [1, 5, 4]

const sorter1 = sorter(longDataSet)
const sorter2 = sorter(shortDataSet)

sorter1(longDataSet) // Output : Sorting with quick sort
sorter2(shortDataSet) // Output : Sorting with bubble sort
```

💢 State
-----
Real world example
> Imagine you are using some drawing application, you choose the paint brush to draw. Now the brush changes it's behavior based on the selected color i.e. if you have chosen red color it will draw in red, if blue then it will be in blue etc.  

In plain words
> It lets you change the behavior of a class when the state changes.

Wikipedia says
> The state pattern is a behavioral software design pattern that implements a state machine in an object-oriented way. With the state pattern, a state machine is implemented by implementing each individual state as a derived class of the state pattern interface, and implementing state transitions by invoking methods defined by the pattern's superclass.
> The state pattern can be interpreted as a strategy pattern which is able to switch the current strategy through invocations of methods defined in the pattern's interface

**Programmatic example**

Let's take an example of text editor, it let's you change the state of text that is typed i.e. if you have selected bold, it starts writing in bold, if italic then in italics etc.

First of all we have our transformation functions

```js
const upperCase = inputString => inputString.toUpperCase()
const lowerCase = inputString => inputString.toLowerCase()
const defaultTransform = inputString => inputString
```
Then we have our editor
```js
class TextEditor {
    constructor(transform) {
        this._transform = transform
    }
    
    setTransform(transform) {
        this._transform = transform
    }
    
    type(words) {
        console.log(this._transform(words))
    }
}
```
And then it can be used as
```js
const editor = new TextEditor(defaultTransform)

editor.type('First line')

editor.setTransform(upperCase)

editor.type('Second line')
editor.type('Third line')

editor.setTransform(lowerCase)

editor.type('Fourth line')
editor.type('Fifth line')

// Output:
// First line
// SECOND LINE
// THIRD LINE
// fourth line
// fifth line
```

📒 Template Method
---------------

Real world example
> Suppose we are getting some house built. The steps for building might look like 
> - Prepare the base of house
> - Build the walls
> - Add roof
> - Add other floors
> The order of these steps could never be changed i.e. you can't build the roof before building the walls etc but each of the steps could be modified for example walls can be made of wood or polyester or stone.

In plain words
> Template method defines the skeleton of how certain algorithm could be performed but defers the implementation of those steps to the children classes.

Wikipedia says
> In software engineering, the template method pattern is a behavioral design pattern that defines the program skeleton of an algorithm in an operation, deferring some steps to subclasses. It lets one redefine certain steps of an algorithm without changing the algorithm's structure.

**Programmatic Example**

Imagine we have a build tool that helps us test, lint, build, generate build reports (i.e. code coverage reports, linting report etc) and deploy our app on the test server.

First of all we have our base class that specifies the skeleton for the build algorithm
```js
class Builder {
    // Template method 
    build() {
        this.test()
        this.lint()
        this.assemble()
        this.deploy()
    }
}
```

Then we can have our implementations

```js
class AndroidBuilder extends Builder {
    test() {
        console.log('Running android tests')
    }
    
    lint() {
        console.log('Linting the android code')
    }
    
    assemble() {
        console.log('Assembling the android build')
    }
    
    deploy() {
        console.log('Deploying android build to server')
    }
}

class IosBuilder extends Builder {
    test() {
        console.log('Running ios tests')
    }
    
    lint() {
        console.log('Linting the ios code')
    }
    
    assemble() {
        console.log('Assembling the ios build')
    }
    
    deploy() {
        console.log('Deploying ios build to server')
    }
}
```
And then it can be used as

```js
const androidBuilder = new AndroidBuilder()
androidBuilder.build()

// Output:
// Running android tests
// Linting the android code
// Assembling the android build
// Deploying android build to server

const iosBuilder = new IosBuilder()
iosBuilder.build()

// Output:
// Running ios tests
// Linting the ios code
// Assembling the ios build
// Deploying ios build to server
```

## 🚦 Wrap Up Folks

And that about wraps it up. I will continue to improve this, so you might want to watch/star this repository to revisit. Also, I have plans on writing the same about the architectural patterns, stay tuned for it.

## 👬 Contribution

- Report issues
- Open pull request with improvements
- Spread the word

## License
MIT © [Soham Kamani](http://sohamkamani.com)
Based on ["Design patterns for humans"](https://github.com/kamranahmedse/design-patterns-for-humans) Copyright 2017 Kamran Ahmed