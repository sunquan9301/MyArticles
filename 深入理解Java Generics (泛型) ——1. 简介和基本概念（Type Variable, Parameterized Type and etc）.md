# 前言

本系列文章包含如下几个**topics**

0. **Generic**相关概念和基本使用（**Type Variable，Parameterized Type and etc**）

1. **Generic**的实现：**Erasure**(擦除)方案

2. **Subtyping and Wildcards**(通配符)的应用场景

3. Enumerated Types

4. **Restrictions on Wildcards** and **unchecking warning**
5. **Covariance and contravariance**(协变与逆变)

5. **Array** and **Reflection**

6. **Collection Framework with Generic**

# Abstraction

本篇文章主要介绍**泛型的发展**，**泛型的基本使用**，相关概念如：**Boxing and Unboxing，Type System、Type Variable、Parameterized Variables**。文章首先通过一个泛型的基本例子来展示泛型的出现对Java语言以及程序的影响，接着阐述泛型相关的基本概念，最后介绍基本的使用方法。

# Introduction

在wikepedia中，Generic如下定义：

> 1.**Generics** are a facility of **generic programming** that were added to the Java programming language in 2004 within **version J2SE 5.0.** 
>
> 2.They were designed to **extend Java's type system** to allow "a type or method to **operate on objects of various types** while providing **compile-time type safety**"
>
> The aspect compile-time type safety was not fully achieved, since it was shown in 2016 that it is not guaranteed in all cases.

- 在J2SE 5.0出现
- 扩展Java 类型系统去支持一个类型或方法操作不同的类型对象，并保证编译器类型安全
- 编译时期的类型安全并没有完全的实现

在Java中，**Generic**被广泛的应用于**Java Collections FrameWork**，大大的提高了代码的可读性，以及**coding**的效率。

**在Java Generic出现之前**，如果想处理一些不同类型的集合如：`Integer`、`String`、`Lists of String`，你可以用同样的类：List 去处理：

```java
list of integers 		               List
list of strings                    List
list of lists of strings           List
```

但是，不得不面对的一个事实是：必须手动的去保证和处理List里面对象的类型。也即：`you must cast it from Object back to Integer(or String or List)`

**在Java Generic出现之后**，则可以用如下方式来区分不同类型的lists

```java
list of integers									  List<Integer>
list of strings											List<String>
list of lists of strings 						List<List<String>>
```

此时编译器保障了要处理list里面的对象类型即：`compiler no explicit cast back to Integer(or String or List<String>)`

## Motivation Examples

```Java
1. List v = new ArrayList();
2. v.add("test"); // A String that cannot be cast to an Integer
3  Integer i = (Integer)v.get(0); // Run time error
```

在Generic出来之前，编译如上代码并不会报错，但是在运行第三行代码的时候会 `throws a runtime exception` `(java.lang.ClassCastException)` ，如果使用泛型重写以上代码，则变为如下：

```Java
1. List<String> v = new ArrayList<String>();
2. v.add("test");
3. Integer i = v.get(0); // (type error)  compilation-time error
```

第一行代码表示该集合只接受`String`类型的对象，因此第三行代码会直接在第三行代码抛出异常。

因此`Generic`的一个重要作用就是：**将代码安全性检查提前到编译期**；

我们再看另外一个求和的例子：

```Java
List<Integer> ints = Arrays.asList(1,2,3); 
int s = 0;
for (int n : ints) { s += n; }
assert s == 6;
```

该例子中，定义了一个`Integer`类型的集合，然后通过`foreach`循环获取集合里面的内容经过`boxing`和`unboxing`处理后做求和操作。

如果不使用`generic`相应的代码如下：

```Java
List ints = Arrays.asList( new Integer[] {
new Integer(1), new Integer(2), new Integer(3)
} );
int s = 0;
for (Iterator it = ints.iterator(); it.hasNext(); ) {
	int n = ((Integer)it.next()).intValue();
  s += n; 
}
assert s == 6;
```

相比较使用`generic`的代码，则需要手动处理`list`里面对象的类型。

因此：**泛型能够省去类型强制转换，并简化代码**

比较如上2种程序，可以看到`generic`通过尖括号为`List`绑定了对象的类型，因此不需要明确的处理和转换对象的类型。



