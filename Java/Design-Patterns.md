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

+ 双重检查：对象在创建的过程的一些步骤可能会被重排序，从而导致在多线程情况下，线程可能会拿到一个不为null但还未初始化完成的对象去使用；对象创建的步骤大概分为3步：

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

​		系统中大量使用该类对象，且各个调用者都需要给它的属性重新赋值；对象之间相同或相似，即只是个别的几个属性不同的时候。通过 new 产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式；创建对象成本较大，例如初始化时间长，占用CPU太多，或者占用网络资源太多等，需要优化资源

+ java.lang.Object#clone()

### 简单工厂模式

**一个抽象产品类，可以派生出多个具体产品类。一个具体工厂类，通过往此工厂的static方法中传入不同参数，产出不同的具体产品类实例。**简单工厂模式有一个具体的工厂类，可以生成多个不同的产品。一旦增加新产品不得不修改工厂逻辑，在产品类型较多时，可能造成逻辑过于复杂

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

### 代理模式

为某个对象提供一个代理对象，并且由代理对象控制对原对象的访问，即通过代理对象访问目标对象。代理模式可以在目标对象实现的基础上，增加一些额外的功能，扩展目标对象的功能。代理模式可以提供非常好的访问控制，应用比较广泛。

##### 角色

+ 抽象主题(Subject)：通过接口或抽象类声明真实主题和代理对象实现的业务方法。
+ 具体主题(Real Subject)：实现了抽象主题中的具体业务，是代理对象所代表的真实对象，是最终要引用的对象。
+ 代理(Proxy)：提供了与真实主题相同的接口，其内部含有对真实主题的引用，它可以访问、控制或扩展真实主题的功能。

##### 实现

+ 静态代理：代理对象需要与目标对象实现一样的接口,所以会有很多代理类，一旦接口增加方法,目标对象与代理对象都要维护。

  ```java
  //抽象主题
  public interface ITeacherDao {
  
      public void teach();
  }
  ```

  ```java
  //目标对象
  public class TeachterDao implements ITeacherDao{
      public void teach() {
          System.out.println("老师授课中...");
      }
  }
  ```

  ```java
  //代理对象
  package cn.personalweb.proxy.staticproxy;
  
  public class TeacherDaoProxy implements ITeacherDao{
  
      //目标对象
      private ITeacherDao target;
  
      public TeacherDaoProxy(ITeacherDao target) {
          this.target = target;
      }
  
      //访问目标对象并做拓展
      public void teach() {
          System.out.println("===开始事物===");
          target.teach();
          System.out.println("===提交事物===");
      }
  }
  ```

+ 动态代理：代理对象动态在内存中构建；JDK动态代理要求被代理类必须实现一个接口，CGLIB不需要被代理类实现接口

  ```java
  //抽象主题
  public interface Subject {
      
      public void request();
  }
  ```

  ```java
  //目标对象
  public class ConcreteSubject implements Subject {
      
      @Override
      public void request() {
          //业务处理逻辑
      }
  }
  ```

  ```java
  //动态生成代理
  public class ProxyHandler implements InvocationHandler {
      //目标对象
      private Object target;
  
      public Object newProxyInstance(Object target) {
          this.target = target;
          Object result = Proxy.newProxyInstance(target.getClass().getClassLoader(),
                  target.getClass().getInterfaces(), this);
          return result;
      }
  
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
          //TODO 原对象方法调用前添加的预处理逻辑
  
          Object ret = null;
          try {
              //调用目标方法
              ret = method.invoke(target, args);
          } catch (Exception e) {
              log.error("调用{}.{}发生异常", target.getClass().getName(), method.getName(), e);
              throw e;
          }
          //TODO 原对象方法调用后添加的后处理逻辑
          return ret;
      }
  }
  ```

##### 应用场景

当无法或不想直接引用某个对象或访问某个对象存在困难时，可以通过代理对象来间接访问。使用代理模式主要有两个目的：一是保护目标对象，二是增强目标对象。

+ 访问控制，代理可仅在客户端凭据满足条件时才将请求传递给服务对象，防止恶意客户端的攻击
+ 远程代理：

### 适配器模式

可以使接口不兼容的类能够相互合作。客户端无法直接调用某个类(可能是接口不兼容的问题)，引入一个中介，这个中介叫作适配器（即目标，供客户端直接使用)，适配器可以帮助客户端去调用那个类，从而解决客户端与那各类不兼容的问题(比如说客户端只能解析json数据，可是有一个服务端只能返回xml数据，那么就要引入一个中介，关于xml转json的适配器，客户端就可以通过这个适配器接受服务端的xml数据了)。从实现方式来说，就是适配器实现了目标类的接口， 并对适配者类进行封装，供客户端直接调用。将目标类和适配者类解耦，通过引入一个中介适配器类来重用现有的适配者类，无须修改原有结构角色

##### 角色

+ 目标接口：目标抽象类定义客户所需的接口，可以是一个抽象类或接口，也可以是具体类。
+ 适配者类：适配者即被适配的角色，它定义了一个已经存在的接口，这个接口需要适配，适配者类一般是一个具体类，包含了客户希望使用的业务方法，在某些情况下可能没有适配者类的源代码。
+ 适配器类：一个可以同时与客户端和服务交互的类， 它在实现客户端接口的同时封装了服务对象。 适配器接受客户端通过适配器接口发起的调用， 并将其转换为适用于被封装服务对象的调用。它是一个转换器，通过继承或引用适配者的对象，把适配者接口转换成目标接口，让客户按目标接口的格式访问适配者。

##### 实现

+ 类适配器模式：适配器通过继承适配者类实现目标接口

```java
//目标(Target):客户直接交互的类
public interface Target {
    void request();
}
```

```java
//适配者类(Adaptee)：客户端无法直接调用，但又需要它
public class Adaptee {
    public void adapteeRequest() {
        System.out.println("被适配者的方法");
    }
}
```

```java
//适配器类(Adapter)：作为中介，帮助客户端去调用适配者
public class Adapter extends Adaptee implements Target{
    @Override
    public void request() {
        //...一些操作...
        super.adapteeRequest();
        //...一些操作...
    }
}
```

