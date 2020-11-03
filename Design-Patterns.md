# Design Patterns

## 创建型模式

创建型模式的主要关注点是“怎样创建对象？”，它的主要特点是“将对象的创建与使用分离”。这样可以降低系统的耦合度，使用者不需要关注对象的创建细节，对象的创建由相关的工厂来完成。

1. 单例（Singleton）模式：某个类只能生成一个实例，该类提供了一个全局访问点供外部获取该实例，其拓展是有限多例模式。
2. 原型（Prototype）模式：将一个对象作为原型，通过对其进行复制而克隆出多个和原型类似的新实例。
3. 工厂方法（Factory Method）模式：定义一个用于创建产品的接口，由子类决定生产什么产品。
4. 抽象工厂（Abstract Factory）模式：提供一个创建产品族的接口，其每个子类可以生产一系列相关的产品。
5. 建造者（Builder）模式：将一个复杂对象分解成多个相对简单的部分，然后根据不同需要分别创建它们，最后构建成该复杂对象。

### 单例模式

一个类在整个软件系统只有一个实例。在有些系统中，可能为了节省内存资源，或者保证数据内容的一致性，或者避免对资源的多重占用，对某些类要求只能创建一个实例，比如操作系统的文件系统、数据库连接池等通常会被设计成单例模式。单例模式的特点有3个：

1. 单例类只有一个实例对象
2. 单例对象是有单例类自行创建的
3. 单例类对外提供了一个访问单例的全局访问点

##### 实现

单例模式的实现方式通常有饿汉式、懒汉式、静态内部类、枚举

+ 饿汉式：类被加载时，它的实例对象就被创建好了

```java
public class Singleton {  
    private static Singleton instance = new Singleton();  
    private Singleton (){}  
    public static Singleton getInstance() {  
    return instance;  
    }  
}
```

+ 懒汉式：类加载时并没有生成单例，只有当第一次使用时，实例对象才会被创建

```java
public class Singleton {  
    private static Singleton instance;  
    private Singleton (){}  
    public static synchronized Singleton getInstance() {  
    if (instance == null) {  
        instance = new Singleton();  
    }  
    return instance;  
    }  
}
```

双重检查：对象在创建的过程的一些步骤可能会被重排序，从而导致在多线程情况下，线程可能会拿到一个不为null但还未初始化完成的对象去使用；对象创建的步骤大概分为3步：

1. 分配对象内存空间
2. 初始化对象
3. 变量引用指向内存地址

第2步和第3步可能被重排序，那么其他线程就可拿到一个不为null但还未初始化的对象。使用volatile可以解决这个问题；当声明对象的引用为volatile后，之前的3行伪代码中的2和3之间的重排序，在多线程环境中将会被禁止

```java
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
    if (singleton == null) {  
        synchronized (Singleton.class) {  
        if (singleton == null) {  
            singleton = new Singleton();  
        }  
        }  
    }  
    return singleton;  
    }  
}
```

+ 静态内部类(Lazy)：在一个静态内部类中创建实例，它并不会是类加载时就会创建实例，而是当需要使用到实例对象，显式调用才会装载静态内部类，从而实例化对象

```java
public class Singleton {
    //静态内部类
    private static class SingletonHolder {  
        private static final Singleton INSTANCE = new Singleton();  
    }
    
    private Singleton (){}  
    
    public static final Singleton getInstance() {  
        return SingletonHolder.INSTANCE;  
    }  

```

+ 枚举

```java
public enum Singleton {  
    INSTANCE;  
    public void whateverMethod() {  
    }  
}
```

##### 应用场景

单例模式保证了系统内存中该类只存在一个对象，节省了系统资源，对于一些需要频繁创建销毁的对象，使用单例模式可以提高系统性能，例如工具类、频繁访问数据库或文件的对象（比如数据源、session工厂）
+ JDK：Runtime ，Runtime类封装了Java运行时的环境。每一个java程序实际上都是启动了一个JVM进程，那么每个JVM进程都是对应这一个Runtime实例，此实例是由JVM为其实例化的。每个 Java 应用程序都有一个 Runtime 类实例，使应用程序能够与其运行的环境相连接。由于Java是单进程的，所以，在一个JVM中，Runtime的实例应该只有一个

