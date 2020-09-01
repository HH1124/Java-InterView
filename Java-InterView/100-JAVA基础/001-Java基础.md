# 基本数据类型
```
byte/8
char/16
short/16
int/32
float/32
long/64
double/64
```
![image](FB2572DFC566471499BE4C9BB974D57A)

# 修饰符
![image](10F795DF5A0643EDBD8D5622D9E995DC)

# 重载和重写
## 重载
- 发生在编译时
- 参数类型、参数个数 可以不同，返回值也可以不同
- 在同类中，进行方法重载

## 重写
- 发生在运行时期
- 参数列表必须相同
- 发生在子父类中

# String
- String 被声明为 final，因此它不可被继承。(Integer 等包装类也不能被继承）
- 在 Java 8 中，String 内部使用 char 数组存储数据。
- 在 Java 9 之后，String 类的实现改用 byte数组存储字符串，同时使用 coder 来标识使用了哪种编码。

## 不可变的好处
- 可以缓存 hash 值，，这样在调用的时候就不需要在计算这个hash值
- 安全性，String 不可变性可以保证参数不可变。
- 线程安全
- 字符串常量池的需要

##  String 和 StringBuffer、StringBuilder
- String是不可变的
- StringBuilder 与 StringBuffer 都继承自 AbstractStringBuilder 类，这两种对象都是可变的
- StringBuffer 是线程安全的，StringBuilder 是线程不安全的

## == 与 equals 的区别
#### ==:
它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象（基本数据类型==比较的是值，引用数据类型==比较的是内存地址）。
#### equals():
它的作用也是判断两个对象是否相等。但它一般有两种使用情况
- 情况1:类没有覆盖 equals()方法。则通过 equals()比较该类的两个对象时，等价于通过“==”比较这两个对象
- 情况2:类覆盖了 equals()方法。一般，我们都覆盖 equals()方法来比较两个对象的内容是否相等；若它们的内容相等，则返回true（即、认为这两个对象相等）。
```

    public class test1 {
        public static void main(String[] args) {
            String a = new String("ab"); // a 为⼀个引⽤
            String b = new String("ab"); // b为另⼀个引⽤,对象的内容⼀样
            String aa = "ab"; // 放在常量池中
            String bb = "ab"; // 从常量池中查找
            if (aa == bb) // true
                System.out.println("aa==bb");
            if (a == b) // false，⾮同⼀对象
                System.out.println("a==b");
            if (a.equals(b)) // true
                System.out.println("aEQb");
            if (42 == 42.0) { // true
                System.out.println("true");
            }
        }
    }
```
说明:

String中的 equals方法是被重写过的，因为 object 的 equals 方法是比较的对象的内存地址，而 String的 equals方法比较的是对象的值
当创建 String类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建个 String对象

#### hashCode
- 如果两个对象相等，那么它们的hashCode()值一定相同。
- 如果两个对象hashCode()相等，它们并不一定相等。
# Object 中的方法

```java
//native方法，用于返回当前运行时对象的Class对象，使用了final关键字修饰，故不允许子类重写。
public final native Class<?> getClass()

//native方法，用于返回对象的哈希码，主要使用在哈希表中，比如JDK中的HashMap。
public native int hashCode() 

//用于比较2个对象的内存地址是否相等，String类对该方法进行了重写用与比较字符串的值是否相等。
public boolean equals(Object obj)

//naitive方法，用于创建并返回当前对象的一份拷贝。一般情况下，对于任何对象 x，表达式 x.clone() != x为true，x.clone().getClass()==x.getClass()为true。Object本身没有实现Cloneable接口，所以不重写clone
方法并且进行调用的话会发生CloneNotSupportedException异常。
protected native Object clone() throws CloneNotSupportedException

//返回类的名字@实例的哈希码的16进制的字符串。建议Object所有的子类都重写这个方法。
public String toString()

//native方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。
public final native void notify()

//native方法，并且不能重写。跟notify一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。
public final native void notifyAll()

//native方法，并且不能重写。暂停线程的执行。注意：sleep方法没有释放锁，而wait方法释放了锁 。timeout是等待时间。
public final native void wait(long timeout) throws InterruptedException

//多了nanos参数，这个参数表示额外时间（以毫微秒为单位，范围是 0-999999）。 所以超时的时间还需要加上nanos毫秒。
public final void wait(long timeout, int nanos) throws InterruptedException

//跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念
public final void wait() throws InterruptedException

//实例被垃圾回收器回收的时候触发的操作
protected void finalize() throws Throwable { }
```

# 关键字
## final
1. 数据
    - 基本数据类型 final 使数值不变；
    - 引用数据类型 final 使引用不变
2. 方法
    - 声明方法不能被子类重写
3. 类
    - 声明类不允许被继承。

## static
1. 静态变量
- 又称为类变量，可以直接通过类名来访问它。静态变量在内存中只存在一份。
2. 静态方法
- 它不依赖于任何实例。所以静态方法必须有实现,只能访问所属类的静态字段和静态方法
3. 静态代码块
- 静态语句块在类初始化时运行一次
- ![image](7377678727214C6991D78C9CE38574CD)
4. 静态内部类
- 非静态内部类依赖于外部类的实例，也就是说需要先创建外部类实例，才能用这个实例去创建非静态内部类。而静态内部类不需要。
- 静态内部类不能访问外部类的非静态的变量和方法。
5. 初始化顺序
- ![image](0E4984D45EAD445BBCAF76713E68B8B7)
- 存在继承的情况下，初始化顺序为：
    - 父类（静态变量、静态语句块）
    - 子类（静态变量、静态语句块）
    - 父类（实例变量、普通语句块）
    - 父类（构造函数）
    - 子类（实例变量、普通语句块）
    - 子类（构造函数）

# 反射
## 怎样实现反射？ 常用的方法？
- 在程序**运行时**获取任意对象的信息
- 三种实现方式
```
1. obj.getClass();

2. Person.class();

3. Class.forName("全路径");
```
- 基本使用类
```
1. Field ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段；
2. Method ：可以使用 invoke() 方法调用与 Method 对象关联的方法；
3. Constructor ：可以用 Constructor 的 newInstance() 创建新的对象。
```
## 反射的优点
- 可扩展性，增加程序的灵活性 降低耦合性
```
例如：
    如果在配置文件中需要获取一个类，可以直接通过反射拿到这个类，后期如果有修改，可以直接在配置文件进行修改

```
## 反射的缺点
### 性能问题
- 反射属于一种解释操作，他的效率会慢很多，反射涉及了动态类型的解析，所以 JVM 无法对这些代码进行优化
### 安全限制
- 使用反射技术要求程序必须在一个没有安全限制的环境中运行
### 内部暴露
- 反射允许访问私有的属性和方法，可能导致程序出现问题
## 项目中或者java本身哪里有用到反射
- spring 中的依赖注入  根据className生成一个具体的实例，
- jdk 动态代理  通过反射机制获得动态代理类的构造函数
- 

# 序列化
无论是何种类型的数据，都会以二进制序列的形式在网络上传送。发送方需要把这个Java对象转换为字节序列，才能在网络上传送；接收方则需要把字节序列再恢复为Java对象。

Java语言内置了序列化和反序列化，通过Serializable接口实现。

## transient
用来关闭字段的序列化。


# 异常
![image](333ABE7F42D4405BBB6D98061503C078)
##  Error
一般为底层的不可恢复的类；
## Exception
定义方法时必须声明所有可能会抛出的exception；在调用这个方法时，必须捕获它的checkedexception，不然就得把它的exception传递下去

分为未检查异常(RuntimeException)和已检查异常(非RuntimeException)

## Runtime Exception
RuntimeException表示系统异常，比较严重，如果出现RuntimeException，那么一定是程序员的错误
几个经典的RunTimeException:
- NullPointerException
- ArrayIndexoutofBoundsException
- IndexOutOfBoundsException

