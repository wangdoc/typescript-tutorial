# TypeScript 装饰器

## 简介

装饰器（Decorator）是一种语法结构，用来在定义时修改类（class）的行为。

在语法上，装饰器有如下几个特征。

（1）第一个字符（或者说前缀）是`@`，后面是一个表达式。

（2）`@`后面的表达式，必须是一个函数（或者执行后可以得到一个函数）。

（3）这个函数接受所修饰对象的一些相关值作为参数。

（4）这个函数要么不返回值，要么返回一个新对象取代所修饰的目标对象。

举例来说，有一个函数`Injectable()`当作装饰器使用，那么需要写成`@Injectable`，然后放在某个类的前面。

```typescript
@Injectable class A {
  // ...
}
```

上面示例中，由于有了装饰器`@Injectable`，类`A`的行为在运行时就会发生改变。

下面就是一个最简单的装饰器。

```typescript
function simpleDecorator() {
  console.log('hi');
}

@simpleDecorator
class A {} // "hi"
```

上面示例中，函数`simpleDecorator()`用作装饰器，附加在类`A`之上，后者在代码解析时就会打印一行日志。

编译上面的代码会报错，提示没有用到装饰器的参数。现在就为装饰器加上参数，让它更像正式运行的代码。

```typescript
function simpleDecorator(
  target:any,
  context:any
) {
  console.log('hi, this is ' + target);
  return target;
}

@simpleDecorator
class A {} // "hi, this is class A {}" 
```

上面的代码就可以顺利通过编译了，代码含义这里先不解释。大家只要理解，类`A`在执行前会先执行装饰器`simpleDecorator()`，并且会向装饰器自动传入参数就可以了。

装饰器有多种形式，基本上只要在`@`符号后面添加表达式都是可以的。下面都是合法的装饰器。

```typescript
@myFunc
@myFuncFactory(arg1, arg2)

@libraryModule.prop
@someObj.method(123)

@(wrap(dict['prop']))
```

注意，`@`后面的表达式，最终执行后得到的应该是一个函数。

相比使用子类改变父类，装饰器更加简洁优雅，缺点是不那么直观，功能也受到一些限制。所以，装饰器一般只用来为类添加某种特定行为。

```javascript
@frozen class Foo {

  @configurable(false)
  @enumerable(true)
  method() {}

  @throttle(500)
  expensiveMethod() {}
}
```

上面示例中，一共有四个装饰器，一个用在类本身（`@frozen`），另外三个用在类的方法（`@configurable`、`@enumerable`、`@throttle`）。它们不仅增加了代码的可读性，清晰地表达了意图，而且提供一种方便的手段，增加或修改类的功能。

## 装饰器的版本

TypeScript 从早期开始，就支持装饰器。但是，装饰器的语法后来发生了变化。ECMAScript 标准委员会最终通过的语法标准，与 TypeScript 早期使用的语法有很大差异。

目前，TypeScript 5.0 同时支持两种装饰器语法。标准语法可以直接使用，传统语法需要打开`--experimentalDecorators`编译参数。

```bash
$ tsc --target ES5 --experimentalDecorators
```

本章介绍装饰器的标准语法，下一章介绍传统语法。

## 装饰器的结构

装饰器函数的类型定义如下。

```typescript
type Decorator = (
  value: DecoratedValue,
  context: {
    kind: string;
    name: string | symbol;
    addInitializer?(initializer: () => void): void;
    static?: boolean;
    private?: boolean;
    access: {
      get?(): unknown;
      set?(value: unknown): void;
    };
  }
) => void | ReplacementValue;
```

上面代码中，`Decorator`是装饰器的类型定义。它是一个函数，使用时会接收到`value`和`context`两个参数。

- `value`：所装饰的对象。
- `context`：上下文对象，TypeScript 提供一个原生接口`ClassMethodDecoratorContext`，描述这个对象。

```typescript
function decorator(
  value:any,
  context:ClassMethodDecoratorContext
) {
  // ...
}
```

