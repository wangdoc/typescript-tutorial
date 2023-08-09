# TypeScript 的函数类型

## 简介

函数的类型声明，需要在声明函数时，给出参数的类型和返回值的类型。

```typescript
function hello(
  txt:string
):void {
  console.log('hello ' + txt);
}
```

上面示例中，函数`hello()`在声明时，需要给出参数`txt`的类型（string），以及返回值的类型（`void`），后者写在参数列表的圆括号后面。`void`类型表示没有返回值，详见后文。

如果不指定参数类型（比如上例不写`txt`的类型），TypeScript 就会推断参数类型，如果缺乏足够信息，就会推断该参数的类型为`any`。

返回值的类型通常可以不写，因为 TypeScript 自己会推断出来。

```typescript
function hello(txt:string) {
  console.log('hello ' + txt);
}
```

上面示例中，由于没有`return`语句，TypeScript 会推断出函数`hello()`没有返回值。

不过，有时候出于文档目的，或者为了防止不小心改掉返回值，还是会写返回值的类型。

如果变量被赋值为一个函数，变量的类型有两种写法。

```typescript
// 写法一
const hello = function (txt:string) {
  console.log('hello ' + txt);
}

// 写法二
const hello:
  (txt:string) => void
= function (txt) {
  console.log('hello ' + txt);
};
```

上面示例中，变量`hello`被赋值为一个函数，它的类型有两种写法。写法一是通过等号右边的函数类型，推断出变量`hello`的类型；写法二则是使用箭头函数的形式，为变量`hello`指定类型，参数的类型写在箭头左侧，返回值的类型写在箭头右侧。

写法二有两个地方需要注意。

首先，函数的参数要放在圆括号里面，不放会报错。

其次，类型里面的参数名（本例是`txt`）是必须的。有的语言的函数类型可以不写参数名（比如 C 语言），但是 TypeScript 不行。如果写成`(string) => void`，TypeScript 会理解成函数有一个名叫 string 的参数，并且这个`string`参数的类型是`any`。

```typescript
type MyFunc = (string, number) => number;
// (string: any, number: any) => number
```

上面示例中，函数类型没写参数名，导致 TypeScript 认为参数类型都是`any`。

函数类型里面的参数名与实际参数名，可以不一致。

```typescript
let f:(x:number) => number;
 
f = function (y:number) {
  return y;
};
```

上面示例中，函数类型里面的参数名为`x`，实际的函数定义里面，参数名为`y`，两者并不相同。

如果函数的类型定义很冗长，或者多个函数使用同一种类型，写法二用起来就很麻烦。因此，往往用`type`命令为函数类型定义一个别名，便于指定给其他变量。

```typescript
type MyFunc = (txt:string) => void;

const hello:MyFunc = function (txt) {
  console.log('hello ' + txt);
};
```

上面示例中，`type`命令为函数类型定义了一个别名`MyFunc`，后面使用就很方便，变量可以指定为这个类型。

函数的实际参数个数，可以少于类型指定的参数个数，但是不能多于，即 TypeScript 允许省略参数。

```typescript
let myFunc:
  (a:number, b:number) => number;

myFunc = (a:number) => a; // 正确

myFunc = (
  a:number, b:number, c:number
) => a + b + c; // 报错
```

上面示例中，变量`myFunc`的类型只能接受两个参数，如果被赋值为只有一个参数的函数，并不报错。但是，被赋值为有三个参数的函数，就会报错。

这是因为 JavaScript 函数在声明时往往有多余的参数，实际使用时可以只传入一部分参数。比如，数组的`forEach()`方法的参数是一个函数，该函数默认有三个参数`(item, index, array) => void`，实际上往往只使用第一个参数`(item) => void`。因此，TypeScript 允许函数传入的参数不足。

```typescript
let x = (a:number) => 0;
let y = (b:number, s:string) => 0;

y = x; // 正确
x = y; // 报错
```

上面示例中，函数`x`只有一个参数，函数`y`有两个参数，`x`可以赋值给`y`，反过来就不行。

如果一个变量要套用另一个函数类型，有一个小技巧，就是使用`typeof`运算符。

```typescript
function add(
  x:number,
  y:number
) {
  return x + y;
}

const myAdd:typeof add = function (x, y) {
  return x + y;
}
```

