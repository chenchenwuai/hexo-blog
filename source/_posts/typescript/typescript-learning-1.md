---
title: TypeScript学习(1)-基础类型
date: 2020-07-10 08:36:22
tags: TypeScript
categories: TypeScript
---
> 学习资源为[TypeScript中文网](https://www.tslang.cn/docs/home.html)

为了方便，无特殊说明，以下的`js`表示 JavaScript，`ts`表示TypeScript
<!--more-->
### 基础类型

基础类型包括 `boolean`,`number`,`string`,`数组`,`元组`,`enum`,`any`,`void`,`null`,`undefined`.`never`,`object`.
使用方法
```typescript typescript
  // 变量:数据类型
  let a: number = 1 // 数字
  let b: string = 'string' // 字符串
  let c: number[] = [1,2,3] // 数组
  let d: [number,string] = [1,'hello world'] // 元组
```

#### 布尔值 boolean
```typescript
  let isTrue: boolean = true // 只能true、false两个值
```

#### 数字 number
ts始于js，所以ts中的所有数字都是浮点数，类型为`number`。除了支持十进制和十六进制字面量，ts还支持es6中引入的二进制和八进制字面量
```typescript
  let decLiteral: number = 6; // 十进制
  let hexLiteral: number = 0xf00d; // 十六进制 0x 数字0+小写字母x
  let binaryLiteral: number = 0b1010; // 二进制 0b
  let octalLiteral: number = 0o744; // 八进制 0o 字母o
```

#### 字符串 string
```typescript
  let nickname: string = 'chenuai'
  let age: number = 26
  let hello: string = 'hello, my name is ' + nickname + ', I am ' + age + 'years old.'
  let hello1: string = `hello, my name is ${nickname}, I am ${age} years old.` // es6写法
```

#### 数组 []
ts表示数组有两种方式
1.元素类型后面加`[]`
```typescript
  let list: string[] = ['chenuai','hello world']
  let list2: number[] = [1,2,3]
```
2.使用数组泛型,之后讲解什么是泛型
```typescript
  let list: Array<number> = [1,2,4]
```

#### 元组(Tuple)  
元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。 比如，你可以定义一对值分别为 `string`和`number`类型的元组

```typescript
  let test: [number,string,boolean] = [27,'chenuai',false] 
  let userinfo: [string,number] = ['chenuai',27] 
  // Error : userinfo[1] = ['chenwuai'] // 必须和定义的顺序一样
  // 但是如果出现数组越界的情况，越界元素值必须是(string|number)类型
  // 例如：
  userinfo[2] = '第三个元素' // true
  userinfo[3] = 4 // true
  userinfo[5] = true // error 此处错误，因为布尔值不是string或者number类型
  // 此处牵扯联合类型，之后讲解。
```

#### 枚举 enum
`enum`类型是对js标准数据类型的一个补充。 像C#等其它语言一样，使用枚举类型可以为一组数值赋予友好的名字
```typescript
  enum Color {Red, Green, Blue} // 默认从0开始编号
  enum videoStatus = { connect = 1, play, pause, stop } // 默认从1开始编号
  enum videoType = { mp4, flv = 2, hls = 4, rtsp } // 默认从0开始，flv为2，hls为4， rtsp没有手动设置值，则它为上一个元素的值+1，所以为5
```
枚举类型提供的一个便利是你可以由枚举的值得到它的名字。 
例如，我们知道颜色数值为2，但是不确定它映射到Color里的哪个名字，我们可以查找相应的名字：
```typescript
  let colorName: string = Color[2]
  console.log(colorName) // blue 因为在Color的枚举里面，blue对应的值为2
```

#### Any
有时候，我们会想要为那些在编程阶段还不清楚类型的变量指定一个类型。 这些值可能来自于动态的内容，比如来自用户输入或第三方代码库。 这种情况下，我们不希望类型检查器对这些值进行检查而是直接让它们通过编译阶段的检查。 那么我们可以使用`any`类型来标记这些变量
```typescript
  // 刚开始学ts时，如果不清楚使用什么类型，可以使用any，
  let notSure: any = 4;
  notSure = "maybe a string instead";
  notSure = false;
```
在对现有代码进行改写的时候，`any`类型是十分有用的，它允许你在编译时可选择地包含或移除类型检查。 你可能认为`Object`类型（见下方Object）有相似的作用，就像它在其它语言中那样。 但是`Object`类型的变量只是允许你给它赋任意值 - 但是却不能够在它上面调用任意的方法，即便它真的有这些方法：
```typescript
  let notSure: any = 4;
  notSure.ifItExists(); // okay, 在运行时，ifItExists方法可能存在
  notSure.toFixed(); // okay, toFixed方法存在

  let prettySure: Object = 4;
  prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'. 属性toFixed 不存在
```
当你只知道一部分数据的类型时，`any`类型也是有用的。 比如，你有一个数组，它包含了不同的类型的数据：
```typescript
  let list: any[] = [1, true, "free"];
  list[1] = 100;
```

#### Void
某种程度上来说，`void`类型像是与`any`类型相反，它表示没有任何类型。 当一个函数没有返回值时，你通常会见到其返回值类型是 `void`：
```typescript
  function warnUser(): void {
    console.log("This is my warning message");
  }
  // 声明一个void类型的变量没有什么大用，因为你只能为它赋予undefined和null：
  let unusable: void = undefined;
```

#### Null 和 Undefined
ts里，`undefined`和`null`两者各自有自己的类型分别叫做`undefined`和`null`。 和 `void`相似，它们的本身的类型用处不是很大：
```typescript
  let u: undefined = undefined;
  let n: null = null;
```
默认情况下`null`和`undefined`是所有类型的子类型。 就是说你可以把 `null`和`undefined`赋值给`number`类型的变量。
然而，当你指定了`--strictNullChecks`标记，`null`和`undefined`只能赋值给`void`和它们各自。 这能避免 很多常见的问题。 也许在某处你想传入一个 `string`或`null`或`undefined`，你可以使用联合类型`string | null | undefined`。为了避免常见问题，尽可能的使用 `--strictNullChecks`

#### Never
`never`类型表示的是那些永不存在的值的类型。 例如， `never`类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型； 变量也可能是 `never`类型，当它们被永不为真的类型保护所约束时。

never类型是任何类型的子类型，也可以赋值给任何类型；然而，没有类型是`never`的子类型或可以赋值给`never`类型（除了`never`本身之外）。 即使 `any`也不可以赋值给`never`。
```typescript
  // 返回never的函数必须存在无法达到的终点
  function error(message: string): never {
      throw new Error(message);
  }

  // 推断的返回值类型为never
  function fail() {
      return error("Something failed");
  }

  // 返回never的函数必须存在无法达到的终点
  function infiniteLoop(): never {
      while (true) {
      }
  }
```

#### Object
`object`表示非原始类型，也就是除`number`，`string`，`boolean`，`symbol`，`null`或`undefined`之外的类型。
使用`object`类型，就可以更好的表示像Object.create这样的API。例如：
```typescript
  declare function create(o: object | null): void;

  create({ prop: 0 }); // OK
  create(null); // OK

  create(42); // Error
  create("string"); // Error
  create(false); // Error
  create(undefined); // Error
```

### 类型断言
有时候你会遇到这样的情况，你会比ts更了解某个值的详细信息。 通常这会发生在你清楚地知道一个实体具有比它现有类型更确切的类型。

通过类型断言这种方式可以告诉编译器，“相信我，我知道自己在干什么”。 类型断言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构。 它没有运行时的影响，只是在编译阶段起作用。 ts会假设你，程序员，已经进行了必须的检查。

类型断言有两种形式。 其一是“尖括号”语法：
```typescript
  let someValue: any = "this is a string";
  let strLength: number = (<string>someValue).length;
```
另一个为`as`语法：
```typescript
  let someValue: any = "this is a string";
  let strLength: number = (someValue as string).length;
```
两种形式是等价的。 至于使用哪个大多数情况下是凭个人喜好；然而，当你在ts里使用`JSX`时，只有 `as`语法断言是被允许的。