```java
public class Runtime {
    private static Runtime currentRuntime = new Runtime();

    public static Runtime getRuntime() {
        return currentRuntime;
    }

    private Runtime() {
    }
```

### 原型模式

用一个已经创建的实例作为原型，通过复制该原型对象来创建一个和原型相同或相似的新对象。这与通过对一个类进行实例化来构造新对象不同，原型实例指定了要创建的对象的种类，并且通过拷贝这些原型创建新的对象。当系统需要创建大量相同的对象时，可以使用原型模式生成对象，用这种方式创建对象非常高效，根本无须知道对象创建的细节。

##### 角色

+ 抽象原型类：声明一个具有克隆自己方法clone()的接口
+ 具体原型类：实现原型接口并实现克隆自己的clone()方法

##### 实现

+ 浅拷贝：当存在对象成员属性时，它并没有将象成员重新复制一份，只是引用指向同一对象成员；对于基本数据类型的成员变量，浅拷贝会直接将属性值复制一份给新对象；对于引用数据类型的成员变量，比如成员变量是某个数组，某个类的对象等，浅拷贝会进行引用传递，也就是将该成员变量的引用值复制给一份新的对象。Java 中的 Object 类提供了浅克隆的 clone() 方法，具体原型类只要实现 Cloneable 接口就可实现对象的浅克隆

抽象原型类

```java
public interface Cloneable {
}
```

具体原型类

```java
public class PrototypeSheep implements Cloneable {

    private String name;
    private int age;
    private String color;

    //实现克隆(浅拷贝)
    @Override
    protected Object clone() {
        PrototypeSheep sheep =null;
        try {
            sheep = (PrototypeSheep)super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return sheep;
    }
```

+ 深拷贝：复制所有基本数据类型的成员变量值，为所有引用数据类型的成员变量申请存储空间，并复制每个引用数据类型成员变量所引用的对象。实现方式：重写clone方法、对象序列化

抽象原型类

```java
public interface Cloneable {
}
```

具体原型类

```java
public class DeepCopySheep implements Serializable,Cloneable {
    private String name;
    private int age;
    private String color;
    private Integer[] nums;

    //深拷贝方式一：重写clone
    @Override
    protected Object clone()  {
        DeepCopySheep deepCopySheep=null;
        try {
            //完成基本数据类型拷贝
            deepCopySheep = (DeepCopySheep)super.clone();
            //引用类型属性单独处理
            deepCopySheep.setNums(this.nums.clone());

        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return deepCopySheep;
    }

    //深拷贝方式二：序列化
    public Object deepClone(){
        //创建流对象
        ByteArrayOutputStream bos =null;
        ObjectOutputStream oos =null;
        ByteArrayInputStream bis=null;
        ObjectInputStream ois=null;
        DeepCopySheep deepCopySheep =null;
        try{
            //将当前对象序列化
            bos = new ByteArrayOutputStream();
            oos = new ObjectOutputStream(bos);
            oos.writeObject(this);

            //再将当前对象反序列化
            bis =new ByteArrayInputStream(bos.toByteArray());
            ois = new ObjectInputStream(bis);
            deepCopySheep =(DeepCopySheep)ois.readObject();

        }catch (Exception e){
            e.printStackTrace();
            return null;
        }finally {
            if (bos!=null){
                try {
                    bos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (oos!=null){
                try {
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (bis!=null){
                try {
                    bis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (ois!=null){
                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return deepCopySheep;
    }
}
```

##### 应用场景

​		系统中大量使用该类对象，且各个调用者都需要给它的属性重新赋值；对象之间相同或相似，即只是个别的几个属性不同的时候。通过 new 产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式；创建对象成本较大，例如初始化时间长，占用CPU太多，或者占用网络资源太多等，需要优化资源。

+ java.lang.Object#clone()

### 简单工厂模式

**一个抽象产品类，可以派生出多个具体产品类。一个具体工厂类，通过往此工厂的static方法中传入不同参数，产出不同的具体产品类实例。**简单工厂模式有一个具体的工厂类，可以生成多个不同的产品。一旦增加新产品不得不修改工厂逻辑，在产品类型较多时，可能造成逻辑过于复杂。

