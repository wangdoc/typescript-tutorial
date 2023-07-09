# global.d.ts

`global.d.ts`文件用来放置全局的 interfaces 和类型。一旦设置，该项目的所有文件，都可以读取这些类型。


在这个文件里面可以全局申明模块。

```typescript
// global.d.ts
declare module 'foo' {
  // Some variable declarations
  export var bar: number; /*sample*/
}
```

然后可以用模块名输入。

```typescript
// anyOtherTsFileInYourProject.ts
import * as foo from 'foo';
// TypeScript assumes (without doing any lookup) that
// foo is {bar:number}
```

该文件的另一个用途是声明一些编译时常量。

```typescript
declare const BUILD_MODE_PRODUCTION: boolean; // can be used for conditional compiling
declare const BUILD_VERSION: string;
```