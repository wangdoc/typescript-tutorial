# TypeScript 类型缩小

TypeScript 变量的值可以变，但是类型通常是不变的。唯一允许的改变，就是类型缩小，就是将变量值的范围缩得更小。

## 手动类型缩小

如果一个变量属于联合类型，所以使用时一般需要缩小类型。

第一种方法是使用`if`判断。

```typescript
function getScore(value: number|string): number {
  if (typeof value === 'number') { // (A)
    // %inferred-type: number
    value;
    return value;
  }
  if (typeof value === 'string') { // (B)
    // %inferred-type: string
    value;
    return value.length;
  }
  throw new Error('Unsupported value: ' + value);
}
```

如果一个值是`any`或`unknown`，你又想对它进行处理，就必须先缩小类型。

```typescript
function parseStringLiteral(stringLiteral: string): string {
  const result: unknown = JSON.parse(stringLiteral);
  if (typeof result === 'string') { // (A)
    return result;
  }
  throw new Error('Not a string literal: ' + stringLiteral);
}
```

下面是另一个例子。

```typescript
interface Book {
  title: null | string;
  isbn: string;
}

function getTitle(book: Book) {
  if (book.title === null) {
    // %inferred-type: null
    book.title;
    return '(Untitled)';
  } else {
    // %inferred-type: string
    book.title;
    return book.title;
  }
}
```

缩小类型的前提是，需要先获取类型。获取类型的几种方法如下。

```typescript
function func(value: Function|Date|number[]) {
  if (typeof value === 'function') {
    // %inferred-type: Function
    value;
  }

  if (value instanceof Date) {
    // %inferred-type: Date
    value;
  }

  if (Array.isArray(value)) {
    // %inferred-type: number[]
    value;
  }
}
```

### typeof 运算符

第二种方法是使用`switch`缩小类型。

```typescript
function getScore(value: number|string): number {
  switch (typeof value) {
    case 'number':
      // %inferred-type: number
      value;
      return value;
    case 'string':
      // %inferred-type: string
      value;
      return value.length;
    default:
      throw new Error('Unsupported value: ' + value);
  }
}
```

### instanceof 运算符

第三种方法是instanceof运算符。它能够检测实例对象与构造函数之间的关系。instanceof运算符的左操作数为实例对象，右操作数为构造函数，若构造函数的prototype属性值存在于实例对象的原型链上，则返回true；否则，返回false。

```typescript
function f(x: Date | RegExp) {
    if (x instanceof Date) {
        x; // Date
    }

    if (x instanceof RegExp) {
        x; // RegExp
    }
}
```

instanceof类型守卫同样适用于自定义构造函数，并对其实例对象进行类型细化。

```typescript
class A {}
class B {}

function f(x: A | B) {
   if (x instanceof A) {
       x; // A
   }

   if (x instanceof B) {
       x; // B
  }
}
```

### in 运算符

第四种方法是使用in运算符。

in运算符是JavaScript中的关系运算符之一，用来判断对象自身或其原型链中是否存在给定的属性，若存在则返回true，否则返回false。in运算符有两个操作数，左操作数为待测试的属性名，右操作数为测试对象。

in类型守卫根据in运算符的测试结果，将右操作数的类型细化为具体的对象类型。

```typescript
interface A {
    x: number;
}
interface B {
    y: string;
}

function f(x: A | B) {
    if ('x' in x) {
        x; // A
    } else {
        x; // B
    }
}
```

```typescript
interface A { a: number }
interface B { b: number }
function pickAB(ab: A | B) {
 if ('a' in ab) {
 ab // Type is A
 } else {
 ab // Type is B
 }
 ab // Type is A | B
}
```

缩小对象的属性，要用`in`运算符。

```typescript
type FirstOrSecond =
  | {first: string}
  | {second: string};

function func(firstOrSecond: FirstOrSecond) {
  if ('second' in firstOrSecond) {
    // %inferred-type: { second: string; }
    firstOrSecond;
  }
}

// 错误
function func(firstOrSecond: FirstOrSecond) {
  // @ts-expect-error: Property 'second' does not exist on
  // type 'FirstOrSecond'. [...]
  if (firstOrSecond.second !== undefined) {
    // ···
  }
}
```

`in`运算符只能用于联合类型，不能用于检查一个属性是否存在。

```typescript
function func(obj: object) {
  if ('name' in obj) {
    // %inferred-type: object
    obj;

    // 报错
    obj.name;
  }
}
```

### 特征属性

对于不同对象之间的区分，还可以人为地为每一类对象设置一个特征属性。

```typescript
interface UploadEvent {
    type: 'upload';
    filename: string;
    contents: string 
}
interface DownloadEvent { type: 'download'; filename: string; }
type AppEvent = UploadEvent | DownloadEvent;

function handleEvent(e: AppEvent) {
 switch (e.type) {
 case 'download':
 e // Type is DownloadEvent
 break;
 case 'upload':
 e; // Type is UploadEvent
 break;
 }
}
```

## any 类型的细化

TypeScript 推断变量类型时，会根据获知的信息，不断改变推断出来的类型，越来越细化。这种现象在`any`身上特别明显。

```typescript
function range(
  start:number,
  limit:number
) {
  const out = []; // 类型为 any[]
  for (let i = start; i < limit; i++) {
    out.push(i);
  }
  return out; // 类型为 number[]
}
```  

上面示例中，变量`out`的类型一开始推断为`any[]`，后来在里面放入数值，类型就变为`number[]`。

再看下面的例子。

```typescript
const result = []; // 类型为 any[]
result.push('a');
result // 类型为 string[]
result.push(1);
result // 类型为 (string | number)[]
```

上面示例中，数组`result`随着成员类型的不同，而不断改变自己的类型。

注意，这种`any`类型的细化，只在打开了编译选项`noImplicitAny`时发生。

这时，如果在变量的推断类型还为`any`时（即没有任何写操作)，就去输出（或读取）该变量，则会报错，因为这时推断还没有完成，无法满足`noImplicitAny`的要求。

```typescript
const result = []; // 类型为 any[]
console.log(typeof result); // 报错
result.push('a'); // 类型为 string[]
```

上面示例中，只有运行完第三行，`result`的类型才能完成第一次推断，所以第二行读取`result`就会报错。

## is 运算符

`is`运算符返回一个布尔值，用来判断左侧的值是否属于右侧的类型。

```typescript
function isInputElement(el: HTMLElement): el is HTMLInputElement {
  return 'value' in el;
}

function getElementContent(el: HTMLElement) {
  if (isInputElement(el)) {
 el; // Type is HTMLInputElement
    return el.value;
 }
 el; // Type is HTMLElement
 return el.textContent;
}
```

```typescript
function isDefined<T>(x: T | undefined): x is T {
 return x !== undefined;
}
```