对于Java来说，不支持多继承，那么一次最多只能适配一个适配者类，并且适配者类不能为final类，否则无法被继承，而目标抽象类也只能定义为接口了。不过，由于适配器类是适配者类的子类，因此重新定义Adaptee的部分行为（子类覆盖父类的部分实现方法），可以在适配器类中置换一些适配者的方法，使得适配器的灵活性更强。

+ 对象适配器模式(基于关联)：通过继承Target并关联一个Adaptee对象使二者产生联系。

```java
//目标(Target):客户直接交互的类
public interface Target {
    void request();
}
```

```java
//适配者类(Adaptee)：客户端无法直接调用，但又需要它
public class Adaptee {
    public void adapteeRequest() {
        System.out.println("被适配者的方法");
    }
}
```

```java
//适配器类(Adapter)：作为中介，帮助客户端去调用适配者
public class Adapter implements Target{
    // 适配者是对象适配器的一个属性
    private Adaptee adaptee = new Adaptee();

    @Override
    public void request() {
        //...
        adaptee.adapteeRequest();
        //...
    }
}
```

一个适配器可以把多种不同的源适配到同一个目标，当需要同时配源类和其子类，可以使用对象适配器

##### 应用场景

系统需要复用现有类，而该类的接口不符合系统的需求，可以使用适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作

+ spring AOP中的适配器模式：在Spring的Aop中，使用的 Advice（通知） 来增强被代理类的功能。

Advice的类型有：MethodBeforeAdvice、AfterReturningAdvice、ThrowsAdvice；在每个类型 Advice 都有对应的拦截器，MethodBeforeAdviceInterceptor、AfterReturningAdviceInterceptor、ThrowsAdviceInterceptor；Spring需要将每个 Advice 都封装成对应的拦截器类型，返回给容器，所以需要使用适配器模式对 Advice 进行转换

+ spring mvc中的适配者模式：spring MVC中的适配器模式主要用于执行目标 Controller 中的请求处理方法。在Spring MVC中，DispatcherServlet 作为用户，HandlerAdapter 作为期望接口，具体的适配器实现类用于对目标类进行适配，Controller 作为需要适配的类。为什么要在 Spring MVC 中使用适配器模式？Spring MVC 中的 Controller 种类众多，不同类型的 Controller 通过不同的方法来对请求进行处理。如果不利用适配器模式的话，DispatcherServlet 直接获取对应类型的 Controller，需要的自行来判断

### 装饰器模式

​		在不改变现有对象结构的情况下，动态地给该对象增加一些职责，增加一些额外功能。它比继承方式更加灵活，继承是静态的，不能在运行期间改变，并且随着扩展功能的增多，子类会很膨胀。装饰者通过是组合关系来对象进行包装，在保持原对象的类结构不变的前提下，为其拓展额外的功能，这种拓展是可以在运行时动态进行的，可以不断对对象的功能进行拓展，并且通过不同的装饰器以及这些装饰类的排列组合，我们可以创造出更多不同行为的组合。例如，衣服：T-shirt、衬衫、外套、雨衣......所有这些衣物都 “扩展” 了你的基本行为， 但它们并不是你的一部分， 如果你不再需要某件衣物， 可以方便地随时脱掉。装饰类Decorator，不管装饰多少层，返回的对象还是Component(因为Decorator本身就是继承自Component的)，实现的还是is-a的关系。多层的装饰是比较复杂的。

##### 角色

