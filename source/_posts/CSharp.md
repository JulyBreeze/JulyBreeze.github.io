---
title: C#
date: 2019-09-08 13:18:36
tags: C#
---

### I.初步介绍

```c#
using System; //使用了这个System的命名空间
namespace HelloworldApplication {//Namespace declaration
//HelloWorldApplication命名空间包含了HelloWorld class
  class HelloWorld {
    //Main方法是所有C#的入口点
    static void Main(String[] args) {
      Console.WriteLine("hello world");
      Console.ReadKey();
      //ReadKey()针对VS.NET用户的
      //这使得程序会等待一个按键的动作
    }
  }
}
```

逐行分析:

```C#
using System; //一个程序一般有多个using语句
namespace aaa { //一个namespace里有一系列的class
  class bbb {

  }
}
```

C#具有：

* 大小写敏感
* 必须;结尾
* 文件名可以和class的名称不同
</br>

C#的命名规则：

* 字母，下划线，@开头，后面可以跟字母、下划线、数字、@
* 区分大小写
* 关键字前加@后，可以作为标识符

### II.基本语法

#### 1.数据类型

```c#
Value types:
  int, bool, char(括在单引号里), float, double, byte, decimal, long, short ...
  //用sizeof(int)可以得到int类型在机器上的存储尺寸

Reference types:
  object, dynamic, string
  1) Object类型的变量可以被分配任何其他类型。
  //Object变量的类型检查是在编译时发生

  2)dynamic
    //动态类型变量可以存储任何type的值，这类变量的类型检查是在运行时发生的
    dynamic <variable_name> = value;

  3)String
  //两种形式进行被分配
  String str = "hello";//(括在双引号里)
  String str = @"hello\world"; //@表示将转义字符\当成普通字符对待
  //即等价于"hello\\world"
  //@字符串中可以任意换行，换行符及缩进空格都计算在字符串长度之内。

Pointer types:
  声明格式：type* identifier;
  eg:
    char* cptr;
    int* iptr;
```

#### 2.类型转换

隐式：不会导致数据丢失，比如从派生类转换为几类
显式: 会造成数据丢失

内置的类型转换方法：

```c#
ToInt32();
ToBoolean();
ToChar();
ToDecimal();
ToDouble();
ToString();
//...
```

### 3.其他

```c#
int num;
//ReadLine() 只接受字符串格式
num = Convert.ToInt32(Console.ReadLine());
//num的赋值来自用户的输入
```

```c#
sizeof();
typeof();
&a //返回变量a的地址
*a //指向一个变量
is //判断是否是某个类型
  Eg. Ford is Car

as //强制转换，即使转换出问题也不会抛出异常
```

#### 4.特殊数据类型Nullable type

1) ?单问号对int，double，bool等无法直接赋值为null的数据类型进行null赋值

```c#
int? i = 3;
//等同于
Nullable<int> i = new Nullable<int> 3;
```

2) ??双问号用于判断一个variable为null时返回一个指定的值。

```c#
//如果nums1为null，就返回10.5
//否则返回num1的值给num2
double num2 = num1 ?? 10.5;
```

#### 5.String类

属性和常用方法：

```c#
//属性
.Length
.Chars[index] //get the char at a specified position in string

//方法
.ToUpper()
.ToLower()

.Contains() //判断是否包含某个string或char
.Substring(startIndex, Len)

.IndexOf() //找到某个string/char在该string中第一次出现的位置
.LastIndexOf()

.StartsWith()
.EndsWith()

.Insert(index, str) //在index处插入str

.Trim() //删除开始和结尾处的空格
.Remove(startIndex, len) //删除从startIndex开始的长度为len的substring
String.Join(SplitCh, array); //array为要合并的char数组，splitCh为合并后的分隔符
```

### III.Struct结构体

struct是**值类型**的数据结构。    
keyword: struct

struct的特点：

* 结构体可以带有方法，属性，事件
* 不能继承其它的struct和class
* 可以implement多个interface
* 成员不能指定为abstract, virtual 或 protected

