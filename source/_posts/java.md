---
title: Java I Basics
date: 2019-04-22 19:03:16
tags: java
---

### I.Most Basics

**1.variables** - primitive and reference

* primitive(8种)

```java
boolean
char
integer:
  byte(8bits), short(16bits), int(32bits), long(64bits)
  其中，int：[-2147483648, 2147483647] //10位
float(32bits): float x = 32.5f; //必须加上f
double(64bits)
```

local variables is within a method, it must be initialized before use.

**2.Class & objects**  
class is the blueprint of objects, instance is another way of saying objects.

Object has instance variables(state) and methods(behavior).

一旦没有指向某object的variable了，这个object就会被GC回收。

**3.Arrays**  
**Arrays are always objects**, whether they hold primitives or object references.

**4.Java is pass everything by value**, which means pass-by-copy.  
如果传入的是一个reference variable, pass a copy of that remote control, 就是传入的是一个复制的地址。

**5.setter & getter**  
用setter和getter的好处是，可以不破坏class的原有代码，而在这两个函数里做想做的事。比如instance variables are private, but setter and getter are public, so other objects can access to it by using setter and getter.

**6.Java Library**  
Every class in java belongs to a package.

```java
import java.util.*;
```

### II.override & overload

Only override achieves Polymorphism. Overloading has nothing to do with inheritance, but it can achieve static polymorphism!

#### override

子类可以复写父类里的方法.  
Override allows subclass to provide a specific implementation of a method that is already provided by one of its super classes.  

* The arguments must be the same; the return type is a subtype of the return type declared in the superclass.
* The method cannot be less accessible, like from public to private.

#### overload

必须是参数的数量或类型不一样，但是对返回的类型没有要求.  
Allow different methods to have the same name, but different signatures differing on the number or type of the input parameters. Overloading cannot distinguish two same signatures with different return type.  
Can vary the access level in any directions, 即overload的方法可以改变access modifier的类型。

Therefore, the return type can be same or different. Just make sure the number or type of input parameters are different.

### static & final

#### static  

variable, method, class, block  
static variables and methods are on the class level which is shared with the objects.

* Static variable is initialized only once, at the start of the execution.
* Static method can be invoked without the need for creating an instance of a class.
* Only nested/inner classes can be static.  
A static class cannot access non-static members of the Outer class. It can access only static members of Outer class.

There is no global variable in Java. The so called "Global variable" declared after class and outside of methods is actually a class-level variable, which called instance variable.  
具体见Java III文章。

#### final

**final变量一旦初始化后不能重新赋值**  
**final方法不能被复写，但是可以被继承**  
**final类不能被继承**: A lot of classes in Java are final to keep security, like String class.

* variable: cannot change the value of final variable
* method: cannot override final method, final method is inherited
* class: cannot extend final class

1)blank final variable:  
A final variable that is not initialized at the time of declaration. It can be initialized only in constructor

2)static blank final variable:  
static final variable that is not initialized at the time of declaration. It can be initialized only in static block.

```java
static final int num;
static{ num = 10; }
```

#### static final

static final variables are constants. Constant variables names should be Capital.
</br>

### Package

Package is to encapsulate a group of classes, sub-packages and interfaces.  
Packages are used for:

* Prevent name conflicts
* Provide control access  
**protected and default are package-level.**

```java
//创建package，通常小写字母，必须放在第一行
package xxx;

import xxx.*; //*通配符
```

### Access modifier

restrict the scope of class, method, variable.  
**class cannot be private or protected**, unless the class is inner class.

4 types: default(no keyword), private, protected, public.

{% asset_img AccessModifier.png access_modifer %}

public > protected > default > private  
对于protected：只有不同package的非子类不能获取  
对于default：只有相同的package才能获取

### Interface

(achieve polymorphism)

* only contain method signatures and fields, cannot contain the implementation of the methods.
* public or default access modifier.
* class that implements an interface must implement all the methods declared in the interface.  
Doesn't need to declare the variables of the interface. Only the methods.
* a class can implement multiple interfaces
* interface cannot be instantiated
* interface can inherit another interface

Those types can implement Interface:

```html
class
abstract class
nested class
Enum
```

Eg.

```java
public interface A {
  ...
}
//注意interface是 EXTENDS interface！
public interface B extends A {
  ...
}
```

```java
public interface Vehicle {
  void changeGear(int a);
  void applyBrakes(int a);
}

class Bicycle implements Vehicle {
  int speed, gear;

  @override
  public void changeGear(int newGear){
    ...
  }

  @override
  public void applyBrakes(int decrement){
    speed -= decrement;
  }
}
```

once a class implements a java interface, can assign an instance of this class to a interface-type variable:

```java
//vehicle is an interface, bicycle is a class
Vehicle veh = new Bicycle();
veh.changeGear(1);
```

#### Generic Interface

Generic interface can be typed - it can be specialized to work with a specific type when used.

Eg, 对比着看一下三个block里的代码：  
Ordinary interface:

```java
public interface MyProducer(){
  public Object produce();
}

public class CarProducer implements MyProducer {
  @Override
  public Object produce(){
    return new Car();
  }
}

MyProducer car_producer = new CarProducer();
//这里用到了 cast, 即(Car)
Car car = (Car) car_producer.produce();
```

But using Generic interface < T >, there is no need to cast the object returned from produce() method:  
因为不同class implement interface method后的return type会不会不同，所以可以在interface里用< T >来解决。

