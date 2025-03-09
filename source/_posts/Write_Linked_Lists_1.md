---
title: ​Write Linked Lists (1)
cover_image: /img/cover.png
abbrlink: '54193186'
date: 2022-04-12 15:44:10
---

移动组的姚姐最近写微机原理作业写得很辛苦，为了找点乐子她决定开始学习Rust（什么）。### A Bad Stack

从实现一个最简单的链表开始。

什么是链表呢？简单来说，它是堆上的一堆按顺序指向彼此的数据。我们可以函数式的尝试给出这样的定义。

```
List a = Empty | Elem a (List a)
```
这是一个递归定义。照着这个定义我们可以写出这样的Rust代码。

```
pub enum List 
```
呼，这很简单嘛，我们可以尝试编译一下。姚姐自信的输入了cargo build。

3 seconds later ~~

damei！我们没有通过编译，这意味着我们的写法是不合法的。这是因为这样编写的递归结构编译器无法确定其中的元素数量，也就不知道需要为其分配多少内存。

为了解决这个问题，我们可以引入一个在Rust中非常常用的智能指针Box< T >。Box允许你将一个值放在堆上而不是栈上。留在栈上的则是指向堆数据的指针。

OK，我们再试一次。

```
pub enum List 
```
这次，我们成功编译了，因为在List中存的是Box的指针，指针的大小是确定的，我们解决了编译器无法确定递归结构需要多少内存的问题。

但这实际上是一个很蠢的定义。我们考虑一个拥有两个元素的列表。

```
[] = 栈() = 堆[Elem A, ptr] -> (Elem B, ptr) -> (Empty *垃圾*)
```
是的，我们不仅创建了一个根本不是节点的节点，此外另外其中的一个节点根本没分配在堆中！这意味着链表的节点的内存布局是不统一的。

我们重新考虑一个链表的潜在的内存布局。

```
[ptr] -> (元素A, ptr) -> (元素B, *null*)
```
在这个布局里，我们在堆里分配了所有的元素，并且和第一个布局相比，我们让多余的垃圾消失了。那么之前提到的内存布局不统一会带来什么样的问题呢，考虑这样一个场景，在两种布局里分别分割一个列表。

```
layout 1:[Elem A, ptr] -> (Elem B, ptr) -> (Elem C, ptr) -> (Empty *junk*)split off C:[Elem A, ptr] -> (Elem B, ptr) -> (Empty *junk*)[Elem C, ptr] -> (Empty *junk*)
```
```
layout 2:[ptr] -> (Elem A, ptr) -> (Elem B, ptr) -> (Elem C, *null*)split off C:[ptr] -> (Elem A, ptr) -> (Elem B, *null*)[ptr] -> (Elem C, *null*)
```
布局2的分割仅仅涉及将B的指针拷贝到栈上，并把原值设置为null。布局1最终还是做了同一件事，但是还得把C从堆中拷贝到栈中。

链表的优点之一就是可以在节点中构建元素，然后可以在列表中随意调换他的位置而不是移动他的内存。我们只需要调整指针，元素就被移动了。但是第一个布局毁掉了这个优点。

在重新编写代码前，我们先来介绍一个被称作空指针优化的特性，它可以帮助我们写出更好的代码。

```
enum Foo 
```
一般来说，enum需要存储一个标签，来指明它代表哪一个enum的变体，但是如果我们有如上特殊类型的enum，空指针优化就会发挥作用，消除标签所占用的内存空间。如果是A变体，整个enum被设置为0，否则，就是B变体。

这样优化的enum和类型在Rust中非常多，这看起来很简单，但其实很重要。这意味着&, &mut, Box, Rc, Arc, Vec，以及其他一些Rust中的重要类型，在放到一个Option中时没有多余开销，这些都是非常常用的编程模型。

好的，我们已经掌握了所有支撑我们写出一个链表数据结构定义的知识，我们可以写出如下的代码。

```
pub struct List enum Link struct Node 
```
我们现在实现了：

* 列表末尾不分配多余垃圾。
* enum享受美妙的空指针优化。
* 所有元素的内存分配一致。
继续实现一个List的构造方法。

```
impl List     }}
```
有了基础的定义和方法，我们接下来实现一个push方法。通过这个方法，我们也可以一窥Rust的神奇之处。

按照我们的现有理解，我们可以写出这样的代码，因为push修改了list，所以我们需要&mut self。

```
impl List ;       self.head = Link::More(new_node);    }}
```
看起来完全没有问题了，但是很可惜我们仍然过不了编译。这是违反Rust所有权机制带来的错误。由于篇幅所限，这里将不会详细介绍Rust的所有权和生命周期等特性，我们的目的在于了解Rust的所有权后实现一个链表。