上面是一个装饰器函数，其中第二个参数`context`的类型就可以写成`ClassMethodDecoratorContext`。

`context`对象的属性，根据所装饰对象的不同而不同，其中只有两个属性（`kind`和`name`）是必有的，其他都是可选的。

（1）`kind`：字符串，表示所装饰对象的类型，可能取以下的值。

- 'class'
- 'method'
- 'getter'
- 'setter'
- 'field'
- 'accessor'

这表示一共有六种类型的装饰器。

（2）`name`：字符串或者 Symbol 值，所装饰对象的名字，比如类名、属性名等。

（3）`addInitializer()`：函数，用来添加类的初始化逻辑。以前，这些逻辑通常放在构造函数里面，对方法进行初始化，现在改成以函数形式传入`addInitializer()`方法。注意，`addInitializer()`没有返回值。

（4）`private`：布尔值，表示所装饰的对象是否为类的私有成员。

（5）`static`：布尔值，表示所装饰的对象是否为类的静态成员。

（6）`access`：一个对象，包含了某个值的 get 和 set 方法。

## 类装饰器

类装饰器的类型描述如下。

```typescript
type ClassDecorator = (
  value: Function,
  context: {
    kind: 'class';
    name: string | undefined;
    addInitializer(initializer: () => void): void;
  }
) => Function | void;
```

类装饰器接受两个参数：`value`（当前类本身）和`context`（上下文对象）。其中，`context`对象的`kind`属性固定为字符串`class`。

类装饰器一般用来对类进行操作，可以不返回任何值，请看下面的例子。

```typescript
function Greeter(value, context) {
  if (context.kind === 'class') {
    value.prototype.greet = function () {
      console.log('你好');
    };
  }
}

@Greeter
class User {}

let u = new User();
u.greet(); // "你好"
```

上面示例中，类装饰器`@Greeter`在类`User`的原型对象上，添加了一个`greet()`方法，实例就可以直接使用该方法。

类装饰器可以返回一个函数，替代当前类的构造方法。

```typescript
function countInstances(value:any, context:any) {
  let instanceCount = 0;

  const wrapper = function (...args:any[]) {
    instanceCount++;
    const instance = new value(...args);
    instance.count = instanceCount;
    return instance;
  } as unknown as typeof MyClass;

  wrapper.prototype = value.prototype; // A
  return wrapper;
}

@countInstances
class MyClass {}

const inst1 = new MyClass();
inst1 instanceof MyClass // true
inst1.count // 1
```

上面示例中，类装饰器`@countInstances`返回一个函数，替换了类`MyClass`的构造方法。新的构造方法实现了实例的计数，每新建一个实例，计数器就会加一，并且对实例添加`count`属性，表示当前实例的编号。

注意，上例为了确保新构造方法继承定义在`MyClass`的原型之上的成员，特别加入`A`行，确保两者的原型对象是一致的。否则，新的构造函数`wrapper`的原型对象，与`MyClass`不同，通不过`instanceof`运算符。

类装饰器也可以返回一个新的类，替代原来所装饰的类。

```typescript
function countInstances(value:any, context:any) {
  let instanceCount = 0;

  return class extends value {
    constructor(...args:any[]) {
      super(...args);
      instanceCount++;
      this.count = instanceCount;
    }
  };
}

@countInstances
class MyClass {}

const inst1 = new MyClass();
inst1 instanceof MyClass // true
inst1.count // 1
```

上面示例中，`@countInstances`返回一个`MyClass`的子类。

下面的例子是通过类装饰器，禁止使用`new`命令新建类的实例。

```typescript
function functionCallable(
  value as any, {kind} as any
) {
  if (kind === 'class') {
    return function (...args) {
      if (new.target !== undefined) {
        throw new TypeError('This function can’t be new-invoked');
      }
      return new value(...args);
    }
  }
}

@functionCallable
class Person {
  constructor(name) {
    this.name = name;
  }
}
const robin = Person('Robin');
robin.name // 'Robin'
```

