# 类型运算

## 运算律

改变成员类型的顺序不影响联合类型的结果类型。

```typescript
type T0 = string | number;
type T1 = number | string;
```

对部分类型成员使用分组运算符不影响联合类型的结果类型。

```typescript
type T0 = (boolean | string) | number;
type T1 = boolean | (string | number);
```

联合类型的成员类型可以进行化简。假设有联合类型“U = T0 | T1”，如果T1是T0的子类型，那么可以将类型成员T1从联合类型U中消去。最后，联合类型U的结果类型为“U = T0”。例如，有联合类型“boolean | true | false”。其中，true类型和false类型是boolean类型的子类型，因此可以将true类型和false类型从联合类型中消去。最终，联合类型“boolean | true | false”的结果类型为boolean类型。

```typescript
type T0 = boolean | true | false;

// 所以T0等同于 T1
type T1 = boolean;
```

### 优先级


`&`的优先级高于`|`。

```typescript
A & B | C & D
// 该类型等同于如下类型：
(A & B) | (C & D)
```

分配律

```typescript
A & (B | C) 
// 等同于
(A & B) | (A & C)
```

一个稍微复杂的类型等式。

```typescript
(A | B) & (C | D) ≡ A & C | A & D | B & C | B & D
```

```typescript
T = (string | 0) & (number | 'a');
T = (string & number) | (string & 'a') | (0 & number) | (0 & 'a');

T = never | 'a' | 0 | never;
T = 'a' | 0;
```


```typescript
function extend<T extends object, U extends object>(first: T, second: U): T & U {
  const result = <T & U>{};
  for (let id in first) {
    (<T>result)[id] = first[id];
  }
  for (let id in second) {
    if (!result.hasOwnProperty(id)) {
      (<U>result)[id] = second[id];
    }
  }

  return result;
}

const x = extend({ a: 'hello' }, { b: 42 });
```

## never 类型

never 可以视为空集。

```typescript
type NeverIntersection = never & string; // Type: never
type NeverUnion = never | string; // Type: string
```

很适合在交叉类型中用作过滤。

```typescript
type OnlyStrings<T> = T extends string ? T : never;
type RedOrBlue = OnlyStrings<"red" | "blue" | 0 | false>;
// Equivalent to: "red" | "blue"
```

范例：https://www.typescriptlang.org/play#example/conditional-types

## unknown 类型

在联合类型中，unknown吸收所有类型。这意味着如果任何组成类型是unknown，则联合类型的计算结果为unknown。

```typescript
// In an intersection everything absorbs unknown
type T00 = unknown & null; // null
type T01 = unknown & undefined; // undefined
type T02 = unknown & null & undefined; // null & undefined (which becomes never)
type T03 = unknown & string; // string
type T04 = unknown & string[]; // string[]
type T05 = unknown & unknown; // unknown
type T06 = unknown & any; // any
// In a union an unknown absorbs everything
type T10 = unknown | null; // unknown
type T11 = unknown | undefined; // unknown
type T12 = unknown | null | undefined; // unknown
type T13 = unknown | string; // unknown
type T14 = unknown | string[]; // unknown
type T15 = unknown | unknown; // unknown
type T16 = unknown | any; // any
// Type variable and unknown in union and intersection
type T20<T> = T & {}; // T & {}
type T21<T> = T | {}; // T | {}
type T22<T> = T & unknown; // T
type T23<T> = T | unknown; // unknown
// unknown in conditional types
type T30<T> = unknown extends T ? true : false; // Deferred
type T31<T> = T extends unknown ? true : false; // Deferred (so it distributes)
type T32<T> = never extends T ? true : false; // true
type T33<T> = T extends never ? true : false; // Deferred
```


```typescript
type UnionType1 = unknown | null; // unknown
type UnionType2 = unknown | undefined; // unknown
type UnionType3 = unknown | string; // unknown
type UnionType4 = unknown | number[]; // unknown
```

该规则的一个例外是any。如果至少有一种构成类型是any，则联合类型的计算结果为any：

