---
title: SpringNote1
date: 2020-05-25 15:22:43
tags: spring
---

### 1. Core Framework

**AOP**: Aspect-oriented programming
add functionality to objects declaratively
allows you to create these applicationwide service
-Logging, security, transactions...

**Core Container**:
Beans, Core, SpEL(spring expression language), Context(hold beans in memory)

**Infrastructure**:
AOP, Aspects, Instrumentation(to work with ), Messaging

**Data Access Layer**:
JDBC, ORM(Object to Relational Mapping, Integration with Hibernate and JPAs), 
Transactions(heavy use of AOP), OXM(), JMS(Java Message Service, to send async messages to a message queue)

Web Layer (Home of spring mvc framework)
Servlet, WebSocket, Web, Portlet

Test Layer:(TDD)
Unit, Integration, Mock


### Spring Projects
spring.io
add-on modules: spring cloud, data, security, batch...


### Set up environment

1.JDK(Java Development Kit) installed. Spring 5 require java 8+
2.Java application server(eg. tomcat server) for web development.
Java IDE(Integrated Development Enviroment) eg.eclipse IDE

Tomcat: choose Full type

After tomcat run(install 9, spring 5 doesn't work on 10 currently because of package rename), go to localhost:8080 to check if running successfully.

How to stop it? In main menu right click, or go to system settings/service/local services

3.Connect Tomcat and Eclipse
On Eclipse, there is a Servers Tab below. Click the link, and choose Apache/Tomcat 9.0

4.Create eclipse project
  First change: Window -> Perspective -> Java
download spring jar files
add jar files to eclipse project and build path
Right click on Project name, choose Properties -> Java Build Path -> Library Tab, choose ClassPath
-> Add jars, select all the jars under the lib folder

### Spring container
Primary functions:
1) Create and manage objects (IOC - inversion颠倒 倒置 of control)
2) Inject object's dependency

Configure Spring container:
1) XML configuration file
2) Java Annotation
3) Java Source code 


Spring Development Process:
1) Configure Spring Beans
```xml
<beans ..>
    <bean id="xxx"
          class="fully qualified name of class(packageName.className)">
    </bean>
</beans>
```
Right click and choose copy fully qualified className in Eclipse.

2) Create Spring Container(or known as ApplicationContext)
There are many specialized applicationContext
```java
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

//to close the context
context.close();
```
3) Retrieve Beans from Container
```java
Coach theCoach = context.getBean("xxx", Coach.class);
//xxx is the bean id, Coach is the interface, while bean class is the implementation
//spring will cast the object
```

### Dependency Injection
outsource交办 the construction and injection of objects to spring.

Two most common injections:
1.Constructor Injection

config file:
```xml
<!-- define the dependency -->
<bean id="abc"
      class="the dependency name">
</bean>

<!-- inject the dependency -->
<bean id="xxx"
      class="xxxx">
    <constructor-arg ref="abc">
</bean>
```

Eclipse: Right click, choose Source-> Generate Constructor using Fields
Then it will generate a contructor automatically.


Main method:
```java
public static void main(String[] args) {
    //load the spring config file
    ClassPathXmalApplicationContext context = new ClassPathXmalApplicationContext("applicationContext.xml");
    
    //retrieve bean from spring container
    Coach theCoach = context.getBean("myCoach", Coach.class);
    
    //call methods on the bean
    System.out.println(theCoach.getDailyWorkout());
    
    //close the context
    context.close();
}
```

2.Setter Injection Step
a)Create setter method in class for injections
b)Configure the dependency injection in spring config file

```xml
//xml
<!-- bean for dependency -->
<bean id="abc"
      class="the dependency name">
</bean>

<!-- setter injection -->
<bean id="xxx"
      class="xxxx">
    <property name="dependencyName" ref="abc">
    <property name="dependencyName", value="xxx">
</bean>
```

```java
//java, capitalize the first letter of dependencyName defined in xml
public void setDependencyName(...) {

}
```

Inject values from Properties Files
1) Create Properties Files
Create a text file
```java
fooEmail = helloworld@gmail.com
```
2) Load File in Spring config
```xml
<context: property-placeholder location="classpath: full name of properties file" />
```
3) Reference values from Properties File
```xml
<properties name="emailAddress" value="${fooEmail}" />
```