```java
public interface MyProducer<T>{
  public T produce();
}

public class CarProducer<T> implements MyProducer<T> {
  public T produce(){
    return (T) new Car();
  }
}

MyProducer<Car> car_producer = new CarProducer<Car>();
Car car = car_producer.produce();

MyProducer<String> car_producer_str = new CarProducer<String>();
String car_str = car_producer_str.produce();
```

Can lock down the generic type of the MyProducer interface when implementing it:

```java
public class CarProducer implements MyProducer<T> {
  public Car produce(){
    return new Car();
  }
}
```

</br>

### Enum

```java
//the enum constants should be in uppercase letters
//xxx相当于是这个group of constants的type name
enum XXX {AA, BB, CC, DD};
XXX myEnum = XXX.A;
```

enum type has a values() method

```java
//loop through
for(XXX myEnum : XXX.values()) {
  ...
}
```

### Abstract

(achieve polymorphism)

abstract class and abstract methods.  
abstract classes **must be extended**, abstract methods **must be overriden**.

**1) Abtract class**  
It is similar to interface, except that it can contain method implementation.  
It provides common method implementation to all the subclasses or to provide default implementation.

* abstract class **cannot be instantiated**!
* abstract class can implement interfaces without providing implementation of interface methods.
* abstract class can have constructor.
* abstract class can have static members.

**2) Abstract method**

* abstract method doesn't have a body, so no curly braces!

**3) 关系**

* abstract methods must exist in a abstract class.
* abstract class can have no abstract methods.
* **subclass of abstract class must implement all the abstract methods**. But if the subclass is also an abstract class, it can choose to implement or not.

抽象class里可以没有抽象method，但有抽象method的class一定是抽象class。

Eg1:

```java
public abstract class Person {
  private String name, gender;
  public Person(String name, gender){
    this.name = name;
    this.gender = gender;
  }

  //abstract method has no body
  public abstract void work();

  public void changeName(String newName){
    this.name = newName;
  }
}

public class Employee extends Person {
  private int id;
  public Employee(String name, String gender, int id){
    super(name, gender);
    this.id = id;
  }

  public void work(){
    if(id == 0) ...
    else ...
  }
}

public static void main(String[] args){
  Person student = new Employee("John", "Male", 0);
  student.work();
}
```

Eg2:  
假设Wolf -> abstract Canine -> abstract Animal。如果Animal中有一个abstract method在Canine中被implement了，那么Wolf就不一定要implement这个method了。但是如果Canine没有implement它，那么Wolf就一定要implement它。  
总结就是abstract super class的subclass chain中的第一个class一定要implement superclass里的abstract methods，后面的就可以随便了。但如果subclass is also an abstract class, 可以选择不implement。

### Four Pillars

* Encapsulation[ɪn,kæpsə'leɪʃən]  
data hiding, wrap variable and methods in a single unit.
* Polymorphism[,pɑlɪ'mɔrfɪzm]  
method override and overload  
具体：different classes can implement the same interface. Each of these classes can provide its own implementation of the interface.
* Abstraction  
detail hiding, only show the relevant details, reduce complexity, act as blueprint.  
achieved by **interface and abstract classes**.
* Inheritance  
class cannot inherit multiple classes

### The way Java works

source code(.java file) -> compiler(create .class file) -> .class file(is made up by bytecode) -> JVM reads and runs bytecode.

#### JVM

Java virtual machine is a run-time engine to run Java applications, it is an **interpreter by executing java code**.

1) diff between JDK, JRE, JVM  
{% asset_img relation.jpg relation %}

JRE是橘黄色的方框里包裹的全部内容，JDK是整个蓝色框里包裹的全部内容。

#### heap & stack

**stack** memory：  
指程序里有一个method时，会为这个方法单独分配一块私属存储空间，用于存储这个方法内部的局部变量。当这个方法结束时，分配给这个方法的stack会释放，这个stack中的变量也将随之释放。  
stack memory is used for static memory allocation and the execution of a thread. 可以理解成一个thread一个stack，复制这个thread上的memory。  
It contains primitive values that are specific to a method and references to the objects in the heap referred from the method.

**heap**：  
stores objects and classes, new出来的对象都放在heap里，它不会随方法的结束而消失。 
方法中的**局部变量使用final修饰后，放在heap中**，而不是stack中。

**比较**：

* Objects stored in heap are globally accessible; stack memory cannot be accessed by other threads.
* heap memory lives during the application execution; Stack memory is short-lived.
* Stack memory size is less than Heap memory, but faster.

#### Garbage Collection

GC **Runs on the heap memory** to **free the memory used by objects** that doesn't have reference.

JVM heap memory is divided into two parts - Young generation and old generation：  
When the young generation is filled, GC is performed called **Minor GC**.  
Old Generation memory contains the objects that are long-lived and survived after many rounds of Minor GC. When old generation memory is full, **Major GC** performs.

### Collection

A Collection represents a single unit of objects.

```html
interfaces: Set, Map, List, Queue, Deque
classes:  Vector, ArrayList, LinkedList, Stack
          PriorityQueue, ArrayDeque,
          HashSet, LinkedHashSet, TreeSet
```

{% asset_img hierarchy.png hierarchy %}

Map does not extend from java.util.Collection, but they're still considered to be part of collection:  
{% asset_img mapDemo.png map %}

虚线表示implements，实线表示extends