```typescript
type UnionType5 = unknown | any; // any
```

在交叉类型中，每种类型都吸收unknown. 这意味着与任何类型相交unknown不会改变结果类型：

```typescript
type IntersectionType1 = unknown & null; // null
type IntersectionType2 = unknown & undefined; // undefined
type IntersectionType3 = unknown & string; // string
type IntersectionType4 = unknown & number[]; // number[]
type IntersectionType5 = unknown & any; // any
```

除非使用`as`断言，首先缩小类型`unknown`类型的范围，然后才可以用于其他类型。

```typescript
const value: unknown = "Hello World";
const someString: string = value as string;
const otherString = someString.toUpperCase(); // "HELLO WORLD"
```

## 联合类型

如果类型是多个值的联合，甚至可以产生插值的效果。

```typescript
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

// 等同于 type AllLocaleIDs = "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
```

## 交叉类型

```typescript
type Brightness = "dark" | "light";
type Color = "blue" | "red";
type BrightnessAndColor = `${Brightness}-${Color}`;
// Equivalent to: "dark-red" | "light-red" | "dark-blue" | "light-blue"
```

如果交叉类型中存在多个相同的成员类型，那么相同的成员类型将被合并为单一成员类型。

```typescript
type T0 = boolean;
type T1 = boolean & boolean;
type T2 = boolean & boolean & boolean;
```

上面示例中，T0、T1和T2都表示同一种类型boolean。

改变成员类型的顺序不影响交叉类型的结果类型。

```typescript
interface Clickable {
    click(): void;
}
interface Focusable {
    focus(): void;
}

type T0 = Clickable & Focusable;
type T1 = Focusable & Clickable;
```
注意，当交叉类型涉及调用签名重载或构造签名重载时便失去了“加法交换律”的性质。因为交叉类型中成员类型的顺序将决定重载签名的顺序，进而将影响重载签名的解析顺序。

```typescript
interface Clickable {
    register(x: any): void;
}
interface Focusable {
    register(x: string): boolean;
}

type ClickableAndFocusable = Clickable & Focusable;
type FocusableAndFocusable = Focusable & Clickable;

function foo(
    clickFocus: ClickableAndFocusable,
    focusClick: FocusableAndFocusable
) {
    let a: void = clickFocus.register('foo');
    let b: boolean = focusClick.register('foo');
}
```

此例第8行和第9行使用不同的成员类型顺序定义了两个交叉类型。第15行，调用“register()”方法的返回值类型为void，说明在ClickableAndFocusable类型中，Clickable接口中定义的“register()”方法具有更高的优先级。第16行，调用“register()”方法的返回值类型为boolean，说明FocusableAndFocusable类型中Focusable接口中定义的“register()”方法具有更高的优先级。此例也说明了调用签名重载的顺序与交叉类型中成员类型的定义顺序是一致的。

对部分类型成员使用分组运算符不影响交叉类型的结果类型。

```typescript
interface Clickable {
  click(): void;
}
interface Focusable {
  focus(): void;
}
interface Scrollable {
  scroll(): void;
}

type T0 = (Clickable & Focusable) & Scrollable;
type T1 = Clickable & (Focusable & Scrollable);
```

上面示例的T0和T1类型是同一种类型。

```typescript
type Combined = { a: number } & { b: string };
type Conflicting = { a: number } & { a: string };
```

只要交叉类型I中任意一个成员类型包含了属性签名M，那么交叉类型I也包含属性签名M。

```typescript
interface A {
    a: boolean;
}

interface B {
    b: string;
}

// 交叉类型如下
{
    a: boolean;
    b: string;
}
```

若交叉类型的属性签名M在所有成员类型中都是可选属性，那么该属性签名在交叉类型中也是可选属性。否则，属性签名M是一个必选属性。

```typescript
interface A {
    x: boolean;
    y?: string;
}
interface B {
    x?: boolean;
    y?: string;
}

// 交叉类型如下
{
    x: boolean;
    y?: string;
}
```