### Bean Scopes
Scope refers to the lifecycle of a bean, like how long bean live, how many instances are created, how is bean shared in the spring environment.

Default scope: *Singleton*
Spring container creates only one instance of the bean. It is cached in memory.
All the requests for this bean will return a shared reference to the same bean


Explicitly specify the scope:
```xml
<!-- prototype means a new object is created for each request -->
<bean id="xxx"
      class="xxxx">
      scope="prototype" />
</bean>
```

During Bean initialization and destruction, can add customized code, this is hook.
The method can not accept any arguments, should be no-arg.
```xml
<!-- prototype means a new object is created for each request -->
<bean id="xxx"
      class="xxxx">
      init-method="methodName" />
      destroy-method="methodName" 
</bean>
```

### Annotations
Annotations are special labels/markers added to Java class, provide meta-data about the class
Processed at compile time or run-time for special processing

In spring, configure the spring beans with annotations to minimize the xml configuration
spring will scan the java classes for special annotations, automatically register the beans in the spring container.

Process:
1) Enable component scanning in Spring config file
2) Add @Component to the java classes
3) retrieve bean from container

Eg.

```xml
<beans ...>
    <context:component-scan base-package="xxx the package name which will be scaned" />
</beans>
```

```java
@Component("here is the bean id")

//Spring supports the default bean id
//it change the first letter of class name to lowercase.
@Component
public class BaseballCoach implements Coach {
    ...
}
```

### Depenency Injection with Annotation and Auto-wiring
auto-wiring:
spring looks for a class that matches the property(by types: class or interface), then inject it automactically.

Autowiring injection types:
- constructor injection
- setter injection
- field injection

#### 1.constructor injection process
1) define the dependency interface and class
```java
//interface
public interface FS {
    public String getFortune();
}

//class
@Component
public class HappyFS implements FS {
    public String getFortune() {
        ....
    }
}
```

2) create a constructor in class for injections
3) configure the dependency injection with @Autowired

```java
@Component
public class BaseballCoach implements Coach {
    private FS fs;
    //constructor, spring inject happyFS(injection) here
    @Autowired
    public BaseballCoach(FS fs) {
        this.fs = fs;
    }
}
```

### Spring MVC
Model-View-Controller



### Request params and Request mapping

```java
@RequestMapping("/processFormVersion")
public String getPerson(HttpServletRequest request, Model model) {
    String name = request.getParameter("studentName");
}
```

```java
@RequestMapping("/processFormVersion")
public String getPerson(@RequestParam("studentName") String name, Model model) {
    //can use variable "name" here
}
```



Spring object factory -> configuration

spring container
primary functions:
1) create and manage objects(IoC)
2) inject objects' dependencies(Dependency injection)	

configurating spring container (3 ways)
1) xml configuration file(legacy)
2) java annotations
3) java source code

spring development process

1) cnfigure spring beans
spring beans is a java object
xxx.xml file:
```java
<beans ... >
	<bean id="can be used to retrieve the bean"
		class="full class name">
	</bean>
</beans>
```
2) create a spring container(the official term is ApplicationContext)
```java
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("xxx.xml");
```
3) retrieve beans from spring container
```java
Coach theCoach = context.getBean(
	"xxx id", Coach.class); Coach.class is interface here
```




### Spring dependency injection

dependency : = helper objects
 

Injection types: (many types), we learn
1. constructor injection

1) define the dependency interface and class
```java
//interface
public interface FortuneService {
    public String getFortune();
}

//class
public class HappyFortuneService implements FortuneService {
    public String getFortune() {
        return "Today is your lucky day";
    }
}
```

2) create a constructor in your class for injections

```java
public class BaseballCoach implements Coach {
    private FortuneService fs;
    //constructor
    public BaseballCoach (FortuneService theFS) {
        fs = theFS;
    }
}
```

3) configure the dependency injection in spring config file
```xml
<!-- define the dependency -->
<bean id="myFS" class="HappyFortuneService class path"> </bean>

<bean id="myCoach" class="xxxxx">
<constructor-arg ref="myFS" /> 
<!-- inject the dependency -->
</bean>	

```

2. setter injection
3. auto-wiring in the annotations 