**!!!Generic会隐式的执行强制转换，那么如果转换失败了怎么办？**

因此Generic具有如下的**保证**：

***Cast-iron guarantee*:**  the implicit casts added by the compilation of generics never fail.

但是这个**guarantee**也有其局限性就是：它只适用于编译器没有发出**unchecked warnings**的情况。

在文章开头Generic的定义就表明：**compile-time type safety was not fully achieved。**

## Java Type System

#### Definition:

这里简单的说一下`java`的类型系统。定义如下：

> In programming languages, a **type system** is a logical system comprising **a set of rules** that **assigns a property called a type** to the various constructs of a computer program, such as **variables、expressions、functions or modules**.

- 包含了一组规则
- 为程序里不同的变量、表达式、函数等赋予一种类型属性

#### Purpose:

`type system`有什么作用呢？如下

> The main purpose of a **type system** is to **reduce possibilities for bugs** in computer programs **by defining interfaces between different parts of a computer program**, and then **checking that the parts have been connected in a consistent way.** 

- 降低bug
- 规范不同接口间调用类型的一致性

这里提到了`checking`，具体分为`statically` **(at compile time)**和`dynamically`**(at run time)**；于此同时就对应着编译报错和运行报错

因此：**类型系统通过类型检查来保障程序代码满足一系列的类型一致性规则，因此来确保程序的正确性**

`java language`有如下2种说话：

1. `The java programming language is a strongly typed language`
2. `The Java programming is a statically typed language`

#### Strongly Typed Language

那什么是`strongly typed language`：

```markdown
If a language specification requires its typing rules strongly (i.e., more or less allowing only those automatic type conversions that do not lose information), one can refer to the process as strongly typed, if not, as weakly typed.
```

简单来说**只允许那些不丢失类型的自动类型转换，则为强类型，反之为弱类型。**

```java
int a = 1;
float b = 2.2;
System.out.println(a+b);
```

如上例子中整型a和浮点型b相加，那么整型将自动转换为浮点型。

```markdown
The more type restrictions that are imposed by the compiler, the more strongly typed a programming language is.
```

#### Statically Typed Language

这里得提到`Type checking`类型检查分为**静态类型检查**和**动态类型检查**，**通过分析源代码来保障程序的类型安全则为静态类型检查，相应的语言则为静态类型语言。**

```markdown
Static type checking is the process of verifying the type safety of a program based on analysis of a program's text (source code).
```

如果一段代码不能通过静态类型检查，则会抛出编译期异常。

## Concepts About Generic

#### Type Variable

> 1. A **type variable** is an unqualified **identifier** used as a type in **class, interface, method**, and **constructor** bodies.
>
> 2. A **type variable** is introduced by the **declaration of a type parameter of a generic class**, interface, method or constructor

- **类型变量**是一个标识符用于类，接口，方法和构造函数
- **类型变量**通过泛型的**类型参数的声明**引入

因此，我们有：

A **class** (**interface**、**method**、**constructor**) is **generic** if it **declares** one or more **type variable**. These type variables are **known as the type parameters** of the **class**  (**interface**、**method**、**constructor**) . It define one or more type variables that act as parameters.

#### Definition

*TypeParameter	:= 	{TypeParameterModifier} Identifier [TypeBound]* 

*TypeParameterModifier	:=	 Annotation* 

*TypeBound	:=*	extends *TypeVariable* extends *ClassOrInterfaceType {AdditionalBound}* 

*AdditionalBound	:=	*& *InterfaceType*

说明如下：

- **TypeParameter** 可以是单独的标识符号如：**T**
- 或者可以为**T1 & … & Tn**
- 每个**T** 可以声明一个边界：` T extend Number`
- 如果没有声明边界，则默认为`object`

#### Example

```java
package TypeVarMembers;

class C {
  public    void mCPublic()    {}
  private   void mCPrivate()   {}
}

interface I {
  void mI();
}

class CT extends C implements I {
  public void mI() {}
}

class Test {
  <T extends C & I> void test(T t) {
    t.mI();           // OK
    t.mCPublic();     // OK
    t.mCPrivate();    // Compile-time error
  } 
}
```

