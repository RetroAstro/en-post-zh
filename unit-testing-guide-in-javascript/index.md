# JavaScript 单元测试指南

## 📖 目录

1. 通则
  + [单元测试](#unit-tests)
  + [设计原则](#design-principles)
2. 指南
  + [尽可能地使用 TDD ( 测试驱动开发 )](#尽可能地使用-TDD-(-测试驱动开发-))
  + [正确地组织测试](#正确地组织测试)
  + [正确地为测试命名](#正确地为测试命名) 
  + [不要在测试中添加注释](#不要在测试中添加注释)
  + [在测试中要避免逻辑](#在测试中要避免逻辑) 
  + [不要写不必要的断言](#不要写不必要的断言) 
  + [正确地初始化应用于所有相关测试的操作](#正确地初始化应用于所有相关测试的操作) 
  + [考虑在测试中使用工厂函数](#考虑在测试中使用工厂函数) 
  + [熟悉你的测试框架 API](#熟悉你的测试框架-API) 
  + [不要在同一测试中测试多个关注点](#不要在同一测试中测试多个关注点) 
  + [要覆盖一般情况与边缘情况](#要覆盖一般情况与边缘情况) 
  + [在应用 TDD 时，总是从编写最简单的失败测试开始](#在应用-TDD-时，总是从编写最简单的失败测试开始) 
  + [在应用 TDD 时，总是在每个测试优先的周期中小步前进](#在应用-TDD-时，总是在每个测试优先的周期中小步前进) 
  + [测试行为，而不是内部实现](#测试行为，而不是内部实现) 
  + [不要 mock 所有数据](#不要-mock-所有数据) 
  + [为每个 bug 创建新的测试](#为每个-bug-创建新的测试) 
  + [不要为复杂的用户交互编写单元测试](#不要为复杂的用户交互编写单元测试) 
  + [测试简单的用户操作](#测试简单的用户操作) 
  + [首先审查测试代码](#首先审查测试代码) 
  + [在编码中实践，通过结对编程学习](#在编码中实践，通过结对编程学习) 
3. [参考资料](#参考资料) 

## 通则

### 单元测试

**单元 = 工作单元** 

它可以是那些在公共 API 中调用的**多个方法和类**且能够：

* 返回值或者抛出异常
* 改变系统的状态
* 进行第三方调用（API、数据库等）

单元测试应该测试一个工作单元的行为：给定一个输入，最终的期望结果可以是上面的任何一种。

**单元测试之间应该是相互隔离且彼此独立的** 

* 每个给定的行为都应该对应一个**单独的测试** 
* 一个测试的执行与执行顺序**不应该影响其他测试** 

**单元测试属于轻量测试**

+ 可重复
+ 快速
+ 一致性
+ 方便读写

**单元测试也是代码** 

它们应该与正在测试的代码具有相同的质量级别。它们也能通过重构来增强代码自身的可维护性与可读性。

• [返回目录](#目录) •

### 设计原则

良好单元测试的关键在于编写**可测试的代码**。下面是一些有用的设计原则：

* 使用良好的命名规范并注释代码（“为什么”而不是“怎么做”），要记住注释并不能代替糟糕的命名和设计
* 不要重复自己，避免重复代码
* **单一职责**：每个对象 / 函数只应该关注一个任务
* 在同一组件中保持**单一抽象级别**（例如，不要将业务逻辑与同一方法中的低层实现细节混合在一起）
* **最小化组件间依赖**：通过封装组件，减少组件之间的信息交换
* **支持可配置性而不是硬编码**，这避免了在测试时复制完全相同的环境
* 适当地引用**设计模式**，尤其是**依赖注入**，因为它能将对象的创建职责与业务逻辑分离

+ 避免全局状态

• [返回目录](#目录) •

## 指南

这些指南的目的在于让你的测试具有：

+ **可读性**
+ **可维护性**
+ **可靠性** 

这是良好单元测试的三大支柱

下文所有的测试用例都以 [Jasmine](http://jasmine.github.io) 框架为基础。

• [返回目录](#目录) •

---------------------------------------

### 尽可能地使用 TDD ( 测试驱动开发 )

TDD is a _design process_, not a testing process. TDD is a robust way of designing software components ("units") interactively so that their behaviour is specified through unit tests.

How? Why?

#### Test-first cycle

1. Write a simple failing test
2. Make the test pass by writing the minimum amount of code, don't bother with code quality
3. Refactor the code by applying design principles/patterns

#### Consequences of the test-first cycle

+ Writing a test first makes the code design testable de facto
+ Writing just the amount of code needed to implement the required functionality makes the resulting codebase minimal, thus more maintainable
+ The codebase can be enhanced using refactoring mechanisms, the tests give you confidence that the new code is not modifying the existing functionalities
+ Cleaning the code in each cycle makes the codebase more maintainable, it is much cheaper to change the code frequently and in small increments
+ Fast feedback for the developers, you know that you don't break anything and that you are evolving the system in a good direction
+ Generates confidence to add features, fix bugs, or explore new designs

Note that code written without a test-first approach is often very hard to test.

• [返回目录](#目录) •

### 正确地组织测试

Don't hesitate to nest your suites to structure logically your tests in subsets.

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

• [返回目录](#目录) •

### 正确地为测试命名

Tests names should be concise, explicit, descriptive and in correct English. Read the output of the spec runner and verify that it is understandable! Keep in mind that someone else will read it too. Tests can be the live documentation of the code.

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

In order to help you write test names properly, you can use the **"unit of work - scenario/context - expected behaviour"** pattern:

```js
describe('[unit of work]', () => {
  it('should [expected behaviour] when [scenario/context]', () => {
  });
});
```

Or whenever you have many tests that follow the same scenario or are related to the same context:

```js
describe('[unit of work]', () => {
  describe('when [scenario/context]', () => {
    it('should [expected behaviour]', () => {
    });
  });
});
```

For example:

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

• [返回目录](#目录) •

### 不要在测试中添加注释

Never. Ever. Tests have a reason to be or not.

Don't comment them out because they are too slow, too complex or produce false negatives. Instead, make them fast, simple and trustworthy. If not, remove them completely.

• [返回目录](#目录) •

### 在测试中要避免逻辑

Always use simple statements. Don't use loops and/or conditionals. If you do, you add a possible entry point for bugs in the test itself:

+ Conditionals: you don't know which path the test will take
+ Loops: you could be sharing state between tests

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

Better: write a test for each type of sanitization. It will give a nice output of all possible cases, improving maintainability.

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

• [返回目录](#目录) •

### 不要写不必要的断言

Remember, unit tests are a design specification of how a certain *behaviour* should work, not a list of observations of everything the code happens to do.

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

This will improve maintainability. Your test is no longer tied to implementation details.

• [返回目录](#目录) •

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

The setup code should apply to all the tests:

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

Consider keeping the setup code minimal to preserve readability and maintainability.

• [返回目录](#目录) •

### 考虑在测试中使用工厂函数

Factories can:

+ help reduce the setup code, especially if you use dependency injection
+ make each test more readable, since the creation is a single function call that can be in the test itself instead of the setup
+ provide flexibility when creating new instances (setting an initial state, for example)

There's a trade-off to find here between applying the DRY principle and readability.

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

Factories are particularly useful when dealing with the DOM:

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

• [返回目录](#目录) •

### 熟悉你的测试框架 API

The API documentation of the testing framework/library should be your bedside book!

Having a good knowledge of the API can help you in reducing the size/complexity of your test code and, in general, help you during development. A simple example:

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

The handy `fit` function used in the example above allows you to execute only one test without having to comment out all the tests below. `fdescribe` does the same for test suites. This could help save a lot of time when developing.

More information on the [Jasmine website](http://jasmine.github.io).

• [返回目录](#目录) •

### 不要在同一测试中测试多个关注点

If a method has several end results, each one should be tested separately. Whenever a bug occurs, it will help you locate the source of the problem.

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

Beware that writing "AND" or "OR" when naming your test smells bad...

• [返回目录](#目录) •

### 要覆盖一般情况与边缘情况

"Strange behaviour" usually happens at the edges... Remember that your tests can be the live documentation of your code.

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

• [返回目录](#目录) •

### 在应用 TDD 时，总是从编写最简单的失败测试开始

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

From there, start building the functionalities incrementally.

• [返回目录](#目录) •

### 在应用 TDD 时，总是在每个测试优先的周期中小步前进

Build your tests suite from the simple case to the more complex ones. Keep in mind the incremental design. Deliver software fast, incrementally, and in short iterations.

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

• [返回目录](#目录) •

### 测试行为，而不是内部实现

**:(**

```js
it('should add a user in memory', () => {
  userManager.addUser('Dr. Falker', 'Joshua');

  expect(userManager._users[0].name).toBe('Dr. Falker');
  expect(userManager._users[0].password).toBe('Joshua');
});
```

A better approach is to test at the same level of the API:

**:)**

```js
it('should add a user in memory', () => {
  userManager.addUser('Dr. Falker', 'Joshua');

  expect(userManager.loginUser('Dr. Falker', 'Joshua')).toBe(true);
});
```

Pro:

+ Changing the internal implementation of a class/object will not necessarily force you to refactor the tests

Con:

+ If a test is failing, we might have to debug to know which part of the code needs to be fixed

Here, a balance has to be found, unit-testing some key parts can be beneficial.

• [返回目录](#目录) •

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

This test fails, because the survey is considered disabled. Let's fix this:

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

This will work... but needs a lot of code. Let's try a simpler approach:

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

We created a permanent storage of data. What happens if we do not properly clean it?
We might affect the other tests. Let's fix this:

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

The `MemoryStorage` used here does not persist data. Nice and easy, with no side effects.

#### Takeaway

The idea to keep in mind is that *dependencies can still be "real" objects*. Don't mock everything because you can.
In particular, consider using the "real" version of the objects if:

+ it leads to a simple, nice and easy tests setup
+ it does not create a shared state between the tests, causing unexpected side effects
+ the code being tested does not make AJAX requests, API calls or browser page reloads
+ the speed of execution of the tests stays *within the limits you fixed*

• [返回目录](#目录) •

### 为每个 bug 创建新的测试

Whenever a bug is found, create a test that replicates the problem **before touching any code**. From there, you can apply TDD as usual to fix it.

• [返回目录](#目录) •

### 不要为复杂的用户交互编写单元测试

Examples of complex user interactions:

+ Filling a form, drag and dropping some items then submitting the form
+ Clicking a tab, clicking an image thumbnail then navigating through a gallery of images previously loaded from a database
+ (...)

These interactions might involve many units of work and should be handled at a higher level by **functional tests**. They will take more time to execute. They could be flaky (false negatives) and they need debugging whenever a failure is reported.

For functional testing, consider using a test automation framework ([Selenium](http://docs.seleniumhq.org/), ...) or QA manual testing.

• [返回目录](#目录) •

### 测试简单的用户操作

Example of simple user actions:

+ Clicking on a link that toggles the visibility of a DOM element
+ Submitting a form that triggers the form validation
+ (...)

These actions can be easily tested **by simulating DOM events**, for example:

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

Note how simple the test is because the UI (DOM) layer does not mix with the business logic layer:

+ a "click" event occurs
+ a public method is called

The next step could be to test the business logic implemented in "showPreview()" or "hidePreview()".

• [返回目录](#目录) •

### 首先审查测试代码

When reviewing code, always start by reading the code of the tests. Tests are mini use cases of the code that you can drill into.

It will help you understand the intent of the developer very quickly (could be just by looking at the names of the tests).

• [返回目录](#目录) •

### 在编码中实践，通过结对编程学习

Because experience is the _only_ teacher. Ultimately, greatness comes from practicing; applying the theory over and over again, using feedback to get better every time.

• [返回目录](#目录) •

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

### BDD (行为驱动开发) 

+ Enrique Amodeo - "Learning Behavior-driven Development with JavaScript": https://www.packtpub.com/application-development/learning-behavior-driven-development-javascript

### 事件

+ Assert(js) Testing Conference 2018: https://www.youtube.com/playlist?list=PLZ66c9_z3umNSrKSb5cmpxdXZcIPNvKGw

### 相关库

+ Jasmine: https://jasmine.github.io/
+ Jest: https://jestjs.io/
+ Mocha: https://mochajs.org/
+ Tape: https://github.com/substack/tape

• [返回目录](#目录) •