上面示例中，类装饰器`@functionCallable`返回一个新的构造方法，里面判断`new.target`是否不为空，如果是的，就表示通过`new`命令调用，从而报错。

类装饰器的上下文对象`context`的`addInitializer()`方法，用来定义一个类的初始化函数，在类完全定义结束后执行。

```typescript
function customElement(name: string) {
  return <Input extends new (...args: any) => any>(
    value: Input,
    context: ClassDecoratorContext
  ) => {
    context.addInitializer(function () {
      customElements.define(name, value);
    });
  };
}

@customElement("hello-world")
class MyComponent extends HTMLElement {
  constructor() {
    super();
  }
  connectedCallback() {
    this.innerHTML = `<h1>Hello World</h1>`;
  }
}
```

上面示例中，类`MyComponent`定义完成后，会自动执行类装饰器`@customElement()`给出的初始化函数，该函数会将当前类注册为指定名称（本例为`<hello-world>`）的自定义 HTML 元素。

## 方法装饰器

方法装饰器用来装饰类的方法（method）。它的类型描述如下。

```typescript
type ClassMethodDecorator = (
  value: Function,
  context: {
    kind: 'method';
    name: string | symbol;
    static: boolean;
    private: boolean;
    access: { get: () => unknown };
    addInitializer(initializer: () => void): void;
  }
) => Function | void;
```

根据上面的类型，方法装饰器是一个函数，接受两个参数：`value`和`context`。

参数`value`是方法本身，参数`context`是上下文对象，有以下属性。

- `kind`：值固定为字符串`method`，表示当前为方法装饰器。
- `name`：所装饰的方法名，类型为字符串或 Symbol 值。
- `static`：布尔值，表示是否为静态方法。该属性为只读属性。
- `private`：布尔值，表示是否为私有方法。该属性为只读属性。
- `access`：对象，包含了方法的存取器，但是只有`get()`方法用来取值，没有`set()`方法进行赋值。
- `addInitializer()`：为方法增加初始化函数。

方法装饰器会改写类的原始方法，实质等同于下面的操作。

```typescript
function trace(decoratedMethod) {
  // ...
}

class C {
  @trace
  toString() {
    return 'C';
  }
}

// `@trace` 等同于
// C.prototype.toString = trace(C.prototype.toString);
```

上面示例中，`@trace`是方法`toString()`的装饰器，它的效果等同于最后一行对`toString()`的改写。

如果方法装饰器返回一个新的函数，就会替代所装饰的原始函数。

```typescript
function replaceMethod() {
  return function () {
    return `How are you, ${this.name}?`;
  }
}

class Person {
  constructor(name) {
    this.name = name;
  }

  @replaceMethod
  hello() {
    return `Hi ${this.name}!`;
  }
}

const robin = new Person('Robin');

robin.hello() // 'How are you, Robin?'
```

上面示例中，装饰器`@replaceMethod`返回的函数，就成为了新的`hello()`方法。

下面是另一个例子。

```typescript
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }

  @log
  greet() {
    console.log(`Hello, my name is ${this.name}.`);
  }
}

function log(originalMethod:any, context:ClassMethodDecoratorContext) {
  const methodName = String(context.name);

  function replacementMethod(this: any, ...args: any[]) {
    console.log(`LOG: Entering method '${methodName}'.`)
    const result = originalMethod.call(this, ...args);
    console.log(`LOG: Exiting method '${methodName}'.`)
    return result;
  }

  return replacementMethod;
}

const person = new Person('张三');
person.greet()
// "LOG: Entering method 'greet'."
// "Hello, my name is 张三."
// "LOG: Exiting method 'greet'."
```

上面示例中，装饰器`@log`的返回值是一个函数`replacementMethod`，替代了原始方法`greet()`。在`replacementMethod()`内部，通过执行`originalMethod.call()`完成了对原始方法的调用。

利用方法装饰器，可以将类的方法变成延迟执行。

