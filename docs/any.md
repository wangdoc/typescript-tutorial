# any 类型，unknown 类型，never 类型

本章介绍 TypeScript 的三种特殊类型，它们可以作为学习 TypeScript 类型系统的起点。

## any 类型

### 基本含义

any 类型表示该位置不限制类型，任意类型的值都可以使用。

```typescript
let x:any;

x = 1; // 正确
x = 'foo'; // 正确
x = true; // 正确
```

上面示例中，变量`x`的类型是`any`，就可以被赋值为任意类型的值。

变量类型一旦设为`any`，TypeScript 实际上会关闭它的类型检查，即使有明显的类型错误，只要句法正确，都不会报错。

```typescript
let x:any = 'hello';
x(1) // 正确
x.foo = 100; // 正确
```

上面示例中，变量`x`的值是一个字符串，但是把它当作函数调用，或者当作对象读取任意属性，TypeScript 编译时都不报错。原因就是`x`的类型是`any`，TypeScript 不对其进行类型检查。

实际项目中，`any`类型往往用于关闭某些变量的类型检查。由于这个原因，应该尽量避免使用`any`类型，否则就失去了使用 TypeScript 的意义。

这个类型的主要设计目的，是为了适配以前老的 JavaScript 项目的迁移。有些年代很久的大型 JavaScript 项目，尤其是别人的代码，很难为每一行适配正确的类型，这时你为那些类型复杂的变量加上`any`，TypeScript 编译时就不会报错。不过，这大概是`any`唯一的适用场合。

总之，TypeScript 认为，只要开发者使用了`any`类型，就表示开发者想要自己来处理这些代码，所以就不对`any`类型进行任何限制，怎么使用都可以。

### 类型推断问题

`any`类型的另一个出现场景是，对于那些开发者没有指定类型、TypeScript 必须自己推断类型的变量，如果这时无法推断出类型，TypeScript 就会认为该变量的类型是`any`。

```typescript
function add(x, y) {
  return x + y;
}

add(1, [1, 2, 3]) // 正确
```

上面示例中，函数`add()`的参数变量`x`和`y`，都没有足够的信息，TypeScript 无法推断出它们的类型，就会认为这些变量的类型是`any`。以至于后面就不再对函数`add()`进行类型检查了，怎么用都可以。

这显然是很糟糕的情况，所以对于那些类型不明显的变量，一定要明确声明类型，防止推断为`any`。

TypeScript 提供了一个编译选项`--noImplicitAny`，只要打开这个选项，推断不出类型就会报错。

```bash
$ tsc --noImplicitAny app.ts
```

上面命令就使用了`--noImplicitAny`编译选项进行编译，这时上面的函数`add()`就会报错。

### 污染问题

`any`类型除了关闭类型检查，还有一个很大的问题，就是它会“污染”其他变量。它可以赋值给其他任何类型的变量（因为没有类型检查），导致其他变量出错。

```typescript
let x:any = 'hello';
let y:number;

y = x; // 正确

y * 123 // 正确
y.toFixed() // 正确
```

上面示例中，变量`x`的类型是`any`，实际的值是一个字符串。数值类型的变量`y`被赋值为`x`，也不会报错。然后，变量`y`继续进行各种数值运算，TypeScript 也检查不出错误，问题就这样留到运行时才会暴露。

污染其他具有正确类型的变量，把错误留到运行时，这就是不宜使用`any`类型的另一个主要原因。

### 顶端类型

前面说过，`any`类型可以被赋值为任何类型的值。在 TypeScript 语言中，如果类型`A`可以被赋值为类型`B`，那么类型`A`称为父类型，类型`B`称为子类型。TypeScript 的一个规则是，凡是可以使用父类型的地方，都可以使用子类型。

由于任何值都可以赋值给`any`类型，所以`any`类型是 TypeScript 所有其他类型的父类型，或者说，所有其他类型都是`any`的子类型。

所以，`any`类型是 TypeScript 的一个基础类型，包含了一切可能的值。所有其他类型都可以看成是它的衍生类型，它又被称为顶端类型（top type）。

## unknown 类型