这里编译报错的原因在于第5行我们尝试将self.head赋值给new_node的next字段，但这意味着我们会将self.head的所有权移动到new_node身上，此时这个函数內只拥有self.head的引用，意味着我们只是借用，并不是真正的拥有这个self.head，但是借用时如果self.head的所有权被移出去了，那self.head自己是什么呢，self的head字段存储着什么呢，这是Rust不允许的行为。可以用这张动图形象的描述这一情景。

解决这个问题的方法有很多，在这里我们可以使用mem::replace这一招。这个有用的函数可以让我们通过替换的方式从一个借用中偷出一个值。

```
pub fn push(&mut self, elem: i32) );    self.head = Link::More(new_node);}
```
这样子，push函数就完成了，接下来我们继续实现pop函数。

```
pub fn pop(&mut self) -> Option<i32>         Link::More(ref node) =>     };    result}
```
意料之中，这同样是一个编译不过的版本，但这里也有细节需要注意，我们在match的匹配中使用了ref node，这意味着我们只是匹配self.head的引用，假如没有这个ref，match会像之前一样，把self.head的所有权传入match的范围内，self.head会失去所有权，这意味着我们犯了和之前的一样的错误，error: cannot move out of borrowed content。

那么我们上面又出现了什么错误呢。

这里有两个问题，首先，我们的赋值操作在我们仅仅拥有一个共享引用的情况下尝试把值移动出node，另外，我们已经借用了node的引用后，我们还在尝试改变self.head的值。

解决这些问题的办法我们依然可以使用替换这一个方法。

```
pub fn pop(&mut self) -> Option<i32>         Link::More(node) =>     };    result}
```
根据Rust的语法，我们还可以简化一下。

```
pub fn pop(&mut self) -> Option<i32>     }}
```
就这样，虽然他还很简单，但我们成功实现了一个类似栈的数据结构，通过list组织，提供了关键的pop和push的方法。但这还不够。

### An Ok Stack

#### Using Option

使用Option改进一下list的数据结构。另外之前使用的mem::replace(&mut option, None)是一种非常常见的习惯用法，以至于option实际上直接把它变成了一个方法:take。match option 也是一种非常常见的习惯用法，它可以调用map实现。map接受一个函数在Some(x)中的x上执行，生成Some(y)中的y。使用闭包我们还可以引用闭包外部的局部变量，这使得我们在处理各种条件逻辑时非常有用。

根据这些我们可以重写一下之前的代码。

```
pub struct List type Link = Option<Box>;struct Node impl List     }    pub fn push(&mut self, elem: i32) );        self.head = Some(new_node);    }  pub fn pop(&mut self) -> Option<i32> )  }}impl Drop for List     }}
```
### Generic

Rust中提供了较好的泛型支持，之前我们已经在Option和Box的使用中接触了一些泛型，接下来我们可以实现一下list中元素的泛型。

```
pub struct List type Link = Option<Box>>;struct Node impl List     }    pub fn push(&mut self, elem: T) );        self.head = Some(new_node);    }    pub fn pop(&mut self) -> Option )    }}impl Drop for List     }}
```
### Peek

一个完整的list不应该只有pop和push这两个函数，我们接下来继续实现一个peek函数，peek函数可以返回一个指向列表头元素的引用（如果它存在的话）。这听起来很简单，让我们试试。

```
pub fn peek(&self) -> Option<&T> )}
```
可惜我们仍来没有通过编译，有了之前的经验相信我们已经能大概猜出来了问题的所在，这同样是因为map会转移self.head的所有权，所有权的生命周期只是维持在这个闭包内，但是我们还需传出对这个节点元素的引用，这显然是不合法的，returns a reference to data owned by the current function。之前我们通过take函数实际上移走的不是self.head的所有权，而是替换出来的元素。

解决这个问题的方法在于我们不能让map转移所有权，我们可以使用as_ref函数。

```
impl Option 
```
```
pub fn peek(&self) -> Option<&T> )}
```
就这样，我们让map内部获得了引用而没有转移走所有权。我们还可以实现一个可变版本的peek函数。

```
pub fn peek_mut(&mut self) -> Option<&mut T> )}
```
到现在为止我们实现了一个比较简单的链表，涉及到了一些Rust中特有的知识作为对Rust的一个简单介绍，之后我们会继续深入的实现和完善这个链表。

> Reference
> 
> * https://rust-unofficial.github.io/too-many-lists/index.html
> * https://doc.rust-lang.org/book/
>