```typescript
function delay(milliseconds: number = 0) {
  return function (value, context) {
    if (context.kind === "method") {
      return function (...args: any[]) {
        setTimeout(() => {
          value.apply(this, args);
        }, milliseconds);
      };
    }
  };
}

class Logger {
  @delay(1000)
  log(msg: string) {
    console.log(`${msg}`);
  }
}

let logger = new Logger();
logger.log("Hello World");
```

上面示例中，方法装饰器`@delay(1000)`将方法`log()`的执行推迟了1秒（1000毫秒）。这里真正的方法装饰器，是`delay()`执行后返回的函数，`delay()`的作用是接收参数，用来设置推迟执行的时间。这种通过高阶函数返回装饰器的做法，称为“工厂模式”，即可以像工厂那样生产出一个模子的装饰器。

方法装饰器的参数`context`对象里面，有一个`addInitializer()`方法。它是一个钩子方法，用来在类的初始化阶段，添加回调函数。这个回调函数就是作为`addInitializer()`的参数传入的，它会在构造方法执行期间执行，早于属性（field）的初始化。

下面是`addInitializer()`方法的一个例子。我们知道，类的方法往往需要在构造方法里面，进行`this`的绑定。

```typescript
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;

    // greet() 绑定 this
    this.greet = this.greet.bind(this);
  }

  greet() {
    console.log(`Hello, my name is ${this.name}.`);
  }
}

const g = new Person('张三').greet;
g() // "Hello, my name is 张三."
```

上面例子中，类`Person`的构造方法内部，将`this`与`greet()`方法进行了绑定。如果没有这一行，将`greet()`赋值给变量`g`进行调用，就会报错了。

`this`的绑定必须放在构造方法里面，因为这必须在类的初始化阶段完成。现在，它可以移到方法装饰器的`addInitializer()`里面。

```typescript
function bound(
  originalMethod:any, context:ClassMethodDecoratorContext
) {
  const methodName = context.name;
  if (context.private) {
    throw new Error(`不能绑定私有方法 ${methodName as string}`);
  }
  context.addInitializer(function () {
    this[methodName] = this[methodName].bind(this);
  });
}
```

上面示例中，绑定`this`转移到了`addInitializer()`方法里面。

下面再看一个例子，通过`addInitializer()`将选定的方法名，放入一个集合。

```typescript
function collect(
  value,
  {name, addInitializer}
) {
  addInitializer(function () {
    if (!this.collectedMethodKeys) {
      this.collectedMethodKeys = new Set();
    }
    this.collectedMethodKeys.add(name);
  });
}

class C {
  @collect
  toString() {}

  @collect
  [Symbol.iterator]() {}
}

const inst = new C();
inst.@collect // new Set(['toString', Symbol.iterator])
```

上面示例中，方法装饰器`@collect`会将所装饰的成员名字，加入一个 Set 集合`collectedMethodKeys`。

## 属性装饰器

属性装饰器用来装饰定义在类顶部的属性（field）。它的类型描述如下。

```typescript
type ClassFieldDecorator = (
  value: undefined,
  context: {
    kind: 'field';
    name: string | symbol;
    static: boolean;
    private: boolean;
    access: { get: () => unknown, set: (value: unknown) => void };
    addInitializer(initializer: () => void): void;
  }
) => (initialValue: unknown) => unknown | void;
```

注意，装饰器的第一个参数`value`的类型是`undefined`，这意味着这个参数实际上没用的，装饰器不能从`value`获取所装饰属性的值。另外，第二个参数`context`对象的`kind`属性的值为字符串`field`，而不是“property”或“attribute”，这一点是需要注意的。

属性装饰器要么不返回值，要么返回一个函数，该函数会自动执行，用来对所装饰属性进行初始化。该函数的参数是所装饰属性的初始值，该函数的返回值是该属性的最终值。

```typescript
function logged(value, context) {
  const { kind, name } = context;
  if (kind === 'field') {
    return function (initialValue) {
      console.log(`initializing ${name} with value ${initialValue}`);
      return initialValue;
    };
  }
}

class Color {
  @logged name = 'green';
}

const color = new Color();
// "initializing name with value green"
```