上面示例中，函数`myAdd()`的类型与函数`add()`是一样的，那么就可以定义成`typeof add`。因为函数名`add`本身不是类型，而是一个值，所以要用`typeof`运算符返回它的类型。

这是一个很有用的技巧，任何需要类型的地方，都可以使用`typeof`运算符从一个值获取类型。

函数类型还可以采用对象的写法。

```typescript
let add:{
  (x:number, y:number):number
};
 
add = function (x, y) {
  return x + y;
};
```

上面示例中，变量`add`的类型就写成了一个对象。

函数类型的对象写法如下。

```typescript
{
  (参数列表): 返回值
}
```

注意，这种写法的函数参数与返回值之间，间隔符是冒号`:`，而不是正常写法的箭头`=>`，因为这里采用的是对象类型的写法，对象的属性名与属性值之间使用的是冒号。

这种写法平时很少用，但是非常合适用在一个场合：函数本身存在属性。

```typescript
function f(x:number) {
  console.log(x);
}

f.version = '1.0';
```

上面示例中，函数`f()`本身还有一个属性`version`。这时，`f`完全就是一个对象，类型就要使用对象的写法。

```typescript
let foo: {
  (x:number): void;
  version: string
} = f;
```

函数类型也可以使用 Interface 来声明，这种写法就是对象写法的翻版，详见《Interface》一章。

```typescript
interface myfn {
  (a:number, b:number): number;
}

var add:myfn = (a, b) => a + b;
```

上面示例中，interface 命令定义了接口`myfn`，这个接口的类型就是一个用对象表示的函数。

## Function 类型

TypeScript 提供 Function 类型表示函数，任何函数都属于这个类型。

```typescript
function doSomething(f:Function) {
  return f(1, 2, 3);
}
```

上面示例中，参数`f`的类型就是`Function`，代表这是一个函数。

Function 类型的值都可以直接执行。

Function 类型的函数可以接受任意数量的参数，每个参数的类型都是`any`，返回值的类型也是`any`，代表没有任何约束，所以不建议使用这个类型，给出函数详细的类型声明会更好。

## 箭头函数

箭头函数是普通函数的一种简化写法，它的类型写法与普通函数类似。

```typescript
const repeat = (
  str:string,
  times:number
):string => str.repeat(times);
```

上面示例中，变量`repeat`被赋值为一个箭头函数，类型声明写在箭头函数的定义里面。其中，参数的类型写在参数名后面，返回值类型写在参数列表的圆括号后面。

注意，类型写在箭头函数的定义里面，与使用箭头函数表示函数类型，写法有所不同。

```typescript
function greet(
  fn:(a:string) => void
):void {
  fn('world');
}
```

上面示例中，函数`greet()`的参数`fn`是一个函数，类型就用箭头函数表示。这时，`fn`的返回值类型要写在箭头右侧，而不是写在参数列表的圆括号后面。

下面再看一个例子。

```typescript
type Person = { name: string };

const people = ['alice', 'bob', 'jan'].map(
  (name):Person => ({name})
);
```

上面示例中，`Person`是一个类型别名，代表一个对象，该对象有属性`name`。变量`people`是数组的`map()`方法的返回值。

`map()`方法的参数是一个箭头函数`(name):Person => ({name})`，该箭头函数的参数`name`的类型省略了，因为可以从`map()`的类型定义推断出来，箭头函数的返回值类型为`Person`。相应地，变量`people`的类型是`Person[]`。

至于箭头后面的`({name})`，表示返回一个对象，该对象有一个属性`name`，它的属性值为变量`name`的值。这里的圆括号是必须的，否则`(name):Person => {name}`的大括号表示函数体，即函数体内有一行语句`name`，同时由于没有`return`语句，这个函数不会返回任何值。

注意，下面两种写法都是不对的。

```typescript
// 错误
(name:Person) => ({name})

// 错误
name:Person => ({name})
```

上面的两种写法在本例中都是错的。第一种写法表示，箭头函数的参数`name`的类型是`Person`，同时没写函数返回值的类型，让 TypeScript 自己去推断。第二种写法中，函数参数缺少圆括号。

## 可选参数

如果函数的某个参数可以省略，则在参数名后面加问号表示。

```typescript
function f(x?:number) {
  // ...
}

f(); // OK
f(10); // OK
```

上面示例中，虽然参数`x`后面有问号，表示该参数可以省略。

