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

* [创建型](#创建型设计模式)
* [结构型](#结构型设计模式)
* [行为型](#行为型设计模式) 


创建型设计模式
==========================

简单地来说

> 创建型模式专注于如何实例化一个对象或者一组相关的对象。

维基百科上的解释

> 在软件工程中，创建型模式是处理对象创建的设计模式，试图根据实际情况使用合适的方式创建对象。基本的对象创建方式可能会导致设计上的问题，或增加设计的复杂度。创建型模式通过以某种方式控制对象的创建来解决问题。

 * [简单工厂模式](#-简单工厂模式)
 * [工厂方法模式](#-工厂方法模式)
 * [抽象工厂模式](#-抽象工厂模式)
 * [建造者模式](#-建造者模式)
 * [原型模式](#-原型模式)
 * [单例模式](#-单例模式)

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

**友情提示：** 当你发现一个函数或者方法中的参数太多（通常两个以上）时，用一个对象来替换这些参数往往是更好的选择。这样做有两个好处：

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

 * [适配器模式](#-适配器模式)
 * [桥模式](#-桥模式)
 * [组合模式](#-组合模式)
 * [装饰器模式](#-装饰器模式)
 * [外观模式](#-外观模式)
 * [享元模式](#-享元模式)
 * [代理模式](#-代理模式) 

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

* [职责链模式](#-职责链模式)
* [命令模式](#-命令模式)
* [迭代器模式](#-迭代器模式)
* [中介者模式](#-中介者模式)
* [备忘录模式](#-备忘录模式)
* [观察者模式](#-观察者模式)
* [访问者模式](#-访问者模式)
* [策略模式](#-策略模式)
* [状态模式](#-状态模式)
* [模版方法模式](#-模版方法模式) 

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

> 一个常见的例子是当你在饭店点餐时，你 ( `Client` ) 会让服务员 ( `Invoker` ) 给你一些食物 ( `Command` ) ，服务员则会将请求递给会做饭的厨师 ( `Receiver` ) 。
>
> 另一个例子是你 ( `Client` ) 可以通过遥控器 ( `Invoker` ) 切换 ( `Command` ) 电视 ( `Receiver` ) 的频道。

简单地来说

> 命令模式允许你将操作封装在对象中。该模式背后的关键思想是提供将客户端与接收者分离的方法。

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
然后我们有一个每个命令都需要实现的 Command 接口，这个接口中包含着很多命令。

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
下面是我们的调用者 ( `Invoker` ) 类，它会接收并执行命令。

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

命令模式还可以用来实现基于事务的系统。你可以保留所有执行命令的操作记录，如果最后的命令被成功执行则没有什么大的问题，否则就需要通过迭代并执行历史记录中命令的 `undo` 方法来撤销所有的操作。

➿ 迭代器模式
--------

现实生活中的例子

> 旧收音机将会是迭代器模式中一个很好的例子，用户可以从某个频道开始，通过按下向前或者向后键按钮来收听不同频道的节目。或者以 MP3 播放器或电视机为例，你可以按下向前或向后键按钮来查看后续的频道，换句话说，它们都提供了一个接口来遍历各个频道、歌曲或者电台。

简单地来说

> 它提供了一种可以访问对象元素但又不暴露底层表示的方法。

维基百科上的解释

> 在面向对象编程中，迭代器模式是一种设计模式，其中迭代器用于遍历容器并访问容器中的元素。 在某些情况下，迭代器模式将算法与容器分离，算法必然是特定于容器的，因此不能解耦。

**编程示例** 

让我们以电台频道为例，首先我们有一个 `RadioStation` 类。

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
然后我们有自己的迭代器类

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
最后它可以这样使用
```js
const stationList = new StationList()

stationList.addStation(new RadioStation(89))
stationList.addStation(new RadioStation(101))
stationList.addStation(new RadioStation(102))
stationList.addStation(new RadioStation(103.2))

stationList.stations.forEach(station => console.log(station.getFrequency()))

stationList.removeStation(new RadioStation(89)) // Will remove station 89
```

👽 中介者模式
-------

现实生活中的例子

> 一个常见的例子是当你在手机上与某人通话时，你和他人的对话信息是通过网络提供商的设备进行传输而不是直接发送。在这个例子中网络提供商就是一个中介者。

简单地来说

> 中介者模式增加了一个第三方对象（称为中介者）来控制两个对象（称为同事）间的交互。该模式有助于减少两个类间通信的耦合程度。因为现在它们并不需要知道各自内部的实现。

维基百科上的解释

> 在软件工程中，中介者模式定义了一个对象，该对象封装了一组对象的交互方式。 由于它可以改变程序的运行时行为，因此这种模式被认为是一种行为型设计模式。

**编程示例** 

下面是一个最简版的聊天室（中介者），用户（同事）能够通过聊天室向其他用户发送消息。

首先，我们来实现聊天室这个中介者类。

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

下面是我们的用户（同事）类

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
最后它可以这样使用
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

💾 备忘录模式
-------
现实生活中的例子

> 让我们以计算器为例，当你在执行一些计算时，最后一次计算的结果总会保存在内存中，这样做是为了方便你回来访问它，当然你也可以通过某些动作按钮来恢复该计算结果。

简单地来说

> 备忘录模式是一种能够捕获并存储当前对象状态的方法，在之后我们能以平稳的方式恢复该对象状态。

维基百科上的解释

> 备忘录模式是一种软件设计模式，它提供了一种将对象恢复到其先前状态的能力（以回滚的方式撤销）。

通常在需要提供某种撤销功能时该模式非常有用。

**编程示例**

让我们以文字编辑器为例，它能够实时地保存文字状态以便恢复到先前的某个时刻。

首先我们有一个用来保存编辑器状态的备忘录对象。

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

然后我们有一个使用备忘录对象的编辑器

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

最后它可以这样使用

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

😎 观察者模式
--------

（又被称作是“发布-订阅”模式）

现实生活中的例子

> 一个合适的例子是，求职者订阅了某个招聘网站，只要有匹配的工作机会，他们就会得到通知。

简单地来说

> 观察者模式定义了对象之间的依赖关系，当一个对象改变其自身状态时，所有依赖它的对象都会得到通知。

维基百科上的解释

> 观察者模式是一种软件设计模式，其中一个称为主体的对象维护其依赖者列表（称为观察者），通过调用主体上的某种方法，任何状态的改变都能够自动通知到所有相关的观察者。

**编程示例**

让我们以求职者找工作为例，首先我们需要创建一个能够收到工作通知的求职者类。

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
然后我们再创建一个求职者能够订阅求职信息的招聘网站
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
最后它可以这样使用
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

🏃 访问者模式
-------
现实生活中的例子

> 假设某人去迪拜旅游，他需要进入迪拜的签证。在到达迪拜后，他可以独自参观迪拜的任何地方而不需要经过许可或者白费一些跑腿的工作，但前提是让他知道这个地方的位置。访问者模式就能够做到这点，它可以帮助你添加要访问的地点，这样你就能访问尽可能多的地方而不需要做多余的工作。

简单地来说

> 访问者模式能够在不修改对象的前提下为对象添加更多的操作。

维基百科上的解释

> 在面向对象编程和软件工程中，访问者设计模式是一种将算法与其运行的对象结构分离的方法。 这种分离的实际结果是能够在不修改这些结构的情况下向现有对象结构添加新操作。 这是遵循开放 / 封闭原则的一种方式。

**编程示例**

让我们以动物园为例，在这里有很多种能够发出不同声音的动物。

首先我们来实现动物类：

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
然后我们来实现访问者类
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

最后它可以这样使用
```js
const monkey = new Monkey()
const lion = new Lion()
const dolphin = new Dolphin()

monkey.accept(speak)    // Ooh oo aa aa!    
lion.accept(speak)      // Roaaar!
dolphin.accept(speak)   // Tuut tutt tuutt!
```
我们本可以直接通过动物类来达到相同的效果，但是如果我们想要向动物类中添加新的动作，就不得不修改动物类内部的方法。通过访问者模式，我们就不必改变动物类。例如，我们想让动物能有跳跃的行为，只需要新增一个访问者类即可。

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
它可以这样使用
```js
monkey.accept(speak)   // Ooh oo aa aa!
monkey.accept(jump)    // Jumped 20 feet high! on to the tree!

lion.accept(speak)     // Roaaar!
lion.accept(jump)      // Jumped 7 feet! Back on the ground! 

dolphin.accept(speak)  // Tuut tutt tuutt! 
dolphin.accept(jump)   // Walked on water a little and disappeared
```

💡 策略模式
--------

现实生活中的例子

> 让我们以排序算法为例，冒泡排序算法在数据集较大的情况下会变得低效，因此我们会采取快速排序算法。虽然在数据集较大的情况下快速排序算法表现更好，但它在很小的数据集下却表现不佳。因此我们使用策略模式，在数据集较大时采用快速排序算法，数据集较小时采用冒泡排序算法。

简单地来说

> 策略模式能够让你基于具体场景切换不同的算法或者策略。

维基百科上的例子

> 在计算机编程中，策略模式是一种行为型设计模式，它可以在运行时选择算法的行为。

**编程示例**

让我们以上面的排序算法为例，通过 JavaScript 中函数是第一等公民的特性，我们可以轻松地实现策略模式。

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

在这里使用不同的策略
```js
const sorter = dataset => {
    if(dataset.length > 5){
        return quickSort
    } else {
        return bubbleSort
    }
}
```
最后它可以这样使用
```js
const longDataSet = [1, 5, 4, 3, 2, 8]
const shortDataSet = [1, 5, 4]

const sorter1 = sorter(longDataSet)
const sorter2 = sorter(shortDataSet)

sorter1(longDataSet) // Output : Sorting with quick sort
sorter2(shortDataSet) // Output : Sorting with bubble sort
```

💢 状态模式
-----
现实生活中的例子

> 假设你正在使用某个画图应用，你选择了画笔来作图。你可以选择不同的颜色来更改画笔原始的颜色。如果你选择了红色，你就能以红色的笔作图，如果你选择了蓝色，你就能以蓝色的笔作图。

简单地来说

> 状态模式能够在状态发生改变时改变类的行为。

维基百科上的解释

> 状态模式是一种行为型设计模式，它以面向对象的方式实现状态机。 使用状态模式，通过将每个单独的状态实现为状态模式接口的派生类来实现状态机，并通过调用由模式的超类定义的方法来实现状态转换。 状态模式可以解释为一种策略模式，它能够通过调用模式接口中定义的方法来切换当前策略。

**编程示例**

让我们以文本编辑器为例，它能够改变文本的显示状态。如果你选择了粗号字体，文本就会变粗，如果你选择了斜体，文本就会显示为斜体。

首先我们实现了文字转换的函数

```js
const upperCase = inputString => inputString.toUpperCase()
const lowerCase = inputString => inputString.toLowerCase()
const defaultTransform = inputString => inputString
```
然后是我们的文本编辑器类
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
最后它可以这样使用
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

📒 模版方法模式
---------------

现实生活中的例子

> 假设我们正在建造一间房子。建造的步骤可能会像下面这样
>
> * 准备房子的基础设施
> * 建造房子的墙
> * 添加房顶
> * 添加地板
>
> 这些步骤的顺序基本不会改变，因为你不可能在建造房子的墙之前去添加地板，但是每个步骤的具体内容却可以发生改变，比如墙的材质可以由木头、聚酯或者石头制成。

简单地来说

> 模版方法模式定义了如何执行某些算法的框架，但是将这些步骤的实现推迟到了子类。

维基百科上的解释

> 在软件工程中，模板方法模式是一种行为型设计模式，它定义了操作中算法的程序框架，将一些步骤推迟到子类。 它允许重新定义算法的某些步骤而不改变算法的结构。

**编程示例** 

假设我们正在编写一个帮助我们测试、规范、构建、生成构建报告（代码覆盖率, 代码规范）的工具，并且最后它还能将应用部署到测试服务器上。

首先我们来实现一个具有基本框架的构建算法类

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

然后我们在子类中实现具体的步骤

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
最后它可以这样使用

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

