---
title: Java III - Risky & Collections & others
date: 2019-08-30 17:47:16
tags: java
---

### I. Static

Math class does not have any instance variables, its constructor is private, which means that you cannot use new keyword to make an instance of Math class.  
理由很简单：the math methods's behavior acts on the arguments, but are never affected by an instance variable state.

所以可以用**static**来表示.

Call a static method using a class name. 比如：

```java
Math.abs();
```

A static variables live in class level instead of the object.  
Eg.

```java
class Duck {
  //this variable count is shared by all instances of Duck class
  private static int count = 0;
  public Duck(){
    count++;
  }
}
```

所以这个变量会持续increment，不会在每次有新的instance出现时被置为0，因为all instances of the same class share a single copy of this static variable.

### II. Risky Behavior

#### try-catch-finally

Try/catch tells compiler that you know an exceptional thing could happen in the method you're handling.  
If there are multiple exceptions, stack the catch block in order.

A finally block is where you put code that must run regardless of an exception. 即使try/catch里有return，finally block still runs. The flow will jumps to the finally, then back to the return.

```java
try {
  //put risky thing here
}catch(AException ex1){
  //if the exceptional situation happens, what should do
}catch(BException ex2){
  
}finally {
  //the code here always run! 
}
```

#### Exception

Exception is a class.  
keyword: throws

Eg.

```java
//risky method could throw the exception
//注意上面是throws，下面if里是throw
public void takeRisk() throws BadException {
  if(somethingWrong) {
    throw new BadException();
  }
}

//this method, which is caller, calls the risky method
//If something wrong, the risky method will throw an exception
//to the caller.
public void crossFingers () {
  try {
    anObject.takeRisk();
  } catch (BadException ex) {
    System.out.println("something wrong");
    ex.printStackTrace();
  }
}
```

The exceptions compiler cares about are called "checked exceptions". But RuntimeException and the subclasses of it will not be checked by compiler.

### III. Data Structure

#### 1.Generic types

angle brackets< > represents generics. The main point of generics is to let you write type-safe collections.

下面是Collection.sort()的api文档：

```java
public static <T extends Comparable<?super T>> void sort(List<T> list)
//<T extends Comparable 表示T must be of type of Comparable
// <?super T>表示the type parameter must be of type T or T's supertype
//写文档需要，extends表示extends or implements
```

Comparable and Comparator are interface:

```java
public interface Comparable<T> {
  int compareTo(T o);
}

public interface Comparator<T> {
  int compare(T o1, T o2);
}
```

#### compare two objects

The default .hashCode() will give each object a unique number, ususally based on the object's memory address on the heap, so no two objects will have the same hashcode.

要比较两个object是否相等，先比较地址，再比较内容。  
1)check reference equality  
use == operator

== operator compares the bits in the variables. If both references point to the same object, the bits wil be identical.

2)check object equality  
override .hashCode() and .equals() which inherit from Object class.

Override the .hashCode() to make sure that two equivalent(指内容上的) objects return the same hashCode.

#### 具体原理

1)How a hashset check for duplicates?  
It uses the object's hashcode to determine where to put the object in the set. The hashcode is like a label on the buckets where hashset stores elements.

It also compares the object's hashcode with all others. But two objects with the same hashcode may not be equal, so if hashset finds a matching hashcode, it will then call .equals() method to check if the object it's looking for is in that buckets.

2)How a treeset check for duplicates and sort?  
Treeset is similar to hashset in that it prevents duplicates.  
It uses each object's compareTo() method for sort.  

#### 例题

```java
Animal[] animals = {new Dog(), new Cat()};
Dog[] dogs = {new Dog(), new Dog()};

takeAnimals(animals);
takeAnimals(dogs);

public void takeAnimals(Animal[] animals){
  for(Animal a : animals){
    a.eat();
  }
}
```

如果把array换成arrayList：

```java
ArrayList<Animal> animals = new ArrayList<>();
animals.add(new Dog());
animals.add(new Cat());

ArrayList<Dogs> dogs = new ArrayList<>();
dogs.add(new Dog());
dogs.add(new Dog());

takeAnimals(animals);
takeAnimals(dogs); //这边会出现compile error

public void takeAnimals(ArrayList<Animal> animals){
  for(Animal a : animals){
    a.eat();
  }
}
```

If you declare a method to take ArrayList< Animal >, it can take only ArrayList< Animal >, not ArrayList< Dog >.

为什么？  
假设允许takeAnimals(dogs)，如果takeAnimals()函数里有一个animals.add(new Cat())的代码的话，那不就会冲突了吗，所以compiler干脆就不允许。

但是为什么array就不害怕这种情况发生呢？  
因为Array types are checked again at runtime, but collection type checks happen only at compile. 所以如果代码如下：

```java
Dog[] dogs = {new Dog(), new Dog()};
takeAnimals(dogs);

public void takeAnimals(Animal[] animals){
  animals[0] = new Cat();
}
```

就会在runtime出现错误。
</br>

有没有一种解决方法可以兼容不同的type呢？  
如果把takeAnimal函数写成如下：

```java
//extends means extends or implements depending on the type
public void takeAnimals(ArrayList<? extends Animal> animals){
  ...
}
```

如果declaration中出现了wildcard < ? >，那么compiler只允许在函数的body中对list中的元素进行操作，但是不允许往list中添加元素。
