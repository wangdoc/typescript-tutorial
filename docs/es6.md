# TypeScript 的 ES6 类型

## `Map<K, V>`

```typescript
let map2 = new Map(); // Key any, value any
let map3 = new Map<string, number>(); // Key string, value number
```

TypeScript 使用 Map 类型，描述 Map 结构。

```typescript
const myMap: Map<boolean,string> = new Map([
  [false, 'no'],
  [true, 'yes'],
]);
```

Map 是一个泛型，使用时，比如给出类型变量。

由于存在类型推断，也可以省略类型参数。

```typescript
const myMap = new Map([
  [false, 'no'],
  [true, 'yes'],
]);
```

## `Set<T>`

## `Promise<T>`

## async 函数

async 函数的的返回值是一个 Promise 对象。

```typescript
const p:Promise<number> = /* ... */;

async function fn(): Promise<number> {
  var i = await p;
  return i + 1;
}
```

## `Iterable<>`

对象只要部署了 Iterator 接口，就可以用`for...of`循环遍历。Generator 函数（生成器）返回的就是一个具有 Iterator 接口的对象。

TypeScript 使用泛型`Iterable<T>`表示具有 Iterator 接口的对象，其中`T`表示 Iterator 接口包含的值类型（每一轮遍历获得的值）。

```typescript
interface Iterable<T> {
  [Symbol.iterator](): Iterator<T>;
}
```

上面是`Iterable<T>`接口的定义，表示一个具有`Symbol.iterator`属性的对象，该属性是一个函数，调用后返回的是一个 Iterator 对象。

Iterator 对象必须具有`next()`方法，另外还具有两个可选方法`return()`和`throw()`，类型表述如下。

```typescript
interface Iterator<T> {
  next(value?: any): IteratorResult<T>;
  return?(value?: any): IteratorResult<T>;
  throw?(e?: any): IteratorResult<T>;
}
```

上面的类型定义中，可以看到`next()`、`return()`、`throw()`这三个方法的返回值是一个部署了`IteratorResult<T>`接口的对象。

`IteratorResult<T>`接口的定义如下。

```typescript
interface IteratorResult<T> {
  done: boolean; //表示遍历是否结束
  value: T; // 当前遍历得到的值
}
```

上面的类型定义表示，Iterator 对象的`next()`等方法的返回值，具有`done`和`value`两个属性。

下面的例子是 Generator 函数返回一个具有 Iterator 接口的对象。

```typescript
function* g():Iterable<string> {
  for (var i = 0; i < 100; i++) {
    yield '';
  }
  yield* otherStringGenerator();
}
```

上面示例中，生成器`g()`返回的类型是`Iterable<string>`，其中`string`表示 Iterator 接口包含的是字符串。

这个例子的类型声明可以省略，因为 TypeScript 可以自己推断出来 Iterator 接口的类型。

```typescript
function* g() {
  for (var i = 0; i < 100; i++) {
    yield ""; // infer string
  }
  yield* otherStringGenerator();
}
```

另外，扩展运算符（`...`）后面的值必须具有 Iterator 接口，下面是一个例子。

```typescript
function toArray<X>(xs: Iterable<X>):X[] {
  return [...xs]
}
```

## Generator 函数

Generator 函数返回一个同时具有 Iterable 接口（具有`[Symbol.iterator]`属性）和 Iterator 接口（具有`next()`方法）的对象，因此 TypeScript 提供了一个泛型`IterableIterator<T>`，表示同时满足`Iterable<T>`和`Iterator<T>`两个接口。

```typescript
interface IterableIterator<T> extends Iterator<T> {
    [Symbol.iterator](): IterableIterator<T>;
}
```

上面类型定义中，`IterableIterator<T>`接口就是在`Iterator<T>`接口的基础上，加上`[Symbol.iterator]`属性。

下面是一个例子。

```typescript
function* createNumbers(): IterableIterator<number> {
  let n = 0;
  while (1) {
    yield n++;
  }
}

let numbers = createNumbers()

// {value: 0, done: false}
numbers.next()

// {value: 1, done: false}
numbers.next()

// {value: 2, done: false}
numbers.next()
```

上面示例中，`createNumbers()`返回的对象`numbers`即具有`next()`方法，也具有`[Symbol.iterator]`属性，所以满足`IterableIterator<T>`接口。

## 参考链接

- [Typing Iterables and Iterators with TypeScript](https://www.geekabyte.io/2019/06/typing-iterables-and-iterators-with.html)