##### 角色

+ 简单工厂：负责实现创建所有实例的内部逻辑。工厂类的创建产品类的方法可以被外界直接调用，创建所需的产品对象。
+ 抽象产品：工厂创建对象的抽象父类，负责描述所有实例共有的公共接口
+ 具体产品：工厂创建具体对象

##### 实现

抽象产品：描述产品的公共接口

```java
abstract  class Product {
    //产品介绍
    abstract void intro();
}
```

具体产品：描述生产的具体产品

```java
//具体产品A
public class AProduct extends Product{
    @Override
    void intro() {
        System.out.println("可乐");
    }
}

//具体产品B
public class BProduct extends Product{
    @Override
    void intro() {
        System.out.println("奶茶");
    }
}

//具体产品C
public class CProduct extends Product{
    @Override
    void intro() {
        System.out.println("咖啡");
    }
}
```

简单工厂：负责实现创建所有实例的内部逻辑，并提供一个外界调用的方法，创建所需的产品对象。

```java
public class Factory {
    public static Product getProduct(String type) {
        switch (type) {
            case "A":
                return new AProduct();
            case "B":
                return new BProduct();
            case "C":
                return new CProduct();
            default:
                return null;
        }
    }
}
```



##### 应用场景

对于产品种类相对较少的情况，考虑使用简单工厂模式。使用简单工厂模式的客户端只需要传入工厂类的参数，不需要关心如何创建对象的逻辑，可以很方便地创建所需产品。

### 工厂方法模式

**一个抽象产品类，可以派生出多个具体产品类。一个抽象工厂类，可以派生出多个具体工厂类。每个具体工厂类只能创建一个具体产品类的实例**。与简单工厂相比，通过**定义工厂父类负责定义创建对象的公共接口，而子类则负责生成具体的对象**。将类的实例化（具体产品的创建）延迟到工厂类的子类（具体工厂）中完成，即由子类来决定应该实例化（创建）哪一个类。

##### 角色

+ 抽象工厂：提供了创建产品的接口，调用者通过它访问具体工厂的工厂方法 newProduct() 来创建产品。
+ 具体工厂：提供了创建产品的接口，调用者通过它访问具体工厂的工厂方法 newProduct() 来创建产品。
+ 抽象产品：定义了产品的规范，描述了产品的主要特性和功能。
+ 具体产品：实现了抽象产品角色所定义的接口，由具体工厂来创建，它同具体工厂之间一一对应。

##### 实现

抽象工厂：描述具体工厂的公共接口

```java
abstract class Factory {
    //生产产品
    abstract Product getProduct();
}
```

抽象产品：负责描述产品的公共接口

```java
abstract  class Product {
    //产品介绍
    abstract void intro();
}
```

具体产品：描述生产的具体产品

```java
//具体产品A
public class ProductA extends Product{
    @Override
    void intro() {
        System.out.println("饮料A");
    }
}

//具体产品B
public class ProductB extends Product{
    @Override
    void intro() {
        System.out.println("饮料B");
    }
}
```

具体工厂：描述具体工厂，创建产品的实例，供外界调用

```java
// 具体工厂A：负责具体的产品A生产
 public class FactoryA extends Factory{
    @Override
    Product getProduct() {
        return new ProductA();
    }
}

//具体工厂B：负责具体的产品B生产
public class FactoryB extends Factory{
    @Override
    Product getProduct() {
        return new ProductB();
    }
}
```

##### 应用场景

+ java.util.Calendar#getInstance()

```java
public static Calendar getInstance(){
        return createCalendar(TimeZone.getDefault(), Locale.getDefault(Locale.Category.FORMAT));
}
```