参数名带有问号，表示该参数的类型实际上是`原始类型|undefined`，它有可能为`undefined`。比如，上例的`x`虽然类型声明为`number`，但是实际上是`number|undefined`。

```typescript
function f(x?:number) {
  return x;
}

f(undefined) // 正确
```

上面示例中，参数`x`是可选的，等同于说`x`可以赋值为`undefined`。

但是，反过来就不成立，类型显式设为`undefined`的参数，就不能省略。

```typescript
function f(x:number|undefined) {
  return x;
}

f() // 报错
```

上面示例中，参数`x`的类型是`number|undefined`，表示要么传入一个数值，要么传入`undefined`，如果省略这个参数，就会报错。

函数的可选参数只能在参数列表的尾部，跟在必选参数的后面。

```typescript
let myFunc:
  (a?:number, b:number) => number; // 报错
```

上面示例中，可选参数在必选参数前面，就报错了。

如果前部参数有可能为空，这时只能显式注明该参数类型可能为`undefined`。

```typescript
let myFunc:
  (
    a:number|undefined,
    b:number
  ) => number;
```

上面示例中，参数`a`有可能为空，就只能显式注明类型包括`undefined`，传参时也要显式传入`undefined`。

函数体内部用到可选参数时，需要判断该参数是否为`undefined`。

```typescript
let myFunc:
  (a:number, b?:number) => number; 

myFunc = function (x, y) {
  if (y === undefined) {
    return x;
  }
  return x + y;
}
```

上面示例中，由于函数的第二个参数为可选参数，所以函数体内部需要判断一下，该参数是否为空。

## 参数默认值

TypeScript 函数的参数默认值写法，与 JavaScript 一致。

设置了默认值的参数，就是可选的。如果不传入该参数，它就会等于默认值。

```typescript
function createPoint(
  x:number = 0,
  y:number = 0
):[number, number] {
  return [x, y];
}

createPoint() // [0, 0]
```

上面示例中，参数`x`和`y`的默认值都是`0`，调用`createPoint()`时，这两个参数都是可以省略的。这里其实可以省略`x`和`y`的类型声明，因为可以从默认值推断出来。

```typescript
function createPoint(
  x = 0, y = 0
) {
  return [x, y];
}
```

可选参数与默认值不能同时使用。

```typescript
// 报错
function f(x?: number = 0) {
  // ...
}
```

上面示例中，`x`是可选参数，还设置了默认值，结果就报错了。

设有默认值的参数，如果传入`undefined`，也会触发默认值。

```typescript
function f(x = 456) {
  return x;
}

f2(undefined) // 456
```

具有默认值的参数如果不位于参数列表的末尾，调用时不能省略，如果要触发默认值，必须显式传入`undefined`。

```typescript
function add(
  x:number = 0,
  y:number
) {
  return x + y;
}

add(1) // 报错
add(undefined, 1) // 正确
```

## 参数解构

函数参数如果存在变量解构，类型写法如下。

```typescript
function f(
  [x, y]: [number, number]
) {
  // ...
}

function sum(
  { a, b, c }: {
     a: number;
     b: number;
     c: number
  }
) {
  console.log(a + b + c);
}
```

参数结构可以结合类型别名（type 命令）一起使用，代码会看起来简洁一些。

```typescript
type ABC = { a:number; b:number; c:number };

function sum({ a, b, c }:ABC) {
  console.log(a + b + c);
}
```

## rest 参数

rest 参数表示函数剩余的所有参数，它可以是数组（剩余参数类型相同），也可能是元组（剩余参数类型不同）。

```typescript
// rest 参数为数组
function joinNumbers(...nums:number[]) {
  // ...
}

// rest 参数为元组
function f(...args:[boolean, number]) {
  // ...
}
```

注意，元组需要声明每一个剩余参数的类型。如果元组里面的参数是可选的，则要使用可选参数。

```typescript
function f(
  ...args: [boolean, string?]
) {}
```

下面是一个 rest 参数的例子。

```typescript
function multiply(n:number, ...m:number[]) {
  return m.map((x) => n * x);
}
```

上面示例中，参数`m`就是 rest 类型，它的类型是一个数组。

rest 参数甚至可以嵌套。

```typescript
function f(...args:[boolean, ...string[]]) {
  // ...
}
```

rest 参数可以与变量解构结合使用。

```typescript
function repeat(
  ...[str, times]: [string, number]
):string {
  return str.repeat(times);
}

// 等同于
function repeat(
  str: string,
  times: number
):string {
  return str.repeat(times);
}
```

