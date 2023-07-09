# 条件类型

```typescript
T extends U ? X : Y
```

上面定义中，T、U、X、Y 代表任意类型。`T extends U`表示类型的测试条件，如果满足此条件，返回类型`X`，否则返回类型`Y`。

下面是一个例子。

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;
```

上面式子定义了一个范型`NonNullable`，用来检测某个类型是否非空。

```typescript
type EmailAddress = string | string[] | null | undefined;

// 等同于 type NonNullableEmailAddress = string | string[];
type NonNullableEmailAddress = NonNullable<EmailAddress>;
```

TypeScript 提供了一些预定义的条件类型，下面逐一介绍。

## `NonNullable<T>`

`NonNullable<T>`从类型`T`里面过滤掉`null`和`undefined`。

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;
```

```typescript
type A = NonNullable<boolean>; // boolean
type B = NonNullable<number | null>; // number
type C = NonNullable<string | undefined>; // string
type D = NonNullable<null | undefined>; // never
```

## `Extract<T, U>`

```typescript
type Extract<T, U> = T extends U ? T : never;
```

`Extract<T, U>`类型表达式相当于提取功能，只要`T`符合`U`就返回`T`，否则就过滤掉。

```typescript
type A = Extract<string | string[], any[]>; // string[]
type B = Extract<(() => void) | null, Function>; // () => void
type C = Extract<200 | 400, 200 | 201>; // 200
type D = Extract<number, boolean>; // never
```

## `Exclude<T, U>`

`Exclude<T, U>`相当于排除功能，只要`T`符合`U`就过滤掉，否则返回`T`。

```typescript
type Exclude<T, U> = T extends U ? never : T;
```

```typescript
type A = Exclude<string | string[], any[]>; // string
type B = Exclude<(() => void) | null, Function>; // null
type C = Exclude<200 | 400, 200 | 201>; // 400
type D = Exclude<number, boolean>; // number
```

## `ReturnType<T>`

`ReturnType<T>`提取函数的返回类型。

```typescript
type ReturnType<T extends (...args: any[]) => any> = T extends (
  ...args: any[]
) => infer R
  ? R
  : any;
```

```typescript
type A = ReturnType<() => string>; // string
type B = ReturnType<() => () => any[]>; // () => any[]
type C = ReturnType<typeof Math.random>; // number
type D = ReturnType<typeof Array.isArray>; // boolean
```

## `Parameters<T>`

`Parameters<T>`提供函数`T`的所有参数类型，它的返回值是一个`tuple`类型，或者`never`(如果 T 不是函数)。

```typescript
type Parameters<T extends (...args: any[]) => any> = T extends (
  ...args: infer P
) => any
  ? P
  : never;
```

```typescript
type A = Parameters<() => void>; // []
type B = Parameters<typeof Array.isArray>; // [any]
type C = Parameters<typeof parseInt>; // [string, (number | undefined)?]
type D = Parameters<typeof Math.max>; // number[]
```

`Array.isArray()`只有一个参数，所以返回的类型是`[any]`，而不是`any[]`。`Math.max()`的参数是任意多个数值，而不是一个数值数组，所以返回的类型是`number[]`，而不是`[number[]]`。

## `ConstructorParameters<T>`

`ConstructorParameters<T>`提取一个构造函数的所有参数类型。它的返回值是一个 tuple 类型，成员是所有参数的类型，如果 T 不是函数，则返回 never。

```typescript
type ConstructorParameters<
  T extends new (...args: any[]) => any
> = T extends new (...args: infer P) => any ? P : never;
```

```typescript
type A = ConstructorParameters<ErrorConstructor>;
// [(string | undefined)?]

type B = ConstructorParameters<FunctionConstructor>;
// string[]

type C = ConstructorParameters<RegExpConstructor>;
// [string, (string | undefined)?]
```

## `InstanceType<T>`

`InstanceType<T>`提取构造函数的返回值的类型，等同于构造函数的`ReturnType<T>`。

```typescript
type InstanceType<T extends new (...args: any[]) => any> = T extends new (
  ...args: any[]
) => infer R
  ? R
  : any;
```

```typescript
type A = InstanceType<ErrorConstructor>; // Error
type B = InstanceType<FunctionConstructor>; // Function
type C = InstanceType<RegExpConstructor>; // RegExp
```
