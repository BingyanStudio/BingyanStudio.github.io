---
title: Build a Simple BF Interpreter Based on TypeScript Types
cover_image: https://image-public.bingyan.net/blog/img/567aaf4e-da04-4754-98d4-1e6ecb13d479.jpeg
date: 2025-04-05 23:08
categories: 前端
author: 主教
abbrlink: 143307b1-f5da
---


TypeScript 的类型系统是图灵完备的（[#14833](https://github.com/microsoft/TypeScript/issues/14833)）。因此我尝试用 TypeScript 的类型系统来实现一个 Brainfxxk 的解释器。

## 什么是 Brainfxxk？

BF 实际上就是一个图灵机（Turing Machine）。以防万一，我贴上 BF 的指令表。

```
> 指针右移
< 指针左移 
. 输入
, 输出
+ 当前位置的值加一
- 当前位置的值减一
[ 如果当前位是0，则直接跳至对应的 ] 处
] 如果当前位是0，则直接跳回对应的 [ 处
```

下面是一段简单的 BF 程序：

```
>++[<++++>-]<[>>++++[<++>-]<<-]>. // Output: 'A'
```

## TypeScript 必要的类型体操

分支判断：

```typescript
type IsNumber<T> = T extends Number ? true : false
```

`infer` 语法，让我们优雅地、非递归地取出我们想要的值：

```typescript
type ArrayRest<T>   = T extends [any, ...infer Rest] ? Rest : never
type ArrayFirst<T>  = T extends [infer First, ...any[]] ? First : never
type StringRest<T>  = T extends `${string}${infer Rest}` ? Rest : never
type StringFirst<T> = T extends `${infer First}${string}` ? First : never
```

数组 `['length']` 属性：

```typescript
type LiteralNum4 = [any, any, any, any]['length'] // ✅ extends 4
```

我们发现，可以声明一个 `any[]` 类型作为 `MyNum`，用其长度代表数值，实现计数：

```typescript
type MyNum = any[]

type AddOne<MyNum> = [any, ...MyNum]  // ['length'] += 1
type SubOne<MyNum> = ArrayRest<MyNum> // ['length'] -= 1
```

`['length']` 结合 `extends` 的分支，<strong>理论上</strong>我们可以用递归表示任何自然数：

```typescript
type MyNum<
        N extends Number = 0,
        Temp extends any[] = []
> = Temp['length'] extends N
    ? Temp
    : MyNum<N, [any, ...Temp]>

// Tests
type Test0 = MyNum['length'] extends 0    // ✅
type Test1 = MyNum<1>['length'] extends 1 // ✅
type Test8 = MyNum<8>['length'] extends 8 // ✅
```

但是 `['length']` 计数有一个缺点：不能表示负数。为此我也实现了一个 `type Int`，用数组模拟内存，使用补码位运算实现任意加减：[tsfuck/src/number.ts (GitHub)](https://github.com/HomeArchbishop/tsfuck/blob/main/src/number.ts)。

## 开始写 Tsfuck 吧

我们设想一下，如果用 JS 或者 TS（非类型系统）实现 BF，思路通常是什么样的。

- 输入 `Program: string` 和 `Input: string`，将 `Input` 逐字符转化为 Ascii 码（`number` 类型）供图灵机读取，以适用加减指令。最后将图灵机输出的每个 Ascii `number` 转化并拼接为 `Output: string`。
- 面向对象设计，实现 `class TuringMachine`。简单的原型如下

```
class TuringMachine {
        Tape:     number[] // 或称 Memory，图灵机「纸带」
        Capacity: number   // 「纸带」容量
        Ptr:      number   // 图灵机「指针」

        GetTape()     : number[]
        GetCapacity() : number
        GetPtr()      : number
}
```

- 对 `Program` 逐字符<strong>迭代</strong>，修改图灵机实例，得到输出。

尝试将这套想法等价翻译进 TS 类型系统，是不是就可以了呢？

### Step 1 出入输出 Char & Int

这个很容易实现。在 BF 中，只有加一减一，且 ascii 码都是非负的。因此直接使用 `\00` 到 `\ff` 构建加减映射表。不用将 `Input: string` 再转化为 `Int` 再加减，直接查表就行了。

```
// Good:
interface AsciiIncreaseMap {
  '\x00': '\x01'
  '\x01': '\x02'
  //...
  '\xff': '\x00'
}
interface AsciiDecreaseMap {
  '\x00': '\xff'
  '\x01': '\x00'
  // ...
  '\xff': '\xfe'
 }
```

### Step 2 实现图灵机

值得注意的是，TS 的类型系统是一个非常典型的 <strong>函数式编程（FP）</strong> 系统。应当充分发挥其 <strong>Data-in & Data-out</strong> 和 <strong>声明即实现</strong> 的特点，设计以下的图灵机：

```typescript
type Tape<infer A, infer B, infer C> = [A, B, C]
type TuringMachine<
  A extends string = any, B extends Char = any, C extends string = any
>= Tape<A, B, C>
```

```
<'',     '\x00',         ''> // Initial
<'',     '\x00',     '\x00'> // After <
<'',     '\x01',     '\x00'> // After +
<'',     '\x00', '\x01\x00'> // After <
<'\x00', '\x01',     '\x00'> // After >
```

这里补充一下，如果使用类 OOP 的思维，用 Array+Ptr 和 Getter 来写，也是可以实现的。`TM` 作为状态输入，每一个泛型运算确实是 pure function。但是实际使用起来，会出现大量的  `TM<GetXX<TM<...>>>`  嵌套，并没有发挥出函数式编程的优势。

```typescript
// Not good...
type TuringMachine = [IntLength, IntType[], IntType, IntType]
type GetL   <TM extends TuringMachine> = TM[0]
type GetTape<TM extends TuringMachine> = TM[1]
type GetCap <TM extends TuringMachine> = TM[2]
type GetPtr <TM extends TuringMachine> = TM[3]
```

### 逐字符解析

TS 类型系统无法使用<strong>迭代</strong>，所以用<strong>递归</strong>代替。伪代码表示为：

```typescript
type Excute<
        P extends Program,
        I extends Input,
        TM extends TuringMachine
> = Excute<RestP<P>, RestI<I>, NextTM<TM>>
```

对上面的每个模块作单元测试，确保运算正确。最终测试如下：

```typescript
type Test1 = Tsfuck<'+++>++++.', ''>   // ✅ extends '\x04'
type Test2 = Tsfuck<'+[>+++<-]>.', ''> // ✅ extends '\x03'
```

Looks nice! 我们实现了正确的 Brainfuck 解析器！ 那上点强度呢？

```typescript
type Test3 = Tsfuck<'++[>+<-]>.', ''>
// 😥 Type instantiation is excessively deep and possibly infinite.
```

Oops... 提示递归层级过深了。怎么办呢？

## 解决递归过深的问题！

什么是递归过深？以上面 `MyNum` 为例：

```typescript
type MyNum<
        N extends Number = 0,
        Temp extends any[] = []
> = Temp['length'] extends N
    ? Temp
    : MyNum<N, [any, ...Temp]>

type Num = MyNum<4>    // ✅ fine
type Num = MyNum<999>  // ✅ fine
type Num = MyNum<1000> // ❎ too deep
```

不难看出，在 N 过大时，类型递归的展开层数超出了编译器允许的最大递归深度，TS 罢工了。

当然我们可以通过修改 TypeScript 的 `lib/_tsc.js` 代码突破限制，但这是不优雅的（不过这很有意思，挖个坑之后分享）。这里给出一种解决方案（Ref：[TypeScript の型の再帰上限を突破する裏技](https://susisu.hatenablog.com/entry/2020/09/12/214343)）：

首先我们小改一下 `MyNum` 的声明：

```typescript
type MyNum<
        N extends Number = 0,
        Temp extends any[] = []
> = Temp['length'] extends N
    ? Temp
    : { _: MyNum<N, [any, ...Temp]> }
```

取代了裸奔的 `any[]`，递归生成的类型结构如下所示：

```typescript
{
        _: {
                _: {
                        _: [any, any, any]
                }
        }
}
```

非常奇妙的是，<strong>这个结构是惰性解析的</strong>，因此可以通过递归构造出<strong>任意长度</strong>的 `any[]` 数组（Source：[Announcing TypeScript 3.7 - (more)-recursive-type-aliases](https://devblogs.microsoft.com/typescript/announcing-typescript-3-7/#(more)-recursive-type-aliases)）。然而，如何从该结构中提取出最终生成的 `any[]` 呢？

<strong>只能再递归</strong>。我们考虑定义一个 `RecurseSub` 将值剥开，再用 `Recurse` 朴素地递归，可以吗？

```typescript
type RecurseSub<T> = T['_']
type Recurse<T> = T extends { _: unknown } ? Recurse<RecurseSub<T>> : T

type NNN = Recurse<MyNum<1000>> // ❎ too deep
```

显而易见，这种递归又回到了 999 层限制。正确的做法是，同样利用<strong>惰性解析</strong>的特性来削减层数。使用 `infer` 推断，将<strong>每两层</strong>结构简化成一层结构，保证它仍然是一个 `{ _: obj }`。

```typescript
type RecurseSub<T> = 
    T extends { _: never } 
                ? never
            : T extends { _: { _: infer U } }
                    ? { _: RecurseSub<U> }
                : T extends { _: infer U }
                        ? U
                    : T

type Recurse<T> = T extends { _: unknown } ? Recurse<RecurseSub<T>> : T
```

```typescript
// fine-tests.ts
type Num = Recurse<MyNum<1000>> // ✅ fine
type Num = Recurse<MyNum<3141>> // ✅ fine

// bad-tests.ts
type Num = Recurse<MyNum<3142>> // ❎ too deep
```

上限提升到了 3141。但是，按照刚才的分析，`MyNum` 的理论上限是 2^999，这里为什么只有 3141 呢？<strong>这也是一个很有意思的问题</strong>，再挖个坑，以后单独讨论。

## Tsfuck2

基于上面的想法，我实现了新版：[Tsfuck2 (GitHub)](https://github.com/HomeArchbishop/tsfuck2)。测试显示，它运行得正确且轻松。

```typescript
const TsfuckTest: Test<[
        IsExtends<Tsfuck<'++[>+++[>+<-]+<-]>.>.', ''>, '\x01\x07'>
        IsExtends<Tsfuck<'>++[<++++>-]<[>>++++[<++>-]<<-]>.', ''>, 'a'>
    IsExtends<Tsfuck<'++++++++++[>+++++++>++++++++++>+++>+<<<<-]>++.>+.+++++++..+++.>++.<<+++++++++++++++.>.+++.------.--------.>+.>.', ''>, 'Hello World!\n'>
] // ✅ Passed
```

复杂一点，算斐波那契数列：

![](https://image-public.bingyan.net/blog/img/JTH4b3gy3owiKIxamRvcvQQInM5.jpeg)

再有趣一点，自举！下面是一个运行在 Tsfuck 上的 Brainfuck 写成的 Brainfuck 解析器运行成功了一段 Brainfuck 程序（我用 TS 写了 BF，再用 BF 实现了 BF）：

![](https://image-public.bingyan.net/blog/img/HBMlb6jFhoBDzUx04LOc6ulSncc.jpeg)

## 一个小细节

如何获得字符串的的第一个字符？ <strong>StringFirst<S></strong><strong> or </strong><strong>S[0]</strong>？ 我使用了 `StringFirst<S>` ，而没有简单地使用 `S[0]`。原因在于 `S` 可能是一个空串，`S[0]` 是 `Char|undefined`，而 `StringFirst<S>` 是 `Char`。因此，尽管在实际使用中都必须保证 `S` 不为空，但只有 `StringFirst<S>`  通过类型检查。

## 后记

TS 类型系统作为一个函数式编程系统，完全具备函数式编程的特性。更多地，例如函数组合，结果缓存的优点，在 TS 中亦有体现。关于这个，或许以后可以另行讨论。

## Repo

- [GitHub - HomeArchbishop/tsfuck](https://github.com/HomeArchbishop/tsfuck)
- [GitHub - HomeArchbishop/tsfuck2](https://github.com/HomeArchbishop/tsfuck2)

## Ref

- [some brainfuck fluff](https://brainfuck.org/)
- [TypeScript の型の再帰上限を突破する裏技](https://susisu.hatenablog.com/entry/2020/09/12/214343)
- [GitHub - susisu/typefuck](https://github.com/susisu/typefuck)
- [Announcing TypeScript 3.7 - TypeScript](https://devblogs.microsoft.com/typescript/announcing-typescript-3-7/#(more)-recursive-type-aliases)

## Also Read

- - [What Writing 2K+ Lines of Brainfuck Taught Me](https://aartaka.me/brainfuck-lessons.html)
- [How to code in BrainF\*ck without Losing Your Mind… - by Saket Upadhyay \| Medium](https://saketupadhyay.medium.com/how-to-code-in-brainf-ck-without-losing-your-mind-6a8fd67b36b4)
- [Build Your Own Brainfuck Interpreter \| Coding Challenges](https://codingchallenges.fyi/challenges/challenge-brainfuck/)
- [TypeScript の型で遊ぶ時、再帰制限を無効化する - Qiita](https://qiita.com/kgtkr/items/eff20225e4bf9b159110)
- [A Practical Guide to Functional Programming \| by Hakan Apohan \| HAVELSAN \| Medium](https://medium.com/havelsan/a-practical-guide-to-functional-programming-dfdda6c35c0f)