## readonly 只读参数

如果函数内部不能修改某个参数，可以在函数定义时，在参数类型前面加上`readonly`关键字，表示这是只读参数。

```typescript
function arraySum(
  arr:readonly number[]
) {
  // ...
  arr[0] = 0; // 报错
}
```

上面示例中，参数`arr`的类型是`readonly number[]`，表示为只读参数。如果函数体内部修改这个数组，就会报错。

## void 类型

void 类型表示函数没有返回值。

```typescript
function f():void {
  console.log('hello');
}
```

上面示例中，函数`f`没有返回值，类型就要写成`void`。

如果返回其他值，就会报错。

```typescript
function f():void {
  return 123; // 报错
}
```

上面示例中，函数`f()`的返回值类型是`void`，但是实际返回了一个数值，编译时就报错了。

void 类型允许返回`undefined`或`null`。

```typescript
function f():void {
  return undefined; // 正确
}

function f():void {
  return null; // 正确
}
```

如果打开了`strictNullChecks`编译选项，那么 void 类型只允许返回`undefined`。如果返回`null`，就会报错。这是因为 JavaScript 规定，如果函数没有返回值，就等同于返回`undefined`。

```typescript
// 打开编译选项 strictNullChecks

function f():void {
  return undefined; // 正确
}

function f():void {
  return null; // 报错
}
```

需要特别注意的是，如果变量、对象方法、函数参数的类型是 void 类型的函数，那么并不代表不能赋值为有返回值的函数。恰恰相反，该变量、对象方法和函数参数可以接受返回任意值的函数，这时并不会报错。

```typescript
type voidFunc = () => void;

const f:voidFunc = () => {
  return 123;
};
```

上面示例中，变量`f`的类型是`voidFunc`，是一个没有返回值的函数类型。但是实际上，`f`的值是一个有返回值的函数（返回`123`），编译时不会报错。

这是因为，这时 TypeScript 认为，这里的 void 类型只是表示该函数的返回值没有利用价值，或者说不应该使用该函数的返回值。只要不用到这里的返回值，就不会报错。

这样设计是有现实意义的。举例来说，数组方法`Array.prototype.forEach(fn)`的参数`fn`是一个函数，而且这个函数应该没有返回值，即返回值类型是`void`。

但是，实际应用中，很多时候传入的函数是有返回值，但是它的返回值不重要，或者不产生作用。

```typescript
const src = [1, 2, 3];
const ret = [];

src.forEach(el => ret.push(el));
```

上面示例中，`push()`有返回值，表示新插入的元素在数组里面的位置。但是，对于`forEach()`方法来说，这个返回值是没有作用的，根本用不到，所以 TypeScript 不会报错。

如果后面使用了这个函数的返回值，就违反了约定，则会报错。

```typescript
type voidFunc = () => void;
 
const f:voidFunc = () => {
  return 123;
};

f() * 2 // 报错
```

上面示例中，最后一行报错了，因为根据类型声明，`f()`没有返回值，但是却用到了它的返回值，因此报错了。

注意，这种情况仅限于变量、对象方法和函数参数，函数字面量如果声明了返回值是 void 类型，还是不能有返回值。

```typescript
function f():void {
  return true; // 报错
}
 
const f3 = function ():void {
  return true; // 报错
};
```

上面示例中，函数字面量声明了返回`void`类型，这时只要有返回值（除了`undefined`和`null`）就会报错。

除了函数，其他变量声明为`void`类型没有多大意义，因为这时只能赋值为`undefined`或者`null`（假定没有打开`strictNullChecks`) 。

```typescript
let foo:void = undefined;

// 没有打开 strictNullChecks 的情况下
let bar:void = null;
```

## never 类型

`never`类型表示肯定不会出现的值。它用在函数的返回值，就表示某个函数肯定不会返回值，即函数不会正常执行结束。

它主要有以下两种情况。

（1）抛出错误的函数。

```typescript
function fail(msg:string):never {
  throw new Error(msg);
}
```

上面示例中，函数`fail()`会抛错，不会正常退出，所以返回值类型是`never`。

注意，只有抛出错误，才是 never 类型。如果显式用`return`语句返回一个 Error 对象，返回值就不是 never 类型。

```typescript
function fail():Error {
  return new Error("Something failed");
}
```