Eg:

```c#
struct Books {
  public string title;
  public string author;
  public int book_id;

  public void getValues(String t, String a, int id) {
    title = t;
    author = a;
    book_id = id;
  }
}

//声明
Books b1;
Books b2;

//赋值
b1.title = "C programming";
b1.author = "John";
b1.book_id = 10;

//或者
Books b1 = new Books();
b1.getValues("C programming", "John", 10);
```


### IV.Enum 枚举

```c#
//声明
enum <enum_name> {
  enumeration list(用逗号分隔)
}

//举例
enum Weekends {Sun, Mon};
//每个符号代表一个整数值，默认情况下，第一个枚举符号的值是0
```

### V.输入和输出

System.IO namespace里有不同的class，用来执行文件操作。

```c#
BufferedStream  字节流的临时存储
FileStream      文件的读写和关闭
```

### VI.面向对象设计

#### 1.access modifier

```html
1.Four types
public
private
protected
internal:  the object is accessible only inside its own assembly but not in other assemblies

2.Two combinations
protected internal
private protected
```

#### 2.class

1)构造函数和析构函数  
class的constructor;  
class的destructor，当class的对象超出范围内执行。其名称是class name前加一个~，不返回值，也不带任何参数。  
析构函数用于在结束程序（比如关闭文件、释放内存等）之前释放资源。析构函数不能继承或重载。

```c#
class Line {
  private double length;
  public Line() {
    //...
  }

  ~Line() {
    Console.WriteLine("delete obj");
  }
}
```

2)静态static成员  
当class members为静态时，表示无论有多少个类的对象被创建，只会有一个该静态成员的副本。

static可以修饰变量和函数。静态函数在对象被创建之前就已经存在了。

```c#

```

#### 3.Inheritance

C#不支持多重继承，但是可以继承多个interface

```c#
class childClass : parentClass {
  //...
}
```

1.Interface  
1)接口的成员包括属性、方法和事件。interface默认是public的，继承接口仍然使用引号。  
2)interface的成员不能有 public、protected、internal、private 等修饰符。  
3)当一个接口实现一个接口，这2个接口中有相同的方法时，可用 new 关键字隐藏父接口中的方法。  
4)接口没有构造函数，所以不能使用new来对interface进行实例化。

```c#
public interface xxx {
  ...
}

class aaa : xxx {
  ...
}
```
如果一个interface继承了其他接口，那么该接口的实现类或结构就要实现所有interface的成员。


#### 4.动态polymorphism

动态多态性是通过abstract class和virtual method实现的。

1)abstract class  
如果在class前加上keyword - sealed，表示这个class不能被继承。abstract class不能被声明为sealed。

2)virtual method  
当一个定义在class里的method需要在派生类中实现时，可以使用virtual。

```c#
public class Shape {
  public int height;
  public int width;

  public virtual void Draw() {
    //aaa
  }
}

//派生类
class Circle : Shape {
  public override void Draw() {
    //bbb
  }
}
```

### VII.Data Structure

```c#
ArrayList
HashTable
SortedList //用key和index来访问element，各项总是按照key值排列
Stack
Queue
BitArray
```

### VIII.高级部分

#### 1.Attribut

语法：

```c#
[attribute(positional_parameters, name_parameter = value ... )] element
```

attribute的name和value是放在[]里的，放在它所应用的元素之前

.Net提供了3中预定义的attributes:

* AttributeUsage  
描述如何使用一个自定义的类
* Conditional
* Obsolete

#### 2.Property

#### 3.Reflection

#### 4.Indexer

#### 5.Delegate

#### 6.Event

Event基本上就是一个用户操作，如按键、点击、鼠标移动等等。  
C#使用事件机制实现线程间的通信。

1.通过event使用delegate

包含event的class用于发布事件，称为publisher class；其他接受该event的类被称为subscriber class。  
event使用publisher-subscriber模型。