```java
//类C有一个public方法和private方法; 
//接口I定义了方法mi(); 
//类CT继承了类C,实现了接口I, 因此包含类C和接口I的所有属性和方法
//类Test定义了一个泛型方法 test; 该方法声明了一个类型变量T; T通过extends声明了它的边界 C & T； 因此与CT一样，T继承了C和T的属性和方法。
```

#### Parameterized Types

> 1. A class or interface declaration that is generic **defines a set of parameterized types**.
>
> 2. A **parameterized type** is a class or interface type of the form **C<T1,...,Tn>**, where **C** is the **name of a generic type** and **<T1,...,Tn>** is a list of **type arguments** that denote
>    a particular *parameterization* of the generic type.

```java
//因此对于 C<T1,...,Tn>
//1.C is the name of a generic type.
//2. T1 ... Tn is a list of type arguments
//3. the number of type arguments must match the number type parameters.
//4. each T is a parameterized type
//5. each T has its bound.(Object if not be declared)
```

**Example**：

```java
List<String>				//ok
List<List<Integer> 	//ok
Map<String>					// error, not enough type arument
Map<String,Integer> //ok
List<int>						//error, primitive types cannot be type arguments 
Pair<String,String,String> //error too many arguments
```

#### Boxing and UnBoxing

Java 类型分为*reference type* or a *primitive type*. 所有的reference types 都是**Object**的子类，可以被赋值为**null**。

```java
Primitive					Reference
byte							Byte
short							Short
int								Integer
long							Long
float							Float
double						Double
boolean						Boolean
char							Character
```

> **Boxing**: Conversion of a primitive type to the corresponding reference type
>
> **UnBoxing:** conversion of the reference type to the corresponding primitive type

泛型会在合适的地方自动的插入*boxing* or *unboxing* 如下相等的2个例子

```java
List<Integer> ints = new ArrayList<Integer>(); ints.add(1);
int n = ints.get(0);
```

```java
List<Integer> ints = new ArrayList<Integer>(); ints.add(Integer.valueOf(1));
int n = ints.get(0).intValue();
```

#### Foreach

```java
1. List<Integer> ints = Arrays.asList(1,2,3); int s = 0;
2. for (int n : ints) { s += n; } 
   //foreach loop even though with the keyword for.
3. assert s == 6;
```

第三行代码等价于如下代码：

```java
for (Iterator<Integer> it = ints. iterator();it.hasNext();) { 
  int n = it.next();
	s += n;
}
```

## Generic 基本使用

#### Class

```java
public interface List<E> extends Collection<E> {
  Iterator<E> iterator();
  boolean add(E e);
  E get(int index);
  E set(int index, E element);
  boolean addAll(Collection<? extends E> c);
  int indexOf(Object o);
}
```

```java
/**上述代码是List的接口定义
1.List是一个泛型接口，List 是对应的接口名字
2.List定一个了一个parameterized Type 为 Type Variable E;
3.bound默认为Object.
4.继承了Collection<E> 接口
5.包含了一系列的方法，这里关注下  
	boolean addAll(Collection<? extends E> c);
		因为父类可以引用它的子类，所以用 <? extends E>来表示。
	int indexOf(Object o);
		这里传入参数的类型是Object, 为什么不是indexOf(E o),因为判断传入的类型可以是任意的，他们只有一个子类Object；如果定义为 indexOf(E o)那么传入的方法不是E或者E的子类的时候就会报 unchecking warning.
**/

```

#### Method

这里为以上的List的实现类加一个Generic Method

```java
public class MyList<E> implements List<E>{
    public static <K,V> List<K> toKeyList(Map<? extends K,V> map){
        List<K> list = new ArrayList<>();
        for(K k:map.keySet()){
            list.add(k);
        }
        return list;
    }
 		//1.toKeyList泛型函数定义了2个parameterized Type， Type Variable K and V
  	//2.传入一个Map map的key值为K以及K的子类，这里把它转换为List.
}
```

以上就是基本的泛型使用。

# Conclusion

本文简单的介绍了Generic相关的基本概念以及其基本使用方法，那么对应日常应用场景有什么用呢？

1. 优秀的开源框架，或多或少都涉及到泛型，帮助你更好的理解开源框架的设计思想
2. 遇到泛型相关的bug（类型出错），能更精准的定位问题
3. Generic涉及了很多优秀的语言设计思想，可以提高自己的编码思维，大大提高了代码的扩展性
4. more….