上面示例中，函数`fail()`返回一个 Error 对象，所以返回值类型是 Error。

（2）无限执行的函数。

```typescript
const sing = function():never {
  while (true) {
    console.log('sing');
};
```

上面示例中，函数`sing()`会永远执行，不会返回，所以返回值类型是`never`。

注意，`never`类型不同于`void`类型。前者表示函数没有执行结束，不可能有返回值；后者表示函数正常执行结束，但是不返回值，或者说返回`undefined`。

```typescript
// 正确
function sing():void {
  console.log('sing');
}

// 报错
function sing():never {
  console.log('sing');
}
```

上面示例中，函数`sing()`虽然没有`return`语句，但实际上是省略了`return undefined`这行语句，真实的返回值是`undefined`。所以，它的返回值类型要写成`void`，而不是`never`，写成`never`会报错。

如果一个函数抛出了异常或者陷入了死循环，那么该函数无法正常返回一个值，因此该函数的返回值类型就是`never`。如果程序中调用了一个返回值类型为`never`的函数，那么就意味着程序会在该函数的调用位置终止，永远不会继续执行后续的代码。

```typescript
function neverReturns():never {
  throw new Error();
}

function f(
  x:string|undefined
) {
  if (x === undefined) {
    neverReturns();
  }

  x; // 推断为 string
}
```

上面示例中，函数`f()`的参数`x`的类型为`string|undefined`。但是，`x`类型为`undefined`时，调用了`neverReturns()`。这个函数不会返回，因此 TypeScript 可以推断出，判断语句后面的那个`x`，类型一定是`string`。

## 局部类型

函数内部允许声明其他类型，该类型只在函数内部有效，称为局部类型。

```typescript
function hello(txt:string) {
  type message = string;
  let newTxt:message = 'hello ' + txt;
  return newTxt;
}

const newTxt:message = hello('world'); // 报错
```

上面示例中，类型`message`是在函数`hello()`内部定义的，只能在函数内部使用。在函数外部使用，就会报错。

## 高阶函数

一个函数的返回值还是一个函数，那么前一个函数就称为高阶函数（higher-order function）。

下面就是一个例子，箭头函数返回的还是一个箭头函数。

```typescript
(someValue: number) => (multiplier: number) => someValue * multiplier;
```

## 函数重载

有些函数可以接受不同类型或不同个数的参数，并且根据参数的不同，会有不同的函数行为。这种根据参数类型不同，执行不同逻辑的行为，称为函数重载（function overload）。

```javascript
reverse('abc') // 'cba'
reverse([1, 2, 3]) // [3, 2, 1]
```

上面示例中，函数`reverse()`可以将参数颠倒输出。参数可以是字符串，也可以是数组。

这意味着，该函数内部有处理字符串和数组的两套逻辑，根据参数类型的不同，分别执行对应的逻辑。这就叫“函数重载”。

TypeScript 对于“函数重载”的类型声明方法是，逐一定义每一种情况的类型。

```typescript
function reverse(str:string):string;
function reverse(arr:any[]):any[];
```

上面示例中，分别对函数`reverse()`的两种参数情况，给予了类型声明。但是，到这里还没有结束，后面还必须对函数`reverse()`给予完整的类型声明。

```typescript
function reverse(str:string):string;
function reverse(arr:any[]):any[];
function reverse(
  stringOrArray:string|any[]
):string|any[] {
  if (typeof stringOrArray === 'string')
    return stringOrArray.split('').reverse().join('');
  else
    return stringOrArray.slice().reverse();
}
```

上面示例中，前两行类型声明列举了重载的各种情况。第三行是函数本身的类型声明，它必须与前面已有的重载声明兼容。

有一些编程语言允许不同的函数参数，对应不同的函数实现。但是，JavaScript 函数只能有一个实现，必须在这个实现当中，处理不同的参数。因此，函数体内部就需要判断参数的类型及个数，并根据判断结果执行不同的操作。

```typescript
function add(
  x:number,
  y:number
):number;
function add(
  x:any[],
  y:any[]
):any[];
function add(
  x:number|any[],
  y:number|any[]
):number|any[] {
  if (typeof x === 'number' && typeof y === 'number') {
    return x + y;
  } else if (Array.isArray(x) && Array.isArray(y)) {
    return [...x, ...y];
  }

  throw new Error('wrong parameters');
}
```

