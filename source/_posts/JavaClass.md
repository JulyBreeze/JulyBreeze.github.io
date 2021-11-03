---
title: Java II - Four Pillar
date: 2019-08-30 11:35:12
tags: java
---


### I. Inheritance

keyword: **extends**

Subclass extends the superclass, it can inherit the members of superclass.  
Private members cannot be inherited.  
Subclass can add new methods and instance variables of its own, it can override the methods it inherits from.  

Def. The members of a class include the variables and methods in the class + anything inherited from a superclass.

Java does not support multiple inheritance using classes, one class can only extend one class but can implement multiple interfaces.(Because of Deadly Diamond of Death problem)

Example:

```java
class Animal {
  makeNoise();
  eat();
  sleep();
  roam();
}

class Canine extends Animal {
  roam();
}

class Wolf extends Canine{
  makeNoise();
  eat();
}
```

```java
Wolf w = new Wolf();
w.makeNoise(); //调用的是Wolf class里的
w.roam(); //调用的是Canine里的
w.eat(); //调用的是Wolf class里的
w.sleep(); //调用的是Animal里的
```

The lowest one wins! JVM starts look first in the wolf class. If it doesn't find, it will walk back up the inheritance hierarchy until it finds a match.

One of the advantages of inheritance is when you only change the code in superclass, the subclasses don't have to be compiled in order to use the new version of the superclass. So put common code in one place, when you want to change that behavior, you can modify it in only one place, and everybody else can see that change.

#### class

Three ways to prevent a class from being subclassed:

* access control  
**A class can't be marked as private**. Inner class is the class as a member of other class, **the inner class can be made private**.
* final  
A final class cannot be inherited, it is the end of inheritance line.
* has only private constructors, which also makes the class cannot be substantiated.

#### keyword - super

If don't want to completely override the superclass method, just want to add more stuff, use keyword super by using super.xxxmethodName().  
Eg.

```java
public void roam() {
  super.roam(); //first calls the inherited version of roam
  //my own roam stuff //then come back to do my own code
}
```

#### Is-A, Has-A

1)Is-A (one direction)  
X extends Y, then X is-A Y.  
Eg. Trangle is-A Shape.

2)Has-A  
X has-A Y, means X has a reference to Y, but X does not extend Y, and vice-versa.  
Eg. Bathroom has-A Tub.

```java
class Bathroom {
  Tub bathtub;
  ...
}

class Tub {
  ...
}
```

#### Design with Inheritance

1)Follow two rules: IS-A and HAS-A!!! Use IS-A to verify your inheritance hierarchy!

Reusable code can be extracted and put in a class.
Eg. The printing code should be in a class Printer. All printable objects can take advantage of via a HAS-A relation.

2)It makes sense to keep inheritance tree shallow.

</br>

### II. Polymorphism

先回顾一下the way declare a reference and create an object:

```java
Dog myHusky = new Dog();
```

* JVM allocates a reference variable named myHusky
* JVM allocates a space in heap for a new Dog object
* Assign this new object to variable.

**With Polymorphism**:  
1)the reference type and object can be different

```java
Animal myHusky = new Dog();
```

**The reference type can be a superclass of the actual object type!!!**  
也就是说可以把子类的object赋值给reference type为父类的变量，即使父类是abstract class！

```java
Superclass variable = new Subclass object;
```

Eg:

```java
Animal[] animals = new Animal[3];
animals[0] = new Dog();
animals[1] = new Cat();
animals[2] = new Wolf();

for(int i = 0; i < 3; i++){
  animals[i].eat();
  animals[i].roam();
}
```

当i = 0时，get the dog's eat(); 当i = 1时，get the cat's eat().

2)**Have polymorphic arguments and return types!**

Eg.

```java
//Vet兽医
class Vet {
  public void command(Animal a) {
    a.makeNoise();
  }
}

class PetOwner {
  public void start() {
    Vet v = new Vet();

    Dog d = new Dog();
    Cat c = new Cat();

    v.command(d); //Dog's makeNoise() works
    v.command(c); //Cat's makeNoise() works
  }
}
```

command() 接收的是一个Animal type的argument。  
As long as the argument you pass in is a subclass of Animal, it will work.

Can write code that doesn't have to change when introducing new subclass types. Vet class is written without any knowledge of the new Animal subtypes.

也就是说如果把method arguments设为父类type，就可以传入子类object作为参数，包括return type也是。**这个非常方便！！**

### III.万物之母Object

Every class in Java extends class **Object**!!

Object class is a non-abstract class, because it's got method implementation that all classes can inherit and use out-of-the-box.  
The most use of instance of type Object is for thread synchronization.

Object class里常见的四个methods：

```java
Class getClass();
boolean equals(Object o);
int hashCode();
String toString();
```

后面这三个methods经常被override。

虽然推荐polymorphic types, 但不能把所有的methods的argement和return type都设为Object type。例子见下：

```java
Dog a = new Dog();
Dog sameDog = getObject(a);//这里会发生编译错误

public Object getObject(Object o) {
  return o;
}
```

