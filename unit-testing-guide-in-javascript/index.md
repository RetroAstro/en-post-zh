> 原文地址：https://github.com/mawrkus/js-unit-testing-guide
>
> 原文作者：https://github.com/mawrkus

## 📖 目录

1. **通则**
  + [单元测试](#unit-tests)
  + [设计原则](#design-principles)
2. **指南** 
  + [尽可能地使用 TDD](#尽可能地使用-TDD)
  + [正确地组织测试](#正确地组织测试)
  + [正确地为测试命名](#正确地为测试命名) 
  + [不要注释测试](#不要注释测试) 
  + [在测试中要避免逻辑](#在测试中要避免逻辑) 
  + [不要写不必要的断言](#不要写不必要的断言) 
  + [正确地初始化应用于所有相关测试的操作](#正确地初始化应用于所有相关测试的操作) 
  + [考虑在测试中使用工厂函数](#考虑在测试中使用工厂函数) 
  + [熟悉你的测试框架 API](#熟悉你的测试框架-API) 
  + [不要在同一测试中测试多个关注点](#不要在同一测试中测试多个关注点) 
  + [要覆盖一般情况与边缘情况](#要覆盖一般情况与边缘情况) 
  + [在应用 TDD 时, 总是从编写最简单的失败测试开始](#在应用-TDD-时-总是从编写最简单的失败测试开始) 
  + [在应用 TDD 时, 总是在每个测试优先周期中小步前进](#在应用-TDD-时-总是在每个测试优先周期中小步前进) 
  + [测试行为, 而不是内部实现](#测试行为-而不是内部实现) 
  + [不要 mock 所有数据](#不要-mock-所有数据) 
  + [为每个 bug 创建新的测试](#为每个-bug-创建新的测试) 
  + [不要为复杂的用户交互编写单元测试](#不要为复杂的用户交互编写单元测试) 
  + [测试简单的用户操作](#测试简单的用户操作) 
  + [首先审查测试代码](#首先审查测试代码) 
  + [在编码中实践, 通过结对编程学习](#在编码中实践-通过结对编程学习) 
3. **[参考资料](#-参考资料)**  

## 通则

### 单元测试

**单元 = 工作单元** 

它可以是那些在公共 API 中调用的**多个方法和类**且能够：

+ 返回值或者抛出异常
+ 改变系统的状态
+ 进行第三方调用（API、数据库等）

单元测试应该测试一个工作单元的行为：给定一个输入，最终的期望结果可以是上面的任何一种。

**单元测试之间应该是相互隔离且彼此独立的** 

+ 每个给定的行为都应该对应一个**单独的测试** 
+ 一个测试的执行与执行顺序**不应该影响其他测试** 

**单元测试属于轻量测试**

+ 可重复
+ 快速
+ 一致性
+ 方便读写

**单元测试也是代码** 

它们应该与正在测试的代码具有相同的质量级别。它们也能通过重构来增强代码自身的可维护性与可读性。

• [返回目录](#-目录) •

### 设计原则

良好单元测试的关键在于编写**可测试的代码**。下面是一些有用的设计原则：

+ 使用良好的命名规范并注释代码（“为什么”而不是“怎么做”），要记住注释并不能代替糟糕的命名和设计
+ 不要重复自己，避免重复代码
+ **单一职责**：每个对象 / 函数只应该关注一个任务
+ 在同一组件中保持**单一抽象级别**（例如，不要将业务逻辑与同一方法中的低层实现细节混合在一起）
+ **最小化组件间依赖**：通过封装组件，减少组件之间的信息交换
+ **支持可配置性而不是硬编码**，这避免了在测试时复制完全相同的环境
+ 适当地引用**设计模式**，尤其是**依赖注入**，因为它能将对象的创建职责与业务逻辑分离
+ 避免全局状态

• [返回目录](#-目录) •

## 指南

这些指南的目的在于让你的测试具有：

+ **可读性**
+ **可维护性**
+ **可靠性** 

这是良好单元测试的三大支柱

下文所有的测试用例都以 [Jasmine](http://jasmine.github.io) 框架为基础。

• [返回目录](#-目录) •

---------------------------------------

### 尽可能地使用 TDD

TDD ( 测试驱动开发 ) 是一个设计过程，而不是测试过程。TDD 是一种以交互式设计软件组件（“单元”）的强大方式，在单元测试中我们就能够指定相关的行为。

#### 测试优先周期

1. 编写一个简单的失败测试
2. 用最少的代码让测试通过，且不要去担心代码质量
3. 使用设计原则 / 模式重构代码

#### 测试优先周期带来的好处

+ 首先编写测试能够让代码设计本身变得易于测试
+ 只需编写要实现功能的所需代码量，就可以让生成的代码库最小化，从而变得更具有可维护性
+ 代码库能够通过重构机制进行增强，因为测试能够保证新的代码并不会改变当前已有的功能
+ 在每个周期中清理代码能够让代码库更易于维护，频繁且少量地修改代码往往更不容易出错
+ 对开发者来说是一种快速反馈，你知道你没有破坏任何东西，并且在让系统往好的方向发展
+ 对于添加新特性、修复 bug 或者探索新设计更加地有自信

注意：那些没有使用 TDD 方式编写的代码往往都难以测试。

• [返回目录](#-目录) •

### 正确地组织测试

通过嵌套测试套件让子集中的测试更有逻辑性。

**:(**

```js
describe('A set of functionalities', () => {
  it('a set of functionalities should do something nice', () => {
  });

  it('a subset of functionalities should do something great', () => {
  });

  it('a subset of functionalities should do something awesome', () => {
  });

  it('another subset of functionalities should also do something great', () => {
  });
});
```

**:)**

```js
describe('A set of functionalities', () => {
  it('should do something nice', () => {
  });

  describe('A subset of functionalities', () => {
    it('should do something great', () => {
    });

    it('should do something awesome', () => {
    });
  });

  describe('Another subset of functionalities', () => {
    it('should also do something great', () => {
    });
  });
});
```

• [返回目录](#-目录) •

### 正确地为测试命名

测试名称应简洁、明确、描述性强且使用正确的英语。通过查看 spec runner 的输出来验证该测试名称是可以理解的。要记住测试代码也会被他人阅读，它也可以是代码的实时文档。

**:(**

```js
describe('MyGallery', () => {
  it('init set correct property when called (thumb size, thumbs count)', () => {
  });

  // ...
});
```

**:)**

```js
describe('The Gallery instance', () => {
  it('should properly calculate the thumb size when initialized', () => {
  });

  it('should properly calculate the thumbs count when initialized', () => {
  });

  // ...
});
```

为了让测试名称更加合理，你可以使用 **“工作单元 - 场景 / 上下文 - 期望行为”** 模式来命名：

```js
describe('[unit of work]', () => {
  it('should [expected behaviour] when [scenario/context]', () => {
  });
});
```

或者当许多测试都在同一场景 / 相关上下文时：

```js
describe('[unit of work]', () => {
  describe('when [scenario/context]', () => {
    it('should [expected behaviour]', () => {
    });
  });
});
```

例如：

**:) :)**

```js
describe('The Gallery instance', () => {
  describe('when initialized', () => {
    it('should properly calculate the thumb size', () => {
    });

    it('should properly calculate the thumbs count', () => {
    });
  });

  // ...
});
```

• [返回目录](#-目录) •

### 不要注释测试

测试总有它存在或者不存在的理由。

不要因为测试太慢、太复杂或者会失败就把它们注释掉，相反我们应该让其变得快速、简单且值得信赖。如果实在不行，就将它们完全移除。

• [返回目录](#-目录) •

### 在测试中要避免逻辑

总是使用简单语句。不要使用循环或者条件语句。如果这样做了，在测试代码中就可能产生 bug ：

+ 条件语句：你不知道在测试时会执行哪条语句
+ 循环语句：你可能在多个测试之间共享状态

**:(**

```js
it('should properly sanitize strings', () => {
  let result;
  const testValues = {
    'Avion'         : 'Avi' + String.fromCharCode(243) + 'n',
    'The-space'     : 'The space',
    'Weird-chars-'  : 'Weird chars!!',
    'file-name.zip' : 'file name.zip',
    'my-name.zip'   : 'my.name.zip'
  };

  for (result in testValues) {
    expect(sanitizeString(testValues[result])).toBe(result);
  }
});
```

**:)**

```js
it('should properly sanitize strings', () => {
  expect(sanitizeString('Avi'+String.fromCharCode(243)+'n')).toBe('Avion');
  expect(sanitizeString('The space')).toBe('The-space');
  expect(sanitizeString('Weird chars!!')).toBe('Weird-chars-');
  expect(sanitizeString('file name.zip')).toBe('file-name.zip');
  expect(sanitizeString('my.name.zip')).toBe('my-name.zip');
});
```

更好的方式：为每种类型编写单独的测试。这样就可以输出所有可能的情况，增强代码的可维护性。

**:) :)**

```js
it('should sanitize a string containing non-ASCII chars', () => {
  expect(sanitizeString('Avi'+String.fromCharCode(243)+'n')).toBe('Avion');
});

it('should sanitize a string containing spaces', () => {
  expect(sanitizeString('The space')).toBe('The-space');
});

it('should sanitize a string containing exclamation signs', () => {
  expect(sanitizeString('Weird chars!!')).toBe('Weird-chars-');
});

it('should sanitize a filename containing spaces', () => {
  expect(sanitizeString('file name.zip')).toBe('file-name.zip');
});

it('should sanitize a filename containing more than one dot', () => {
  expect(sanitizeString('my.name.zip')).toBe('my-name.zip');
});
```

• [返回目录](#-目录) •

### 不要写不必要的断言

请记住，单元测试是特定行为应该如何工作的设计规范，而不是对代码执行的所有操作的观察列表。

**:(**

```js
it('should multiply the number passed as parameter and subtract one', () => {
  const multiplySpy = spyOn(Calculator, 'multiple').and.callThrough();
  const subtractSpy = spyOn(Calculator, 'subtract').and.callThrough();

  const result = Calculator.compute(21.5);

  expect(multiplySpy).toHaveBeenCalledWith(21.5, 2);
  expect(subtractSpy).toHaveBeenCalledWith(43, 1);
  expect(result).toBe(42);
});
```

**:)**

```js
it('should multiply the number passed as parameter and subtract one', () => {
  const result = Calculator.compute(21.5);
  expect(result).toBe(42);
});
```

这样做能够提高代码的可维护性，因为你的测试不再与代码的实现细节有关。

• [返回目录](#-目录) •

### 正确地初始化应用于所有相关测试的操作

**:(**

```js
describe('Saving the user profile', () => {
  let profileModule;
  let notifyUserSpy;
  let onCompleteSpy;

  beforeEach(() => {
    profileModule = new ProfileModule();
    notifyUserSpy = spyOn(profileModule, 'notifyUser');
    onCompleteSpy = jasmine.createSpy();
  });

  it('should send the updated profile data to the server', () => {
    jasmine.Ajax.install();

    profileModule.save();

    const request = jasmine.Ajax.requests.mostRecent();

    expect(request.url).toBe('/profiles/1');
    expect(request.method).toBe('POST');
    expect(request.data()).toEqual({ username: 'mawrkus' });

    jasmine.Ajax.uninstall();
  });

  it('should notify the user', () => {
    jasmine.Ajax.install();

    profileModule.save();

    expect(notifyUserSpy).toHaveBeenCalled();

    jasmine.Ajax.uninstall();
  });

  it('should properly execute the callback passed as parameter', () => {
    jasmine.Ajax.install();

    profileModule.save(onCompleteSpy);

    jasmine.Ajax.uninstall();

    expect(onCompleteSpy).toHaveBeenCalled();
  });
});
```

初始化代码应该正确地应用于所有相关的测试：

**:)**

```js
describe('Saving the user profile', () => {
  let profileModule;

  beforeEach(() => {
    jasmine.Ajax.install();
    profileModule = new ProfileModule();
  });

  afterEach( () => {
    jasmine.Ajax.uninstall();
  });

  it('should send the updated profile data to the server', () => {
    profileModule.save();

    const request = jasmine.Ajax.requests.mostRecent();

    expect(request.url).toBe('/profiles/1');
    expect(request.method).toBe('POST');

  });

  it('should notify the user', () => {
    spyOn(profileModule, 'notifyUser');

    profileModule.save();

    expect(profileModule.notifyUser).toHaveBeenCalled();
  });

  it('should properly execute the callback passed as parameter', () => {
    const onCompleteSpy = jasmine.createSpy();

    profileModule.save(onCompleteSpy);

    expect(onCompleteSpy).toHaveBeenCalled();
  });
});
```

考虑将初始化代码限制在最小的相关上下文中，以保持代码的可读性与可维护性。

• [返回目录](#-目录) •

### 考虑在测试中使用工厂函数

工厂函数能够：

+ 帮助你减少初始化代码，特别是当你使用依赖注入时。
+ 让每个测试更具可读性，因为创建仅仅是一个函数调用，所以可以将其应用在测试本身而不是测试夹具中。
+ 在创建新的实例是更具灵活性（例如，设置初始状态）

我们需要在 DRY 原则与可读性之间寻找平衡点。

**:(**

```js
describe('User profile module', () => {
  let profileModule;
  let pubSub;

  beforeEach(() => {
    const element = document.getElementById('my-profile');
    pubSub = new PubSub({ sync: true });

    profileModule = new ProfileModule({
      element,
      pubSub,
      likes: 0
    });
  });

  it('should publish a topic when a new "like" is given', () => {
    spyOn(pubSub, 'notify');
    profileModule.incLikes();
    expect(pubSub.notify).toHaveBeenCalledWith('likes:inc', { count: 1 });
  });

  it('should retrieve the correct number of likes', () => {
    profileModule.incLikes();
    profileModule.incLikes();
    expect(profileModule.getLikes()).toBe(2);
  });
});
```

**:)**

```js
describe('User profile module', () => {
  function createProfileModule({
    element = document.getElementById('my-profile'),
    likes = 0,
    pubSub = new PubSub({ sync: true })
  }) {
    return new ProfileModule({ element, likes, pubSub });
  }

  it('should publish a topic when a new "like" is given', () => {
    const pubSub = jasmine.createSpyObj('pubSub', ['notify']);
    const profileModule = createProfileModule({ pubSub });

    profileModule.incLikes();

    expect(pubSub.notify).toHaveBeenCalledWith('likes:inc');
  });

  it('should retrieve the correct number of likes', () => {
    const profileModule = createProfileModule({ likes: 40 });

    profileModule.incLikes();
    profileModule.incLikes();

    expect(profileModule.getLikes()).toBe(42);
  });
});
```

在处理与 DOM 相关的事务时，工厂函数尤其有用：

**:(**

```js
describe('The search component', () => {
  describe('when the search button is clicked', () => {
    let container;
    let form;
    let searchInput;
    let submitInput;

    beforeEach(() => {
      fixtures.inject(`<div id="container">
        <form class="js-form" action="/search">
          <input type="search">
          <input type="submit" value="Search">
        </form>
      </div>`);

      container = document.getElementById('container');
      form = container.getElementsByClassName('js-form')[0];
      searchInput = form.querySelector('input[type=search]');
      submitInput = form.querySelector('input[type=submith]');
    });

    it('should validate the text entered', () => {
      const search = new Search({ container });
      spyOn(search, 'validate');

      search.init();

      input(searchInput, 'peace');
      click(submitInput);

      expect(search.validate).toHaveBeenCalledWith('peace');
    });

    // ...
  });
});
```

**:)**

```js
function createHTMLFixture() {
  fixtures.inject(`<div id="container">
    <form class="js-form" action="/search">
      <input type="search">
      <input type="submit" value="Search">
    </form>
  </div>`);

  const container = document.getElementById('container');
  const form = container.getElementsByClassName('js-form')[0];
  const searchInput = form.querySelector('input[type=search]');
  const submitInput = form.querySelector('input[type=submith]');

  return {
    container,
    form,
    searchInput,
    submitInput
  };
}

describe('The search component', () => {
  describe('when the search button is clicked', () => {
    it('should validate the text entered', () => {
      const { container, form, searchInput, submitInput } = createHTMLFixture();
      const search = new Search({ container });
      spyOn(search, 'validate');

      search.init();

      input(searchInput, 'peace');
      click(submitInput);

      expect(search.validate).toHaveBeenCalledWith('peace');
    });

    // ...
  });
});
```

• [返回目录](#-目录) •

### 熟悉你的测试框架 API

你应该十分熟悉测试框架 / 库中的 API 文档。

熟悉 API 能够减少测试代码的大小 / 复杂度，并且能在开发过程中为你提供帮助。一个简单的例子：

**:(**

```js
it('should call a method with the proper arguments', () => {
  const foo = {
    bar: jasmine.createSpy(),
    baz: jasmine.createSpy()
  };

  foo.bar('qux');

  expect(foo.bar).toHaveBeenCalled();
  expect(foo.bar.calls.argsFor(0)).toEqual(['qux']);
});

/*it('should do more but not now', () => {
});

it('should do much more but not now', () => {
});*/
```

**:)**

```js
fit('should call once a method with the proper arguments', () => {
  const foo = jasmine.createSpyObj('foo', ['bar', 'baz']);

  foo.bar('baz');

  expect(foo.bar).toHaveBeenCalledWith('baz');
});

it('should do something else but not now', () => {
});

it('should do something else but not now', () => {
});
```

#### Note

上面示例中使用的 **`fit`** 函数允许你只执行一个测试，而不必注释掉下面所有的测试。这可以帮助你在开发时节省大量的时间。

想要了解更多请参考 [Jasmine](http://jasmine.github.io) 的官方网站。

• [返回目录](#-目录) •

### 不要在同一测试中测试多个关注点

如果一个方法中有多个期望结果，那么应该分别测试每个结果。这样当 bug 出现时，便能更快地定位到问题的源头。

**:(**

```js
it('should send the profile data to the server and update the profile view properly', () => {
  // expect(...)to(...);
  // expect(...)to(...);
});
```

**:)**

```js
it('should send the profile data to the server', () => {
  // expect(...)to(...);
});

it('should update the profile view properly', () => {
  // expect(...)to(...);
});
```

当你的测试命名中含有 “and” 或 “or” 时，就意味着产生了代码的坏味道。

• [返回目录](#-目录) •

### 要覆盖一般情况与边缘情况

“奇怪的行为”总是在边界情况下发生 ...... 要记住你的测试应该作为代码的实时文档。

**:(**

```js
it('should properly calculate a RPN expression', () => {
  const result = RPN('5 1 2 + 4 * - 10 /');
  expect(result).toBe(-0.7);
});
```

**:)**

```js
describe('The RPN expression evaluator', () => {
  it('should return null when the expression is an empty string', () => {
    const result = RPN('');
    expect(result).toBeNull();
  });

  it('should return the same value when the expression holds a single value', () => {
    const result = RPN('42');
    expect(result).toBe(42);
  });

  it('should properly calculate an expression', () => {
    const result = RPN('5 1 2 + 4 * - 10 /');
    expect(result).toBe(-0.7);
  });

  it('should throw an error whenever an invalid expression is passed', () => {
    const compute = () => RPN('1 + - 1');
    expect(compute).toThrow();
  });
});
```

• [返回目录](#-目录) •

### 在应用 TDD 时, 总是从编写最简单的失败测试开始

**:(**

```js
it('should suppress all chars that appear multiple times', () => {
  expect(keepUniqueChars('Hello Fostonic !!')).toBe('HeFstnic');
});
```

**:)**

```js
it('should return an empty string when passed an empty string', () => {
  expect(keepUniqueChars('')).toBe('');
});
```

从这里开始，逐步构建功能。

• [返回目录](#-目录) •

### 在应用 TDD 时, 总是在每个测试优先周期中小步前进

构建你的测试套件，从简单到复杂。请记住增量设计，快速、增量、短迭代地交付软件。

**:(**

```js
it('should return null when the expression is an empty string', () => {
  const result = RPN('');
  expect(result).toBeNull();
});

it('should properly calculate a RPN expression', () => {
  const result = RPN('5 1 2 + 4 * - 10 /');
  expect(result).toBe(-0.7);
});
```

**:)**

```js
describe('The RPN expression evaluator', () => {
  it('should return null when the expression is an empty string', () => {
    const result = RPN('');
    expect(result).toBeNull();
  });

  it('should return the same value when the expression holds a single value', () => {
    const result = RPN('42');
    expect(result).toBe(42);
  });

  describe('Additions-only expressions', () => {
    it('should properly calculate a simple addition', () => {
      const result = RPN('41 1 +');
      expect(result).toBe(42);
    });

    it('should properly calculate a complex addition', () => {
      const result = RPN('2 9 + 15 3 + + 7 6 + +');
      expect(result).toBe(42);
    });
  });

  // ...

  describe('Complex expressions', () => {
    it('should properly calculate an expression containing all 4 operators', () => {
      const result = RPN('5 1 2 + 4 * - 10 /');
      expect(result).toBe(-0.7);
    });
  });
});
```

• [返回目录](#-目录) •

### 测试行为, 而不是内部实现

**:(**

```js
it('should add a user in memory', () => {
  userManager.addUser('Dr. Falker', 'Joshua');

  expect(userManager._users[0].name).toBe('Dr. Falker');
  expect(userManager._users[0].password).toBe('Joshua');
});
```

更好的方式是在相同级别的 API 上进行测试：

**:)**

```js
it('should add a user in memory', () => {
  userManager.addUser('Dr. Falker', 'Joshua');

  expect(userManager.loginUser('Dr. Falker', 'Joshua')).toBe(true);
});
```

优点：

+ 改变内部类 / 对象的实现不一定会强制你去重构相关的测试代码

缺点：

+ 如果某个测试失败，我们就得通过调试来定位哪段代码需要被修复

在这里我们需要找到一个平衡点，但单元测试中的关键部分是有益的。

• [返回目录](#-目录) •

### 不要 mock 所有数据

**:(**

```js
describe('when the user has already visited the page', () => {
  // storage.getItem('page-visited', '1') === '1'
  describe('when the survey is not disabled', () => {
    // storage.getItem('survey-disabled') === null
    it('should display the survey', () => {
      const storage = jasmine.createSpyObj('storage', ['setItem', 'getItem']);
      storage.getItem.and.returnValue('1'); // ouch.

      const surveyManager = new SurveyManager(storage);
      spyOn(surveyManager, 'display');

      surveyManager.start();

      expect(surveyManager.display).toHaveBeenCalled();
    });
  });

  // ...
});
```

上面的测试其实是失败的，因为 survey 一定是 disabled 。让我们来修复这个问题：

**:)**

```js
describe('when the user has already visited the page', () => {
  // storage.getItem('page-visited', '1') === '1'
  describe('when the survey is not disabled', () => {
    // storage.getItem('survey-disabled') === null
    it('should display the survey', () => {
      const storage = jasmine.createSpyObj('storage', ['setItem', 'getItem']);
      storage.getItem.and.callFake(key => {
        switch (key) {
          case 'page-visited':
            return '1';

          case 'survey-disabled':
            return null;
        }

        return null;
      }); // ouch.

      const surveyManager = new SurveyManager(storage);
      spyOn(surveyManager, 'display');

      surveyManager.start();

      expect(surveyManager.display).toHaveBeenCalled();
    });
  });

  // ...
});
```

现在这个测试能够正常工作 ...... 但这需要不少的代码。让我们来尝试更简单的方法：

**:(**

```js
describe('when the user has already visited the page', () => {
  // storage.getItem('page-visited', '1') === '1'
  describe('when the survey is not disabled', () => {
    // storage.getItem('survey-disabled') === null
    it('should display the survey', () => {
      const storage = window.localStorage; // ouch.
      storage.setItem('page-visited', '1');

      const surveyManager = new SurveyManager();
      spyOn(surveyManager, 'display');

      surveyManager.start();

      expect(surveyManager.display).toHaveBeenCalled();
    });
  });

  // ...
});
```

我们创建了一个永久性的数据存储。如果我们没有正确地将其清除，可能就会影响到其他的测试。让我们来修复这个问题：

**:) :)**

```js
describe('when the user has already visited the page', () => {
  // storage.getItem('page-visited', '1') === '1'
  describe('when the survey is not disabled', () => {
    // storage.getItem('survey-disabled') === null
    it('should display the survey', () => {
      const storage = new MemoryStorage(); // see https://github.com/tatsuyaoiw/webstorage
      storage.setItem('page-visited', '1');

      const surveyManager = new SurveyManager(storage);
      spyOn(surveyManager, 'display');

      surveyManager.start();

      expect(surveyManager.display).toHaveBeenCalled();
    });
  });
});
```

**`MemoryStorage`** 并不会持久化数据，因此也不会产生副作用。

#### 注意

需要记住的是依赖关系仍可以是“真实”的对象。不要 mock 所有的数据。特别地，在下面的场景中我们应该使用“真实”的对象：

+ 在测试中能够简单、轻松地初始化
+ 不会在多个测试之间创建共享状态，从而导致意外的副作用
+ 被测试的代码不会执行 AJAX 请求、第三方 API 调用或者浏览器页面刷新操作
+ 测试的执行速度应该保持在你的修复范围内

• [返回目录](#-目录) •

### 为每个 bug 创建新的测试

每当发现一个 bug ，我们就应该在**修改任何代码之前**创建新的测试来重现问题，然后再采用 TDD 修复问题。

• [返回目录](#-目录) •

### 不要为复杂的用户交互编写单元测试

复杂用户操作示例：

+ 填写表单，拖拽某个元素然后再提交表单
+ 点击选项卡，点击图像缩略图然后再浏览之前从数据库中加载的图像库
+ ( ...... )

这些交互可能会涉及大量的工作单元，在这种情况下我们应该采用更高级的**功能测试**。它们可能会花费更多的时间来完成。它们可能是片状的，并且在报告失败时需要进行调试。

对于功能测试，考虑使用自动化测试框架 ( [Selenium](https://www.seleniumhq.org/) ) 或者进行手动的 QA 测试。

• [返回目录](#-目录) •

### 测试简单的用户操作

简单用户操作示例：

+ 通过点击链接来切换 DOM 元素的可见性
+ 提交表单进而触发相关的表单验证
+ ( ...... ) 

我们可以通过**模拟 DOM 事件**轻松地测试这些操作，例如：

```js
describe('clicking on the "Preview profile" link', () => {
  it('should show the profile preview if it is hidden', () => {
    const previewLink = document.createElement('a');
    const profileModule = createProfileModule({ previewLink, previewIsVisible: false });

    spyOn(profileModule, 'showPreview');

    click(previewLink);

    expect(profileModule.showPreview).toHaveBeenCalled();
  });

  it('should hide the profile preview if it is displayed', () => {
    const previewLink = document.createElement('a');
    const profileModule = createProfileModule({ previewLink, previewIsVisible: true });

    spyOn(profileModule, 'hidePreview');

    click(previewLink);

    expect(profileModule.hidePreview).toHaveBeenCalled();
  });
});
```

可以看到上面的测试是多么的简单，这是因为 UI ( DOM ) 层并没有与逻辑层耦合：

+ “点击”事件被触发
+ 公共方法被调用

下一步就应该测试 “showPreview()” 或 “hidePreview()” 方法中实现的业务逻辑。

• [返回目录](#-目录) •

### 首先审查测试代码

总是以审查测试代码为先。测试是能够让你深入研究代码的迷你示例。

它可以帮助你快速了解开发者的意图（通过查看测试名称就可以实现）。

• [返回目录](#-目录) •

### 在编码中实践, 通过结对编程学习

经验就是老师。毕竟，实践出真知。反复地应用理论，并通过反馈得到更好的结果。

• [返回目录](#-目录) •

## 📙 参考资料

### 最佳实践

+ Roy Osherove - "JS Unit Testing Good Practices and Horrible Mistakes": https://www.youtube.com/watch?v=iP0Vl-vU3XM
+ Steven Sanderson - "Writing Great Unit Tests: Best and Worst Practices": http://blog.stevensanderson.com/2009/08/24/writing-great-unit-tests-best-and-worst-practises/
+ Rebecca Murphy - "Writing Testable JavaScript": http://alistapart.com/article/writing-testable-javascript
+ YUI Team - "Writing Effective JavaScript Unit Tests with YUI Test": http://yuiblog.com/blog/2009/01/05/effective-tests/
+ Colin Snover - "Testable code best practices": http://www.sitepen.com/blog/2014/07/11/testable-code-best-practices/
+ Miško Hevery - "The Clean Code Talks -- Unit Testing": https://www.youtube.com/watch?v=wEhu57pih5w
+ José Armesto - "Unit Testing sucks (and it’s our fault)": https://www.youtube.com/watch?v=GZ9iZsMAZFQ
+ TDD - From the Inside Out or the Outside In?: https://8thlight.com/blog/georgina-mcfadyen/2016/06/27/inside-out-tdd-vs-outside-in.html

### 整洁代码

+ Clean code cheat sheet: http://www.planetgeek.ch/2014/11/18/clean-code-cheat-sheet-v-2-4/
+ Addy Osmani - "Learning JavaScript Design Patterns": http://addyosmani.com/resources/essentialjsdesignpatterns/book/

### BDD ( 行为驱动开发 ) 

+ Enrique Amodeo - "Learning Behavior-driven Development with JavaScript": https://www.packtpub.com/application-development/learning-behavior-driven-development-javascript

### 事件

+ Assert(js) Testing Conference 2018: https://www.youtube.com/playlist?list=PLZ66c9_z3umNSrKSb5cmpxdXZcIPNvKGw

### 相关库

+ Jasmine: https://jasmine.github.io/
+ Jest: https://jestjs.io/
+ Mocha: https://mochajs.org/
+ Tape: https://github.com/substack/tape

• [返回目录](#-目录) •