上面示例中，函数`add()`内部使用`if`代码块，分别处理参数的两种情况。

注意，重载的个别类型描述与函数的具体实现之间，不能有其他代码，否则报错。

另外，虽然函数的具体实现里面，有完整的类型声明。但是，函数实际调用的类型，以前面的类型声明为准。比如，上例的函数实现，参数类型和返回值类型都是`number|any[]`，但不意味着参数类型为`number`时返回值类型为`any[]`。

函数重载的每个类型声明之间，以及类型声明与函数实现的类型之间，不能有冲突。

```typescript
// 报错
function fn(x:boolean):void;
function fn(x:string):void;
function fn(x:number|string) {
  console.log(x);
}
```

上面示例中，函数重载的类型声明与函数实现是冲突的，导致报错。

重载声明的排序很重要，因为 TypeScript 是按照顺序进行检查的，一旦发现符合某个类型声明，就不再往下检查了，所以类型最宽的声明应该放在最后面，防止覆盖其他类型声明。

```typescript
function f(x:any):number;
function f(x:string): 0|1;
function f(x:any):any {
  // ...
}

const a:0|1 = f('hi'); // 报错
```

上面声明中，第一行类型声明`x:any`范围最宽，导致函数`f()`的调用都会匹配这行声明，无法匹配第二行类型声明，所以最后一行调用就报错了，因为等号两侧类型不匹配，左侧类型是`0|1`，右侧类型是`number`。这个函数重载的正确顺序是，第二行类型声明放到第一行的位置。

对象的方法也可以使用重载。

```typescript
class StringBuilder {
  #data = '';

  add(num:number): this;
  add(bool:boolean): this;
  add(str:string): this;
  add(value:any): this {
    this.#data += String(value);
    return this;
  }

  toString() {
    return this.#data;
  }
}
```

上面示例中，方法`add()`也使用了函数重载。

函数重载也可以用来精确描述函数参数与返回值之间的对应关系。

```typescript
function createElement(
  tag:'a'
):HTMLAnchorElement;
function createElement(
  tag:'canvas'
):HTMLCanvasElement;
function createElement(
  tag:'table'
):HTMLTableElement;
function createElement(
  tag:string
):HTMLElement {
  // ...
}
```

上面示例中，函数重载精确描述了参数`tag`的三个值，所对应的不同的函数返回值。

这个示例的函数重载，也可以用对象表示。

```typescript
type CreateElement = {
  (tag:'a'): HTMLAnchorElement;
  (tag:'canvas'): HTMLCanvasElement;
  (tag:'table'): HTMLTableElement;
  (tag:string): HTMLElement;
}
```

由于重载是一种比较复杂的类型声明方法，为了降低复杂性，一般来说，如果可以的话，应该优先使用联合类型替代函数重载。

```typescript
// 写法一
function len(s:string):number;
function len(arr:any[]):number;
function len(x:any):number {
  return x.length;
}

// 写法二
function len(x:any[]|string):number {
  return x.length;
}
```

上面示例中，写法二使用联合类型，要比写法一的函数重载简单很多。

## 构造函数

JavaScript 语言使用构造函数，生成对象的实例。

构造函数的最大特点，就是必须使用`new`命令调用。

```typescript
const d = new Date();
```

上面示例中，`date()`就是一个构造函数，使用`new`命令调用，返回 Date 对象的实例。

构造函数的类型写法，就是在参数列表前面加上`new`命令。

```typescript
class Animal {
  numLegs:number = 4;
}

type AnimalConstructor = new () => Animal;

function create(c:AnimalConstructor):Animal {
  return new c();
}

const a = create(Animal);
```

上面示例中，类型`AnimalConstructor`就是一个构造函数，而函数`create()`需要传入一个构造函数。在 JavaScript 中，类（class）本质上是构造函数，所以`Animal`这个类可以传入`create()`。

构造函数还有另一种类型写法，就是采用对象形式。

```typescript
type F = {
  new (s:string): object;
};
```

上面示例中，类型 F 就是一个构造函数。类型写成一个可执行对象的形式，并且在参数列表前面要加上`new`命令。

某些函数既是构造函数，又可以当作普通函数使用，比如`Date()`。这时，类型声明可以写成下面这样。

```typescript
type F = {
  new (s:string): object;
  (n?:number): number;
}
```

上面示例中，F 既可以当作普通函数执行，也可以当作构造函数使用。