分析：  
getObject()函数返回的type是Object，而compiler不允许把return的内容赋值给其它非Object类型的类型！  
简单点说就是，如果一个instance被一个类型为Object的变量所引用，那么这个instance就不能再被它的真实类型的变量所引用。

改成下面这样就可以通过编译了：

```java
Dog a = new Dog();
Object sameDog = getObject(a);

public Object getObject(Object o) {
  return o;
}
```

<font size=4>**Compiler decides whether a method can be called based on the reference type, not on the actual object type！！！**</font>  
举例：

```java
//假设一个list设定为hold Object-type的元素
//但是我们往里add的时候add的是Dog-type的instance
//但在compiler眼中，obj仍然是Object-type的instance。
Object obj = list.get(0);
int num = obj.hashCode();
obj.bark(); //编译错误，因为Object class里没有bark()
```

compiler不知道这是Dog类型的instance，即使放入list的时候是Dog-type instance.  
我在想，为什么前面那个animal的例子就行，这个不行。是不是list.get(0)不要赋值给Object，而直接用list.get().bark就不会出错了呢。

#### inner object

由于每个class都继承自Object，所以对象object contains everything it inheris from each of its superclass，所以每个object也是instance of Object.

比如：

```java
ButtonBoard bb = new ButtonBoard();
Object oo = bb;
```

bb这个reference可以获取Object和ButtonBoard的所有accessible(通常指public) members。虽然oo也和bb一样，指向了同一个object，但是oo只能access Object里的members，不能获取到ButtonBoard里的独有members。原因见上面加粗字体。

#### cast

A reference variable of Object-type cannot be assigned to any other reference type without a cast.

```java
Dog d = (Dog) list.get(0);
```

A cast can be used to assign a reference variable of one type to a reference variable of subtype. 就是说借助cast，可以把父类type的obj赋值给其子类type的变量。

### IV. Interface

keywords: interface, implements

interface里所有的methods都是默认abstract and public，可以不写出来。  
interface也可以inherit one or more interfaces, 用**extends** keyword.

class implements interface, interface extends interface.

```java
public interface A {}
public interface B extends A {}
public class implements A {}
```

### V. How to choose class, subclass, abstract class, interface?

1. If your class doesn't pass IS-A test for any other classes, choose CLASS.

2. If you need to make a specific version of a class and need to override or add some new behaviors, choose SUBCLASS.

3. If you want to define a template for a group of subclasses, and you have at least some implementation code that all subclasses could use, or you don't want a class to be instantiated, choose ABSTRACT CLASS.

4. If you want to define a role that other classes can play, regardless of where those classes are in the inheritance tree, choose INTERFACE.

### VI.Life of an Object

**1.Defintion**

* Instance variables  
declared inside a class but outside the method
* Local variables  
declared inside a method, including method parameters

**2.Memory**  
Two areas: objects live in heap; method invocations调用 and local variables live in stack.

**3.Constructor**  
The only way to invoke a constructor is with the keyword new. Even abstract class has constructors.  
**Constructors are not inherited.**

The compiler gets involved with constructor-making only if you don't write any constructors. If you write a  constructor with arguments and you still need a constructor with no arguments, you need to build that no-arg constructor yourself.

**4.Constructor chain**  
All instance variables from every class in the inheritance tree have to be declared and initialized, so when a constructor runs, it immediately calls its superclass constructor.

The superclass parts of an objects have to be fully-formed before the subclass parts can be constructed.

总结，父类的constructor先运行，然后是子类的

**5.super()**  
A call to super() in the constructor puts the superclass constructor on the top of the stack, and it is the only way to call a super constructors. If the superclass has overloaded constructors, only the no-arg one is called.

If you don't provide a constructor, the compiler will puts one like:

```java
//this is a constructor
public ClassName(...) {
  super();
}
```

If you provide a constructor but without super(), the compiler will put a call to super() in each of your overloaded constructors. 即super()由compiler默认加上，如果父类里有多个constructor，compiler默认所call的super()调用的是没有参数的那个父类constructor。

super() can have arguments. Eg:

```java
public class Animal {
  private String name;
  public Animal(String theName) {
    name = theName;
  }
}

public class Dog extends Animal {
  public Dog (String name) {
    //it sends the name up to the stack to the Animal constructor
    super(name);
  }
}
```

**6.this()**  
如果有很多的constructors，这些constructors都有common part，如果在每一个constructor中都写上这个common code会比较麻烦。如果希望不管哪一个constructor第一个被invoke时，the real constructor is called to finish that common part, 就可以使用this().

this() and super() **must be in the first line**, so they cannot apprear in constructors at the same time.

```java
class Mini extends Car {
  Color color;
  public Mini(){
    //calls the real constructor, the one has super()
    this(Color.RED);
  }

  //the real constructor
  public Mini(Color c){
    super("Mini");
    color = c;
  }
}
```

**7.life of variables**  
A local variable lives only within the method it is declared in.  
A instance variable lives as long as the object does.

For object:  
If a reference variable stops pointing to an object, that object is eligible for GC, GC will destory some or all of this eligible objects to keep from running out of RAM.