+ 抽象构件(Component)：具体构件和抽象装饰类的共同父类，声明了在具体构件中实现的业务方法
+ 具体构件(ConcreteComponent)：它是抽象构件类的子类，用于定义具体的构件对象，实现了在抽象构件中声明的方法，装饰器可以给它增加额外的职责（方法）
+ 抽象装饰(Decorator)：拥有一个构件Component(即被封装对象，可以是具体构件或装饰）装饰基类会将所有操作委派给被封装的对象。
+ 具体装饰(ConcreteDecorator)：定义了可动态添加到部件的额外行为。 具体装饰类会重写装饰基类的方法， 并在调用父类方法之前或之后进行额外的行为。

##### 实现

```java
//抽象构件
public interface Component {
    public void sampleOperation();
}
```

```java
//具体构件
public class ConcreteComponent implements Component {
    @Override
    public void sampleOperation() {
        // 写相关的业务代码
    }
}
```

````java
//抽象装饰
public class Decorator implements Component{
    //构件引用
    private Component component;
    
    public Decorator(Component component){
        this.component = component;
    }

    @Override
    public void sampleOperation() {
        // 直接委派给构件
        component.sampleOperation();
    }   
}
````

```java
//具体装饰
public class ConcreteDecoratorA extends Decorator {

    public ConcreteDecoratorA(Component component) {
        super(component);
    }
    
    @Override
    public void sampleOperation() {
      // 写相关的业务代码
　　　super.sampleOperation();
     // 写相关的业务代码
    }
}
```

如何对Final类进行增强：先将final类封装成一个可继承的类作为具体构件类，然后就可以这个构件类作为基类，创建他的子类抽象装饰类(持有一个具体构件的引用)，在重写具体构件方法时直接调用具体构件的方法；而装饰器的子类在重写父类的方法时，先直接调用父类的方法，然后作其他增强

```java
//最终类
public final class Material{
    
    public void add(){
        System.out.println("add");
    }
    
    public void delete(){
        System.out.println("change");
    }
    
    public void change(){
        System.out.println("add");
    }

}
```

```java
//具体构建类
public abstract class Component{
    Material material;
    
    //防止子类装饰器报错
    public Component(){};
    
    public Component(Material material){
        this.material = material;
    }
    
    
    public void add(){
        material.add();
    }
    
    public void delete(){
        material.delete();
    }
    
    public void change(){
        material.change();
    }
    
}
```

```java
//抽象装饰类
public class Decorator extends Component {
    Component component;
    public Decorator(Component component) {
        this.component=component;
    }
    
    @Override
    public void add(){
        component.add();
    }
    
    @Override
    public void delete(){
        component.delete();
    }
    
    @Override
    public void change(){
        component.change();
    }
}
```

```java
//具体装饰
public class ConcreteDecoratorA extends Decorator {

    public ConcreteDecoratorA(Component component) {
        super(component);
    }
    
    @Override
    public void add(){
        //其他业务
        super.add();
        //其他业务
    }
    
    @Override
    public void delete(){
        //其他业务
        super.delete();
        //其他业务
    }
    
    @Override
    public void change(){
        //其他业务
        super.change();
        //其他业务
    }
}
```

##### 应用场景

+ 透明性要求

  装饰模式对客户端的透明性要求程序不要声明给一个ConcreteComponent类型的变量，而应当声明一个Component类型的变量。即

  ```java
  //具体构件
  Component comA = new ConcreteComponentA();
  //具体装饰A
  Component decA = new ConcreteDecoratorA(comA);
  //具体装饰B
  Component decB = new ConcreteDecoratorA(decA);
  ```

  另一种装饰模式是半透明的，装饰器的用意是在不改变接口的前提下，增强所考虑的类的性能，在增强性能的时候，具体装饰器往往要建立新的方法。这意味着客户端可以声明ConcreteDecorator类型的变量，从而可以调用ConcreteDecorator类中才有的方法

  ```java
  //具体构件
  Component comA = new ConcreteComponentA();
  //具体装饰A
  ConcreteDecoratorA decA = new ConcreteDecoratorA(comA);
  //ConcreteDecoratorA特有方法swim()
  decA.swim()
  //具体装饰B
  ConcreteDecoratorB decB = new ConcreteDecoratorB(decA);
  //ConcreteDecoratorB特有方法fly()
  decB.fly()
  ```

+ JAVA I/O标准库：整个Java IO体系都是基于字符流(InputStream/OutputStream) 和 字节流(Reader/Writer)作为基类

  ```java
  //抽象构件
  public abstract class InputStream implements Closeable {
  
  }
  ```

  ```java
  //具体构件
  ByteArrayInputStream、FileInputStream、StringBufferInputStream
  ```

  ```java
  //抽象装饰
  public class FilterInputStream extends InputStream {
     
      protected volatile InputStream in;
      
      protected FilterInputStream(InputStream in) {
          this.in = in;
  }
  ```

  ```java
  //具体装饰(BufferedInputStream在调用其read()读取数据时会委托基础构件来进行更底层的操作，而它自己所起的装饰作用就是缓冲)
  public class BufferedInputStream extends FilterInputStream {
      
      public synchronized int read() throws IOException {
          if (pos >= count) {
              fill();
              if (pos >= count)
                  return -1;
          }
          return getBufIfOpen()[pos++] & 0xff;
      }
      
      private void fill() throws IOException {
          byte[] buffer = getBufIfOpen();
          if (markpos < 0)
              pos = 0;            /* no mark: throw away the buffer */
          else if (pos >= buffer.length)  /* no room left in buffer */
              if (markpos > 0) {  
                  int sz = pos - markpos;
                  System.arraycopy(buffer, markpos, buffer, 0, sz);
                  pos = sz;
                  markpos = 0;
              } else if (buffer.length >= marklimit) {
                  markpos = -1;   
                  pos = 0;        /* drop buffer contents */
              } else if (buffer.length >= MAX_BUFFER_SIZE) {
                  throw new OutOfMemoryError("Required array size too large");
              } else {            /* grow buffer */
                  int nsz = (pos <= MAX_BUFFER_SIZE - pos) ?
                          pos * 2 : MAX_BUFFER_SIZE;
                  if (nsz > marklimit)
                      nsz = marklimit;
                  byte nbuf[] = new byte[nsz];
                  System.arraycopy(buffer, 0, nbuf, 0, pos);
                  if (!bufUpdater.compareAndSet(this, buffer, nbuf)) {
                      throw new IOException("Stream closed");
                  }
                  buffer = nbuf;
              }
          count = pos;
          //委托基础构件来进行更底层的操作
          int n = getInIfOpen().read(buffer, pos, buffer.length - pos);
          if (n > 0)
              count = n + pos;
      }
      
      private InputStream getInIfOpen() throws IOException {
          //抽象装饰类中的构件 protected volatile InputStream in;
          InputStream input = in;
          if (input == null)
              throw new IOException("Stream closed");
          return input;
      }
      
  }
  ```

  



### 外观模式

是一种通过为多个复杂的子系统提供一个一致的接口，而使这些子系统更加容易被访问的模式。该模式对外有一个统一接口，外部应用程序不用关心内部子系统的具体细节，这样会大大降低应用程序的复杂度，提高了程序的可维护性。通过引入一个外观角色来简化客户端与子系统之间的交互，为复杂的子系统调用提供一个统一的入口，降低子系统与客户端的耦合度，且客户端调用非常方便。外观模式的目的不是给予子系统添加新的功能接口，而是为了让外部减少与子系统内多个模块的交互，松散耦合，从而让外部能够更简单地使用子系统。外观模式的本质是：封装交互，简化调用。

+ 外观(Facade)：在外观角色中可以知道相关的（一个或者多个）子系统的功能和责任；在正常情况下，它将所有从客户端发来的请求委派到相应的子系统去，传递给相应的子系统对象处理。
+ 子系统(Sub System)：每一个子系统可以不是一个单独的类，而是一个类的集合，它实现子系统的功能；每一个子系统都可以被客户端直接调用，或者被外观角色调用，它处理由外观类传过来的请求；子系统并不知道外观的存在，对于子系统而言，外观角色仅仅是另外一个客户端而已。

##### 实现

```java
//外观角色
class Facade
{
    //子系统
    private SubSystem01 obj1=new SubSystem01();
    private SubSystem02 obj2=new SubSystem02();
    private SubSystem03 obj3=new SubSystem03();
    public void method()
    {
        obj1.method1();
        obj2.method2();
        obj3.method3();
    }
}
```

```java
//子系统角色
class SubSystem01
{
    public  void method1()
    {
        System.out.println("子系统01的method1()被调用！");
    }   
}
//子系统角色
class SubSystem02
{
    public  void method2()
    {
        System.out.println("子系统02的method2()被调用！");
    }   
}
//子系统角色
class SubSystem03
{
    public  void method3()
    {
        System.out.println("子系统03的method3()被调用！");
    }   
}
```

```java
//客户端
public class FacadePattern
{
    public static void main(String[] args)
    {
        Facade f=new Facade();
        f.method();
    }
}
```

##### 应用场景

+ SLF4J：SLF4J 是简单的日志外观模式框架，抽象了各种日志框架例如 Logback、Log4j、Commons-logging 和 JDK 自带的 logging 实现接口。它使得用户可以在部署时使用自己想要的日志框架。


### 享元模式

##### 角色

##### 实现

##### 应用场景

### 桥接模式

##### 角色

##### 实现

##### 应用场景

### 组合模式

##### 角色

##### 实现

##### 应用场景

## 行为型模式

行为型模式用于描述程序在运行时复杂的流程控制，即描述多个类或对象之间怎样相互协作共同完成单个对象都无法单独完成的任务，它涉及算法与对象间职责的分配。行为型模式分为类行为模式和对象行为模式，前者采用继承机制来在类间分派行为，后者采用组合或聚合在对象间分配行为。由于组合关系或聚合关系比继承关系耦合度低，满足“合成复用原则”，所以对象行为模式比类行为模式具有更大的灵活性。除了模板方法模式和解释器模式是类行为型模式，其他的全部属于对象行为型模式

1. 模板方法（Template Method）模式：定义一个操作中的算法骨架，将算法的一些步骤延迟到子类中，使得子类在可以不改变该算法结构的情况下重定义该算法的某些特定步骤。
2. 策略（Strategy）模式：定义了一系列算法，并将每个算法封装起来，使它们可以相互替换，且算法的改变不会影响使用算法的客户。
3. 命令（Command）模式：将一个请求封装为一个对象，使发出请求的责任和执行请求的责任分割开。
4. 职责链（Chain of Responsibility）模式：把请求从链中的一个对象传到下一个对象，直到请求被响应为止。通过这种方式去除对象之间的耦合。
5. 状态（State）模式：允许一个对象在其内部状态发生改变时改变其行为能力。
6. 观察者（Observer）模式：多个对象间存在一对多关系，当一个对象发生改变时，把这种改变通知给其他多个对象，从而影响其他对象的行为。
7. 中介者（Mediator）模式：定义一个中介对象来简化原有对象之间的交互关系，降低系统中对象间的耦合度，使原有对象之间不必相互了解。
8. 迭代器（Iterator）模式：提供一种方法来顺序访问聚合对象中的一系列数据，而不暴露聚合对象的内部表示。
9. 访问者（Visitor）模式：在不改变集合元素的前提下，为一个集合中的每个元素提供多种访问方式，即每个元素有多个访问者对象访问。
10. 备忘录（Memento）模式：在不破坏封装性的前提下，获取并保存一个对象的内部状态，以便以后恢复它。
11. 解释器（Interpreter）模式：提供如何定义语言的文法，以及对语言句子的解释方法，即解释器。

### 模板方法模式

在父类中定义了一个算法的框架， 允许子类在不修改结构的情况下重写算法的特定步骤。即算法的流程执行顺序是由父类控制的，父类会实现其中一些固定不变的步骤，而另外一些可变步骤则由子类去实现，从而让固定的算法流程而产生不同的结果。父类封装了具体流程以及实现部分不变行为，其它可变行为交由子类进行具体实现；框架流程由父类限定，子类无法更改；子类可以针对流程某些步骤进行具体实现，实现不会改变流程结构。去除子类中的重复代码，很好的代码复用

##### 角色

+ 抽象类(Abstract Class)：负责给出一个算法的轮廓和骨架。它由一个模板方法和若干个基本方法构成，定义了一系列基本操作，这些基本操作可以是具体的，也可以是抽象的，每一个基本操作对应算法的一个步骤，在其子类中可以重定义或实现这些步骤。同时，在抽象类中实现了一个模板方法(Template Method)，用于定义一个算法的框架，模板方法不仅可以调用在抽象类中实现的基本方法，也可以调用在抽象类的子类中实现的基本方法，还可以调用其他对象中的方法。
+ 具体子类(Concrete Class)：用于实现在父类中声明的抽象基本操作以完成子类特定算法的步骤，也可以覆盖在父类中已经实现的具体基本操作。

一个模板方法是定义在抽象类中的、把基本操作方法组合在一起形成一个总算法或一个总行为的方法。这个模板方法定义在抽象类中，并由子类不加以修改地完全继承下来。模板方法是一个具体方法，它给出了一个顶层逻辑框架，而逻辑的组成步骤在抽象类中可以是具体方法，也可以是抽象方法。基本方法是实现算法各个步骤的方法，是模板方法的组成部分。基本方法又可以分为三种：抽象方法(Abstract Method)、具体方法(Concrete Method)和钩子方法(Hook Method)。

+ 抽象方法：一个抽象方法由抽象类声明、由其具体子类实现。
+ 具体方法：一个具体方法由一个抽象类或具体类声明并实现，其子类可以进行覆盖也可以直接继承。
+ 钩子方法：可以与一些具体步骤 “挂钩” ，以实现在不同条件下执行模板方法中的不同步骤

##### 实现

```java
//抽象类
public abstract class ACourse {

    protected final void makeCourse() {
        this.makePPT();
        this.makeVideo();
        if (needWriteArticle()) {
            this.writeArticle();
        }
        this.packageCourse();
    }

    final void makePPT() {
        System.out.println("1. 制作PPT");
    }

    final void makeVideo() {
        System.out.println("2. 制作视频");
    }

    final void writeArticle() {
        System.out.println("3. 编写课程笔记");
    }

    //钩子方法
    protected boolean needWriteArticle() {
        return false;
    }

    abstract void packageCourse();
}
```

```java
//子类 JavaCourse，实现了抽象方法 packageCourse，重写了钩子方法 needWriteArticle
public class JavaCourse extends ACourse {
    @Override
    void packageCourse() {
        System.out.println("4. 提供Java课程源代码");
    }

    @Override
    protected boolean needWriteArticle() {
        return true;
    }
}
```

```java
//子类 FECourse，实现了抽象方法 packageCourse，重写了钩子方法 needWriteArticle，其中把钩子方法的结果交给客户端确定
public class FECourse extends ACourse {
    private boolean needWriteArticleFlag = false;
    @Override
    void packageCourse() {
        System.out.println("4.1 提供课程的前端代码");
        System.out.println("4.2 提供课程的图片等多媒体素材");
    }

    public FECourse(boolean needWriteArticleFlag) {
        this.needWriteArticleFlag = needWriteArticleFlag;
    }

    @Override
    protected boolean needWriteArticle() {
        return this.needWriteArticleFlag;
    }
}
```

##### 应用场景

+ Mybatis BaseExecutor接口中的模板方法模式：Executor 是 Mybatis 的核心接口之一，其中定义了数据库操作的基本方法

  ```java
  public interface Executor {
    ResultHandler NO_RESULT_HANDLER = null;
    // 执行 update、insert、delete 三种类型的SQL语句
    int update(MappedStatement ms, Object parameter) throws SQLException;
    // 执行selete类型的SQL语句，返回值分为结果对象列表或游标对象
    <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey cacheKey, BoundSql boundSql) throws SQLException;
    /
    <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException;
  
    <E> Cursor<E> queryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds) throws SQLException; 
    // 批量执行SQL语句
    List<BatchResult> flushStatements() throws SQLException;
    // 提交事务
    void commit(boolean required) throws SQLException;
    // 回滚事务
    void rollback(boolean required) throws SQLException;
    // 创建缓存中用到的CacheKey对象
    CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql);
    // 根据CacheKey对象查找缓存
    boolean isCached(MappedStatement ms, CacheKey key);
    // 清空一级缓存
    void clearLocalCache();
    // 延迟加载一级缓存中的数据
    void deferLoad(MappedStatement ms, MetaObject resultObject, String property, CacheKey key, Class<?> targetType);
    // 获取事务对象
    Transaction getTransaction();
    // 关闭Executor对象
    void close(boolean forceRollback);
    // 检测Executor是否已关闭
    boolean isClosed();
  
    void setExecutorWrapper(Executor executor);
  
  }
  
  ```

  BaseExecutor 中主要提供了缓存管理和事务管理的基本功能，继承 BaseExecutor 的子类只需要实现四个基本方法来完成数据库的相关操作即可，这四个方法分别是：doUpdate() 方法、doQuery() 方法、doQueryCursor() 方法、doFlushStatement() 方法，其余功能都在 BaseExecutor 中实现。BaseExecutor的部分代码如下，其中的 query() 方法首先会创建 CacheKey 对象，并根据 CacheKey 对象查找一级缓存，如果缓存命中则返回缓存中记录的结果对象，如果未命中则查询数据库得到结果集，之后将结果集映射成结果对象并保存到一级缓存中，同时返回结果对象。BaseExecutor 的子类有四个分别是 SimpleExecotor、ReuseExecutor、BatchExecutor、ClosedExecutor，由于这里使用了模板方法模式，一级缓存等固定不变的操作都封装到了 BaseExecutor 中，因此子类就不必再关心一级缓存等操作，只需要专注实现4个基本方法的实现即可。

  ```java
  public abstract class BaseExecutor implements Executor {
    protected Transaction transaction;
    protected Executor wrapper;
  
    protected ConcurrentLinkedQueue<DeferredLoad> deferredLoads;
    protected PerpetualCache localCache;
    protected PerpetualCache localOutputParameterCache;
    protected Configuration configuration;
  
    protected int queryStack = 0;
    private boolean closed;
    
    @Override
    public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
      ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
      if (closed) {
        throw new ExecutorException("Executor was closed.");
      }
      if (queryStack == 0 && ms.isFlushCacheRequired()) {
        clearLocalCache();
      }
      List<E> list;
      try {
        queryStack++;
        list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
        if (list != null) {
          handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
        } else {
          list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
        }
      } finally {
        queryStack--;
      }
      if (queryStack == 0) {
        for (DeferredLoad deferredLoad : deferredLoads) {
          deferredLoad.load();
        }
        // issue #601
        deferredLoads.clear();
        if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
          // issue #482
          clearLocalCache();
        }
      }
      return list;
    }
    
    protected abstract int doUpdate(MappedStatement ms, Object parameter)
        throws SQLException;
  
    protected abstract List<BatchResult> doFlushStatements(boolean isRollback)
        throws SQLException;
  
    protected abstract <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql)
        throws SQLException;
  
    protected abstract <E> Cursor<E> doQueryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds, BoundSql boundSql)
        throws SQLException;
  
    // 省略....
  }
  
  ```

+ AQS
+ Spring容器初始化： IOC 容器初始化时中的模板方法，不管是 XML 还是注解的方式，对于核心容器启动流程都是一致的。AbstractApplicationContext 的 refresh 方法实现了 IOC 容器启动的主要逻辑。一个 refresh() 方法包含了好多其他步骤方法，像不像我们说的 模板方法，getBeanFactory() 、refreshBeanFactory() 是子类必须实现的抽象方法，postProcessBeanFactory() 是钩子方法。ClassPathXmlApplicationContext 和 AnnotationConfigApplicationContext 启动入口

### 策略模式

定义一系列算法， 并将每个算法独立封装起来，它们之间是可以相互替换的。当实现某一个功能存在多种算法或者策略，我们可以根据环境或者条件的不同选择不同的算法或者策略来完成该功能。客户端必须知道所有的策略类，并自行决定使用哪一个策略类。这就意味着客户端必须理解这些算法的区别，以便适时选择恰当的算法。换言之，策略模式只适用于客户端知道所有的算法或行为的情况。客户端每次只能使用一个策略类，不支持使用一个策略类完成部分功能后再使用另一个策略类来完成剩余功能的情况。

##### 角色

+ 抽象策略类(Strategy)：定义了一个公共接口，各种不同的算法以不同的方式实现这个接口，环境角色使用这个接口调用不同的算法，一般使用接口或抽象类实现。
+ 具体策略类(ConcreteStrategy)：实现了抽象策略定义的接口，不同的策略提供不同的算法实现。
+ 环境类(Context)：持有一个策略类的引用，最终给客户端调用

##### 实现

```java
//抽象策略类
interface Strategy
{   
    public void strategyMethod();    //策略方法
}
```

```java
//具体策略类A
class ConcreteStrategyA implements Strategy
{
    public void strategyMethod()
    {
        System.out.println("具体策略A的策略方法被访问！");
    }
}

//具体策略类B
class ConcreteStrategyB implements Strategy
{
  public void strategyMethod()
  {
      System.out.println("具体策略B的策略方法被访问！");
  }
}
```

```java
//环境类
class Context{
    private Strategy strategy;
    
    public Strategy getStrategy(){
        return strategy;
    }
    
    public void setStrategy(Strategy strategy){
        this.strategy=strategy;
    }
    
    //调用具体策略
    public void strategyMethod(){
        strategy.strategyMethod();
    }
}
```

```java
//客户端
public static void main(String[] args){
        Context c=new Context();
    	//选择A策略
        Strategy s=new ConcreteStrategyA();
        c.setStrategy(s);
        c.strategyMethod();
        System.out.println("-----------------");
    	//选择B策略
        s=new ConcreteStrategyB();
        c.setStrategy(s);
        c.strategyMethod();
    }
```

##### 应用场景

+ Java Comparator：java.util.Comparator 接口是比较器接口，可以通过 Collections. sort(List,Comparator) 和 Arrays.sort(Object[],Comparator) 对集合和数据进行排序，向 Collections.sort() 和 Arrays.sort() 分别传入不同的比较器即可实现不同的排序效果（升序或降序）这里 Comparator 接口充当了抽象策略角色，两个比较器 DescSortor()和 AscSortor 则充当了具体策略角色，Collections 和 Arrays 则是环境角色
+ Spring Bean 实例化中的策略模式：Spring实例化Bean有三种方式：构造器实例化、静态工厂实例化、实例工厂实例化。具体实例化Bean的过程中，Spring中角色分工很明确，创建对象的时候先通过 ConstructorResolver 找到对应的实例化方法和参数，再通过实例化策略 InstantiationStrategy 进行实例化，根据创建对象的三个分支( 工厂方法、有参构造方法、无参构造方法 ), InstantiationStrategy 提供了三个接口方法，InstantiationStrategy 为实例化策略接口，扮演抽象策略角色，有两种具体策略类，分别为 SimpleInstantiationStrategy 和 CglibSubclassingInstantiationStrategy

### 命令模式

将一个请求封装为一个对象，使发出请求的责任和执行请求的责任分割开。命令模式能将调用操作的对象与实现该操作的对象解耦。命令模式是将行为请求者和行为实现者解耦合的方式。对命令进行封装，将命令和执行命令分隔开。请求的一方发出命令，要求执行某些操作，接受一方收到命令，执行这些操作的真正实现。客户端通过调用者执行命令，命令会传到接受者那里去执行。

##### 角色

+ 抽象命令类(Command)：声明执行命令的接口，拥有执行命令的抽象方法 execute()。
+ 具体命令类(Concrete Command)：是抽象命令类的具体实现类，它拥有接收者对象，并通过调用接收者的功能来完成命令要执行的操作。
+ 实现者/接受者(Receiver)：执行命令功能的相关操作，是具体命令对象业务的真正实现者。
+ 调用者/请求者(Invoker)：是请求的发送者，它通常拥有很多的命令对象，并通过访问命令对象来执行相关请求，它不直接访问接收者。

##### 实现

```java
//抽象命令
interface Command
{
    public abstract void execute();
}
```

```java
//具体命令
class ConcreteCommand implements Command
{
    private Receiver receiver;
    ConcreteCommand()
    {
        receiver=new Receiver();
    }
    public void execute()
    {
        receiver.action();
    }
}
```

```java
//接收者
class Receiver
{
    public void action()
    {
        System.out.println("接收者的action()方法被调用...");
    }
}
```

```java
//调用者
class Invoker
{
    private Command command;
    public Invoker(Command command)
    {
        this.command=command;
    }
    public void setCommand(Command command)
    {
        this.command=command;
    }
    public void call()
    {
        System.out.println("调用者执行命令command...");
        command.execute();
    }
}
```

##### 应用场景

+ JDK Runnable：Runable是一个典型命令模式，Runnable担当命令的角色，Thread充当的是调用者，start方法就是其执行方法，而接受者可以由用户自定义

### 观察者模式

定义对象之间的一种一对多依赖关系，使得每当某一个对象的状态发生改变时，会通知所关联依赖的所有对象，让他们得到更新。观察者模式包含两类对象：观察目标和观察者，一个目标可以有任意数目的与之相依赖的观察者，一旦观察目标的状态发生改变，它会将自身的状态通知给所有的观察者。观察者模式多用于实现订阅功能的场景。观察者模式可以实现表示层和数据逻辑层的分离，定义了稳定的消息更新传递机制，并抽象了更新接口，使得可以有各种各样不同的表示层充当具体观察者角色。观察者模式在观察目标和观察者之间建立一个抽象的耦合。观察目标只需要维持一个抽象观察者的集合，无须了解其具体观察者。如果一个观察目标对象有很多直接和间接观察者，将所有的观察者都通知到会花费很多时间。

##### 角色

+ 抽象目标(Subject)：把所有观察者对象的统一聚集管理，一个目标都可以有一个或多个观察者，目标提供一系列对观察者对象增删以及通知等操作。目标类可以是接口，也可以是抽象类或具体类。
+ 具体目标(ConcreateSubject)：主要实现目标中的通知方法，当内部状态发生改变时，通知所有注册过的观察者对象。
+ 抽象观察者(Observer)：定义观察者在目标状态发生改变时做出的反应。即需要接受目标通知。观察者通常被定义为接口，该接口声明了更新数据的方法update()。
+ 具体观察者(ConcreateObserver)：实现抽象观察者中定义的抽象方法，以便在得到目标的更改通知时更新自身的状态；可以执行一些操作来回应发布者的通知。 如果需要，具体观察者角色可以保持一个指向具体主题对象的引用。

##### 实现

目标定义一系列对观察者进行增加、删除、通知的操作，每当目标自身状态发生变换时，主动通知每一位观察者

```java
// 抽象目标
abstract class Subject {
    // 保存注册的观察者对象
    private List<Observer> list = new ArrayList<>();
    //注册观察者对象
    public void attach (Observer observer){
        list.add(observer);
      System.out.println("注册一个观察者:"+observer.getClass().getName());
    }
    //删除观察者对象
     
    public void delete (Observer observer){
        list.remove(observer);
        System.out.println("删除一个观察者:"+observer);
    }
    //通知所有注册的观察者对象
    public void notifyObserver (String newState){
        for (Observer observer : list) {
            observer.update(newState);
        }
    }
}
```

```java
// 具体目标
class ConcreteSubject extends Subject{
    private String state ;
    public String getState (){
        return state ;
    }
    
    //状态发生改变时通知所有观察者
    public void change (String newState){
        state = newState;
        System.out.println("主题状态:"+state);
        //状态发生改变，通知各个观察者
        this.notifyObserver(state);
    }
}
```

```java
// 抽象观察者
interface Observer {
    //接受通知并做出相应更新
    void update (String state);
}
```

```java
// 具体观察者
class ConcreteObserver implements Observer{
    private String name ;
    // 观察者状态
    private String observerState ;
    public ConcreteObserver (String name){
        this.name = name ;
    }
    //对目标状态更新做相应的操作
    @Override
    public void update(String state) {
        //这里是观察者对象与目标对象保持一致
        observerState = state ;
        System.out.println("【"+this.name+"】状态:"+observerState);
    }
}
```

##### 应用场景

需要在系统中创建一个触发链，A对象的行为将影响B对象，B对象的行为将影响C对象……，可以使用观察者模式创建一种链式触发机制。一个对象的改变将导致一个或多个其他对象也发生改变，而并不知道具体有多少对象将发生改变，也不知道这些对象是谁。

+ JAVA语言的java.utill类库里面，提供了一个Observable类以及一个Observer接口，构成JAVA语言对观察者模式的支持.

  ```java
  //观察者
  public interface Observer {
      void update(Observable o, Object arg);
  }
  ```

  ```java
  //目标类
  public class Observable {
      private boolean changed = false;
      private Vector<Observer> obs = new Vector();
  
      public Observable() {
      }
      // 用于注册新的观察者对象到向量中
      public synchronized void addObserver(Observer var1) {
          if (var1 == null) {
              throw new NullPointerException();
          } else {
              if (!this.obs.contains(var1)) {
                  this.obs.addElement(var1);
              }
  
          }
      }
      // 用于删除向量中的某一个观察者对象
      public synchronized void deleteObserver(Observer var1) {
          this.obs.removeElement(var1);
      }
  
      public void notifyObservers() {
          this.notifyObservers((Object)null);
      }
      // 通知方法，用于在方法内部循环调用向量中每一个观察者的update()方法
      public void notifyObservers(Object var1) {
          Object[] var2;
          synchronized(this) {
              if (!this.changed) {
                  return;
              }
  
              var2 = this.obs.toArray();
              this.clearChanged();
          }
  
          for(int var3 = var2.length - 1; var3 >= 0; --var3) {
              ((Observer)var2[var3]).update(this, var1);
          }
  
      }
      // 用于清空向量，即删除向量中所有观察者对象
      public synchronized void deleteObservers() {
          this.obs.removeAllElements();
      }
      // 该方法被调用后会设置一个boolean类型的内部标记变量changed的值为true，表示观察目标对象的状态发生了变化
      protected synchronized void setChanged() {
          this.changed = true;
      }
      // 用于将changed变量的值设为false，表示对象状态不再发生改变或者已经通知了所有的观察者对象，调用了它们的update()方法
      protected synchronized void clearChanged() {
          this.changed = false;
      }
      // 返回对象状态是否改变
      public synchronized boolean hasChanged() {
          return this.changed;
      }
      // 返回向量中观察者的数量
      public synchronized int countObservers() {
          return this.obs.size();
      }
  }
  ```

### 状态模式

对有状态的对象，把复杂的“判断逻辑”提取到不同的状态对象中，允许状态对象在其内部状态发生改变时改变其行为。将与特定状态相关的行为局部化到一个状态中，并且将不同状态的行为分割开来

##### 角色

+ 抽象状态(State)：定义一个接口，用以封装环境对象中的特定状态所对应的行为。
+ 具体状态(Concrete State)：实现抽象状态所对应的行为。
+ 环境(Context)：定义了客户感兴趣的接口，维护一个当前状态，并将与状态相关的操作委托给当前状态对象来处理。

##### 实现

```java
//抽象状态类
public interface IWaterState {

    void printState();
}
```

```java
//冰水
public class IceWaterState implements IWaterState {

    @Override
    public void printState() {
        System.out.println("Now state: Ice Water");
    }
}
```

```java
//温水
public class WarmWaterState implements IWaterState {
    @Override
    public void printState() {
        System.out.println("Now state: Warm Water");
    }
}
```

```java
//沸水
public class BoilingWaterState implements IWaterState {
    @Override
    public void printState() {
        System.out.println("Now state: Boiling Water");
    }
}
```

```java
//环境类
public class WaterContext {
    private IWaterState mIWaterState;

    public IWaterState getIWaterState() {
        return mIWaterState;
    }

    public void setIWaterState(int i) {
        if (i == 0) {
            mIWaterState = new IceWaterState();
            return;
        }

        if (i == 1) {
            mIWaterState = new WarmWaterState();
            return;
        }

        if (i == 2) {
            mIWaterState = new BoilingWaterState();
            return;
        }
    }
}
```

##### 应用场景

### 迭代器模式

提供一种方法来访问聚合对象，而不用暴露这个对象的内部表示在不暴露集合底层表现形式 （列表、 栈和树等） 的情况下遍历集合中所有的元素。

##### 角色

+ 抽象迭代器(Iterator)：声明了遍历集合所需的操作： 获取下一个元素、 获取当前位置和重新开始迭代等。
+ 具体迭代器(ConcreteIterator)：完成对聚合对象的遍历，同时在具体迭代器中通过游标来记录在聚合对象中所处的当前位置，
+ 抽象聚合类(Aggregate)：存储和管理元素对象，声明一个createIterator()方法用于创建一个迭代器对象，充当抽象迭代器工厂角色。
+ 具体聚合类(ConcreteAggregate)：实现了在抽象聚合类中声明的createIterator()方法，该方法返回一个与该具体聚合类对应的具体迭代器ConcreteIterator实例。

##### 实现

```java
//抽象聚合类
public interface StudentAggregate {
    void addStudent(Student student);
    void removeStudent(Student student);
    StudentIterator getStudentIterator();
}
```

```java
//具体聚合类
public class StudentAggregateImpl implements StudentAggregate {
    private List<Student> list;  // 学生列表

    public StudentAggregateImpl() {
        this.list = new ArrayList<Student>();
    }

    @Override
    public void addStudent(Student student) {
        this.list.add(student);
    }

    @Override
    public void removeStudent(Student student) {
        this.list.remove(student);
    }

    @Override
    public StudentIterator getStudentIterator() {
        return new StudentIteratorImpl(list);
    }
}
```

```java
//抽象迭代器
public interface StudentIterator {
    boolean hashNext();
    Student next();
}
```

```java
//具体迭代器
public class StudentIteratorImpl implements StudentIterator{
    private List<Student> list;
    private int position = 0;
    private Student currentStudent;

    public StudentIteratorImpl(List<Student> list) {
        this.list = list;
    }

    @Override
    public boolean hashNext() {
        return position < list.size();
    }

    @Override
    public Student next() {
        currentStudent = list.get(position);
        position ++;
        return currentStudent;
    }
}
```

##### 应用场景

访问一个聚合对象的内容而无须暴露它的内部表示。将聚合对象的访问与内部数据的存储分离，使得访问聚合对象时无须了解其内部实现细节。

+ Java集合中的迭代器模式：java.util.ArrayList

  ```java
  public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
      transient Object[] elementData;
      private int size;
      
      
      public Iterator<E> iterator() {
          return new Itr();
      }
  ```

  ```java
  //具体迭代器
  private class Itr implements Iterator<E> {
          int cursor;       // index of next element to return
          int lastRet = -1; // index of last element returned; -1 if no such
          int expectedModCount = modCount;
  
          Itr() {}
  
          public boolean hasNext() {
              return cursor != size;
          }
  
          @SuppressWarnings("unchecked")
          public E next() {
              checkForComodification();
              int i = cursor;
              if (i >= size)
                  throw new NoSuchElementException();
              Object[] elementData = ArrayList.this.elementData;
              if (i >= elementData.length)
                  throw new ConcurrentModificationException();
              cursor = i + 1;
              return (E) elementData[lastRet = i];
          }
      ......
      }
  ```

  ```java
  //抽象迭代器
  public interface Iterator<E> {
      
      boolean hasNext();
  
      E next();
  
      default void remove() {
          throw new UnsupportedOperationException("remove");
      }
  
      default void forEachRemaining(Consumer<? super E> action) {
          Objects.requireNonNull(action);
          while (hasNext())
              action.accept(next());
      }
  }
  ```

### 职责链模式

##### 角色

##### 实现

##### 应用场景

### 中介者模式

##### 角色

##### 实现

##### 应用场景

### 访问者模式

##### 角色

##### 实现

##### 应用场景

### 备忘录模式

##### 角色

##### 实现

##### 应用场景

### 解释器模式

##### 角色

##### 实现

##### 应用场景

1. 装饰模式、代理模式、适配器模式区别：代理模式只是原来对象的一个替身，装饰模式是对原对象的功能增强，适配器模式是要改变原对象的接口；适配的目的是匹配两个不同的接口，装饰的目的是增强接口的实现。代理的目的是有代理对象控制原对象，不改变接口。装饰器模式关注于在一个对象上动态的添加方法，然而代理模式关注于控制对对象的访问。对于代理模式客户端最终访问的是同一个接口，而对于适配器模式，客户端最终访问的是不同的接口。

   + 装饰模式：原有功能不能满足现有需求，通过扩展接口来增强功能。特点是被装饰类和所有的装饰类必须实现同一个接口，而且必须持有被装饰的对象，可以无限装饰。装饰模式是在一个原有类A的基础之上增加了某些新的功能变成另一个类B（A和B是属于同一类的，满足is-a关系，注重的是覆盖和扩展）

   + 代理模式：通过代理对象来控制访问另一个对象（被代理对象）的方法，不对该方法进行直接操作。主要是隔离，隔离调用类和被调用类的关系，通过一个中介代理类去调用。代理模式是将一个类A转换成具体的操作类B（A负责对B实现访问）
   + 适配器模式：解决接口不匹配的问题，适配器模式是将一个类A通过某种方式转换成另一个类B（A和B没有必然的联系）适配器模式中适配器则扩展了待适配类新的功能。

代理模式和适配器模式最大的区别，代理模式是与原对象实现同一个接口，而适配器类则是匹配新接口即实现一个新的接口。

代理模式注重的是隔离限制，让外部不能访问你实际的调用对象，比如权限控制，装饰模式注重的是功能的拓展，在同一个方法下实现更多的功能。

适配器模式与装饰模式的区别时，适配器改变接口，适配器和被适配者没有必然的联系。装饰器则不改变接口，装饰者与被装饰者有相同的接口，在不改变被装饰者的代码的同时又能给被装饰者添加新的行为，扩展之后依旧保留OOP关系，满足is-a关系，注重的是覆盖和扩展







如果要处理的对象太多，可以考虑使用抽象解决，面向一个抽象而非众多具体，可以解除对具体的耦合，



优先使用组合而不是继承：继承的优点是子类可以通过继承父类，那么可以调用父类中定义的方法和属性，从而达到代码重用的目的。重用父类的代码以外，还可以扩展自身的属性和方法，来描述子类特有的特征和行为。但继承的缺点是父类的属性和方法，子类是无条件继承的；并且从父类继承而来的实现是静态的，不能在运行时发生改变，不够灵活。使用聚合/组合可以解决继承的缺陷，继承是一种is-a的关系，而组合是一种has-a的关系，把一些特征和行为抽取出来，形成工具类。然后通过聚合/组合成为当前类的属性。再调用其中的属性和行为达到代码重用的目的。用”has-a”(有什么或用什么)去替代”is-a”(是什么)。我们可以选择一个类中是否应该具有某种行为，从而决定应该聚合那些类，不应该聚合那些类。这样，通过聚合/组合关系，也可以避免继承所带的方法污染问题。所以，使用聚合/组合，具有很强的代码重用性和灵活性。聚合/组合复用也可以在运行时动态进行。新对象可以使用聚合/组合关系，将新的责任委派到合适的对象。