上面示例中，属性装饰器`@logged`装饰属性`name`。`@logged`的返回值是一个函数，该函数用来对属性`name`进行初始化，它的参数`initialValue`就是属性`name`的初始值`green`。新建实例对象`color`时，该函数会自动执行。

属性装饰器的返回值函数，可以用来更改属性的初始值。

```typescript
function twice() {
  return initialValue => initialValue * 2;
}

class C {
  @twice
  field = 3;
}

const inst = new C();
inst.field // 6
```

上面示例中，属性装饰器`@twice`返回一个函数，该函数的返回值是属性`field`的初始值乘以2，所以属性`field`的最终值是6。

属性装饰器的上下文对象`context`的`access`属性，提供所装饰属性的存取器，请看下面的例子。

```typescript
let acc;

function exposeAccess(
  value, {access}
) {
  acc = access;
}

class Color {
  @exposeAccess
  name = 'green'
}

const green = new Color();
green.name // 'green'

acc.get(green) // 'green'

acc.set(green, 'red');
green.name // 'red'
```

上面示例中，`access`包含了属性`name`的存取器，可以对该属性进行取值和赋值。

## getter 装饰器，setter 装饰器

getter 装饰器和 setter 装饰器，是分别针对类的取值器（getter）和存值器（setter）的装饰器。它们的类型描述如下。

```typescript
type ClassGetterDecorator = (
  value: Function,
  context: {
    kind: 'getter';
    name: string | symbol;
    static: boolean;
    private: boolean;
    access: { get: () => unknown };
    addInitializer(initializer: () => void): void;
  }
) => Function | void;

type ClassSetterDecorator = (
  value: Function,
  context: {
    kind: 'setter';
    name: string | symbol;
    static: boolean;
    private: boolean;
    access: { set: (value: unknown) => void };
    addInitializer(initializer: () => void): void;
  }
) => Function | void;
```

注意，getter 装饰器的上下文对象`context`的`access`属性，只包含`get()`方法；setter 装饰器的`access`属性，只包含`set()`方法。

这两个装饰器要么不返回值，要么返回一个函数，取代原来的取值器或存值器。

下面的例子是将取值器的结果，保存为一个属性，加快后面的读取。

```typescript
class C {
  @lazy
  get value() {
    console.log('正在计算……');
    return '开销大的计算结果';
  }
}

function lazy(
  value:any,
  {kind, name}:any
) {
  if (kind === 'getter') {
    return function (this:any) {
      const result = value.call(this);
      Object.defineProperty(
        this, name,
        {
          value: result,
          writable: false,
        }
      );
      return result;
    };
  }
  return;
}

const inst = new C();
inst.value
// 正在计算……
// '开销大的计算结果'
inst.value
// '开销大的计算结果'
```

上面示例中，第一次读取`inst.value`，会进行计算，然后装饰器`@lazy`将结果存入只读属性`value`，后面再读取这个属性，就不会进行计算了。

## accessor 装饰器

装饰器语法引入了一个新的属性修饰符`accessor`。

```typescript
class C {
  accessor x = 1;
}
```

上面示例中，`accessor`修饰符等同于为属性`x`自动生成取值器和存值器，它们作用于私有属性`x`。也就是说，上面的代码等同于下面的代码。

```typescript
class C {
  #x = 1;

  get x() {
    return this.#x;
  }

  set x(val) {
    this.#x = val;
  }
}
```

`accessor`也可以与静态属性和私有属性一起使用。

```typescript
class C {
  static accessor x = 1;
  accessor #y = 2;
}
```

accessor 装饰器的类型如下。

```typescript
type ClassAutoAccessorDecorator = (
  value: {
    get: () => unknown;
    set(value: unknown) => void;
  },
  context: {
    kind: "accessor";
    name: string | symbol;
    access: { get(): unknown, set(value: unknown): void };
    static: boolean;
    private: boolean;
    addInitializer(initializer: () => void): void;
  }
) => {
  get?: () => unknown;
  set?: (value: unknown) => void;
  init?: (initialValue: unknown) => unknown;
} | void;
```