```java
private static Calendar createCalendar(TimeZone zone,
                                           Locale aLocale)
    {
        CalendarProvider provider =
            LocaleProviderAdapter.getAdapter(CalendarProvider.class, aLocale)
                                 .getCalendarProvider();
        if (provider != null) {
            try {
                return provider.getInstance(zone, aLocale);
            } catch (IllegalArgumentException iae) {
                // fall back to the default instantiation
            }
        }

        Calendar cal = null;

        if (aLocale.hasExtensions()) {
            String caltype = aLocale.getUnicodeLocaleType("ca");
            if (caltype != null) {
                switch (caltype) {
                case "buddhist":
                cal = new BuddhistCalendar(zone, aLocale);
                    break;
                case "japanese":
                    cal = new JapaneseImperialCalendar(zone, aLocale);
                    break;
                case "gregory":
                    cal = new GregorianCalendar(zone, aLocale);
                    break;
                }
            }
        }
        if (cal == null) {

            if (aLocale.getLanguage() == "th" && aLocale.getCountry() == "TH") {
                cal = new BuddhistCalendar(zone, aLocale);
            } else if (aLocale.getVariant() == "JP" && aLocale.getLanguage() == "ja"
                       && aLocale.getCountry() == "JP") {
                cal = new JapaneseImperialCalendar(zone, aLocale);
            } else {
                cal = new GregorianCalendar(zone, aLocale);
            }
        }
        return cal;
    }
```

### 抽象工厂模式

**提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类；具体的工厂负责实现具体的产品实例。** 解决的问题：每个工厂只能创建一类产品（工厂方法模式）

抽象工厂模式与工厂方法模式最大的区别：抽象工厂中每个工厂可以创建多种类的产品；而工厂方法每个工厂只能创建一类

##### 角色

+ 抽象工厂：提供了创建产品的接口，它包含多个创建产品的方法 newProduct()，可以创建多个不同等级的产品。
+ 具体工厂：主要是实现抽象工厂中的多个抽象方法，完成具体产品的创建。
+ 抽象产品族：定义了不同种类产品的公共规范
+ 抽象产品：定义了产品的规范，描述了不同产品的主要特性和功能，抽象工厂模式有多个抽象产品。
+ 具体产品：实现了抽象产品角色所定义的接口，由具体工厂来创建，它同具体工厂之间是多对一的关系。

##### 实现

抽象工厂：描述具体工厂的公共接口

```java
abstract class Factory {
    //生产A类产品
    abstract Product getProductA();
    //生产B类产品
    abstract Product getProductB();
}
```

抽象产品族：描述抽象产品的公共接口

```java
//抽象产品族 (食品)
abstract class Product {
    //产品介绍
    abstract void intro();
}
```

抽象产品：描述具体产品的公共接口

```java
//抽象A类产品 (饮料)
abstract class ProductA extends Product{
    @Override
    abstract void intro();
}


//抽象B类产品  (零食)
 abstract class ProductB extends Product{
    @Override
    abstract void intro();
}
```

具体产品：具体产品

```java
//具体产品A  (矿泉水)
public  class ProductAa extends ProductA{
    @Override
    void intro() {
        System.out.println("矿泉水");
    }
}

//抽象产品B  (面包)
public class ProductBb extends ProductB{
    @Override
    void intro() {
        System.out.println("面包");
    }
}
```

具体工厂

```java
//生产多种产品
public class FactoryA extends Factory{
    
     //A类产品：饮料
    @Override
    Product getProductA() {
        return new ProductAa();
    }
    
    //B类产品：食物
    @Override
    Product getProductB() {
        return new ProductBb();
    }
}
```

##### 应用场景

+ java.sql.DriverManager#getConnection()
+  java.sql.Connection#createStatement()
+ java.sql.Statement#executeQuery()

> 简单工厂：**由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类（继承自一个父类或接口）的实例。**
>
> 工厂方法：**定义工厂父类负责定义创建对象的公共接口，而子类则负责生成具体的对象**
>
> 抽象工厂：**提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类；具体的工厂负责实现具体的产品实例。**



>1. **工厂方法模式**解决了**简单工厂模式**的“开放 - 关闭原则
>2. 抽象工厂模式解决了**工厂方法模式**一个具体工厂只能创建一类产品

### 建造者模式

将一个复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示。使用多个简单的对象一步一步构建成一个复杂的对象，有点像造房子一样一步步从地基做起到万丈高楼。将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示；在用户不知道对象的建造过程和细节的情况下就可以直接创建复杂的对象。用户只需要给出指定复杂对象的类型和内容，建造者模式负责按顺序创建复杂对象（把内部的建造过程和细节隐藏起来）

与工厂模式的区别是：建造者模式更加关注与零件装配的顺序，一般用来创建更为复杂的对象