为了解决`any`类型“污染”其他变量的问题，TypeScript 3.0 引入了[`unknown`类型](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html#new-unknown-top-type)。它与`any`含义相同，表示类型不确定，但是使用上有一些限制，可以视为严格版的`any`。

`unknown`跟`any`的相似之处，在于所有类型的值都可以分配给`unknown`类型。

```typescript
let x:unknown;

x = true; // 正确
x = 42; // 正确
x = 'Hello World'; // 正确
```

上面示例中，变量`value`的类型是`unknown`，可以赋值为各种类型的值。这与`any`的行为一致。

`unknown`类型跟`any`类型的不同之处在于，它不能直接使用。主要有以下几个限制。

首先，`unknown`类型的变量，不能直接赋值给其他类型的变量（除了`any`类型和`unknown`类型）。

```typescript
let v:unknown = 123;

let v1:boolean = v; // 报错
let v2:number = v; // 报错
```

上面示例中，变量`v`是`unknown`类型，赋值给`any`和`unknown`以外类型的变量都会报错，这就避免了污染问题，从而克服了`any`类型的一大缺点。

另外，也不能直接调用`unknown`类型变量的方法和属性。

```typescript
let v1:unknown = { foo: 123 };
v1.foo  // 报错

let v2:unknown = 'hello';
v2.trim() // 报错

let v3:unknown = (n = 0) => n + 1;
v3() // 报错
```

上面示例中，直接调用`unknown`类型变量的属性和方法，或者直接当作函数执行，都会报错。

再次，`unknown`类型变量能够进行的运算是有限的，只能进行比较运算（运算符`==`、`===`、`!=`、`!==`、`||`、`&&`、`?`）、取反运算（运算符`!`）、`typeof`运算符和`instanceof`运算符这几种，其他运算都会报错。

```typescript
let a:unknown = 1;

a + 1 // 报错
a === 1 // 正确
```

上面示例中，`unknown`类型的变量`a`进行加法运算会报错，因为这是不允许的运算。但是，进行比较运算就是可以的。

那么，怎么才能使用`unknown`类型变量呢？

答案是只有经过“类型细化”（refine），`unknown`类型变量才可以使用。所谓“类型细化”，就是缩小`unknown`变量的类型范围，确保不会出错。

```typescript
let a:unknown = 1;

if (typeof a === 'number') {
  let r = a + 10; // 正确
}
```

上面示例中，`unknown`类型的变量`a`经过`typeof`运算以后，能够确定实际类型是`number`，就能用于加法运算了。这就是“类型细化”，即将一个不确定的类型细化为更明确的类型。

下面是另一个例子。

```typescript
let s:unknown = 'hello';

if (typeof s === 'string') {
  s.length; // 正确
}
```

上面示例中，确定变量`s`的类型为字符串以后，才能调用它的`length`属性。

这样设计的目的是，只有明确`unknown`变量的实际类型，才允许使用它，防止像`any`那样可以随意乱用，“污染”其他变量。类型细化以后再使用，就不会报错。

总之，`unknown`可以看作是更安全的`any`，凡是需要设为`any`的地方，通常都应该优先考虑设为`unknown`。

由于`unknown`类型的变量也可以被赋值为任意其他类型，所以其他类型（除了`any`）都可以视为它的子类型。所以，它和`any`一样都属于 TypeScript 的顶端类型。

## never 类型

类型也可能是空集，即不包含任何类型。为了逻辑的完整性，TypeScript 把这种情况也当作一种类型，叫做`never`类型。

`never`类型表示不可能的类型，也就是不可能有任何值属于这个类型。

```typescript
let x: never;
```

上面示例中，变量`x`的类型是`never`，就不可能赋给它任何值，都会报错。

`never`类型的使用场景，主要是在一些类型运算之中，保证类型运算的完整性，详见后面章节。另外，不可能返回值的函数，返回值的类型就可以写成`never`，详见《函数》一章。

如果一个变量可能有多种类型（即联合类型），通常需要使用分支处理每一种类型。这时，处理所有可能的类型之后，剩余的情况就属于`never`类型。

```typescript
function fn(x:string|number) {
  if (typeof x === 'string') {
    // ...
  } else if (typeof x === 'number') {
    // ...
  } else {
    x; // never 类型
  }
}
```

上面示例中，参数变量`x`可能是字符串，也可能是数组，判断了这两种情况后，剩下的`else`分支里面，`x`就是`never`类型了。

任何类型的变量都可以赋值为`never`类型。

```typescript
function f():never {
  throw new Error('Error');
}

let v1:number = f(); // 正确
let v2:string = f(); // 正确
let v3:string = f(); // 正确
```

上面示例中，函数`f()`会抛错，所以返回值类型可以写成`never`，即不可能返回任何值。各种其他类型的变量都可以赋值为`f()`的运行结果（`never`类型）。

前面说过，在 TypeScript 中，如果类型`A`可以被赋值为类型`B`，那么类型`B`就称为类型`A`的子类型。所以，`never`类型可以视为所有其他类型的子类型，表示不包含任何可能的值，这种情况叫做“尾端类型”（bottom type），`never`是 TypeScript 唯一的尾端类型。
