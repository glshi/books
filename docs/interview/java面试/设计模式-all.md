<!-- GFM-TOC -->
* [一、概述](#一概述)
* [二、创建型](#二创建型)
    * [1. 单例（Singleton）](#1-单例singleton)
    * [2. 简单工厂（Simple Factory）](#2-简单工厂simple-factory)
    * [3. 工厂方法（Factory Method）](#3-工厂方法factory-method)
    * [4. 抽象工厂（Abstract Factory）](#4-抽象工厂abstract-factory)
    * [5. 生成器（Builder）](#5-生成器builder)
    * [6. 原型模式（Prototype）](#6-原型模式prototype)
* [三、行为型](#三行为型)
    * [1. 责任链（Chain Of Responsibility）](#1-责任链chain-of-responsibility)
    * [2. 命令（Command）](#2-命令command)
    * [3. 解释器（Interpreter）](#3-解释器interpreter)
    * [4. 迭代器（Iterator）](#4-迭代器iterator)
    * [5. 中介者（Mediator）](#5-中介者mediator)
    * [6. 备忘录（Memento）](#6-备忘录memento)
    * [7. 观察者（Observer）](#7-观察者observer)
    * [8. 状态（State）](#8-状态state)
    * [9. 策略（Strategy）](#9-策略strategy)
    * [10. 模板方法（Template Method）](#10-模板方法template-method)
    * [11. 访问者（Visitor）](#11-访问者visitor)
    * [12. 空对象（Null）](#12-空对象null)
* [四、结构型](#四结构型)
    * [1. 适配器（Adapter）](#1-适配器adapter)
    * [2. 桥接（Bridge）](#2-桥接bridge)
    * [3. 组合（Composite）](#3-组合composite)
    * [4. 装饰（Decorator）](#4-装饰decorator)
    * [5. 外观（Facade）](#5-外观facade)
    * [6. 享元（Flyweight）](#6-享元flyweight)
    * [7. 代理（Proxy）](#7-代理proxy)
* [参考资料](#参考资料)
<!-- GFM-TOC -->



# 设计模式

本节介绍设计模式相关内容。

# 一、概述

设计模式是解决问题的方案，学习现有的设计模式可以做到经验复用。

拥有设计模式词汇，在沟通时就能用更少的词汇来讨论，并且不需要了解底层细节。

# 二、创建型

## 1. 单例（Singleton）

### Intent

确保一个类只有一个实例，并提供该实例的全局访问点。

### Class Diagram

使用一个私有构造函数、一个私有静态变量以及一个公有静态函数来实现。

私有构造函数保证了不能通过构造函数来创建对象实例，只能通过公有静态函数返回唯一的私有静态变量。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/eca1f422-8381-409b-ad04-98ef39ae38ba.png"/> </div><br>

### Implementation

#### Ⅰ 懒汉式-线程不安全

以下实现中，私有静态变量 uniqueInstance 被延迟实例化，这样做的好处是，如果没有用到该类，那么就不会实例化 uniqueInstance，从而节约资源。

这个实现在多线程环境下是不安全的，如果多个线程能够同时进入 `if (uniqueInstance == null)` ，并且此时 uniqueInstance 为 null，那么会有多个线程执行 `uniqueInstance = new Singleton();` 语句，这将导致实例化多次 uniqueInstance。

```java
public class Singleton {

    private static Singleton uniqueInstance;

    private Singleton() {
    }

    public static Singleton getUniqueInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
}
```

#### Ⅱ 饿汉式-线程安全

线程不安全问题主要是由于 uniqueInstance 被实例化多次，采取直接实例化 uniqueInstance 的方式就不会产生线程不安全问题。

但是直接实例化的方式也丢失了延迟实例化带来的节约资源的好处。

```java
private static Singleton uniqueInstance = new Singleton();
```

#### Ⅲ 懒汉式-线程安全

只需要对 getUniqueInstance() 方法加锁，那么在一个时间点只能有一个线程能够进入该方法，从而避免了实例化多次 uniqueInstance。

但是当一个线程进入该方法之后，其它试图进入该方法的线程都必须等待，即使 uniqueInstance 已经被实例化了。这会让线程阻塞时间过长，因此该方法有性能问题，不推荐使用。

```java
public static synchronized Singleton getUniqueInstance() {
    if (uniqueInstance == null) {
        uniqueInstance = new Singleton();
    }
    return uniqueInstance;
}
```

#### Ⅳ 双重校验锁-线程安全

uniqueInstance 只需要被实例化一次，之后就可以直接使用了。加锁操作只需要对实例化那部分的代码进行，只有当 uniqueInstance 没有被实例化时，才需要进行加锁。

双重校验锁先判断 uniqueInstance 是否已经被实例化，如果没有被实例化，那么才对实例化语句进行加锁。

```java
public class Singleton {

    private volatile static Singleton uniqueInstance;

    private Singleton() {
    }

    public static Singleton getUniqueInstance() {
        if (uniqueInstance == null) {
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

考虑下面的实现，也就是只使用了一个 if 语句。在 uniqueInstance == null 的情况下，如果两个线程都执行了 if 语句，那么两个线程都会进入 if 语句块内。虽然在 if 语句块内有加锁操作，但是两个线程都会执行 `uniqueInstance = new Singleton();` 这条语句，只是先后的问题，那么就会进行两次实例化。因此必须使用双重校验锁，也就是需要使用两个 if 语句：第一个 if 语句用来避免 uniqueInstance 已经被实例化之后的加锁操作，而第二个 if 语句进行了加锁，所以只能有一个线程进入，就不会出现 uniqueInstance == null 时两个线程同时进行实例化操作。

```java
if (uniqueInstance == null) {
    synchronized (Singleton.class) {
        uniqueInstance = new Singleton();
    }
}
```

uniqueInstance 采用 volatile 关键字修饰也是很有必要的， `uniqueInstance = new Singleton();` 这段代码其实是分为三步执行：

1. 为 uniqueInstance 分配内存空间
2. 初始化 uniqueInstance
3. 将 uniqueInstance 指向分配的内存地址

但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1>3>2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T<sub>1</sub> 执行了 1 和 3，此时 T<sub>2</sub> 调用 getUniqueInstance() 后发现 uniqueInstance 不为空，因此返回 uniqueInstance，但此时 uniqueInstance 还未被初始化。

使用 volatile 可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。

#### Ⅴ 静态内部类实现

当 Singleton 类被加载时，静态内部类 SingletonHolder 没有被加载进内存。只有当调用 `getUniqueInstance()` 方法从而触发 `SingletonHolder.INSTANCE` 时 SingletonHolder 才会被加载，此时初始化 INSTANCE 实例，并且 JVM 能确保 INSTANCE 只被实例化一次。

这种方式不仅具有延迟初始化的好处，而且由 JVM 提供了对线程安全的支持。

```java
public class Singleton {

    private Singleton() {
    }

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getUniqueInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

#### Ⅵ 枚举实现

```java
public enum Singleton {

    INSTANCE;

    private String objName;


    public String getObjName() {
        return objName;
    }


    public void setObjName(String objName) {
        this.objName = objName;
    }


    public static void main(String[] args) {

        // 单例测试
        Singleton firstSingleton = Singleton.INSTANCE;
        firstSingleton.setObjName("firstName");
        System.out.println(firstSingleton.getObjName());
        Singleton secondSingleton = Singleton.INSTANCE;
        secondSingleton.setObjName("secondName");
        System.out.println(firstSingleton.getObjName());
        System.out.println(secondSingleton.getObjName());

        // 反射获取实例测试
        try {
            Singleton[] enumConstants = Singleton.class.getEnumConstants();
            for (Singleton enumConstant : enumConstants) {
                System.out.println(enumConstant.getObjName());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

```html
firstName
secondName
secondName
secondName
```

该实现可以防止反射攻击。在其它实现中，通过 setAccessible() 方法可以将私有构造函数的访问级别设置为 public，然后调用构造函数从而实例化对象，如果要防止这种攻击，需要在构造函数中添加防止多次实例化的代码。该实现是由 JVM 保证只会实例化一次，因此不会出现上述的反射攻击。

该实现在多次序列化和序列化之后，不会得到多个实例。而其它实现需要使用 transient 修饰所有字段，并且实现序列化和反序列化的方法。

### Examples

- Logger Classes
- Configuration Classes
- Accesing resources in shared mode
- Factories implemented as Singletons

### JDK

- [java.lang.Runtime#getRuntime()](http://docs.oracle.com/javase/8/docs/api/java/lang/Runtime.html#getRuntime%28%29)
- [java.awt.Desktop#getDesktop()](http://docs.oracle.com/javase/8/docs/api/java/awt/Desktop.html#getDesktop--)
- [java.lang.System#getSecurityManager()](http://docs.oracle.com/javase/8/docs/api/java/lang/System.html#getSecurityManager--)

## 2. 简单工厂（Simple Factory）

### Intent

在创建一个对象时不向客户暴露内部细节，并提供一个创建对象的通用接口。

### Class Diagram

简单工厂把实例化的操作单独放到一个类中，这个类就成为简单工厂类，让简单工厂类来决定应该用哪个具体子类来实例化。

这样做能把客户类和具体子类的实现解耦，客户类不再需要知道有哪些子类以及应当实例化哪个子类。客户类往往有多个，如果不使用简单工厂，那么所有的客户类都要知道所有子类的细节。而且一旦子类发生改变，例如增加子类，那么所有的客户类都要进行修改。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/40c0c17e-bba6-4493-9857-147c0044a018.png"/> </div><br>

### Implementation

```java
public interface Product {
}
```

```java
public class ConcreteProduct implements Product {
}
```

```java
public class ConcreteProduct1 implements Product {
}
```

```java
public class ConcreteProduct2 implements Product {
}
```

以下的 Client 类包含了实例化的代码，这是一种错误的实现。如果在客户类中存在这种实例化代码，就需要考虑将代码放到简单工厂中。

```java
public class Client {

    public static void main(String[] args) {
        int type = 1;
        Product product;
        if (type == 1) {
            product = new ConcreteProduct1();
        } else if (type == 2) {
            product = new ConcreteProduct2();
        } else {
            product = new ConcreteProduct();
        }
        // do something with the product
    }
}
```

以下的 SimpleFactory 是简单工厂实现，它被所有需要进行实例化的客户类调用。

```java
public class SimpleFactory {

    public Product createProduct(int type) {
        if (type == 1) {
            return new ConcreteProduct1();
        } else if (type == 2) {
            return new ConcreteProduct2();
        }
        return new ConcreteProduct();
    }
}
```

```java
public class Client {

    public static void main(String[] args) {
        SimpleFactory simpleFactory = new SimpleFactory();
        Product product = simpleFactory.createProduct(1);
        // do something with the product
    }
}
```

## 3. 工厂方法（Factory Method）

### Intent

定义了一个创建对象的接口，但由子类决定要实例化哪个类。工厂方法把实例化操作推迟到子类。

### Class Diagram

在简单工厂中，创建对象的是另一个类，而在工厂方法中，是由子类来创建对象。

下图中，Factory 有一个 doSomething() 方法，这个方法需要用到一个产品对象，这个产品对象由 factoryMethod() 方法创建。该方法是抽象的，需要由子类去实现。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/f4d0afd0-8e78-4914-9e60-4366eaf065b5.png"/> </div><br>

### Implementation

```java
public abstract class Factory {
    abstract public Product factoryMethod();
    public void doSomething() {
        Product product = factoryMethod();
        // do something with the product
    }
}
```

```java
public class ConcreteFactory extends Factory {
    public Product factoryMethod() {
        return new ConcreteProduct();
    }
}
```

```java
public class ConcreteFactory1 extends Factory {
    public Product factoryMethod() {
        return new ConcreteProduct1();
    }
}
```

```java
public class ConcreteFactory2 extends Factory {
    public Product factoryMethod() {
        return new ConcreteProduct2();
    }
}
```

### JDK

- [java.util.Calendar](http://docs.oracle.com/javase/8/docs/api/java/util/Calendar.html#getInstance--)
- [java.util.ResourceBundle](http://docs.oracle.com/javase/8/docs/api/java/util/ResourceBundle.html#getBundle-java.lang.String-)
- [java.text.NumberFormat](http://docs.oracle.com/javase/8/docs/api/java/text/NumberFormat.html#getInstance--)
- [java.nio.charset.Charset](http://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html#forName-java.lang.String-)
- [java.net.URLStreamHandlerFactory](http://docs.oracle.com/javase/8/docs/api/java/net/URLStreamHandlerFactory.html#createURLStreamHandler-java.lang.String-)
- [java.util.EnumSet](https://docs.oracle.com/javase/8/docs/api/java/util/EnumSet.html#of-E-)
- [javax.xml.bind.JAXBContext](https://docs.oracle.com/javase/8/docs/api/javax/xml/bind/JAXBContext.html#createMarshaller--)

## 4. 抽象工厂（Abstract Factory）

### Intent

提供一个接口，用于创建   **相关的对象家族**  。

### Class Diagram

抽象工厂模式创建的是对象家族，也就是很多对象而不是一个对象，并且这些对象是相关的，也就是说必须一起创建出来。而工厂方法模式只是用于创建一个对象，这和抽象工厂模式有很大不同。

抽象工厂模式用到了工厂方法模式来创建单一对象，AbstractFactory 中的 createProductA() 和 createProductB() 方法都是让子类来实现，这两个方法单独来看就是在创建一个对象，这符合工厂方法模式的定义。

至于创建对象的家族这一概念是在 Client 体现，Client 要通过 AbstractFactory 同时调用两个方法来创建出两个对象，在这里这两个对象就有很大的相关性，Client 需要同时创建出这两个对象。

从高层次来看，抽象工厂使用了组合，即 Cilent 组合了 AbstractFactory，而工厂方法模式使用了继承。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/e2190c36-8b27-4690-bde5-9911020a1294.png"/> </div><br>

### Implementation

```java
public class AbstractProductA {
}
```

```java
public class AbstractProductB {
}
```

```java
public class ProductA1 extends AbstractProductA {
}
```

```java
public class ProductA2 extends AbstractProductA {
}
```

```java
public class ProductB1 extends AbstractProductB {
}
```

```java
public class ProductB2 extends AbstractProductB {
}
```

```java
public abstract class AbstractFactory {
    abstract AbstractProductA createProductA();
    abstract AbstractProductB createProductB();
}
```

```java
public class ConcreteFactory1 extends AbstractFactory {
    AbstractProductA createProductA() {
        return new ProductA1();
    }

    AbstractProductB createProductB() {
        return new ProductB1();
    }
}
```

```java
public class ConcreteFactory2 extends AbstractFactory {
    AbstractProductA createProductA() {
        return new ProductA2();
    }

    AbstractProductB createProductB() {
        return new ProductB2();
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        AbstractFactory abstractFactory = new ConcreteFactory1();
        AbstractProductA productA = abstractFactory.createProductA();
        AbstractProductB productB = abstractFactory.createProductB();
        // do something with productA and productB
    }
}
```

### JDK

- [javax.xml.parsers.DocumentBuilderFactory](http://docs.oracle.com/javase/8/docs/api/javax/xml/parsers/DocumentBuilderFactory.html)
- [javax.xml.transform.TransformerFactory](http://docs.oracle.com/javase/8/docs/api/javax/xml/transform/TransformerFactory.html#newInstance--)
- [javax.xml.xpath.XPathFactory](http://docs.oracle.com/javase/8/docs/api/javax/xml/xpath/XPathFactory.html#newInstance--)

## 5. 生成器（Builder）

### Intent

封装一个对象的构造过程，并允许按步骤构造。

### Class Diagram

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/db5e376d-0b3e-490e-a43a-3231914b6668.png"/> </div><br>

### Implementation

以下是一个简易的 StringBuilder 实现，参考了 JDK 1.8 源码。

```java
public class AbstractStringBuilder {
    protected char[] value;

    protected int count;

    public AbstractStringBuilder(int capacity) {
        count = 0;
        value = new char[capacity];
    }

    public AbstractStringBuilder append(char c) {
        ensureCapacityInternal(count + 1);
        value[count++] = c;
        return this;
    }

    private void ensureCapacityInternal(int minimumCapacity) {
        // overflow-conscious code
        if (minimumCapacity - value.length > 0)
            expandCapacity(minimumCapacity);
    }

    void expandCapacity(int minimumCapacity) {
        int newCapacity = value.length * 2 + 2;
        if (newCapacity - minimumCapacity < 0)
            newCapacity = minimumCapacity;
        if (newCapacity < 0) {
            if (minimumCapacity < 0) // overflow
                throw new OutOfMemoryError();
            newCapacity = Integer.MAX_VALUE;
        }
        value = Arrays.copyOf(value, newCapacity);
    }
}
```

```java
public class StringBuilder extends AbstractStringBuilder {
    public StringBuilder() {
        super(16);
    }

    @Override
    public String toString() {
        // Create a copy, don't share the array
        return new String(value, 0, count);
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        StringBuilder sb = new StringBuilder();
        final int count = 26;
        for (int i = 0; i < count; i++) {
            sb.append((char) ('a' + i));
        }
        System.out.println(sb.toString());
    }
}
```

```html
abcdefghijklmnopqrstuvwxyz
```

### JDK

- [java.lang.StringBuilder](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuilder.html)
- [java.nio.ByteBuffer](http://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html#put-byte-)
- [java.lang.StringBuffer](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuffer.html#append-boolean-)
- [java.lang.Appendable](http://docs.oracle.com/javase/8/docs/api/java/lang/Appendable.html)
- [Apache Camel builders](https://github.com/apache/camel/tree/0e195428ee04531be27a0b659005e3aa8d159d23/camel-core/src/main/java/org/apache/camel/builder)

## 6. 原型模式（Prototype）

### Intent

使用原型实例指定要创建对象的类型，通过复制这个原型来创建新对象。

### Class Diagram

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/b8922f8c-95e6-4187-be85-572a509afb71.png"/> </div><br>

### Implementation

```java
public abstract class Prototype {
    abstract Prototype myClone();
}
```

```java
public class ConcretePrototype extends Prototype {

    private String filed;

    public ConcretePrototype(String filed) {
        this.filed = filed;
    }

    @Override
    Prototype myClone() {
        return new ConcretePrototype(filed);
    }

    @Override
    public String toString() {
        return filed;
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        Prototype prototype = new ConcretePrototype("abc");
        Prototype clone = prototype.myClone();
        System.out.println(clone.toString());
    }
}
```

```html
abc
```

### JDK

- [java.lang.Object#clone()](http://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#clone%28%29)

# 三、行为型

## 1. 责任链（Chain Of Responsibility）

### Intent

使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链发送该请求，直到有一个对象处理它为止。

### Class Diagram

- Handler：定义处理请求的接口，并且实现后继链（successor）

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/ca9f23bf-55a4-47b2-9534-a28e35397988.png"/> </div><br>

### Implementation

```java
public abstract class Handler {

    protected Handler successor;


    public Handler(Handler successor) {
        this.successor = successor;
    }


    protected abstract void handleRequest(Request request);
}
```

```java
public class ConcreteHandler1 extends Handler {

    public ConcreteHandler1(Handler successor) {
        super(successor);
    }


    @Override
    protected void handleRequest(Request request) {
        if (request.getType() == RequestType.TYPE1) {
            System.out.println(request.getName() + " is handle by ConcreteHandler1");
            return;
        }
        if (successor != null) {
            successor.handleRequest(request);
        }
    }
}
```

```java
public class ConcreteHandler2 extends Handler {

    public ConcreteHandler2(Handler successor) {
        super(successor);
    }


    @Override
    protected void handleRequest(Request request) {
        if (request.getType() == RequestType.TYPE2) {
            System.out.println(request.getName() + " is handle by ConcreteHandler2");
            return;
        }
        if (successor != null) {
            successor.handleRequest(request);
        }
    }
}
```

```java
public class Request {

    private RequestType type;
    private String name;


    public Request(RequestType type, String name) {
        this.type = type;
        this.name = name;
    }


    public RequestType getType() {
        return type;
    }


    public String getName() {
        return name;
    }
}

```

```java
public enum RequestType {
    TYPE1, TYPE2
}
```

```java
public class Client {

    public static void main(String[] args) {

        Handler handler1 = new ConcreteHandler1(null);
        Handler handler2 = new ConcreteHandler2(handler1);

        Request request1 = new Request(RequestType.TYPE1, "request1");
        handler2.handleRequest(request1);

        Request request2 = new Request(RequestType.TYPE2, "request2");
        handler2.handleRequest(request2);
    }
}
```

```html
request1 is handle by ConcreteHandler1
request2 is handle by ConcreteHandler2
```

### JDK

- [java.util.logging.Logger#log()](http://docs.oracle.com/javase/8/docs/api/java/util/logging/Logger.html#log%28java.util.logging.Level,%20java.lang.String%29)
- [Apache Commons Chain](https://commons.apache.org/proper/commons-chain/index.html)
- [javax.servlet.Filter#doFilter()](http://docs.oracle.com/javaee/7/api/javax/servlet/Filter.html#doFilter-javax.servlet.ServletRequest-javax.servlet.ServletResponse-javax.servlet.FilterChain-)

## 2. 命令（Command）

### Intent

将命令封装成对象中，具有以下作用：

- 使用命令来参数化其它对象
- 将命令放入队列中进行排队
- 将命令的操作记录到日志中
- 支持可撤销的操作

### Class Diagram

- Command：命令
- Receiver：命令接收者，也就是命令真正的执行者
- Invoker：通过它来调用命令
- Client：可以设置命令与命令的接收者

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/c44a0342-f405-4f17-b750-e27cf4aadde2.png"/> </div><br>

### Implementation

设计一个遥控器，可以控制电灯开关。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/e6bded8e-41a0-489a-88a6-638e88ab7666.jpg"/> </div><br>

```java
public interface Command {
    void execute();
}
```

```java
public class LightOnCommand implements Command {
    Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.on();
    }
}
```

```java
public class LightOffCommand implements Command {
    Light light;

    public LightOffCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.off();
    }
}
```

```java
public class Light {

    public void on() {
        System.out.println("Light is on!");
    }

    public void off() {
        System.out.println("Light is off!");
    }
}
```

```java
/**
 * 遥控器
 */
public class Invoker {
    private Command[] onCommands;
    private Command[] offCommands;
    private final int slotNum = 7;

    public Invoker() {
        this.onCommands = new Command[slotNum];
        this.offCommands = new Command[slotNum];
    }

    public void setOnCommand(Command command, int slot) {
        onCommands[slot] = command;
    }

    public void setOffCommand(Command command, int slot) {
        offCommands[slot] = command;
    }

    public void onButtonWasPushed(int slot) {
        onCommands[slot].execute();
    }

    public void offButtonWasPushed(int slot) {
        offCommands[slot].execute();
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        Invoker invoker = new Invoker();
        Light light = new Light();
        Command lightOnCommand = new LightOnCommand(light);
        Command lightOffCommand = new LightOffCommand(light);
        invoker.setOnCommand(lightOnCommand, 0);
        invoker.setOffCommand(lightOffCommand, 0);
        invoker.onButtonWasPushed(0);
        invoker.offButtonWasPushed(0);
    }
}
```

### JDK

- [java.lang.Runnable](http://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html)
- [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki)
- [javax.swing.Action](http://docs.oracle.com/javase/8/docs/api/javax/swing/Action.html)

## 3. 解释器（Interpreter）

### Intent

为语言创建解释器，通常由语言的语法和语法分析来定义。

### Class Diagram

- TerminalExpression：终结符表达式，每个终结符都需要一个 TerminalExpression。
- Context：上下文，包含解释器之外的一些全局信息。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/2b125bcd-1b36-43be-9b78-d90b076be549.png"/> </div><br>

### Implementation

以下是一个规则检验器实现，具有 and 和 or 规则，通过规则可以构建一颗解析树，用来检验一个文本是否满足解析树定义的规则。

例如一颗解析树为 D And (A Or (B C))，文本 "D A" 满足该解析树定义的规则。

这里的 Context 指的是 String。

```java
public abstract class Expression {
    public abstract boolean interpret(String str);
}
```

```java
public class TerminalExpression extends Expression {

    private String literal = null;

    public TerminalExpression(String str) {
        literal = str;
    }

    public boolean interpret(String str) {
        StringTokenizer st = new StringTokenizer(str);
        while (st.hasMoreTokens()) {
            String test = st.nextToken();
            if (test.equals(literal)) {
                return true;
            }
        }
        return false;
    }
}
```

```java
public class AndExpression extends Expression {

    private Expression expression1 = null;
    private Expression expression2 = null;

    public AndExpression(Expression expression1, Expression expression2) {
        this.expression1 = expression1;
        this.expression2 = expression2;
    }

    public boolean interpret(String str) {
        return expression1.interpret(str) && expression2.interpret(str);
    }
}
```

```java
public class OrExpression extends Expression {
    private Expression expression1 = null;
    private Expression expression2 = null;

    public OrExpression(Expression expression1, Expression expression2) {
        this.expression1 = expression1;
        this.expression2 = expression2;
    }

    public boolean interpret(String str) {
        return expression1.interpret(str) || expression2.interpret(str);
    }
}
```

```java
public class Client {

    /**
     * 构建解析树
     */
    public static Expression buildInterpreterTree() {
        // Literal
        Expression terminal1 = new TerminalExpression("A");
        Expression terminal2 = new TerminalExpression("B");
        Expression terminal3 = new TerminalExpression("C");
        Expression terminal4 = new TerminalExpression("D");
        // B C
        Expression alternation1 = new OrExpression(terminal2, terminal3);
        // A Or (B C)
        Expression alternation2 = new OrExpression(terminal1, alternation1);
        // D And (A Or (B C))
        return new AndExpression(terminal4, alternation2);
    }

    public static void main(String[] args) {
        Expression define = buildInterpreterTree();
        String context1 = "D A";
        String context2 = "A B";
        System.out.println(define.interpret(context1));
        System.out.println(define.interpret(context2));
    }
}
```

```html
true
false
```

### JDK

- [java.util.Pattern](http://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html)
- [java.text.Normalizer](http://docs.oracle.com/javase/8/docs/api/java/text/Normalizer.html)
- All subclasses of [java.text.Format](http://docs.oracle.com/javase/8/docs/api/java/text/Format.html)
- [javax.el.ELResolver](http://docs.oracle.com/javaee/7/api/javax/el/ELResolver.html)

## 4. 迭代器（Iterator）

### Intent

提供一种顺序访问聚合对象元素的方法，并且不暴露聚合对象的内部表示。

### Class Diagram

- Aggregate 是聚合类，其中 createIterator() 方法可以产生一个 Iterator；
- Iterator 主要定义了 hasNext() 和 next() 方法。
- Client 组合了 Aggregate，为了迭代遍历 Aggregate，也需要组合 Iterator。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/89292ae1-5f13-44dc-b508-3f035e80bf89.png"/> </div><br>

### Implementation

```java
public interface Aggregate {
    Iterator createIterator();
}
```

```java
public class ConcreteAggregate implements Aggregate {

    private Integer[] items;

    public ConcreteAggregate() {
        items = new Integer[10];
        for (int i = 0; i < items.length; i++) {
            items[i] = i;
        }
    }

    @Override
    public Iterator createIterator() {
        return new ConcreteIterator<Integer>(items);
    }
}
```

```java
public interface Iterator<Item> {

    Item next();

    boolean hasNext();
}
```

```java
public class ConcreteIterator<Item> implements Iterator {

    private Item[] items;
    private int position = 0;

    public ConcreteIterator(Item[] items) {
        this.items = items;
    }

    @Override
    public Object next() {
        return items[position++];
    }

    @Override
    public boolean hasNext() {
        return position < items.length;
    }
}
```

```java
public class Client {

    public static void main(String[] args) {
        Aggregate aggregate = new ConcreteAggregate();
        Iterator<Integer> iterator = aggregate.createIterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

### JDK

- [java.util.Iterator](http://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html)
- [java.util.Enumeration](http://docs.oracle.com/javase/8/docs/api/java/util/Enumeration.html)

## 5. 中介者（Mediator）

### Intent

集中相关对象之间复杂的沟通和控制方式。

### Class Diagram

- Mediator：中介者，定义一个接口用于与各同事（Colleague）对象通信。
- Colleague：同事，相关对象

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/30d6e95c-2e3c-4d32-bf4f-68128a70bc05.png"/> </div><br>

### Implementation

Alarm（闹钟）、CoffeePot（咖啡壶）、Calendar（日历）、Sprinkler（喷头）是一组相关的对象，在某个对象的事件产生时需要去操作其它对象，形成了下面这种依赖结构：

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/82cfda3b-b53b-4c89-9fdb-26dd2db0cd02.jpg"/> </div><br>

使用中介者模式可以将复杂的依赖结构变成星形结构：

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/5359cbf5-5a79-4874-9b17-f23c53c2cb80.jpg"/> </div><br>

```java
public abstract class Colleague {
    public abstract void onEvent(Mediator mediator);
}
```

```java
public class Alarm extends Colleague {

    @Override
    public void onEvent(Mediator mediator) {
        mediator.doEvent("alarm");
    }

    public void doAlarm() {
        System.out.println("doAlarm()");
    }
}
```

```java
public class CoffeePot extends Colleague {
    @Override
    public void onEvent(Mediator mediator) {
        mediator.doEvent("coffeePot");
    }

    public void doCoffeePot() {
        System.out.println("doCoffeePot()");
    }
}
```

```java
public class Calender extends Colleague {
    @Override
    public void onEvent(Mediator mediator) {
        mediator.doEvent("calender");
    }

    public void doCalender() {
        System.out.println("doCalender()");
    }
}
```

```java
public class Sprinkler extends Colleague {
    @Override
    public void onEvent(Mediator mediator) {
        mediator.doEvent("sprinkler");
    }

    public void doSprinkler() {
        System.out.println("doSprinkler()");
    }
}
```

```java
public abstract class Mediator {
    public abstract void doEvent(String eventType);
}
```

```java
public class ConcreteMediator extends Mediator {
    private Alarm alarm;
    private CoffeePot coffeePot;
    private Calender calender;
    private Sprinkler sprinkler;

    public ConcreteMediator(Alarm alarm, CoffeePot coffeePot, Calender calender, Sprinkler sprinkler) {
        this.alarm = alarm;
        this.coffeePot = coffeePot;
        this.calender = calender;
        this.sprinkler = sprinkler;
    }

    @Override
    public void doEvent(String eventType) {
        switch (eventType) {
            case "alarm":
                doAlarmEvent();
                break;
            case "coffeePot":
                doCoffeePotEvent();
                break;
            case "calender":
                doCalenderEvent();
                break;
            default:
                doSprinklerEvent();
        }
    }

    public void doAlarmEvent() {
        alarm.doAlarm();
        coffeePot.doCoffeePot();
        calender.doCalender();
        sprinkler.doSprinkler();
    }

    public void doCoffeePotEvent() {
        // ...
    }

    public void doCalenderEvent() {
        // ...
    }

    public void doSprinklerEvent() {
        // ...
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        Alarm alarm = new Alarm();
        CoffeePot coffeePot = new CoffeePot();
        Calender calender = new Calender();
        Sprinkler sprinkler = new Sprinkler();
        Mediator mediator = new ConcreteMediator(alarm, coffeePot, calender, sprinkler);
        // 闹钟事件到达，调用中介者就可以操作相关对象
        alarm.onEvent(mediator);
    }
}
```

```java
doAlarm()
doCoffeePot()
doCalender()
doSprinkler()
```

### JDK

- All scheduleXXX() methods of [java.util.Timer](http://docs.oracle.com/javase/8/docs/api/java/util/Timer.html)
- [java.util.concurrent.Executor#execute()](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executor.html#execute-java.lang.Runnable-)
- submit() and invokeXXX() methods of [java.util.concurrent.ExecutorService](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html)
- scheduleXXX() methods of [java.util.concurrent.ScheduledExecutorService](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html)
- [java.lang.reflect.Method#invoke()](http://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html#invoke-java.lang.Object-java.lang.Object...-)

## 6. 备忘录（Memento）

### Intent

在不违反封装的情况下获得对象的内部状态，从而在需要时可以将对象恢复到最初状态。

### Class Diagram

- Originator：原始对象
- Caretaker：负责保存好备忘录
- Menento：备忘录，存储原始对象的的状态。备忘录实际上有两个接口，一个是提供给 Caretaker 的窄接口：它只能将备忘录传递给其它对象；一个是提供给 Originator 的宽接口，允许它访问到先前状态所需的所有数据。理想情况是只允许 Originator 访问本备忘录的内部状态。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/50678f34-694f-45a4-91c6-34d985c83fee.png"/> </div><br>

### Implementation

以下实现了一个简单计算器程序，可以输入两个值，然后计算这两个值的和。备忘录模式允许将这两个值存储起来，然后在某个时刻用存储的状态进行恢复。

实现参考：[Memento Pattern - Calculator Example - Java Sourcecode](https://www.oodesign.com/memento-pattern-calculator-example-java-sourcecode.html)

```java
/**
 * Originator Interface
 */
public interface Calculator {

    // Create Memento
    PreviousCalculationToCareTaker backupLastCalculation();

    // setMemento
    void restorePreviousCalculation(PreviousCalculationToCareTaker memento);

    int getCalculationResult();

    void setFirstNumber(int firstNumber);

    void setSecondNumber(int secondNumber);
}
```

```java
/**
 * Originator Implementation
 */
public class CalculatorImp implements Calculator {

    private int firstNumber;
    private int secondNumber;

    @Override
    public PreviousCalculationToCareTaker backupLastCalculation() {
        // create a memento object used for restoring two numbers
        return new PreviousCalculationImp(firstNumber, secondNumber);
    }

    @Override
    public void restorePreviousCalculation(PreviousCalculationToCareTaker memento) {
        this.firstNumber = ((PreviousCalculationToOriginator) memento).getFirstNumber();
        this.secondNumber = ((PreviousCalculationToOriginator) memento).getSecondNumber();
    }

    @Override
    public int getCalculationResult() {
        // result is adding two numbers
        return firstNumber + secondNumber;
    }

    @Override
    public void setFirstNumber(int firstNumber) {
        this.firstNumber = firstNumber;
    }

    @Override
    public void setSecondNumber(int secondNumber) {
        this.secondNumber = secondNumber;
    }
}
```

```java
/**
 * Memento Interface to Originator
 *
 * This interface allows the originator to restore its state
 */
public interface PreviousCalculationToOriginator {
    int getFirstNumber();
    int getSecondNumber();
}
```

```java
/**
 *  Memento interface to CalculatorOperator (Caretaker)
 */
public interface PreviousCalculationToCareTaker {
    // no operations permitted for the caretaker
}
```

```java
/**
 * Memento Object Implementation
 * <p>
 * Note that this object implements both interfaces to Originator and CareTaker
 */
public class PreviousCalculationImp implements PreviousCalculationToCareTaker,
        PreviousCalculationToOriginator {

    private int firstNumber;
    private int secondNumber;

    public PreviousCalculationImp(int firstNumber, int secondNumber) {
        this.firstNumber = firstNumber;
        this.secondNumber = secondNumber;
    }

    @Override
    public int getFirstNumber() {
        return firstNumber;
    }

    @Override
    public int getSecondNumber() {
        return secondNumber;
    }
}
```

```java
/**
 * CareTaker object
 */
public class Client {

    public static void main(String[] args) {
        // program starts
        Calculator calculator = new CalculatorImp();

        // assume user enters two numbers
        calculator.setFirstNumber(10);
        calculator.setSecondNumber(100);

        // find result
        System.out.println(calculator.getCalculationResult());

        // Store result of this calculation in case of error
        PreviousCalculationToCareTaker memento = calculator.backupLastCalculation();

        // user enters a number
        calculator.setFirstNumber(17);

        // user enters a wrong second number and calculates result
        calculator.setSecondNumber(-290);

        // calculate result
        System.out.println(calculator.getCalculationResult());

        // user hits CTRL + Z to undo last operation and see last result
        calculator.restorePreviousCalculation(memento);

        // result restored
        System.out.println(calculator.getCalculationResult());
    }
}
```

```html
110
-273
110
```

### JDK

- java.io.Serializable

## 7. 观察者（Observer）

### Intent

定义对象之间的一对多依赖，当一个对象状态改变时，它的所有依赖都会收到通知并且自动更新状态。

主题（Subject）是被观察的对象，而其所有依赖者（Observer）称为观察者。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/7a3c6a30-c735-4edb-8115-337288a4f0f2.jpg" width="600"/> </div><br>

### Class Diagram

主题（Subject）具有注册和移除观察者、并通知所有观察者的功能，主题是通过维护一张观察者列表来实现这些操作的。

观察者（Observer）的注册功能需要调用主题的 registerObserver() 方法。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/a8c8f894-a712-447c-9906-5caef6a016e3.png"/> </div><br>

### Implementation

天气数据布告板会在天气信息发生改变时更新其内容，布告板有多个，并且在将来会继续增加。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/b1df9732-86ce-4d69-9f06-fba1db7b3b5a.jpg"/> </div><br>

```java
public interface Subject {
    void registerObserver(Observer o);

    void removeObserver(Observer o);

    void notifyObserver();
}
```

```java
public class WeatherData implements Subject {
    private List<Observer> observers;
    private float temperature;
    private float humidity;
    private float pressure;

    public WeatherData() {
        observers = new ArrayList<>();
    }

    public void setMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        notifyObserver();
    }

    @Override
    public void registerObserver(Observer o) {
        observers.add(o);
    }

    @Override
    public void removeObserver(Observer o) {
        int i = observers.indexOf(o);
        if (i >= 0) {
            observers.remove(i);
        }
    }

    @Override
    public void notifyObserver() {
        for (Observer o : observers) {
            o.update(temperature, humidity, pressure);
        }
    }
}
```

```java
public interface Observer {
    void update(float temp, float humidity, float pressure);
}
```

```java
public class StatisticsDisplay implements Observer {

    public StatisticsDisplay(Subject weatherData) {
        weatherData.reisterObserver(this);
    }

    @Override
    public void update(float temp, float humidity, float pressure) {
        System.out.println("StatisticsDisplay.update: " + temp + " " + humidity + " " + pressure);
    }
}
```

```java
public class CurrentConditionsDisplay implements Observer {

    public CurrentConditionsDisplay(Subject weatherData) {
        weatherData.registerObserver(this);
    }

    @Override
    public void update(float temp, float humidity, float pressure) {
        System.out.println("CurrentConditionsDisplay.update: " + temp + " " + humidity + " " + pressure);
    }
}
```

```java
public class WeatherStation {
    public static void main(String[] args) {
        WeatherData weatherData = new WeatherData();
        CurrentConditionsDisplay currentConditionsDisplay = new CurrentConditionsDisplay(weatherData);
        StatisticsDisplay statisticsDisplay = new StatisticsDisplay(weatherData);

        weatherData.setMeasurements(0, 0, 0);
        weatherData.setMeasurements(1, 1, 1);
    }
}
```

```html
CurrentConditionsDisplay.update: 0.0 0.0 0.0
StatisticsDisplay.update: 0.0 0.0 0.0
CurrentConditionsDisplay.update: 1.0 1.0 1.0
StatisticsDisplay.update: 1.0 1.0 1.0
```

### JDK

- [java.util.Observer](http://docs.oracle.com/javase/8/docs/api/java/util/Observer.html)
- [java.util.EventListener](http://docs.oracle.com/javase/8/docs/api/java/util/EventListener.html)
- [javax.servlet.http.HttpSessionBindingListener](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSessionBindingListener.html)
- [RxJava](https://github.com/ReactiveX/RxJava)

## 8. 状态（State）

### Intent

允许对象在内部状态改变时改变它的行为，对象看起来好像修改了它所属的类。

### Class Diagram

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/79df886f-fdc3-4020-a07f-c991bb58e0d8.png"/> </div><br>

### Implementation

糖果销售机有多种状态，每种状态下销售机有不同的行为，状态可以发生转移，使得销售机的行为也发生改变。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/396be981-3f2c-4fd9-8101-dbf9c841504b.jpg" width="600"/> </div><br>

```java
public interface State {
    /**
     * 投入 25 分钱
     */
    void insertQuarter();

    /**
     * 退回 25 分钱
     */
    void ejectQuarter();

    /**
     * 转动曲柄
     */
    void turnCrank();

    /**
     * 发放糖果
     */
    void dispense();
}
```

```java
public class HasQuarterState implements State {

    private GumballMachine gumballMachine;

    public HasQuarterState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
    }

    @Override
    public void insertQuarter() {
        System.out.println("You can't insert another quarter");
    }

    @Override
    public void ejectQuarter() {
        System.out.println("Quarter returned");
        gumballMachine.setState(gumballMachine.getNoQuarterState());
    }

    @Override
    public void turnCrank() {
        System.out.println("You turned...");
        gumballMachine.setState(gumballMachine.getSoldState());
    }

    @Override
    public void dispense() {
        System.out.println("No gumball dispensed");
    }
}
```

```java
public class NoQuarterState implements State {

    GumballMachine gumballMachine;

    public NoQuarterState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
    }

    @Override
    public void insertQuarter() {
        System.out.println("You insert a quarter");
        gumballMachine.setState(gumballMachine.getHasQuarterState());
    }

    @Override
    public void ejectQuarter() {
        System.out.println("You haven't insert a quarter");
    }

    @Override
    public void turnCrank() {
        System.out.println("You turned, but there's no quarter");
    }

    @Override
    public void dispense() {
        System.out.println("You need to pay first");
    }
}
```

```java
public class SoldOutState implements State {

    GumballMachine gumballMachine;

    public SoldOutState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
    }

    @Override
    public void insertQuarter() {
        System.out.println("You can't insert a quarter, the machine is sold out");
    }

    @Override
    public void ejectQuarter() {
        System.out.println("You can't eject, you haven't inserted a quarter yet");
    }

    @Override
    public void turnCrank() {
        System.out.println("You turned, but there are no gumballs");
    }

    @Override
    public void dispense() {
        System.out.println("No gumball dispensed");
    }
}
```

```java
public class SoldState implements State {

    GumballMachine gumballMachine;

    public SoldState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
    }

    @Override
    public void insertQuarter() {
        System.out.println("Please wait, we're already giving you a gumball");
    }

    @Override
    public void ejectQuarter() {
        System.out.println("Sorry, you already turned the crank");
    }

    @Override
    public void turnCrank() {
        System.out.println("Turning twice doesn't get you another gumball!");
    }

    @Override
    public void dispense() {
        gumballMachine.releaseBall();
        if (gumballMachine.getCount() > 0) {
            gumballMachine.setState(gumballMachine.getNoQuarterState());
        } else {
            System.out.println("Oops, out of gumballs");
            gumballMachine.setState(gumballMachine.getSoldOutState());
        }
    }
}
```

```java
public class GumballMachine {

    private State soldOutState;
    private State noQuarterState;
    private State hasQuarterState;
    private State soldState;

    private State state;
    private int count = 0;

    public GumballMachine(int numberGumballs) {
        count = numberGumballs;
        soldOutState = new SoldOutState(this);
        noQuarterState = new NoQuarterState(this);
        hasQuarterState = new HasQuarterState(this);
        soldState = new SoldState(this);

        if (numberGumballs > 0) {
            state = noQuarterState;
        } else {
            state = soldOutState;
        }
    }

    public void insertQuarter() {
        state.insertQuarter();
    }

    public void ejectQuarter() {
        state.ejectQuarter();
    }

    public void turnCrank() {
        state.turnCrank();
        state.dispense();
    }

    public void setState(State state) {
        this.state = state;
    }

    public void releaseBall() {
        System.out.println("A gumball comes rolling out the slot...");
        if (count != 0) {
            count -= 1;
        }
    }

    public State getSoldOutState() {
        return soldOutState;
    }

    public State getNoQuarterState() {
        return noQuarterState;
    }

    public State getHasQuarterState() {
        return hasQuarterState;
    }

    public State getSoldState() {
        return soldState;
    }

    public int getCount() {
        return count;
    }
}
```

```java
public class Client {

    public static void main(String[] args) {
        GumballMachine gumballMachine = new GumballMachine(5);

        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();

        gumballMachine.insertQuarter();
        gumballMachine.ejectQuarter();
        gumballMachine.turnCrank();

        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();
        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();
        gumballMachine.ejectQuarter();

        gumballMachine.insertQuarter();
        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();
        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();
        gumballMachine.insertQuarter();
        gumballMachine.turnCrank();
    }
}
```

```html
You insert a quarter
You turned...
A gumball comes rolling out the slot...
You insert a quarter
Quarter returned
You turned, but there's no quarter
You need to pay first
You insert a quarter
You turned...
A gumball comes rolling out the slot...
You insert a quarter
You turned...
A gumball comes rolling out the slot...
You haven't insert a quarter
You insert a quarter
You can't insert another quarter
You turned...
A gumball comes rolling out the slot...
You insert a quarter
You turned...
A gumball comes rolling out the slot...
Oops, out of gumballs
You can't insert a quarter, the machine is sold out
You turned, but there are no gumballs
No gumball dispensed
```

## 9. 策略（Strategy）

### Intent

定义一系列算法，封装每个算法，并使它们可以互换。

策略模式可以让算法独立于使用它的客户端。

### Class Diagram

- Strategy 接口定义了一个算法族，它们都实现了  behavior() 方法。
- Context 是使用到该算法族的类，其中的 doSomething() 方法会调用 behavior()，setStrategy(Strategy) 方法可以动态地改变 strategy 对象，也就是说能动态地改变 Context 所使用的算法。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/cd1be8c2-755a-4a66-ad92-2e30f8f47922.png"/> </div><br>

### 与状态模式的比较

状态模式的类图和策略模式类似，并且都是能够动态改变对象的行为。但是状态模式是通过状态转移来改变 Context 所组合的 State 对象，而策略模式是通过 Context 本身的决策来改变组合的 Strategy 对象。所谓的状态转移，是指 Context 在运行过程中由于一些条件发生改变而使得 State 对象发生改变，注意必须要是在运行过程中。

状态模式主要是用来解决状态转移的问题，当状态发生转移了，那么 Context 对象就会改变它的行为；而策略模式主要是用来封装一组可以互相替代的算法族，并且可以根据需要动态地去替换 Context 使用的算法。

### Implementation

设计一个鸭子，它可以动态地改变叫声。这里的算法族是鸭子的叫声行为。

```java
public interface QuackBehavior {
    void quack();
}
```

```java
public class Quack implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("quack!");
    }
}
```

```java
public class Squeak implements QuackBehavior{
    @Override
    public void quack() {
        System.out.println("squeak!");
    }
}
```

```java
public class Duck {

    private QuackBehavior quackBehavior;

    public void performQuack() {
        if (quackBehavior != null) {
            quackBehavior.quack();
        }
    }

    public void setQuackBehavior(QuackBehavior quackBehavior) {
        this.quackBehavior = quackBehavior;
    }
}
```

```java
public class Client {

    public static void main(String[] args) {
        Duck duck = new Duck();
        duck.setQuackBehavior(new Squeak());
        duck.performQuack();
        duck.setQuackBehavior(new Quack());
        duck.performQuack();
    }
}
```

```html
squeak!
quack!
```

### JDK

- java.util.Comparator#compare()
- javax.servlet.http.HttpServlet
- javax.servlet.Filter#doFilter()

## 10. 模板方法（Template Method）

### Intent

定义算法框架，并将一些步骤的实现延迟到子类。

通过模板方法，子类可以重新定义算法的某些步骤，而不用改变算法的结构。

### Class Diagram

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/ac6a794b-68c0-486c-902f-8d988eee5766.png"/> </div><br>

### Implementation

冲咖啡和冲茶都有类似的流程，但是某些步骤会有点不一样，要求复用那些相同步骤的代码。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/11236498-1417-46ce-a1b0-e10054256955.png"/> </div><br>

```java
public abstract class CaffeineBeverage {

    final void prepareRecipe() {
        boilWater();
        brew();
        pourInCup();
        addCondiments();
    }

    abstract void brew();

    abstract void addCondiments();

    void boilWater() {
        System.out.println("boilWater");
    }

    void pourInCup() {
        System.out.println("pourInCup");
    }
}
```

```java
public class Coffee extends CaffeineBeverage {
    @Override
    void brew() {
        System.out.println("Coffee.brew");
    }

    @Override
    void addCondiments() {
        System.out.println("Coffee.addCondiments");
    }
}
```

```java
public class Tea extends CaffeineBeverage {
    @Override
    void brew() {
        System.out.println("Tea.brew");
    }

    @Override
    void addCondiments() {
        System.out.println("Tea.addCondiments");
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        CaffeineBeverage caffeineBeverage = new Coffee();
        caffeineBeverage.prepareRecipe();
        System.out.println("-----------");
        caffeineBeverage = new Tea();
        caffeineBeverage.prepareRecipe();
    }
}
```

```html
boilWater
Coffee.brew
pourInCup
Coffee.addCondiments
-----------
boilWater
Tea.brew
pourInCup
Tea.addCondiments
```

### JDK

- java.util.Collections#sort()
- java.io.InputStream#skip()
- java.io.InputStream#read()
- java.util.AbstractList#indexOf()

## 11. 访问者（Visitor）

### Intent

为一个对象结构（比如组合结构）增加新能力。

### Class Diagram

- Visitor：访问者，为每一个 ConcreteElement 声明一个 visit 操作
- ConcreteVisitor：具体访问者，存储遍历过程中的累计结果
- ObjectStructure：对象结构，可以是组合结构，或者是一个集合。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/79c6f036-bde6-4393-85a3-ef36a0327bd2.png"/> </div><br>

### Implementation

```java
public interface Element {
    void accept(Visitor visitor);
}
```

```java
class CustomerGroup {

    private List<Customer> customers = new ArrayList<>();

    void accept(Visitor visitor) {
        for (Customer customer : customers) {
            customer.accept(visitor);
        }
    }

    void addCustomer(Customer customer) {
        customers.add(customer);
    }
}
```

```java
public class Customer implements Element {

    private String name;
    private List<Order> orders = new ArrayList<>();

    Customer(String name) {
        this.name = name;
    }

    String getName() {
        return name;
    }

    void addOrder(Order order) {
        orders.add(order);
    }

    public void accept(Visitor visitor) {
        visitor.visit(this);
        for (Order order : orders) {
            order.accept(visitor);
        }
    }
}
```

```java
public class Order implements Element {

    private String name;
    private List<Item> items = new ArrayList();

    Order(String name) {
        this.name = name;
    }

    Order(String name, String itemName) {
        this.name = name;
        this.addItem(new Item(itemName));
    }

    String getName() {
        return name;
    }

    void addItem(Item item) {
        items.add(item);
    }

    public void accept(Visitor visitor) {
        visitor.visit(this);

        for (Item item : items) {
            item.accept(visitor);
        }
    }
}
```

```java
public class Item implements Element {

    private String name;

    Item(String name) {
        this.name = name;
    }

    String getName() {
        return name;
    }

    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}
```

```java
public interface Visitor {
    void visit(Customer customer);

    void visit(Order order);

    void visit(Item item);
}
```

```java
public class GeneralReport implements Visitor {

    private int customersNo;
    private int ordersNo;
    private int itemsNo;

    public void visit(Customer customer) {
        System.out.println(customer.getName());
        customersNo++;
    }

    public void visit(Order order) {
        System.out.println(order.getName());
        ordersNo++;
    }

    public void visit(Item item) {
        System.out.println(item.getName());
        itemsNo++;
    }

    public void displayResults() {
        System.out.println("Number of customers: " + customersNo);
        System.out.println("Number of orders:    " + ordersNo);
        System.out.println("Number of items:     " + itemsNo);
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        Customer customer1 = new Customer("customer1");
        customer1.addOrder(new Order("order1", "item1"));
        customer1.addOrder(new Order("order2", "item1"));
        customer1.addOrder(new Order("order3", "item1"));

        Order order = new Order("order_a");
        order.addItem(new Item("item_a1"));
        order.addItem(new Item("item_a2"));
        order.addItem(new Item("item_a3"));
        Customer customer2 = new Customer("customer2");
        customer2.addOrder(order);

        CustomerGroup customers = new CustomerGroup();
        customers.addCustomer(customer1);
        customers.addCustomer(customer2);

        GeneralReport visitor = new GeneralReport();
        customers.accept(visitor);
        visitor.displayResults();
    }
}
```

```html
customer1
order1
item1
order2
item1
order3
item1
customer2
order_a
item_a1
item_a2
item_a3
Number of customers: 2
Number of orders:    4
Number of items:     6
```

### JDK

- javax.lang.model.element.Element and javax.lang.model.element.ElementVisitor
- javax.lang.model.type.TypeMirror and javax.lang.model.type.TypeVisitor

## 12. 空对象（Null）

### Intent

使用什么都不做的空对象来代替 NULL。

一个方法返回 NULL，意味着方法的调用端需要去检查返回值是否是 NULL，这么做会导致非常多的冗余的检查代码。并且如果某一个调用端忘记了做这个检查返回值，而直接使用返回的对象，那么就有可能抛出空指针异常。

### Class Diagram

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/22870bbe-898f-4c17-a31a-d7c5ee5d1c10.png"/> </div><br>

### Implementation

```java
public abstract class AbstractOperation {
    abstract void request();
}
```

```java
public class RealOperation extends AbstractOperation {
    @Override
    void request() {
        System.out.println("do something");
    }
}
```

```java
public class NullOperation extends AbstractOperation{
    @Override
    void request() {
        // do nothing
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        AbstractOperation abstractOperation = func(-1);
        abstractOperation.request();
    }

    public static AbstractOperation func(int para) {
        if (para < 0) {
            return new NullOperation();
        }
        return new RealOperation();
    }
}
```

# 四、结构型

## 1. 适配器（Adapter）

### Intent

把一个类接口转换成另一个用户需要的接口。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/3d5b828e-5c4d-48d8-a440-281e4a8e1c92.png"/> </div><br>

### Class Diagram

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/ff5152fc-4ff3-44c4-95d6-1061002c364a.png"/> </div><br>

### Implementation

鸭子（Duck）和火鸡（Turkey）拥有不同的叫声，Duck 的叫声调用 quack() 方法，而 Turkey 调用 gobble() 方法。

要求将 Turkey 的 gobble() 方法适配成 Duck 的 quack() 方法，从而让火鸡冒充鸭子！

```java
public interface Duck {
    void quack();
}
```

```java
public interface Turkey {
    void gobble();
}
```

```java
public class WildTurkey implements Turkey {
    @Override
    public void gobble() {
        System.out.println("gobble!");
    }
}
```

```java
public class TurkeyAdapter implements Duck {
    Turkey turkey;

    public TurkeyAdapter(Turkey turkey) {
        this.turkey = turkey;
    }

    @Override
    public void quack() {
        turkey.gobble();
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        Turkey turkey = new WildTurkey();
        Duck duck = new TurkeyAdapter(turkey);
        duck.quack();
    }
}
```

### JDK

- [java.util.Arrays#asList()](http://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#asList%28T...%29)
- [java.util.Collections#list()](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#list-java.util.Enumeration-)
- [java.util.Collections#enumeration()](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#enumeration-java.util.Collection-)
- [javax.xml.bind.annotation.adapters.XMLAdapter](http://docs.oracle.com/javase/8/docs/api/javax/xml/bind/annotation/adapters/XmlAdapter.html#marshal-BoundType-)

## 2. 桥接（Bridge）

### Intent

将抽象与实现分离开来，使它们可以独立变化。

### Class Diagram

- Abstraction：定义抽象类的接口
- Implementor：定义实现类接口

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/2a1f8b0f-1dd7-4409-b177-a381c58066ad.png"/> </div><br>

### Implementation

RemoteControl 表示遥控器，指代 Abstraction。

TV 表示电视，指代 Implementor。

桥接模式将遥控器和电视分离开来，从而可以独立改变遥控器或者电视的实现。

```java
public abstract class TV {
    public abstract void on();

    public abstract void off();

    public abstract void tuneChannel();
}
```

```java
public class Sony extends TV {
    @Override
    public void on() {
        System.out.println("Sony.on()");
    }

    @Override
    public void off() {
        System.out.println("Sony.off()");
    }

    @Override
    public void tuneChannel() {
        System.out.println("Sony.tuneChannel()");
    }
}
```

```java
public class RCA extends TV {
    @Override
    public void on() {
        System.out.println("RCA.on()");
    }

    @Override
    public void off() {
        System.out.println("RCA.off()");
    }

    @Override
    public void tuneChannel() {
        System.out.println("RCA.tuneChannel()");
    }
}
```

```java
public abstract class RemoteControl {
    protected TV tv;

    public RemoteControl(TV tv) {
        this.tv = tv;
    }

    public abstract void on();

    public abstract void off();

    public abstract void tuneChannel();
}
```

```java
public class ConcreteRemoteControl1 extends RemoteControl {
    public ConcreteRemoteControl1(TV tv) {
        super(tv);
    }

    @Override
    public void on() {
        System.out.println("ConcreteRemoteControl1.on()");
        tv.on();
    }

    @Override
    public void off() {
        System.out.println("ConcreteRemoteControl1.off()");
        tv.off();
    }

    @Override
    public void tuneChannel() {
        System.out.println("ConcreteRemoteControl1.tuneChannel()");
        tv.tuneChannel();
    }
}
```

```java
public class ConcreteRemoteControl2 extends RemoteControl {
    public ConcreteRemoteControl2(TV tv) {
        super(tv);
    }

    @Override
    public void on() {
        System.out.println("ConcreteRemoteControl2.on()");
        tv.on();
    }

    @Override
    public void off() {
        System.out.println("ConcreteRemoteControl2.off()");
        tv.off();
    }

    @Override
    public void tuneChannel() {
        System.out.println("ConcreteRemoteControl2.tuneChannel()");
        tv.tuneChannel();
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        RemoteControl remoteControl1 = new ConcreteRemoteControl1(new RCA());
        remoteControl1.on();
        remoteControl1.off();
        remoteControl1.tuneChannel();
        RemoteControl remoteControl2 = new ConcreteRemoteControl2(new Sony());
         remoteControl2.on();
         remoteControl2.off();
         remoteControl2.tuneChannel();
    }
}
```

### JDK

- AWT (It provides an abstraction layer which maps onto the native OS the windowing support.)
- JDBC

## 3. 组合（Composite）

### Intent

将对象组合成树形结构来表示“整体/部分”层次关系，允许用户以相同的方式处理单独对象和组合对象。

### Class Diagram

组件（Component）类是组合类（Composite）和叶子类（Leaf）的父类，可以把组合类看成是树的中间节点。

组合对象拥有一个或者多个组件对象，因此组合对象的操作可以委托给组件对象去处理，而组件对象可以是另一个组合对象或者叶子对象。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/2b8bfd57-b4d1-4a75-bfb0-bcf1fba4014a.png"/> </div><br>

### Implementation

```java
public abstract class Component {
    protected String name;

    public Component(String name) {
        this.name = name;
    }

    public void print() {
        print(0);
    }

    abstract void print(int level);

    abstract public void add(Component component);

    abstract public void remove(Component component);
}
```

```java
public class Composite extends Component {

    private List<Component> child;

    public Composite(String name) {
        super(name);
        child = new ArrayList<>();
    }

    @Override
    void print(int level) {
        for (int i = 0; i < level; i++) {
            System.out.print("--");
        }
        System.out.println("Composite:" + name);
        for (Component component : child) {
            component.print(level + 1);
        }
    }

    @Override
    public void add(Component component) {
        child.add(component);
    }

    @Override
    public void remove(Component component) {
        child.remove(component);
    }
}
```

```java
public class Leaf extends Component {
    public Leaf(String name) {
        super(name);
    }

    @Override
    void print(int level) {
        for (int i = 0; i < level; i++) {
            System.out.print("--");
        }
        System.out.println("left:" + name);
    }

    @Override
    public void add(Component component) {
        throw new UnsupportedOperationException(); // 牺牲透明性换取单一职责原则，这样就不用考虑是叶子节点还是组合节点
    }

    @Override
    public void remove(Component component) {
        throw new UnsupportedOperationException();
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        Composite root = new Composite("root");
        Component node1 = new Leaf("1");
        Component node2 = new Composite("2");
        Component node3 = new Leaf("3");
        root.add(node1);
        root.add(node2);
        root.add(node3);
        Component node21 = new Leaf("21");
        Component node22 = new Composite("22");
        node2.add(node21);
        node2.add(node22);
        Component node221 = new Leaf("221");
        node22.add(node221);
        root.print();
    }
}
```

```html
Composite:root
--left:1
--Composite:2
----left:21
----Composite:22
------left:221
--left:3
```

### JDK

- javax.swing.JComponent#add(Component)
- java.awt.Container#add(Component)
- java.util.Map#putAll(Map)
- java.util.List#addAll(Collection)
- java.util.Set#addAll(Collection)

## 4. 装饰（Decorator）

### Intent

为对象动态添加功能。

### Class Diagram

装饰者（Decorator）和具体组件（ConcreteComponent）都继承自组件（Component），具体组件的方法实现不需要依赖于其它对象，而装饰者组合了一个组件，这样它可以装饰其它装饰者或者具体组件。所谓装饰，就是把这个装饰者套在被装饰者之上，从而动态扩展被装饰者的功能。装饰者的方法有一部分是自己的，这属于它的功能，然后调用被装饰者的方法实现，从而也保留了被装饰者的功能。可以看到，具体组件应当是装饰层次的最低层，因为只有具体组件的方法实现不需要依赖于其它对象。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/6b833bc2-517a-4270-8a5e-0a5f6df8cd96.png"/> </div><br>

### Implementation

设计不同种类的饮料，饮料可以添加配料，比如可以添加牛奶，并且支持动态添加新配料。每增加一种配料，该饮料的价格就会增加，要求计算一种饮料的价格。

下图表示在 DarkRoast 饮料上新增新添加 Mocha 配料，之后又添加了 Whip 配料。DarkRoast 被 Mocha 包裹，Mocha 又被 Whip 包裹。它们都继承自相同父类，都有 cost() 方法，外层类的 cost() 方法调用了内层类的 cost() 方法。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/c9cfd600-bc91-4f3a-9f99-b42f88a5bb24.jpg" width="600"/> </div><br>

```java
public interface Beverage {
    double cost();
}
```

```java
public class DarkRoast implements Beverage {
    @Override
    public double cost() {
        return 1;
    }
}
```

```java
public class HouseBlend implements Beverage {
    @Override
    public double cost() {
        return 1;
    }
}
```

```java
public abstract class CondimentDecorator implements Beverage {
    protected Beverage beverage;
}
```

```java
public class Milk extends CondimentDecorator {

    public Milk(Beverage beverage) {
        this.beverage = beverage;
    }

    @Override
    public double cost() {
        return 1 + beverage.cost();
    }
}
```

```java
public class Mocha extends CondimentDecorator {

    public Mocha(Beverage beverage) {
        this.beverage = beverage;
    }

    @Override
    public double cost() {
        return 1 + beverage.cost();
    }
}
```

```java
public class Client {

    public static void main(String[] args) {
        Beverage beverage = new HouseBlend();
        beverage = new Mocha(beverage);
        beverage = new Milk(beverage);
        System.out.println(beverage.cost());
    }
}
```

```html
3.0
```

### 设计原则

类应该对扩展开放，对修改关闭：也就是添加新功能时不需要修改代码。饮料可以动态添加新的配料，而不需要去修改饮料的代码。

不可能把所有的类设计成都满足这一原则，应当把该原则应用于最有可能发生改变的地方。

### JDK

- java.io.BufferedInputStream(InputStream)
- java.io.DataInputStream(InputStream)
- java.io.BufferedOutputStream(OutputStream)
- java.util.zip.ZipOutputStream(OutputStream)
- java.util.Collections#checked[List|Map|Set|SortedSet|SortedMap]()

## 5. 外观（Facade）

### Intent

提供了一个统一的接口，用来访问子系统中的一群接口，从而让子系统更容易使用。

### Class Diagram

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/f9978fa6-9f49-4a0f-8540-02d269ac448f.png"/> </div><br>

### Implementation

观看电影需要操作很多电器，使用外观模式实现一键看电影功能。

```java
public class SubSystem {
    public void turnOnTV() {
        System.out.println("turnOnTV()");
    }

    public void setCD(String cd) {
        System.out.println("setCD( " + cd + " )");
    }

    public void startWatching(){
        System.out.println("startWatching()");
    }
}
```

```java
public class Facade {
    private SubSystem subSystem = new SubSystem();

    public void watchMovie() {
        subSystem.turnOnTV();
        subSystem.setCD("a movie");
        subSystem.startWatching();
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        Facade facade = new Facade();
        facade.watchMovie();
    }
}
```

### 设计原则

最少知识原则：只和你的密友谈话。也就是说客户对象所需要交互的对象应当尽可能少。

## 6. 享元（Flyweight）

### Intent

利用共享的方式来支持大量细粒度的对象，这些对象一部分内部状态是相同的。

### Class Diagram

- Flyweight：享元对象
- IntrinsicState：内部状态，享元对象共享内部状态
- ExtrinsicState：外部状态，每个享元对象的外部状态不同

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/5f5c22d5-9c0e-49e1-b5b0-6cc7032724d4.png"/> </div><br>

### Implementation

```java
public interface Flyweight {
    void doOperation(String extrinsicState);
}
```

```java
public class ConcreteFlyweight implements Flyweight {

    private String intrinsicState;

    public ConcreteFlyweight(String intrinsicState) {
        this.intrinsicState = intrinsicState;
    }

    @Override
    public void doOperation(String extrinsicState) {
        System.out.println("Object address: " + System.identityHashCode(this));
        System.out.println("IntrinsicState: " + intrinsicState);
        System.out.println("ExtrinsicState: " + extrinsicState);
    }
}
```

```java
public class FlyweightFactory {

    private HashMap<String, Flyweight> flyweights = new HashMap<>();

    Flyweight getFlyweight(String intrinsicState) {
        if (!flyweights.containsKey(intrinsicState)) {
            Flyweight flyweight = new ConcreteFlyweight(intrinsicState);
            flyweights.put(intrinsicState, flyweight);
        }
        return flyweights.get(intrinsicState);
    }
}
```

```java
public class Client {

    public static void main(String[] args) {
        FlyweightFactory factory = new FlyweightFactory();
        Flyweight flyweight1 = factory.getFlyweight("aa");
        Flyweight flyweight2 = factory.getFlyweight("aa");
        flyweight1.doOperation("x");
        flyweight2.doOperation("y");
    }
}
```

```html
Object address: 1163157884
IntrinsicState: aa
ExtrinsicState: x
Object address: 1163157884
IntrinsicState: aa
ExtrinsicState: y
```

### JDK

Java 利用缓存来加速大量小对象的访问时间。

- java.lang.Integer#valueOf(int)
- java.lang.Boolean#valueOf(boolean)
- java.lang.Byte#valueOf(byte)
- java.lang.Character#valueOf(char)

## 7. 代理（Proxy）

### Intent

控制对其它对象的访问。

### Class Diagram

代理有以下四类：

- 远程代理（Remote Proxy）：控制对远程对象（不同地址空间）的访问，它负责将请求及其参数进行编码，并向不同地址空间中的对象发送已经编码的请求。
- 虚拟代理（Virtual Proxy）：根据需要创建开销很大的对象，它可以缓存实体的附加信息，以便延迟对它的访问，例如在网站加载一个很大图片时，不能马上完成，可以用虚拟代理缓存图片的大小信息，然后生成一张临时图片代替原始图片。
- 保护代理（Protection Proxy）：按权限控制对象的访问，它负责检查调用者是否具有实现一个请求所必须的访问权限。
- 智能代理（Smart Reference）：取代了简单的指针，它在访问对象时执行一些附加操作：记录对象的引用次数；当第一次引用一个对象时，将它装入内存；在访问一个实际对象前，检查是否已经锁定了它，以确保其它对象不能改变它。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/9b679ff5-94c6-48a7-b9b7-2ea868e828ed.png"/> </div><br>

### Implementation

以下是一个虚拟代理的实现，模拟了图片延迟加载的情况下使用与图片大小相等的临时内容去替换原始图片，直到图片加载完成才将图片显示出来。

```java
public interface Image {
    void showImage();
}
```

```java
public class HighResolutionImage implements Image {

    private URL imageURL;
    private long startTime;
    private int height;
    private int width;

    public int getHeight() {
        return height;
    }

    public int getWidth() {
        return width;
    }

    public HighResolutionImage(URL imageURL) {
        this.imageURL = imageURL;
        this.startTime = System.currentTimeMillis();
        this.width = 600;
        this.height = 600;
    }

    public boolean isLoad() {
        // 模拟图片加载，延迟 3s 加载完成
        long endTime = System.currentTimeMillis();
        return endTime - startTime > 3000;
    }

    @Override
    public void showImage() {
        System.out.println("Real Image: " + imageURL);
    }
}
```

```java
public class ImageProxy implements Image {

    private HighResolutionImage highResolutionImage;

    public ImageProxy(HighResolutionImage highResolutionImage) {
        this.highResolutionImage = highResolutionImage;
    }

    @Override
    public void showImage() {
        while (!highResolutionImage.isLoad()) {
            try {
                System.out.println("Temp Image: " + highResolutionImage.getWidth() + " " + highResolutionImage.getHeight());
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        highResolutionImage.showImage();
    }
}
```

```java
public class ImageViewer {

    public static void main(String[] args) throws Exception {
        String image = "http://image.jpg";
        URL url = new URL(image);
        HighResolutionImage highResolutionImage = new HighResolutionImage(url);
        ImageProxy imageProxy = new ImageProxy(highResolutionImage);
        imageProxy.showImage();
    }
}
```

### JDK

- java.lang.reflect.Proxy
- RMI

# 参考资料

- 弗里曼. Head First 设计模式 [M]. 中国电力出版社, 2007.
- Gamma E. 设计模式: 可复用面向对象软件的基础 [M]. 机械工业出版社, 2007.
- Bloch J. Effective java[M]. Addison-Wesley Professional, 2017.
- [Design Patterns](http://www.oodesign.com/)
- [Design patterns implemented in Java](http://java-design-patterns.com/)
- [The breakdown of design patterns in JDK](http://www.programering.com/a/MTNxAzMwATY.html)



## **一.设计模式的思维导向图**

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190430134807749-1487199226.png)

 

 

## **二. 设计模式的七大原则：**

　设计模式（面向对象）有七大原则，分别是：

　　**1.开放-封闭原则**

　　　　**通俗：对扩展开发，对修改关闭**

　　**2.单一职责原则**

　　　　**通俗：一个类只做一件事**

　　**3.依赖倒转原则**

　　　　**通俗：类似IOC，采用接口编程**

　　**4.迪米特法则（也称为最小知识原则）**

　　　　**通俗：高内聚，低耦合**

　　**5.接口隔离原则**

　　　　**通俗：细节接口**

　　**6.合成/聚合复用原则**

　　　　**通俗：避免使用继承**

　　**7.里氏代换原则**

　　　　**通俗：子类不能去修改父类的功能** 

 

## 三、结构性模式：

### 1、适配器模式：

> ***常用于将一个新接口适配旧******接口***

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428230235157-1264165116.png)

 

 

PS：***在我们业务代码中经常有新旧接口适配需求，可以采用该模式。\***

### 2、桥接模式：

> ***将抽象和抽象的具体实现进行解耦，这样可以使得抽象和抽象的具体实现可以独立进行变化。\***

 ![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428230504411-1081545395.png)

 

PS：***这个模式，其实我们每天都在用到，但是你可能却浑然不知。只要你用到面向接口编程，其实都是在用桥接模式。\***

### 3、组合模式

> ***让客户端看起来在处理单个对象和对象的组合是平等的，换句话说，某个类型的方法同时也接受自身类型作为参数\***。（So in other words methods on a type accepting the same type）

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428230532452-220408516.png)

 

PS：从上面那句英文我们就可以得知，***组合模式常用于递归操作的优化上，\***比如每个公司都有个boss系统，都会有什么菜单的功能。比如一级菜单下有二级菜单，二级菜单又有三级菜单。删除一级菜单的时候需要不断删除子菜单，那么这个设计模式你可以试试。***总之，凡是有级联操作的，你都可以尝试这个设计模式。\***

### 4、装饰者模式

> ***动态的给一个对象附加额外的功能，因此它也是子类化的一种替代方法。该设计模式在JDK中广泛运用\***，以下只是列举一小部分

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428230646537-989775832.png)

 

PS：***这个模式使用就太广了，我们常用的AOP，既有动态代理，也有装饰者的味道。\***

### 5、门面模式

> ***为一组组件，接口，抽象或子系统提供简化的接口。\***

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428230743274-1010622874.png)

 

PS：***我们每天使用的SLFJ日志就是门面日志，比如我们使用Dubbo，向外提供的服务就尽量采用门面模式，然后服务在调用各种service做聚合\***。

 

SLF4J，即简单日志门面（**Simple Logging Facade for Java**），**不是具体的日志解决方案，它只服务于各种各样的日志系统。按照官方的说法，SLF4J是一个用于日志系统的简单Facade，允许最终用户在部署其应用时使用其所希望的日志System.**

 

实际上，SLF4J所提供的核心API是一些接口以及一个LoggerFactory的工厂类。从某种程度上，**SLF4J有点类似JDBC，不过比JDBC更简单**，在JDBC中，你需要指定驱动程序，而在使用SLF4J的时候，不需要在代码中或配置文件中指定你打算使用那个具体的日志系统。如同使用JDBC基本不用考虑具体数据库一样，SLF4J提供了统一的记录日志的接口，只要按照其提供的方法记录即可，最终日志的格式、记录级别、输出方式等通过具体日志系统的配置来实现，因此可以在应用中灵活切换日志系统。



 

 

### 6、享元模式

> ***使用缓存来减少对小对象的访问时间\***

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428230831818-1063194607.png)

 

PS：***只要用到了缓存，基本都是在使用享元模式\***。很多同学都说自己的项目太low了，都没有用到什么设计模式，这不是开玩笑吗，你用个map缓存几个对象，基本上都运用了享元的思想。

### 7、代理模式

> ***代理模式用于向较简单的对象代替创建复杂或耗时的对象。\***

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428230915402-1771521941.png)

 

PS：***代理模式用得很广泛，基本所有大家知道的开源框架，都用到了动态代理。\***

## 四、创建模式

### 1、抽象工厂模式

> ***抽象工厂模式提供了一个协议来生成一系列的相关或者独立的对象，而不用指定具体对象的类型。它使得应用程序能够和使用的框架的具体实现进行解耦。在JDK和许多开源框架，比如Spring中随处可见，它们很容易被发现。任何用于创建对象但返回接口或抽象类的，就是抽象工厂模式了。（any method that is used to create an object but still returns a interface or abstract class）\***

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428231049003-1733509276.png)

 

PS：***从英文就可以得出，该模式可以与策略模式结合使用。\***

### 2、建造者模式

> ***用于通过定义一个类来简化复杂对象的创建，该类的目的是构建另一个类的实例。构建器模式还允许实现Fluent接口。\***

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428231123239-1859680663.png)

 

PS：这个在我们业务代码中使用的场景太广泛了。***比如订单系统大部分项目都有，订单对象就是一个复杂对象，我们就可以采用建造者模式来做。\***

### 3、工厂方法

> ***只是一个返回实际类型的方法。\***

 ![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428231526931-1230437933.png)

 

PS：这个属于大家都会的设计模式，不多过介绍。

### 4、原型模式

> ***使得类的实例能够生成自身的拷贝。如果创建一个对象的实例非常复杂且耗时时，就可以使用这种模式，而不重新创建一个新的实例，你可以拷贝一个对象并直接修改它。\***

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428231629334-326007537.png)

 

PS：***这个你以为是冷门的设计模式，其实错了，这个是大热门的设计模式。比如我们业务代码，经常要各种DTO、BO、DO、VO转换，其实就可以参考原型设计模式的思想来做。\***

### 5、单例模式

> ***用来确保类只有一个实例。\***Joshua Bloch在Effetive Java中建议到，还有一种方法就是使用枚举。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428231713765-1108565643.png)

 

PS：***在平时开发中，单例是我们用得最多的了，因为Spring的bean，默认就是单例级别的。单例属于大家基本都会的设计模式。\***

## 五、行为模式

### 1、责任链

> ***通过把请求从一个对象传递到链条中下一个对象的方式来解除对象之间的耦合，直到请求被处理完毕。链中的对象是同一接口或抽象类的不同实现。\***

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

 

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428231807492-1963485437.png)

 

PS：***凡是带有`Filter`关键词的，基本都在用这个设计模式。在业务代码使用的场景实在是太多了，用到拦截器的地方基本都在用这个设计模式。\***

### 2、命令模式

> ***将命令包装在对象中，以便可以将其存储，传递到方法中，并像任何其他对象一样返回。\***

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428231853918-1090832987.png)

 

PS：***命令模式使用频率较高，和策略模式比较像，具体区别可以搜索一下。如果用过`Activiti`工作流引擎的朋友可以看一下里面的源码，很多地方都用到了命令模式。\***

### 3、解释器模式

> ***此模式通常描述为该语言定义语法并使用该语法来解释该格式的语句。（This pattern generally describes defining a grammar for that language and using that grammar to interpret statements in that format.）\***

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428231924737-158492068.png)

 

PS：这个比较冷门，肥朝没怎么用过，你用过的话可以留言告诉肥朝。

### 4、迭代器模式

> ***提供一个统一的方式来访问集合中的对象。\***

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428232026947-563773884.png)

 

PS：***这个中间件和基础框架组的同学可能用得比较多，业务代码的话用得不多，不过JDK中的这种使用很经典，可以看看。\***

### 5、中介者模式

> ***通过使用一个中间对象来进行消息分发以及减少类之间的直接依赖。\***

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428232052186-1058798955.png)

 

PS：看到这个描述不用我多说什么，业务代码使用的场景太多了***。比如你们用MQ，其实就是在用中介者模式。\***所以肥朝一再强调，即使是每天CRUD，关注肥朝一起学习，也能给你的CRUD项目，加上美颜+滤镜（设计模式）的加强效果。

### 6、备忘录模式

> ***生成对象状态的一个快照，以便对象可以恢复原始状态而不用暴露自身的内容。比如Date对象通过自身内部的一个long值来实现备忘录模式。\***

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

 

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428232217422-57081267.png)

 

PS：这个在业务中使用得不多，据肥朝了解其中一种场景是，你要把数据丢到MQ，但是MQ暂时不可用，那么你把数据暂存到DB，后面再轮询丢到MQ。如果你有更好的场景，留言告诉肥朝。

### 7、空对象模式

> ***它允许您抽象空对象的处理。\***

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428232259835-624516650.png)

 

PS：这个业务代码用得不多，但是JDK中的这几个方法我们倒是挺常用的。

### 8、观察者模式

> ***用于为组件提供一种灵活地向感兴趣的接收者广播消息的方式。\***

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428232338337-989394543.png)

 

PS：***我们业务代码一般是基于Zookeeper来做观察者的。基本上用到ZK的地方，都是在用观察者模式，比如分布式锁，比如服务发现等**。*

### *9、状态模式*

> ***允许您在运行时根据内部状态轻松更改对象的行为。\***

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428232426206-988641658.png)

 

PS：***这个在业务代码用得就太广泛了，我就不信你们系统还没有“状态”了。比如我们常见的订单状态或者各种XX状态，都可以用得上***。

### 10、策略模式

> ***使用这个模式来将一组算法封装成一系列对象。通过调用这些对象可以灵活的改变程序的功能。\***

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428232615879-1851548814.png)

 

PS：***这个太高频了，常用于优化大量的`if-else`,\***如果这个设计模式都不会，出去不要说关注过肥朝的公众号！

### 11、模板方法模式

> ***让子类可以重写方法的一部分，而不是整个重写，你可以控制子类需要重写那些操作。\***

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428232648986-2136715856.png)

 

PS：这个模式也是非常高频的模式。业务代码中经常遇到有很多相同的部分，***我们可以做一个抽象类，子类来实现差异化\***，如果还不知道的，赶紧搜索一下，再次强调，非常高频。

### 12、访问者模式

> ***提供一个方便的可维护的方式来操作一组对象。它使得你在不改变操作的对象前提下，可以修改或者扩展对象的行为。\***

![img](https://img2018.cnblogs.com/blog/667853/201904/667853-20190428232732391-570510028.png)

 

**一、设计模式的分类**

总体来说设计模式分为三大类：

创建型模式，共五种：工厂方法模式、抽象工厂模式、单例模式、建造者模式、原型模式。

结构型模式，共七种：适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式。

行为型模式，共十一种：策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。

其实还有两类：并发型模式和线程池模式。用一个图片来整体描述一下：

![点击查看原始大小图片](http://dl.iteye.com/upload/attachment/0083/1179/57a92d42-4d84-3aa9-a8b9-63a0b02c2c36.jpg)

 

**二、设计模式的六大原则**

**1、开闭原则（Open Close Principle）**

开闭原则就是说**对扩展开放，对修改关闭**。在程序需要进行拓展的时候，不能去修改原有的代码，实现一个热插拔的效果。所以一句话概括就是：为了使程序的扩展性好，易于维护和升级。想要达到这样的效果，我们需要使用接口和抽象类，后面的具体设计中我们会提到这点。

**2、里氏代换原则（Liskov Substitution Principle）**

里氏代换原则(Liskov Substitution Principle LSP)面向对象设计的基本原则之一。  里氏代换原则中说，任何基类可以出现的地方，子类一定可以出现。  LSP是继承复用的基石，只有当衍生类可以替换掉基类，软件单位的功能不受到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为。里氏代换原则是对“开-闭”原则的补充。实现“开-闭”原则的关键步骤就是抽象化。而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。—— From Baidu 百科

**3、依赖倒转原则（Dependence Inversion Principle）**

这个是开闭原则的基础，具体内容：真对接口编程，依赖于抽象而不依赖于具体。

**4、接口隔离原则（Interface Segregation Principle）**

这个原则的意思是：使用多个隔离的接口，比使用单个接口要好。还是一个降低类之间的耦合度的意思，从这儿我们看出，其实设计模式就是一个软件的设计思想，从大型软件架构出发，为了升级和维护方便。所以上文中多次出现：降低依赖，降低耦合。

**5、迪米特法则（最少知道原则）（Demeter Principle）**

为什么叫最少知道原则，就是说：一个实体应当尽量少的与其他实体之间发生相互作用，使得系统功能模块相对独立。

**6、合成复用原则（Composite Reuse Principle）**

原则是尽量使用合成/聚合的方式，而不是使用继承。

**三、Java的23中设计模式**

从这一块开始，我们详细介绍Java中23种设计模式的概念，应用场景等情况，并结合他们的特点及设计模式的原则进行分析。

**1、工厂方法模式（Factory Method）**

工厂方法模式分为三种：

***11、普通工厂模式\***，就是建立一个工厂类，对实现了同一接口的一些类进行实例的创建。首先看下关系图：

![img](http://dl.iteye.com/upload/attachment/0083/1180/421a1a3f-6777-3bca-85d7-00fc60c1ae8b.png)

举例如下：（我们举一个发送邮件和短信的例子）

首先，创建二者的共同接口：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public interface Sender { 
2.   public void Send(); 
3. } 

其次，创建实现类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class MailSender implements Sender { 
2.   @Override 
3.   public void Send() { 
4. ​    System.out.println("this is mailsender!"); 
5.   } 
6. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class SmsSender implements Sender { 
2.  
3.   @Override 
4.   public void Send() { 
5. ​    System.out.println("this is sms sender!"); 
6.   } 
7. } 

最后，建工厂类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class SendFactory { 
2.  
3.   public Sender produce(String type) { 
4. ​    if ("mail".equals(type)) { 
5. ​      return new MailSender(); 
6. ​    } else if ("sms".equals(type)) { 
7. ​      return new SmsSender(); 
8. ​    } else { 
9. ​      System.out.println("请输入正确的类型!"); 
10. ​      return null; 
11. ​    } 
12.   } 
13. } 

我们来测试下：

1. public class FactoryTest { 
2.  
3.   public static void main(String[] args) { 
4. ​    SendFactory factory = new SendFactory(); 
5. ​    Sender sender = factory.produce("sms"); 
6. ​    sender.Send(); 
7.   } 
8. } 

输出：this is sms sender!

***22、多个工厂方法模式***，是对普通工厂方法模式的改进，在普通工厂方法模式中，如果传递的字符串出错，则不能正确创建对象，而多个工厂方法模式是提供多个工厂方法，分别创建对象。关系图：

![点击查看原始大小图片](http://dl.iteye.com/upload/attachment/0083/1181/84673ccf-ef89-3774-b5cf-6d2523cd03e5.jpg)

将上面的代码做下修改，改动下SendFactory类就行，如下：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)public class SendFactory {  

  public Sender produceMail(){  

1. ​    return new MailSender(); 
2.   } 
3.    
4.   public Sender produceSms(){ 
5. ​    return new SmsSender(); 
6.   } 
7. } 

测试类如下：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class FactoryTest { 
2.  
3.   public static void main(String[] args) { 
4. ​    SendFactory factory = new SendFactory(); 
5. ​    Sender sender = factory.produceMail(); 
6. ​    sender.Send(); 
7.   } 
8. } 

输出：this is mailsender!

***33、静态工厂方法模式***，将上面的多个工厂方法模式里的方法置为静态的，不需要创建实例，直接调用即可。

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class SendFactory { 
2.    
3.   public static Sender produceMail(){ 
4. ​    return new MailSender(); 
5.   } 
6.    
7.   public static Sender produceSms(){ 
8. ​    return new SmsSender(); 
9.   } 
10. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class FactoryTest { 
2.  
3.   public static void main(String[] args) {   
4. ​    Sender sender = SendFactory.produceMail(); 
5. ​    sender.Send(); 
6.   } 
7. } 

输出：this is mailsender!

总体来说，工厂模式适合：凡是出现了大量的产品需要创建，并且具有共同的接口时，可以通过工厂方法模式进行创建。在以上的三种模式中，第一种如果传入的字符串有误，不能正确创建对象，第三种相对于第二种，不需要实例化工厂类，所以，大多数情况下，我们会选用第三种——静态工厂方法模式。

**2、抽象工厂模式（Abstract Factory）**

工厂方法模式有一个问题就是，类的创建依赖工厂类，也就是说，如果想要拓展程序，必须对工厂类进行修改，这违背了闭包原则，所以，从设计角度考虑，有一定的问题，如何解决？就用到抽象工厂模式，创建多个工厂类，这样一旦需要增加新的功能，直接增加新的工厂类就可以了，不需要修改之前的代码。因为抽象工厂不太好理解，我们先看看图，然后就和代码，就比较容易理解。

![点击查看原始大小图片](http://dl.iteye.com/upload/attachment/0083/1185/34a0f8de-16e0-3cd5-9f69-257fcb2be742.jpg)

请看例子：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public interface Sender { 
2.   public void Send(); 
3. } 

两个实现类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class MailSender implements Sender { 
2.   @Override 
3.   public void Send() { 
4. ​    System.out.println("this is mailsender!"); 
5.   } 
6. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class SmsSender implements Sender { 
2.  
3.   @Override 
4.   public void Send() { 
5. ​    System.out.println("this is sms sender!"); 
6.   } 
7. } 

两个工厂类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class SendMailFactory implements Provider { 
2.    
3.   @Override 
4.   public Sender produce(){ 
5. ​    return new MailSender(); 
6.   } 
7. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class SendSmsFactory implements Provider{ 
2.  
3.   @Override 
4.   public Sender produce() { 
5. ​    return new SmsSender(); 
6.   } 
7. } 

在提供一个接口：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public interface Provider { 
2.   public Sender produce(); 
3. } 

测试类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class Test { 
2.  
3.   public static void main(String[] args) { 
4. ​    Provider provider = new SendMailFactory(); 
5. ​    Sender sender = provider.produce(); 
6. ​    sender.Send(); 
7.   } 
8. } 

其实这个模式的好处就是，如果你现在想增加一个功能：发及时信息，则只需做一个实现类，实现Sender接口，同时做一个工厂类，实现Provider接口，就OK了，无需去改动现成的代码。这样做，拓展性较好！

**3、单例模式（\**Singleton\**）**

单例对象（Singleton）是一种常用的设计模式。在Java应用中，单例对象能保证在一个JVM中，该对象只有一个实例存在。这样的模式有几个好处：

1、某些类创建比较频繁，对于一些大型的对象，这是一笔很大的系统开销。

2、省去了new操作符，降低了系统内存的使用频率，减轻GC压力。

3、有些类如交易所的核心交易引擎，控制着交易流程，如果该类可以创建多个的话，系统完全乱了。（比如一个军队出现了多个司令员同时指挥，肯定会乱成一团），所以只有使用单例模式，才能保证核心交易服务器独立控制整个流程。

首先我们写一个简单的单例类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class Singleton { 
2.  
3.   /* 持有私有静态实例，防止被引用，此处赋值为null，目的是实现延迟加载 */ 
4.   private static Singleton instance = null; 
5.  
6.   /* 私有构造方法，防止被实例化 */ 
7.   private Singleton() { 
8.   } 
9.  
10.   /* 静态工程方法，创建实例 */ 
11.   public static Singleton getInstance() { 
12. ​    if (instance == null) { 
13. ​      instance = new Singleton(); 
14. ​    } 
15. ​    return instance; 
16.   } 
17.  
18.   /* 如果该对象被用于序列化，可以保证对象在序列化前后保持一致 */ 
19.   public Object readResolve() { 
20. ​    return instance; 
21.   } 
22. } 


这个类可以满足基本要求，但是，像这样毫无线程安全保护的类，如果我们把它放入多线程的环境下，肯定就会出现问题了，如何解决？我们首先会想到对getInstance方法加synchronized关键字，如下：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public static synchronized Singleton getInstance() { 
2. ​    if (instance == null) { 
3. ​      instance = new Singleton(); 
4. ​    } 
5. ​    return instance; 
6.   } 

但是，synchronized关键字锁住的是这个对象，这样的用法，在性能上会有所下降，因为每次调用getInstance()，都要对对象上锁，事实上，只有在第一次创建对象的时候需要加锁，之后就不需要了，所以，这个地方需要改进。我们改成下面这个：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public static Singleton getInstance() { 
2. ​    if (instance == null) { 
3. ​      synchronized (instance) { 
4. ​        if (instance == null) { 
5. ​          instance = new Singleton(); 
6. ​        } 
7. ​      } 
8. ​    } 
9. ​    return instance; 
10.   } 

似乎解决了之前提到的问题，将synchronized关键字加在了内部，也就是说当调用的时候是不需要加锁的，只有在instance为null，并创建对象的时候才需要加锁，性能有一定的提升。但是，这样的情况，还是有可能有问题的，看下面的情况：在Java指令中创建对象和赋值操作是分开进行的，也就是说instance = new  Singleton();语句是分两步执行的。但是JVM并不保证这两个操作的先后顺序，也就是说有可能JVM会为新的Singleton实例分配空间，然后直接赋值给instance成员，然后再去初始化这个Singleton实例。这样就可能出错了，我们以A、B两个线程为例：

a>A、B线程同时进入了第一个if判断

b>A首先进入synchronized块，由于instance为null，所以它执行instance = new Singleton();

c>由于JVM内部的优化机制，JVM先画出了一些分配给Singleton实例的空白内存，并赋值给instance成员（注意此时JVM没有开始初始化这个实例），然后A离开了synchronized块。

d>B进入synchronized块，由于instance此时不是null，因此它马上离开了synchronized块并将结果返回给调用该方法的程序。

e>此时B线程打算使用Singleton实例，却发现它没有被初始化，于是错误发生了。

所以程序还是有可能发生错误，其实程序在运行过程是很复杂的，从这点我们就可以看出，尤其是在写多线程环境下的程序更有难度，有挑战性。我们对该程序做进一步优化：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. private static class SingletonFactory{      
2. ​    private static Singleton instance = new Singleton();      
3.   }      
4.   public static Singleton getInstance(){      
5. ​    return SingletonFactory.instance;      
6.   }  

实际情况是，单例模式使用内部类来维护单例的实现，JVM内部的机制能够保证当一个类被加载的时候，这个类的加载过程是线程互斥的。这样当我们第一次调用getInstance的时候，JVM能够帮我们保证instance只被创建一次，并且会保证把赋值给instance的内存初始化完毕，这样我们就不用担心上面的问题。同时该方法也只会在第一次调用的时候使用互斥机制，这样就解决了低性能问题。这样我们暂时总结一个完美的单例模式：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class Singleton { 
2.  
3.   /* 私有构造方法，防止被实例化 */ 
4.   private Singleton() { 
5.   } 
6.  
7.   /* 此处使用一个内部类来维护单例 */ 
8.   private static class SingletonFactory { 
9. ​    private static Singleton instance = new Singleton(); 
10.   } 
11.  
12.   /* 获取实例 */ 
13.   public static Singleton getInstance() { 
14. ​    return SingletonFactory.instance; 
15.   } 
16.  
17.   /* 如果该对象被用于序列化，可以保证对象在序列化前后保持一致 */ 
18.   public Object readResolve() { 
19. ​    return getInstance(); 
20.   } 
21. } 

其实说它完美，也不一定，如果在构造函数中抛出异常，实例将永远得不到创建，也会出错。所以说，十分完美的东西是没有的，我们只能根据实际情况，选择最适合自己应用场景的实现方法。也有人这样实现：因为我们只需要在创建类的时候进行同步，所以只要将创建和getInstance()分开，单独为创建加synchronized关键字，也是可以的：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class SingletonTest { 
2.  
3.   private static SingletonTest instance = null; 
4.  
5.   private SingletonTest() { 
6.   } 
7.  
8.   private static synchronized void syncInit() { 
9. ​    if (instance == null) { 
10. ​      instance = new SingletonTest(); 
11. ​    } 
12.   } 
13.  
14.   public static SingletonTest getInstance() { 
15. ​    if (instance == null) { 
16. ​      syncInit(); 
17. ​    } 
18. ​    return instance; 
19.   } 
20. } 

考虑性能的话，整个程序只需创建一次实例，所以性能也不会有什么影响。

**补充：采用"影子实例"的办法为单例对象的属性同步更新**

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class SingletonTest { 
2.  
3.   private static SingletonTest instance = null; 
4.   private Vector properties = null; 
5.  
6.   public Vector getProperties() { 
7. ​    return properties; 
8.   } 
9.  
10.   private SingletonTest() { 
11.   } 
12.  
13.   private static synchronized void syncInit() { 
14. ​    if (instance == null) { 
15. ​      instance = new SingletonTest(); 
16. ​    } 
17.   } 
18.  
19.   public static SingletonTest getInstance() { 
20. ​    if (instance == null) { 
21. ​      syncInit(); 
22. ​    } 
23. ​    return instance; 
24.   } 
25.  
26.   public void updateProperties() { 
27. ​    SingletonTest shadow = new SingletonTest(); 
28. ​    properties = shadow.getProperties(); 
29.   } 
30. } 

通过单例模式的学习告诉我们：

1、单例模式理解起来简单，但是具体实现起来还是有一定的难度。

2、synchronized关键字锁定的是对象，在用的时候，一定要在恰当的地方使用（注意需要使用锁的对象和过程，可能有的时候并不是整个对象及整个过程都需要锁）。

到这儿，单例模式基本已经讲完了，结尾处，笔者突然想到另一个问题，就是采用类的静态方法，实现单例模式的效果，也是可行的，此处二者有什么不同？

首先，静态类不能实现接口。（从类的角度说是可以的，但是那样就破坏了静态了。因为接口中不允许有static修饰的方法，所以即使实现了也是非静态的）

其次，单例可以被延迟初始化，静态类一般在第一次加载是初始化。之所以延迟加载，是因为有些类比较庞大，所以延迟加载有助于提升性能。

再次，单例类可以被继承，他的方法可以被覆写。但是静态类内部方法都是static，无法被覆写。

最后一点，单例类比较灵活，毕竟从实现上只是一个普通的Java类，只要满足单例的基本需求，你可以在里面随心所欲的实现一些其它功能，但是静态类不行。从上面这些概括中，基本可以看出二者的区别，但是，从另一方面讲，我们上面最后实现的那个单例模式，内部就是用一个静态类来实现的，所以，二者有很大的关联，只是我们考虑问题的层面不同罢了。两种思想的结合，才能造就出完美的解决方案，就像HashMap采用数组+链表来实现一样，其实生活中很多事情都是这样，单用不同的方法来处理问题，总是有优点也有缺点，最完美的方法是，结合各个方法的优点，才能最好的解决问题！

**4、建造者模式（Builder）**

工厂类模式提供的是创建单个类的模式，而建造者模式则是将各种产品集中起来进行管理，用来创建复合对象，所谓复合对象就是指某个类具有不同的属性，其实建造者模式就是前面抽象工厂模式和最后的Test结合起来得到的。我们看一下代码：

还和前面一样，一个Sender接口，两个实现类MailSender和SmsSender。最后，建造者类如下：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class Builder { 
2.    
3.   private List<Sender> list = new ArrayList<Sender>(); 
4.    
5.   public void produceMailSender(int count){ 
6. ​    for(int i=0; i<count; i++){ 
7. ​      list.add(new MailSender()); 
8. ​    } 
9.   } 
10.    
11.   public void produceSmsSender(int count){ 
12. ​    for(int i=0; i<count; i++){ 
13. ​      list.add(new SmsSender()); 
14. ​    } 
15.   } 
16. } 

测试类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class Test { 
2.  
3.   public static void main(String[] args) { 
4. ​    Builder builder = new Builder(); 
5. ​    builder.produceMailSender(10); 
6.   } 
7. } 

从这点看出，建造者模式将很多功能集成到一个类里，这个类可以创造出比较复杂的东西。所以与工程模式的区别就是：工厂模式关注的是创建单个产品，而建造者模式则关注创建符合对象，多个部分。因此，是选择工厂模式还是建造者模式，依实际情况而定。

**5、原型模式（Prototype）**

原型模式虽然是创建型的模式，但是与工程模式没有关系，从名字即可看出，该模式的思想就是将一个对象作为原型，对其进行复制、克隆，产生一个和原对象类似的新对象。本小结会通过对象的复制，进行讲解。在Java中，复制对象是通过clone()实现的，先创建一个原型类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class Prototype implements Cloneable { 
2.  
3.   public Object clone() throws CloneNotSupportedException { 
4. ​    Prototype proto = (Prototype) super.clone(); 
5. ​    return proto; 
6.   } 
7. } 

很简单，一个原型类，只需要实现Cloneable接口，覆写clone方法，此处clone方法可以改成任意的名称，因为Cloneable接口是个空接口，你可以任意定义实现类的方法名，如cloneA或者cloneB，因为此处的重点是super.clone()这句话，super.clone()调用的是Object的clone()方法，而在Object类中，clone()是native的，具体怎么实现，我会在另一篇文章中，关于解读Java中本地方法的调用，此处不再深究。在这儿，我将结合对象的浅复制和深复制来说一下，首先需要了解对象深、浅复制的概念：

浅复制：将一个对象复制后，基本数据类型的变量都会重新创建，而引用类型，指向的还是原对象所指向的。

深复制：将一个对象复制后，不论是基本数据类型还有引用类型，都是重新创建的。简单来说，就是深复制进行了完全彻底的复制，而浅复制不彻底。

此处，写一个深浅复制的例子：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8194653)[copy](http://blog.csdn.net/zhangerqing/article/details/8194653)

1. public class Prototype implements Cloneable, Serializable { 
2.  
3.   private static final long serialVersionUID = 1L; 
4.   private String string; 
5.  
6.   private SerializableObject obj; 
7.  
8.   /* 浅复制 */ 
9.   public Object clone() throws CloneNotSupportedException { 
10. ​    Prototype proto = (Prototype) super.clone(); 
11. ​    return proto; 
12.   } 
13.  
14.   /* 深复制 */ 
15.   public Object deepClone() throws IOException, ClassNotFoundException { 
16.  
17. ​    /* 写入当前对象的二进制流 */ 
18. ​    ByteArrayOutputStream bos = new ByteArrayOutputStream(); 
19. ​    ObjectOutputStream oos = new ObjectOutputStream(bos); 
20. ​    oos.writeObject(this); 
21.  
22. ​    /* 读出二进制流产生的新对象 */ 
23. ​    ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray()); 
24. ​    ObjectInputStream ois = new ObjectInputStream(bis); 
25. ​    return ois.readObject(); 
26.   } 
27.  
28.   public String getString() { 
29. ​    return string; 
30.   } 
31.  
32.   public void setString(String string) { 
33. ​    this.string = string; 
34.   } 
35.  
36.   public SerializableObject getObj() { 
37. ​    return obj; 
38.   } 
39.  
40.   public void setObj(SerializableObject obj) { 
41. ​    this.obj = obj; 
42.   } 
43.  
44. } 
45.  
46. class SerializableObject implements Serializable { 
47.   private static final long serialVersionUID = 1L; 
48. } 

 

要实现深复制，需要采用流的形式读入当前对象的二进制输入，再写出二进制数据对应的对象。

我们接着讨论设计模式，上篇文章我讲完了5种创建型模式，这章开始，我将讲下7种结构型模式：适配器模式、装饰模式、代理模式、外观模式、桥接模式、组合模式、享元模式。其中对象的适配器模式是各种模式的起源，我们看下面的图：

![点击查看原始大小图片](http://dl.iteye.com/upload/attachment/0083/1187/e28698b9-994e-3fa8-8810-16f30e7cf3e3.jpg)

 适配器模式将某个类的接口转换成客户端期望的另一个接口表示，目的是消除由于接口不匹配所造成的类的兼容性问题。主要分为三类：类的适配器模式、对象的适配器模式、接口的适配器模式。首先，我们来看看**类的适配器模式**，先看类图：

![img](http://dl.iteye.com/upload/attachment/0083/1189/6b2d13aa-7cc7-3e98-9764-bdcb2c64f795.jpg)

核心思想就是：有一个Source类，拥有一个方法，待适配，目标接口时Targetable，通过Adapter类，将Source的功能扩展到Targetable里，看代码：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class Source { 
2.  
3.   public void method1() { 
4. ​    System.out.println("this is original method!"); 
5.   } 
6. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public interface Targetable { 
2.  
3.   /* 与原类中的方法相同 */ 
4.   public void method1(); 
5.  
6.   /* 新类的方法 */ 
7.   public void method2(); 
8. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class Adapter extends Source implements Targetable { 
2.  
3.   @Override 
4.   public void method2() { 
5. ​    System.out.println("this is the targetable method!"); 
6.   } 
7. } 

Adapter类继承Source类，实现Targetable接口，下面是测试类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class AdapterTest { 
2.  
3.   public static void main(String[] args) { 
4. ​    Targetable target = new Adapter(); 
5. ​    target.method1(); 
6. ​    target.method2(); 
7.   } 
8. } 

输出：

this is original method!
this is the targetable method!

这样Targetable接口的实现类就具有了Source类的功能。

**对象的适配器模式**

基本思路和类的适配器模式相同，只是将Adapter类作修改，这次不继承Source类，而是持有Source类的实例，以达到解决兼容性的问题。看图：

![img](http://dl.iteye.com/upload/attachment/0083/1191/0aabe35b-5b79-3ead-838f-9d4b6fbd774d.jpg)

 

只需要修改Adapter类的源码即可：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class Wrapper implements Targetable { 
2.  
3.   private Source source; 
4.    
5.   public Wrapper(Source source){ 
6. ​    super(); 
7. ​    this.source = source; 
8.   } 
9.   @Override 
10.   public void method2() { 
11. ​    System.out.println("this is the targetable method!"); 
12.   } 
13.  
14.   @Override 
15.   public void method1() { 
16. ​    source.method1(); 
17.   } 
18. } 

测试类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class AdapterTest { 
2.  
3.   public static void main(String[] args) { 
4. ​    Source source = new Source(); 
5. ​    Targetable target = new Wrapper(source); 
6. ​    target.method1(); 
7. ​    target.method2(); 
8.   } 
9. } 

输出与第一种一样，只是适配的方法不同而已。

第三种适配器模式是**接口的适配器模式**，接口的适配器是这样的：有时我们写的一个接口中有多个抽象方法，当我们写该接口的实现类时，必须实现该接口的所有方法，这明显有时比较浪费，因为并不是所有的方法都是我们需要的，有时只需要某一些，此处为了解决这个问题，我们引入了接口的适配器模式，借助于一个抽象类，该抽象类实现了该接口，实现了所有的方法，而我们不和原始的接口打交道，只和该抽象类取得联系，所以我们写一个类，继承该抽象类，重写我们需要的方法就行。看一下类图：

![img](http://dl.iteye.com/upload/attachment/0083/1193/a604fca8-e0c6-3e4e-b00a-49da21595b4e.jpg)

这个很好理解，在实际开发中，我们也常会遇到这种接口中定义了太多的方法，以致于有时我们在一些实现类中并不是都需要。看代码：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public interface Sourceable { 
2.    
3.   public void method1(); 
4.   public void method2(); 
5. } 

抽象类Wrapper2：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public abstract class Wrapper2 implements Sourceable{ 
2.    
3.   public void method1(){} 
4.   public void method2(){} 
5. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class SourceSub1 extends Wrapper2 { 
2.   public void method1(){ 
3. ​    System.out.println("the sourceable interface's first Sub1!"); 
4.   } 
5. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class SourceSub2 extends Wrapper2 { 
2.   public void method2(){ 
3. ​    System.out.println("the sourceable interface's second Sub2!"); 
4.   } 
5. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class WrapperTest { 
2.  
3.   public static void main(String[] args) { 
4. ​    Sourceable source1 = new SourceSub1(); 
5. ​    Sourceable source2 = new SourceSub2(); 
6. ​     
7. ​    source1.method1(); 
8. ​    source1.method2(); 
9. ​    source2.method1(); 
10. ​    source2.method2(); 
11.   } 
12. } 

测试输出：

the sourceable interface's first Sub1!
the sourceable interface's second Sub2!

达到了我们的效果！

 讲了这么多，总结一下三种适配器模式的应用场景：

类的适配器模式：当希望将**一个类**转换成满足**另一个新接口**的类时，可以使用类的适配器模式，创建一个新类，继承原有的类，实现新的接口即可。

对象的适配器模式：当希望将一个对象转换成满足另一个新接口的对象时，可以创建一个Wrapper类，持有原类的一个实例，在Wrapper类的方法中，调用实例的方法就行。

接口的适配器模式：当不希望实现一个接口中所有的方法时，可以创建一个抽象类Wrapper，实现所有方法，我们写别的类的时候，继承抽象类即可。

**7、装饰模式（Decorator）**

顾名思义，装饰模式就是给一个对象增加一些新的功能，而且是动态的，要求装饰对象和被装饰对象实现同一个接口，装饰对象持有被装饰对象的实例，关系图如下：

![img](http://dl.iteye.com/upload/attachment/0083/1195/e1b8b6a3-0150-31ae-8f77-7c3d888b6f80.jpg)

Source类是被装饰类，Decorator类是一个装饰类，可以为Source类动态的添加一些功能，代码如下：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public interface Sourceable { 
2.   public void method(); 
3. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class Source implements Sourceable { 
2.  
3.   @Override 
4.   public void method() { 
5. ​    System.out.println("the original method!"); 
6.   } 
7. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class Decorator implements Sourceable { 
2.  
3.   private Sourceable source; 
4.    
5.   public Decorator(Sourceable source){ 
6. ​    super(); 
7. ​    this.source = source; 
8.   } 
9.   @Override 
10.   public void method() { 
11. ​    System.out.println("before decorator!"); 
12. ​    source.method(); 
13. ​    System.out.println("after decorator!"); 
14.   } 
15. } 

测试类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class DecoratorTest { 
2.  
3.   public static void main(String[] args) { 
4. ​    Sourceable source = new Source(); 
5. ​    Sourceable obj = new Decorator(source); 
6. ​    obj.method(); 
7.   } 
8. } 

输出：

before decorator!
the original method!
after decorator!

装饰器模式的应用场景：

1、需要扩展一个类的功能。

2、动态的为一个对象增加功能，而且还能动态撤销。（继承不能做到这一点，继承的功能是静态的，不能动态增删。）

缺点：产生过多相似的对象，不易排错！

**8、代理模式（Proxy）**

其实每个模式名称就表明了该模式的作用，代理模式就是多一个代理类出来，替原对象进行一些操作，比如我们在租房子的时候回去找中介，为什么呢？因为你对该地区房屋的信息掌握的不够全面，希望找一个更熟悉的人去帮你做，此处的代理就是这个意思。再如我们有的时候打官司，我们需要请律师，因为律师在法律方面有专长，可以替我们进行操作，表达我们的想法。先来看看关系图：![img](http://dl.iteye.com/upload/attachment/0083/1197/ea094ad9-efc5-337d-a8e8-ce9223511144.jpg)

 

根据上文的阐述，代理模式就比较容易的理解了，我们看下代码：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public interface Sourceable { 
2.   public void method(); 
3. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class Source implements Sourceable { 
2.  
3.   @Override 
4.   public void method() { 
5. ​    System.out.println("the original method!"); 
6.   } 
7. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class Proxy implements Sourceable { 
2.  
3.   private Source source; 
4.   public Proxy(){ 
5. ​    super(); 
6. ​    this.source = new Source(); 
7.   } 
8.   @Override 
9.   public void method() { 
10. ​    before(); 
11. ​    source.method(); 
12. ​    atfer(); 
13.   } 
14.   private void atfer() { 
15. ​    System.out.println("after proxy!"); 
16.   } 
17.   private void before() { 
18. ​    System.out.println("before proxy!"); 
19.   } 
20. } 

测试类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class ProxyTest { 
2.  
3.   public static void main(String[] args) { 
4. ​    Sourceable source = new Proxy(); 
5. ​    source.method(); 
6.   } 
7.  
8. } 

输出：

before proxy!
the original method!
after proxy!

代理模式的应用场景：

如果已有的方法在使用的时候需要对原有的方法进行改进，此时有两种办法：

1、修改原有的方法来适应。这样违反了“对扩展开放，对修改关闭”的原则。

2、就是采用一个代理类调用原有的方法，且对产生的结果进行控制。这种方法就是代理模式。

使用代理模式，可以将功能划分的更加清晰，有助于后期维护！

**9、外观模式（Facade）**

外观模式是为了解决类与类之家的依赖关系的，像spring一样，可以将类和类之间的关系配置到配置文件中，而外观模式就是将他们的关系放在一个Facade类中，降低了类类之间的耦合度，该模式中没有涉及到接口，看下类图：（我们以一个计算机的启动过程为例）

![img](http://dl.iteye.com/upload/attachment/0083/1199/eebe2103-6ced-35f2-8664-3a2e8a557f81.jpg)

我们先看下实现类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class CPU { 
2.    
3.   public void startup(){ 
4. ​    System.out.println("cpu startup!"); 
5.   } 
6.    
7.   public void shutdown(){ 
8. ​    System.out.println("cpu shutdown!"); 
9.   } 
10. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class Memory { 
2.    
3.   public void startup(){ 
4. ​    System.out.println("memory startup!"); 
5.   } 
6.    
7.   public void shutdown(){ 
8. ​    System.out.println("memory shutdown!"); 
9.   } 
10. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class Disk { 
2.    
3.   public void startup(){ 
4. ​    System.out.println("disk startup!"); 
5.   } 
6.    
7.   public void shutdown(){ 
8. ​    System.out.println("disk shutdown!"); 
9.   } 
10. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class Computer { 
2.   private CPU cpu; 
3.   private Memory memory; 
4.   private Disk disk; 
5.    
6.   public Computer(){ 
7. ​    cpu = new CPU(); 
8. ​    memory = new Memory(); 
9. ​    disk = new Disk(); 
10.   } 
11.    
12.   public void startup(){ 
13. ​    System.out.println("start the computer!"); 
14. ​    cpu.startup(); 
15. ​    memory.startup(); 
16. ​    disk.startup(); 
17. ​    System.out.println("start computer finished!"); 
18.   } 
19.    
20.   public void shutdown(){ 
21. ​    System.out.println("begin to close the computer!"); 
22. ​    cpu.shutdown(); 
23. ​    memory.shutdown(); 
24. ​    disk.shutdown(); 
25. ​    System.out.println("computer closed!"); 
26.   } 
27. } 

User类如下：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class User { 
2.  
3.   public static void main(String[] args) { 
4. ​    Computer computer = new Computer(); 
5. ​    computer.startup(); 
6. ​    computer.shutdown(); 
7.   } 
8. } 

输出：

start the computer!
cpu startup!
memory startup!
disk startup!
start computer finished!
begin to close the computer!
cpu shutdown!
memory shutdown!
disk shutdown!
computer closed!

如果我们没有Computer类，那么，CPU、Memory、Disk他们之间将会相互持有实例，产生关系，这样会造成严重的依赖，修改一个类，可能会带来其他类的修改，这不是我们想要看到的，有了Computer类，他们之间的关系被放在了Computer类里，这样就起到了解耦的作用，这，就是外观模式！

**10、桥接模式（Bridge）**

桥接模式就是把事物和其具体实现分开，使他们可以各自独立的变化。桥接的用意是：**将抽象化与实现化解耦，使得二者可以独立变化**，像我们常用的JDBC桥DriverManager一样，JDBC进行连接数据库的时候，在各个数据库之间进行切换，基本不需要动太多的代码，甚至丝毫不用动，原因就是JDBC提供统一接口，每个数据库提供各自的实现，用一个叫做数据库驱动的程序来桥接就行了。我们来看看关系图：

![img](http://dl.iteye.com/upload/attachment/0083/1201/35f0b172-b976-3757-bb51-c65d5c9ce68e.jpg)

实现代码：

先定义接口：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public interface Sourceable { 
2.   public void method(); 
3. } 

分别定义两个实现类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class SourceSub1 implements Sourceable { 
2.  
3.   @Override 
4.   public void method() { 
5. ​    System.out.println("this is the first sub!"); 
6.   } 
7. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class SourceSub2 implements Sourceable { 
2.  
3.   @Override 
4.   public void method() { 
5. ​    System.out.println("this is the second sub!"); 
6.   } 
7. } 

定义一个桥，持有Sourceable的一个实例：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public abstract class Bridge { 
2.   private Sourceable source; 
3.  
4.   public void method(){ 
5. ​    source.method(); 
6.   } 
7.    
8.   public Sourceable getSource() { 
9. ​    return source; 
10.   } 
11.  
12.   public void setSource(Sourceable source) { 
13. ​    this.source = source; 
14.   } 
15. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class MyBridge extends Bridge { 
2.   public void method(){ 
3. ​    getSource().method(); 
4.   } 
5. } 

测试类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class BridgeTest { 
2.    
3.   public static void main(String[] args) { 
4. ​     
5. ​    Bridge bridge = new MyBridge(); 
6. ​     
7. ​    /*调用第一个对象*/ 
8. ​    Sourceable source1 = new SourceSub1(); 
9. ​    bridge.setSource(source1); 
10. ​    bridge.method(); 
11. ​     
12. ​    /*调用第二个对象*/ 
13. ​    Sourceable source2 = new SourceSub2(); 
14. ​    bridge.setSource(source2); 
15. ​    bridge.method(); 
16.   } 
17. } 

output：

this is the first sub!
this is the second sub!

这样，就通过对Bridge类的调用，实现了对接口Sourceable的实现类SourceSub1和SourceSub2的调用。接下来我再画个图，大家就应该明白了，因为这个图是我们JDBC连接的原理，有数据库学习基础的，一结合就都懂了。

![img](http://dl.iteye.com/upload/attachment/0083/1203/6f713d07-1409-3312-99c9-fa6b0909f0b2.jpg)

**11、组合模式（Composite）**

组合模式有时又叫**部分-整体**模式在处理类似树形结构的问题时比较方便，看看关系图：

![img](http://dl.iteye.com/upload/attachment/0083/1205/09cab656-5ff9-380e-9df1-326339ac3509.jpg)

直接来看代码：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class TreeNode { 
2.    
3.   private String name; 
4.   private TreeNode parent; 
5.   private Vector<TreeNode> children = new Vector<TreeNode>(); 
6.    
7.   public TreeNode(String name){ 
8. ​    this.name = name; 
9.   } 
10.  
11.   public String getName() { 
12. ​    return name; 
13.   } 
14.  
15.   public void setName(String name) { 
16. ​    this.name = name; 
17.   } 
18.  
19.   public TreeNode getParent() { 
20. ​    return parent; 
21.   } 
22.  
23.   public void setParent(TreeNode parent) { 
24. ​    this.parent = parent; 
25.   } 
26.    
27.   //添加孩子节点 
28.   public void add(TreeNode node){ 
29. ​    children.add(node); 
30.   } 
31.    
32.   //删除孩子节点 
33.   public void remove(TreeNode node){ 
34. ​    children.remove(node); 
35.   } 
36.    
37.   //取得孩子节点 
38.   public Enumeration<TreeNode> getChildren(){ 
39. ​    return children.elements(); 
40.   } 
41. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class Tree { 
2.  
3.   TreeNode root = null; 
4.  
5.   public Tree(String name) { 
6. ​    root = new TreeNode(name); 
7.   } 
8.  
9.   public static void main(String[] args) { 
10. ​    Tree tree = new Tree("A"); 
11. ​    TreeNode nodeB = new TreeNode("B"); 
12. ​    TreeNode nodeC = new TreeNode("C"); 
13. ​     
14. ​    nodeB.add(nodeC); 
15. ​    tree.root.add(nodeB); 
16. ​    System.out.println("build the tree finished!"); 
17.   } 
18. } 

使用场景：将多个对象组合在一起进行操作，常用于表示树形结构中，例如二叉树，数等。

**12、享元模式（Flyweight）**

享元模式的主要目的是实现对象的共享，即共享池，当系统中对象多的时候可以减少内存的开销，通常与工厂模式一起使用。

![img](http://dl.iteye.com/upload/attachment/0083/1207/f7aae0dd-b250-3829-bb07-49d87069bfbb.jpg)

FlyWeightFactory负责创建和管理享元单元，当一个客户端请求时，工厂需要检查当前对象池中是否有符合条件的对象，如果有，就返回已经存在的对象，如果没有，则创建一个新对象，FlyWeight是超类。一提到共享池，我们很容易联想到Java里面的JDBC连接池，想想每个连接的特点，我们不难总结出：适用于作共享的一些个对象，他们有一些共有的属性，就拿数据库连接池来说，url、driverClassName、username、password及dbname，这些属性对于每个连接来说都是一样的，所以就适合用享元模式来处理，建一个工厂类，将上述类似属性作为内部数据，其它的作为外部数据，在方法调用时，当做参数传进来，这样就节省了空间，减少了实例的数量。

看个例子：

![img](http://dl.iteye.com/upload/attachment/0083/1209/53bc0bf4-cafb-3a12-8574-e20a525f2b72.jpg)

看下数据库连接池的代码：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8239539)[copy](http://blog.csdn.net/zhangerqing/article/details/8239539)

1. public class ConnectionPool { 
2.    
3.   private Vector<Connection> pool; 
4.    
5.   /*公有属性*/ 
6.   private String url = "jdbc:mysql://localhost:3306/test"; 
7.   private String username = "root"; 
8.   private String password = "root"; 
9.   private String driverClassName = "com.mysql.jdbc.Driver"; 
10.  
11.   private int poolSize = 100; 
12.   private static ConnectionPool instance = null; 
13.   Connection conn = null; 
14.  
15.   /*构造方法，做一些初始化工作*/ 
16.   private ConnectionPool() { 
17. ​    pool = new Vector<Connection>(poolSize); 
18.  
19. ​    for (int i = 0; i < poolSize; i++) { 
20. ​      try { 
21. ​        Class.forName(driverClassName); 
22. ​        conn = DriverManager.getConnection(url, username, password); 
23. ​        pool.add(conn); 
24. ​      } catch (ClassNotFoundException e) { 
25. ​        e.printStackTrace(); 
26. ​      } catch (SQLException e) { 
27. ​        e.printStackTrace(); 
28. ​      } 
29. ​    } 
30.   } 
31.  
32.   /* 返回连接到连接池 */ 
33.   public synchronized void release() { 
34. ​    pool.add(conn); 
35.   } 
36.  
37.   /* 返回连接池中的一个数据库连接 */ 
38.   public synchronized Connection getConnection() { 
39. ​    if (pool.size() > 0) { 
40. ​      Connection conn = pool.get(0); 
41. ​      pool.remove(conn); 
42. ​      return conn; 
43. ​    } else { 
44. ​      return null; 
45. ​    } 
46.   } 
47. } 

 

通过连接池的管理，实现了数据库连接的共享，不需要每一次都重新创建连接，节省了数据库重新创建的开销，提升了系统的性能！本章讲解了7种结构型模式，因为篇幅的问题，剩下的11种行为型模式，

本章是关于设计模式的最后一讲，会讲到第三种设计模式——行为型模式，共11种：策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。这段时间一直在写关于设计模式的东西，终于写到一半了，写博文是个很费时间的东西，因为我得为读者负责，不论是图还是代码还是表述，都希望能尽量写清楚，以便读者理解，我想不论是我还是读者，都希望看到高质量的博文出来，从我本人出发，我会一直坚持下去，不断更新，源源动力来自于读者朋友们的不断支持，我会尽自己的努力，写好每一篇文章！希望大家能不断给出意见和建议，共同打造完美的博文！

 

 

先来张图，看看这11中模式的关系：

第一类：通过父类与子类的关系进行实现。第二类：两个类之间。第三类：类的状态。第四类：通过中间类

![img](http://dl.iteye.com/upload/attachment/0083/1211/5e2feb58-4170-3c07-a370-ed99bdcab223.jpg)

**13、策略模式（strategy）**

策略模式定义了一系列算法，并将每个算法封装起来，使他们可以相互替换，且算法的变化不会影响到使用算法的客户。需要设计一个接口，为一系列实现类提供统一的方法，多个实现类实现该接口，设计一个抽象类（可有可无，属于辅助类），提供辅助函数，关系图如下：

![img](http://dl.iteye.com/upload/attachment/0083/1213/2319a2c3-7ebd-3ee3-b389-1548074ea9c6.jpg)

图中ICalculator提供同意的方法，
AbstractCalculator是辅助类，提供辅助方法，接下来，依次实现下每个类：

首先统一接口：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public interface ICalculator { 
2.   public int calculate(String exp); 
3. } 

辅助类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public abstract class AbstractCalculator { 
2.    
3.   public int[] split(String exp,String opt){ 
4. ​    String array[] = exp.split(opt); 
5. ​    int arrayInt[] = new int[2]; 
6. ​    arrayInt[0] = Integer.parseInt(array[0]); 
7. ​    arrayInt[1] = Integer.parseInt(array[1]); 
8. ​    return arrayInt; 
9.   } 
10. } 

三个实现类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class Plus extends AbstractCalculator implements ICalculator { 
2.  
3.   @Override 
4.   public int calculate(String exp) { 
5. ​    int arrayInt[] = split(exp,"\\+"); 
6. ​    return arrayInt[0]+arrayInt[1]; 
7.   } 
8. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class Minus extends AbstractCalculator implements ICalculator { 
2.  
3.   @Override 
4.   public int calculate(String exp) { 
5. ​    int arrayInt[] = split(exp,"-"); 
6. ​    return arrayInt[0]-arrayInt[1]; 
7.   } 
8.  
9. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class Multiply extends AbstractCalculator implements ICalculator { 
2.  
3.   @Override 
4.   public int calculate(String exp) { 
5. ​    int arrayInt[] = split(exp,"\\*"); 
6. ​    return arrayInt[0]*arrayInt[1]; 
7.   } 
8. } 

简单的测试类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class StrategyTest { 
2.  
3.   public static void main(String[] args) { 
4. ​    String exp = "2+8"; 
5. ​    ICalculator cal = new Plus(); 
6. ​    int result = cal.calculate(exp); 
7. ​    System.out.println(result); 
8.   } 
9. } 

输出：10

策略模式的决定权在用户，系统本身提供不同算法的实现，新增或者删除算法，对各种算法做封装。因此，策略模式多用在算法决策系统中，外部用户只需要决定用哪个算法即可。

**14、模板方法模式（Template Method）**

解释一下模板方法模式，就是指：一个抽象类中，有一个主方法，再定义1...n个方法，可以是抽象的，也可以是实际的方法，定义一个类，继承该抽象类，重写抽象方法，通过调用抽象类，实现对子类的调用，先看个关系图：

![img](http://dl.iteye.com/upload/attachment/0083/1215/c3d57775-ddf9-302b-9dfe-c65967518d3c.jpg)

就是在AbstractCalculator类中定义一个主方法calculate，calculate()调用spilt()等，Plus和Minus分别继承AbstractCalculator类，通过对AbstractCalculator的调用实现对子类的调用，看下面的例子：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public abstract class AbstractCalculator { 
2.    
3.   /*主方法，实现对本类其它方法的调用*/ 
4.   public final int calculate(String exp,String opt){ 
5. ​    int array[] = split(exp,opt); 
6. ​    return calculate(array[0],array[1]); 
7.   } 
8.    
9.   /*被子类重写的方法*/ 
10.   abstract public int calculate(int num1,int num2); 
11.    
12.   public int[] split(String exp,String opt){ 
13. ​    String array[] = exp.split(opt); 
14. ​    int arrayInt[] = new int[2]; 
15. ​    arrayInt[0] = Integer.parseInt(array[0]); 
16. ​    arrayInt[1] = Integer.parseInt(array[1]); 
17. ​    return arrayInt; 
18.   } 
19. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class Plus extends AbstractCalculator { 
2.  
3.   @Override 
4.   public int calculate(int num1,int num2) { 
5. ​    return num1 + num2; 
6.   } 
7. } 

测试类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class StrategyTest { 
2.  
3.   public static void main(String[] args) { 
4. ​    String exp = "8+8"; 
5. ​    AbstractCalculator cal = new Plus(); 
6. ​    int result = cal.calculate(exp, "\\+"); 
7. ​    System.out.println(result); 
8.   } 
9. } 

我跟踪下这个小程序的执行过程：首先将exp和"\\+"做参数，调用AbstractCalculator类里的calculate(String,String)方法，在calculate(String,String)里调用同类的split()，之后再调用calculate(int ,int)方法，从这个方法进入到子类中，执行完return num1 +  num2后，将值返回到AbstractCalculator类，赋给result，打印出来。正好验证了我们开头的思路。

**15、观察者模式（Observer）**

包括这个模式在内的接下来的四个模式，都是类和类之间的关系，不涉及到继承，学的时候应该  记得归纳，记得本文最开始的那个图。观察者模式很好理解，类似于邮件订阅和RSS订阅，当我们浏览一些博客或wiki时，经常会看到RSS图标，就这的意思是，当你订阅了该文章，如果后续有更新，会及时通知你。其实，简单来讲就一句话：当一个对象变化时，其它依赖该对象的对象都会收到通知，并且随着变化！对象之间是一种一对多的关系。先来看看关系图：

![img](http://dl.iteye.com/upload/attachment/0083/1233/d588525c-fbad-3040-971c-69b2716c67a4.jpg)

我解释下这些类的作用：MySubject类就是我们的主对象，Observer1和Observer2是依赖于MySubject的对象，当MySubject变化时，Observer1和Observer2必然变化。AbstractSubject类中定义着需要监控的对象列表，可以对其进行修改：增加或删除被监控对象，且当MySubject变化时，负责通知在列表内存在的对象。我们看实现代码：

一个Observer接口：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public interface Observer { 
2.   public void update(); 
3. } 

两个实现类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class Observer1 implements Observer { 
2.  
3.   @Override 
4.   public void update() { 
5. ​    System.out.println("observer1 has received!"); 
6.   } 
7. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class Observer2 implements Observer { 
2.  
3.   @Override 
4.   public void update() { 
5. ​    System.out.println("observer2 has received!"); 
6.   } 
7.  
8. } 

Subject接口及实现类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public interface Subject { 
2.    
3.   /*增加观察者*/ 
4.   public void add(Observer observer); 
5.    
6.   /*删除观察者*/ 
7.   public void del(Observer observer); 
8.    
9.   /*通知所有的观察者*/ 
10.   public void notifyObservers(); 
11.    
12.   /*自身的操作*/ 
13.   public void operation(); 
14. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public abstract class AbstractSubject implements Subject { 
2.  
3.   private Vector<Observer> vector = new Vector<Observer>(); 
4.   @Override 
5.   public void add(Observer observer) { 
6. ​    vector.add(observer); 
7.   } 
8.  
9.   @Override 
10.   public void del(Observer observer) { 
11. ​    vector.remove(observer); 
12.   } 
13.  
14.   @Override 
15.   public void notifyObservers() { 
16. ​    Enumeration<Observer> enumo = vector.elements(); 
17. ​    while(enumo.hasMoreElements()){ 
18. ​      enumo.nextElement().update(); 
19. ​    } 
20.   } 
21. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class MySubject extends AbstractSubject { 
2.  
3.   @Override 
4.   public void operation() { 
5. ​    System.out.println("update self!"); 
6. ​    notifyObservers(); 
7.   } 
8.  
9. } 


测试类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class ObserverTest { 
2.  
3.   public static void main(String[] args) { 
4. ​    Subject sub = new MySubject(); 
5. ​    sub.add(new Observer1()); 
6. ​    sub.add(new Observer2()); 
7. ​     
8. ​    sub.operation(); 
9.   } 
10.  
11. } 

输出：

update self!
observer1 has received!
observer2 has received!

 这些东西，其实不难，只是有些抽象，不太容易整体理解，建议读者：**根据关系图，新建项目，自己写代码（或者参考我的代码）,\**按照\**总体思路走一遍，这样才能体会它的思想，理解起来容易！** 

**16、迭代子模式（Iterator）**

顾名思义，迭代器模式就是顺序访问聚集中的对象，一般来说，集合中非常常见，如果对集合类比较熟悉的话，理解本模式会十分轻松。这句话包含两层意思：一是需要遍历的对象，即聚集对象，二是迭代器对象，用于对聚集对象进行遍历访问。我们看下关系图：

 ![img](http://dl.iteye.com/upload/attachment/0083/1217/f7571a69-3c85-3fe1-b781-e460563a40a8.jpg)

这个思路和我们常用的一模一样，MyCollection中定义了集合的一些操作，MyIterator中定义了一系列迭代操作，且持有Collection实例，我们来看看实现代码：

两个接口：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public interface Collection { 
2.    
3.   public Iterator iterator(); 
4.    
5.   /*取得集合元素*/ 
6.   public Object get(int i); 
7.    
8.   /*取得集合大小*/ 
9.   public int size(); 
10. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public interface Iterator { 
2.   //前移 
3.   public Object previous(); 
4.    
5.   //后移 
6.   public Object next(); 
7.   public boolean hasNext(); 
8.    
9.   //取得第一个元素 
10.   public Object first(); 
11. } 

两个实现：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class MyCollection implements Collection { 
2.  
3.   public String string[] = {"A","B","C","D","E"}; 
4.   @Override 
5.   public Iterator iterator() { 
6. ​    return new MyIterator(this); 
7.   } 
8.  
9.   @Override 
10.   public Object get(int i) { 
11. ​    return string[i]; 
12.   } 
13.  
14.   @Override 
15.   public int size() { 
16. ​    return string.length; 
17.   } 
18. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class MyIterator implements Iterator { 
2.  
3.   private Collection collection; 
4.   private int pos = -1; 
5.    
6.   public MyIterator(Collection collection){ 
7. ​    this.collection = collection; 
8.   } 
9.    
10.   @Override 
11.   public Object previous() { 
12. ​    if(pos > 0){ 
13. ​      pos--; 
14. ​    } 
15. ​    return collection.get(pos); 
16.   } 
17.  
18.   @Override 
19.   public Object next() { 
20. ​    if(pos<collection.size()-1){ 
21. ​      pos++; 
22. ​    } 
23. ​    return collection.get(pos); 
24.   } 
25.  
26.   @Override 
27.   public boolean hasNext() { 
28. ​    if(pos<collection.size()-1){ 
29. ​      return true; 
30. ​    }else{ 
31. ​      return false; 
32. ​    } 
33.   } 
34.  
35.   @Override 
36.   public Object first() { 
37. ​    pos = 0; 
38. ​    return collection.get(pos); 
39.   } 
40.  
41. } 

测试类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class Test { 
2.  
3.   public static void main(String[] args) { 
4. ​    Collection collection = new MyCollection(); 
5. ​    Iterator it = collection.iterator(); 
6. ​     
7. ​    while(it.hasNext()){ 
8. ​      System.out.println(it.next()); 
9. ​    } 
10.   } 
11. } 

输出：A B C D E

此处我们貌似模拟了一个集合类的过程，感觉是不是很爽？其实JDK中各个类也都是这些基本的东西，加一些设计模式，再加一些优化放到一起的，只要我们把这些东西学会了，掌握好了，我们也可以写出自己的集合类，甚至框架！

**17、责任链模式（Chain of Responsibility）**

下来我们将要谈谈责任链模式，有多个对象，每个对象持有对下一个对象的引用，这样就会形成一条链，请求在这条链上传递，直到某一对象决定处理该请求。但是发出者并不清楚到底最终那个对象会处理该请求，所以，责任链模式可以实现，在隐瞒客户端的情况下，对系统进行动态的调整。先看看关系图：

 ![img](http://dl.iteye.com/upload/attachment/0083/1219/729a82ce-0987-347c-a4f1-bf64dee59ddb.jpg)

 

Abstracthandler类提供了get和set方法，方便MyHandle类设置和修改引用对象，MyHandle类是核心，实例化后生成一系列相互持有的对象，构成一条链。

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public interface Handler { 
2.   public void operator(); 
3. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public abstract class AbstractHandler { 
2.    
3.   private Handler handler; 
4.  
5.   public Handler getHandler() { 
6. ​    return handler; 
7.   } 
8.  
9.   public void setHandler(Handler handler) { 
10. ​    this.handler = handler; 
11.   } 
12.    
13. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class MyHandler extends AbstractHandler implements Handler { 
2.  
3.   private String name; 
4.  
5.   public MyHandler(String name) { 
6. ​    this.name = name; 
7.   } 
8.  
9.   @Override 
10.   public void operator() { 
11. ​    System.out.println(name+"deal!"); 
12. ​    if(getHandler()!=null){ 
13. ​      getHandler().operator(); 
14. ​    } 
15.   } 
16. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class Test { 
2.  
3.   public static void main(String[] args) { 
4. ​    MyHandler h1 = new MyHandler("h1"); 
5. ​    MyHandler h2 = new MyHandler("h2"); 
6. ​    MyHandler h3 = new MyHandler("h3"); 
7.  
8. ​    h1.setHandler(h2); 
9. ​    h2.setHandler(h3); 
10.  
11. ​    h1.operator(); 
12.   } 
13. } 

输出：

h1deal!
h2deal!
h3deal!

此处强调一点就是，链接上的请求可以是一条链，可以是一个树，还可以是一个环，模式本身不约束这个，需要我们自己去实现，同时，在一个时刻，命令只允许由一个对象传给另一个对象，而不允许传给多个对象。

 **18、命令模式（Command）**

命令模式很好理解，举个例子，司令员下令让士兵去干件事情，从整个事情的角度来考虑，司令员的作用是，发出口令，口令经过传递，传到了士兵耳朵里，士兵去执行。这个过程好在，三者相互解耦，任何一方都不用去依赖其他人，只需要做好自己的事儿就行，司令员要的是结果，不会去关注到底士兵是怎么实现的。我们看看关系图：

![img](http://dl.iteye.com/upload/attachment/0083/1239/98cda4fc-00b1-300d-a25b-63229f0f1cbd.jpg)

Invoker是调用者（司令员），Receiver是被调用者（士兵），MyCommand是命令，实现了Command接口，持有接收对象，看实现代码：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public interface Command { 
2.   public void exe(); 
3. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class MyCommand implements Command { 
2.  
3.   private Receiver receiver; 
4.    
5.   public MyCommand(Receiver receiver) { 
6. ​    this.receiver = receiver; 
7.   } 
8.  
9.   @Override 
10.   public void exe() { 
11. ​    receiver.action(); 
12.   } 
13. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class Receiver { 
2.   public void action(){ 
3. ​    System.out.println("command received!"); 
4.   } 
5. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class Invoker { 
2.    
3.   private Command command; 
4.    
5.   public Invoker(Command command) { 
6. ​    this.command = command; 
7.   } 
8.  
9.   public void action(){ 
10. ​    command.exe(); 
11.   } 
12. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8243942)[copy](http://blog.csdn.net/zhangerqing/article/details/8243942)

1. public class Test { 
2.  
3.   public static void main(String[] args) { 
4. ​    Receiver receiver = new Receiver(); 
5. ​    Command cmd = new MyCommand(receiver); 
6. ​    Invoker invoker = new Invoker(cmd); 
7. ​    invoker.action(); 
8.   } 
9. } 

输出：command received!

这个很哈理解，命令模式的目的就是达到命令的发出者和执行者之间解耦，实现请求和执行分开，熟悉Struts的同学应该知道，Struts其实就是一种将请求和呈现分离的技术，其中必然涉及命令模式的思想！

其实每个设计模式都是很重要的一种思想，看上去很熟，其实是因为我们在学到的东西中都有涉及，尽管有时我们并不知道，其实在Java本身的设计之中处处都有体现，像AWT、JDBC、集合类、IO管道或者是Web框架，里面设计模式无处不在。因为我们篇幅有限，很难讲每一个设计模式都讲的很详细，不过我会尽我所能，尽量在有限的空间和篇幅内，把意思写清楚了，更好让大家明白。本章不出意外的话，应该是设计模式最后一讲了，首先还是上一下上篇开头的那个图：

![点击查看原始大小图片](http://dl.iteye.com/upload/attachment/0083/1221/5d3c9b85-c281-3c48-999c-d27095c6ec9f.jpg)

本章讲讲第三类和第四类。

**19、备忘录模式（Memento）**

主要目的是保存一个对象的某个状态，以便在适当的时候恢复对象，个人觉得叫备份模式更形象些，通俗的讲下：假设有原始类A，A中有各种属性，A可以决定需要备份的属性，备忘录类B是用来存储A的一些内部状态，类C呢，就是一个用来存储备忘录的，且只能存储，不能修改等操作。做个图来分析一下：

![img](http://dl.iteye.com/upload/attachment/0083/1223/853d5c5a-9b7b-3341-a72e-abd3cbc3c81f.jpg)

Original类是原始类，里面有需要保存的属性value及创建一个备忘录类，用来保存value值。Memento类是备忘录类，Storage类是存储备忘录的类，持有Memento类的实例，该模式很好理解。直接看源码：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class Original { 
2.    
3.   private String value; 
4.    
5.   public String getValue() { 
6. ​    return value; 
7.   } 
8.  
9.   public void setValue(String value) { 
10. ​    this.value = value; 
11.   } 
12.  
13.   public Original(String value) { 
14. ​    this.value = value; 
15.   } 
16.  
17.   public Memento createMemento(){ 
18. ​    return new Memento(value); 
19.   } 
20.    
21.   public void restoreMemento(Memento memento){ 
22. ​    this.value = memento.getValue(); 
23.   } 
24. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class Memento { 
2.    
3.   private String value; 
4.  
5.   public Memento(String value) { 
6. ​    this.value = value; 
7.   } 
8.  
9.   public String getValue() { 
10. ​    return value; 
11.   } 
12.  
13.   public void setValue(String value) { 
14. ​    this.value = value; 
15.   } 
16. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class Storage { 
2.    
3.   private Memento memento; 
4.    
5.   public Storage(Memento memento) { 
6. ​    this.memento = memento; 
7.   } 
8.  
9.   public Memento getMemento() { 
10. ​    return memento; 
11.   } 
12.  
13.   public void setMemento(Memento memento) { 
14. ​    this.memento = memento; 
15.   } 
16. } 

测试类：

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class Test { 
2.  
3.   public static void main(String[] args) { 
4. ​     
5. ​    // 创建原始类 
6. ​    Original origi = new Original("egg"); 
7.  
8. ​    // 创建备忘录 
9. ​    Storage storage = new Storage(origi.createMemento()); 
10.  
11. ​    // 修改原始类的状态 
12. ​    System.out.println("初始化状态为：" + origi.getValue()); 
13. ​    origi.setValue("niu"); 
14. ​    System.out.println("修改后的状态为：" + origi.getValue()); 
15.  
16. ​    // 回复原始类的状态 
17. ​    origi.restoreMemento(storage.getMemento()); 
18. ​    System.out.println("恢复后的状态为：" + origi.getValue()); 
19.   } 
20. } 

输出：

初始化状态为：egg
修改后的状态为：niu
恢复后的状态为：egg

简单描述下：新建原始类时，value被初始化为egg，后经过修改，将value的值置为niu，最后倒数第二行进行恢复状态，结果成功恢复了。其实我觉得这个模式叫“备份-恢复”模式最形象。

**20、状态模式（State）**

核心思想就是：当对象的状态改变时，同时改变其行为，很好理解！就拿QQ来说，有几种状态，在线、隐身、忙碌等，每个状态对应不同的操作，而且你的好友也能看到你的状态，所以，状态模式就两点：1、可以通过改变状态来获得不同的行为。2、你的好友能同时看到你的变化。看图：

![img](http://dl.iteye.com/upload/attachment/0083/1225/006156d2-f41f-3019-a194-b872a59ca426.jpg)

State类是个状态类，Context类可以实现切换，我们来看看代码：

 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. package com.xtfggef.dp.state; 
2.  
3. /** 
4.  \* 状态类的核心类 
5.  \* 2012-12-1 
6.  \* @author erqing 
7.  \* 
8.  */ 
9. public class State { 
10.    
11.   private String value; 
12.    
13.   public String getValue() { 
14. ​    return value; 
15.   } 
16.  
17.   public void setValue(String value) { 
18. ​    this.value = value; 
19.   } 
20.  
21.   public void method1(){ 
22. ​    System.out.println("execute the first opt!"); 
23.   } 
24.    
25.   public void method2(){ 
26. ​    System.out.println("execute the second opt!"); 
27.   } 
28. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. package com.xtfggef.dp.state; 
2.  
3. /** 
4.  \* 状态模式的切换类  2012-12-1 
5.  \* @author erqing 
6.  \* 
7.  */ 
8. public class Context { 
9.  
10.   private State state; 
11.  
12.   public Context(State state) { 
13. ​    this.state = state; 
14.   } 
15.  
16.   public State getState() { 
17. ​    return state; 
18.   } 
19.  
20.   public void setState(State state) { 
21. ​    this.state = state; 
22.   } 
23.  
24.   public void method() { 
25. ​    if (state.getValue().equals("state1")) { 
26. ​      state.method1(); 
27. ​    } else if (state.getValue().equals("state2")) { 
28. ​      state.method2(); 
29. ​    } 
30.   } 
31. } 

测试类：

 

 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class Test { 
2.  
3.   public static void main(String[] args) { 
4. ​     
5. ​    State state = new State(); 
6. ​    Context context = new Context(state); 
7. ​     
8. ​    //设置第一种状态 
9. ​    state.setValue("state1"); 
10. ​    context.method(); 
11. ​     
12. ​    //设置第二种状态 
13. ​    state.setValue("state2"); 
14. ​    context.method(); 
15.   } 
16. } 

输出：

 

execute the first opt!
execute the second opt!

根据这个特性，状态模式在日常开发中用的挺多的，尤其是做网站的时候，我们有时希望根据对象的某一属性，区别开他们的一些功能，比如说简单的权限控制等。

**21、访问者模式（Visitor）**

访问者模式把数据结构和作用于结构上的操作解耦合，使得操作集合可相对自由地演化。访问者模式适用于数据结构相对稳定算法又易变化的系统。因为访问者模式使得算法操作增加变得容易。若系统数据结构对象易于变化，经常有新的数据对象增加进来，则不适合使用访问者模式。访问者模式的优点是增加操作很容易，因为增加操作意味着增加新的访问者。访问者模式将有关行为集中到一个访问者对象中，其改变不影响系统数据结构。其缺点就是增加新的数据结构很困难。—— From 百科

简单来说，访问者模式就是一种分离对象数据结构与行为的方法，通过这种分离，可达到为一个被访问者动态添加新的操作而无需做其它的修改的效果。简单关系图：

![img](http://dl.iteye.com/upload/attachment/0083/1227/96bd38f3-2888-3cc5-b90f-0e7542dc5845.jpg)

来看看原码：一个Visitor类，存放要访问的对象，

 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public interface Visitor { 
2.   public void visit(Subject sub); 
3. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class MyVisitor implements Visitor { 
2.  
3.   @Override 
4.   public void visit(Subject sub) { 
5. ​    System.out.println("visit the subject："+sub.getSubject()); 
6.   } 
7. } 

Subject类，accept方法，接受将要访问它的对象，getSubject()获取将要被访问的属性，

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public interface Subject { 
2.   public void accept(Visitor visitor); 
3.   public String getSubject(); 
4. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class MySubject implements Subject { 
2.  
3.   @Override 
4.   public void accept(Visitor visitor) { 
5. ​    visitor.visit(this); 
6.   } 
7.  
8.   @Override 
9.   public String getSubject() { 
10. ​    return "love"; 
11.   } 
12. } 

测试：

 

 

 

 

 

 

 

 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class Test { 
2.  
3.   public static void main(String[] args) { 
4. ​     
5. ​    Visitor visitor = new MyVisitor(); 
6. ​    Subject sub = new MySubject(); 
7. ​    sub.accept(visitor);   
8.   } 
9. } 

输出：visit the subject：love

 

 

 

 

 

 

 

该模式适用场景：如果我们想为一个现有的类增加新功能，不得不考虑几个事情：1、新功能会不会与现有功能出现兼容性问题？2、以后会不会再需要添加？3、如果类不允许修改代码怎么办？面对这些问题，最好的解决方法就是使用访问者模式，访问者模式适用于数据结构相对稳定的系统，把数据结构和算法解耦。

**22、中介者模式（Mediator）**

中介者模式也是用来降低类类之间的耦合的，因为如果类类之间有依赖关系的话，不利于功能的拓展和维护，因为只要修改一个对象，其它关联的对象都得进行修改。如果使用中介者模式，只需关心和Mediator类的关系，具体类类之间的关系及调度交给Mediator就行，这有点像spring容器的作用。先看看图：

![img](http://dl.iteye.com/upload/attachment/0083/1229/f1f2cc36-ab27-32fa-9906-9cdee2c2b625.jpg)

User类统一接口，User1和User2分别是不同的对象，二者之间有关联，如果不采用中介者模式，则需要二者相互持有引用，这样二者的耦合度很高，为了解耦，引入了Mediator类，提供统一接口，MyMediator为其实现类，里面持有User1和User2的实例，用来实现对User1和User2的控制。这样User1和User2两个对象相互独立，他们只需要保持好和Mediator之间的关系就行，剩下的全由MyMediator类来维护！基本实现：

 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public interface Mediator { 
2.   public void createMediator(); 
3.   public void workAll(); 
4. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class MyMediator implements Mediator { 
2.  
3.   private User user1; 
4.   private User user2; 
5.    
6.   public User getUser1() { 
7. ​    return user1; 
8.   } 
9.  
10.   public User getUser2() { 
11. ​    return user2; 
12.   } 
13.  
14.   @Override 
15.   public void createMediator() { 
16. ​    user1 = new User1(this); 
17. ​    user2 = new User2(this); 
18.   } 
19.  
20.   @Override 
21.   public void workAll() { 
22. ​    user1.work(); 
23. ​    user2.work(); 
24.   } 
25. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public abstract class User { 
2.    
3.   private Mediator mediator; 
4.    
5.   public Mediator getMediator(){ 
6. ​    return mediator; 
7.   } 
8.    
9.   public User(Mediator mediator) { 
10. ​    this.mediator = mediator; 
11.   } 
12.  
13.   public abstract void work(); 
14. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class User1 extends User { 
2.  
3.   public User1(Mediator mediator){ 
4. ​    super(mediator); 
5.   } 
6.    
7.   @Override 
8.   public void work() { 
9. ​    System.out.println("user1 exe!"); 
10.   } 
11. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class User2 extends User { 
2.  
3.   public User2(Mediator mediator){ 
4. ​    super(mediator); 
5.   } 
6.    
7.   @Override 
8.   public void work() { 
9. ​    System.out.println("user2 exe!"); 
10.   } 
11. } 

测试类：

 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class Test { 
2.  
3.   public static void main(String[] args) { 
4. ​    Mediator mediator = new MyMediator(); 
5. ​    mediator.createMediator(); 
6. ​    mediator.workAll(); 
7.   } 
8. } 

输出：

user1 exe!
user2 exe!

**23、解释器模式（Interpreter）**

解释器模式是我们暂时的最后一讲，一般主要应用在OOP开发中的编译器的开发中，所以适用面比较窄。

![img](http://dl.iteye.com/upload/attachment/0083/1231/c87e402e-a355-3761-9ce3-7978956ba475.jpg)

Context类是一个上下文环境类，Plus和Minus分别是用来计算的实现，代码如下：

 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public interface Expression { 
2.   public int interpret(Context context); 
3. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class Plus implements Expression { 
2.  
3.   @Override 
4.   public int interpret(Context context) { 
5. ​    return context.getNum1()+context.getNum2(); 
6.   } 
7. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class Minus implements Expression { 
2.  
3.   @Override 
4.   public int interpret(Context context) { 
5. ​    return context.getNum1()-context.getNum2(); 
6.   } 
7. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class Context { 
2.    
3.   private int num1; 
4.   private int num2; 
5.    
6.   public Context(int num1, int num2) { 
7. ​    this.num1 = num1; 
8. ​    this.num2 = num2; 
9.   } 
10.    
11.   public int getNum1() { 
12. ​    return num1; 
13.   } 
14.   public void setNum1(int num1) { 
15. ​    this.num1 = num1; 
16.   } 
17.   public int getNum2() { 
18. ​    return num2; 
19.   } 
20.   public void setNum2(int num2) { 
21. ​    this.num2 = num2; 
22.   } 
23.    
24.    
25. } 

**[java]** [view plain](http://blog.csdn.net/zhangerqing/article/details/8245537)[copy](http://blog.csdn.net/zhangerqing/article/details/8245537)

1. public class Test { 
2.  
3.   public static void main(String[] args) { 
4.  
5. ​    // 计算9+2-8的值 
6. ​    int result = new Minus().interpret((new Context(new Plus() 
7. ​        .interpret(new Context(9, 2)), 8))); 
8. ​    System.out.println(result); 
9.   } 
10. } 

最后输出正确的结果：3。

 

基本就这样，解释器模式用来做各种各样的解释器，如正则表达式等的解释器等等！



资源：http://download.csdn.net/detail/zhangerqing/4835830

 

原文链接：**http://blog.csdn.net/zhangerqing**