##### 角色

+ 产品：描述一个由一系列部件组成较为复杂的对象
+ 抽象建造者：描述具体建造者的公共接口，一般用来定义建造细节的方法，并不涉及具体的对象部件的创建
+ 具体建造者：描述具体建造者，并实现抽象建造者公共接口
+ 指挥者：调用具体建造者来创建复杂对象（产品）的各个部分，并按照一定顺序（流程）来建造复杂对象。

##### 实现

产品

```java
public class Product {
    private String buildA;
    private String buildB;
    private String buildC;
    private String buildD;
}
```

抽象建造者

```java
abstract class Builder {
    //地基
    abstract void bulidA();
    //钢筋工程
    abstract void bulidB();
    //铺电线
    abstract void bulidC();
    //粉刷
    abstract void bulidD();
    //完工-获取产品
    abstract Product getProduct();
}
```

具体建造者

```java
public class ConcreteBuilder extends Builder{
    private Product product;
    public ConcreteBuilder() {
        product = new Product();
    }
    @Override
    void bulidA() {
        product.setBuildA("地基");
    }
    @Override
    void bulidB() {
        product.setBuildB("钢筋工程");
    }
    @Override
    void bulidC() {
        product.setBuildC("铺电线");
    }
    @Override
    void bulidD() {
        product.setBuildD("粉刷");
    }
    @Override
    Product getProduct() {
        return product;
    }
}
```

指挥者

```java
public class Director {
    //指挥工人按顺序造房
    public Product create(Builder builder) {
        builder.bulidA();
        builder.bulidB();
        builder.bulidC();
        builder.bulidD();
        return builder.getProduct();
    }
}
```

##### 应用场景

+ StringBuilder和StringBuffer的append()方法使用了建造者模式。StringBuilder把构建者的角色交给了其的父类AbstractStringBuilder。append方法相当于建造者模式中建造者的对象组合方法，把boolean, char, int, long, float, double, String, char[], CharSequence, String, StringBuilder, StringBuffer, Object等对象组合成程序员需要的字符串。最关键也是常用的方法是public StringBuilder append(String str)，其他许多不同类型的append方法实际是引用了字符串的append方法。该方法的具体实现在StringBuilder的基类AbstractStringBuilder中。

  将产品（String）与建造过程（StringBuider）分离

StringBuilder：指挥者+具体建造者

```java
public final class StringBuilder
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence
```

以append方法为例，最终调用的是父类的append()

```java
@Override
public StringBuilder append(String str) {
    super.append(str);
    return this;
}
```

AbstractStringBuilder：建造者(只是无法实例化的建造者)

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
}
}
```

Appendable：抽象建造者

```java
public interface Appendable {

    Appendable append(CharSequence csq) throws IOException;
   
    Appendable append(CharSequence csq, int start, int end) throws IOException;
  
    Appendable append(char c) throws IOException;
}
```

## 结构模式

结构型模式描述如何将类或对象按某种布局组成更大的结构。它分为类结构型模式和对象结构型模式，前者采用继承机制来组织接口和类，后者釆用组合或聚合来组合对象。除了适配器模式分为类结构型模式和对象结构型模式两种，其他的全部属于对象结构型模式

1. 代理（Proxy）模式：为某对象提供一种代理以控制对该对象的访问。即客户端通过代理间接地访问该对象，从而限制、增强或修改该对象的一些特性。
2. 适配器（Adapter）模式：将一个类的接口转换成客户希望的另外一个接口，使得原本由于接口不兼容而不能一起工作的那些类能一起工作。
3. 桥接（Bridge）模式：将抽象与实现分离，使它们可以独立变化。它是用组合关系代替继承关系来实现的，从而降低了抽象和实现这两个可变维度的耦合度。
4. 装饰（Decorator）模式：动态地给对象增加一些职责，即增加其额外的功能。
5. 外观（Facade）模式：为多个复杂的子系统提供一个一致的接口，使这些子系统更加容易被访问。
6. 享元（Flyweight）模式：运用共享技术来有效地支持大量细粒度对象的复用。
7. 组合（Composite）模式：将对象组合成树状层次结构，使用户对单个对象和组合对象具有一致的访问性。

