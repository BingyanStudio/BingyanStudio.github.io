---
title: Golang泛型
cover_image: /img/cover.png
abbrlink: 300b489c
date: 2022-03-28 08:43:26
---

# Golang泛型

## 泛型使用

### 泛型函数

```
func Add[T any](a, b T) T
```
其中T为类型，any为类型约束，相当于没有约束（底层实现为空接口）.

### 泛型结构体

```
type array[T any] struct 
```
### 约束

```
type Inter interface func AddInInt[T Inter](a, b T) T//也可以这样写func AddInInt[T ~int | ~int8 | ~int16 | ~int32| ~int64](a, b T) T
```
go使用interface作为约束，其约束了T泛型的实际类型。

其中 “|” 操作符就是 “或” 的意思，“~”操作符代表粗略匹配（底层实现相同即可），如

```
func AddInInt1[T ~int](a, b T) Tfunc AddInInt2[T int](a, b T) Ttype MyInt intfunc main() 
```
当然，约束的内容除了基础类型之外，约束的内容是方法也可以。

```
func Print[S Stringer](s S) type Stringer interface 
```
当约束中既有基础类型约束，又有方法约束时，其必须二者都满足，如

```
type Stringer interface type MyInt intfunc (a MyInt) String() string //MyInt满足Stringer约束type onlyInt inttype onlyString stringfunc (a onlyString) String() string //onlyInt与onlyString都不满足
```
约束可以嵌套

```
type Stringer interface type Inter interface type SS interface //其等价为//type Stringer interface -----------------------------------------------------------------------------------------type Stringer interface type Inter interface //其等价为//type Stringer interface -----------------------------------------------------------------------------------------type Stringer interface type Inter interface //其虽然不会报错但显然没有任何一种类型满足约束，因为不可能有任何一种类型同时满足：1、底层为int16，2、实现了方法String()，3、实现了Inter约束，因为Inter约束要求底层为int，与要求的第一点（底层为int16）不可能同时实现//}
```
### 泛型约束

定义

```
type LockInter[T any] interface 
```
使用

```
func test[T LockInter](t T)T{}//或者func test[T LockInter[T1],T1 any](t T)T{}
```
## 泛型原理

### Go

Go泛型实现原理与C++模板相同，对于每一种泛型实例生成对应的代码进行调用。

### java

#### 类型擦除

Java的泛型采用了类型擦除来实现，在编译期间，所有的泛型信息都会被擦掉（被擦为原始类型，无约束则为Object），如在代码中定义List和List等类型，在编译后都会变成List，JVM看到的只是List。

```
public class Test }
```
在上面这个例子中，ArrayList与ArrayList为两种ArrayList数组，但最终打印结果为true，因为泛型类型String和Integer都被擦除掉了，只剩下原始类型Object。

```
public class Test //输出1 asd    }}
```
这个例子中直接调用add只能存储Integer，因为泛型类型实例为Integer（编译器会检查代码中泛型的类型，然后进行类型擦除，再进行编译），但通过运行时利用反射调用add()方法，可以存储字符串，说明ArrayList实例在编译之后Integer被擦除成原始类型Object

#### 原始类型

原始类型指擦除去泛型信息，在字节码中的类型变量的真正类型。定义一个泛型时，相应的原始类型会被自动提供，类型变量擦除，并使用其限定类型（无限定用Object）替换。

```
class V<T>       public void setValue(T  value)   } //原始类型为//class V   //    public void setValue(Object  value)   //}public class V1<T extends Comparable> //原始类型为Comparable
```
#### 类型擦除引起的问题

##### 自动类型转换

因为类型擦除，所有的泛型类型变量最后都会被替换为原始类型，既然都被替换为原始类型，那么为什么在获取的时候，不需要进行强制类型转换？

ArrayList.get()方法：

```
public E get(int index) 
```
可以看到，在return之前，会根据泛型变量进行强制类型转换。假设泛型类型变量为Date，虽然泛型信息会被擦除掉，但是会将(E) elementData[index]，编译为(Date)elementData[index]。所以不用自己进行强制类型转换。当存取一个泛型域时也会自动插入强制类型转换。假设V类的value域是public的，那么表达式：

```
Date date = V.value;
```
也会自动地在结果字节码中插入强制类型转换。

##### 泛型类型无法当做真实的类型使用

```
public  void Method(T t)
```
##### 泛型类型无法用作方法重载

```
public void printList(List list)public void printList(List list)
```
因为在编译时泛型被擦除，被擦除后这两种方法就没有任何区，所以这种写法是错误的。

##### 与多态的冲突(重写&重载)

```
class V<T>       public void setValue(T value)   }class DateV extends V<Date>       @Override      public Date getValue()   }
```
我们本意是在子类DateV中对父类V的两个方法进行重写，但是若进行了类型擦除后，父类就会变成：

```
class V       public void setValue(Object  value)   }  
```
而子类中两个重写的方法：

```
@Override  public void setValue(Date value)   @Override  public Date getValue() 
```
对setValue方法，父类类型是Object，而子类是Date，参数类型不同，这看起来不会是重写，而是重载。

我们进行测试一下：

```
public static void main(String[] args) throws ClassNotFoundException 
```
如果是重载，那么子类中有两个setValue方法，一个是Object类型，一个是Date类型，但通过编译失败信息可以看出，没有参数为Object类型的setValue方法，可见其确实是重写而不是重载。JVM解决这个问题的方法是桥方法：编译器会在DataV子类中自动生成两个桥方法来重写父类的方法：

```
public Object getValue() //虚拟机通过参数类型和返回类型来确定一个方法，所以两个getValue方法可以同时存在（但自己编写java代码时不允许这样）public void setValue(Object value)   
```