accessor 装饰器的`value`参数，是一个包含`get()`方法和`set()`方法的对象。该装饰器可以不返回值，或者返回一个新的对象，用来取代原来的`get()`方法和`set()`方法。此外，装饰器返回的对象还可以包括一个`init()`方法，用来改变私有属性的初始值。

下面是一个例子。

```typescript
class C {
  @logged accessor x = 1;
}

function logged(value, { kind, name }) {
  if (kind === "accessor") {
    let { get, set } = value;

    return {
      get() {
        console.log(`getting ${name}`);

        return get.call(this);
      },

      set(val) {
        console.log(`setting ${name} to ${val}`);

        return set.call(this, val);
      },

      init(initialValue) {
        console.log(`initializing ${name} with value ${initialValue}`);
        return initialValue;
      }
    };
  }
}

let c = new C();

c.x;
// getting x

c.x = 123;
// setting x to 123
```

上面示例中，装饰器`@logged`为属性`x`的存值器和取值器，加上了日志输出。

## 装饰器的执行顺序

装饰器的执行分为两个阶段。

（1）评估（evaluation）：计算`@`符号后面的表达式的值，得到的应该是函数。

（2）应用（application）：将评估装饰器后得到的函数，应用于所装饰对象。

也就是说，装饰器的执行顺序是，先评估所有装饰器表达式的值，再将其应用于当前类。

应用装饰器时，顺序依次为方法装饰器和属性装饰器，然后是类装饰器。

请看下面的例子。

```typescript
function d(str:string) {
  console.log(`评估 @d(): ${str}`);
  return (
    value:any, context:any
  ) => console.log(`应用 @d(): ${str}`);
}

function log(str:string) {
  console.log(str);
  return str;
}

@d('类装饰器')
class T {
  @d('静态属性装饰器')
  static staticField = log('静态属性值');

  @d('原型方法')
  [log('计算方法名')]() {}

  @d('实例属性')
  instanceField = log('实例属性值');
}
```

上面示例中，类`T`有四种装饰器：类装饰器、静态属性装饰器、方法装饰器、属性装饰器。

它的运行结果如下。

```typescript
// "评估 @d(): 类装饰器"
// "评估 @d(): 静态属性装饰器"
// "评估 @d(): 原型方法"
// "计算方法名"
// "评估 @d(): 实例属性"
// "应用 @d(): 原型方法"
// "应用 @d(): 静态属性装饰器"
// "应用 @d(): 实例属性"
// "应用 @d(): 类装饰器"
// "静态属性值"
```

可以看到，类载入的时候，代码按照以下顺序执行。

（1）装饰器评估：这一步计算装饰器的值，首先是类装饰器，然后是类内部的装饰器，按照它们出现的顺序。

注意，如果属性名或方法名是计算值（本例是“计算方法名”），则它们在对应的装饰器评估之后，也会进行自身的评估。

（2）装饰器应用：实际执行装饰器函数，将它们与对应的方法和属性进行结合。

原型方法的装饰器首先应用，然后是静态属性和静态方法装饰器，接下来是实例属性装饰器，最后是类装饰器。

注意，“实例属性值”在类初始化的阶段并不执行，直到类实例化时才会执行。

如果一个方法或属性有多个装饰器，则内层的装饰器先执行，外层的装饰器后执行。

```typescript
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }

  @bound
  @log
  greet() {
    console.log(`Hello, my name is ${this.name}.`);
  }
}
```

上面示例中，`greet()`有两个装饰器，内层的`@log`先执行，外层的`@bound`针对得到的结果再执行。

## 参考链接

- [JavaScript metaprogramming with the 2022-03 decorators API](https://2ality.com/2022/10/javascript-decorators.html)
- [TS 5.0 Beta: New Decorators Are Here!](https://plainenglish.io/blog/ts-5-0-beta-new-decorators-are-here), Bytefer

