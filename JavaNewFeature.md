

# Java版本新特性

[TOC]

## 一、Java9

### 1.模块化

```java
module java.module01 {
  //依赖于java.module02模块
  requires java.module02;
  //将module内部的某些“包”暴露
  exports java.module01.entity;
}
```

### 2.JShell

提供一个交互式的编程工具，在该工具中，开发人员可以在一个命令界面即时编写、编辑和执行java代码片段，而无需创建java源文件。

**特点**：

- 即时反馈。
- 代码片段。
- 自动导入。`JShell`自动导入常用Java包和类。
- 脚本化。可以保存多个命令在脚本文件中，通过`JShell`命令执行。

**JShell的使用**：

1. 启动`JShell`

```shell
#input commad in console directly.
jshell
| 欢迎使用 JShell -- 版本 21.0.3
| 要大致了解该版本, 请键入: /help intro
```

2. 变量声明

```shell
jshell> int a = 1;
a ==> 1
```

3. 创建方法

```shell
jshell> int add(int x , int y) {
   ...>     return x + y;
   ...> }
|  已创建 方法 add(int,int)
```

4. 调用方法

```shell
jshell> add(a,b)
$4 ==> 3
```

5. 运行表达式

```shell
jshell> 1 + 2
$5 ==> 3
```

6. 创建Java类

```shell
jshell> public class SKTest{
   ...>     public void test() {
   ...>         System.out.println("sk java");
   ...>     }
   ...> }
|  已创建 类 SKTest

jshell> SKTest sktest = new SKTest();
sktest ==> SKTest@70177ecd

jshell> sktest.
# 此处按tab键可以自动提示所有方法
clone()       equals(       finalize()    getClass()    hashCode()    
notify()      notifyAll()   test()        toString()    wait(         
jshell> sktest.test()
sk java
```



7. 定义接口，并实现

```shell
jshell> public interface UserService{
   ...>     void test();
   ...> }
|  已创建 接口 UserService

jshell> public class UserServiceImpl implements UserService {
   ...>     public void test() {
   ...>         System.out.println("interface java");
   ...>     }
   ...> }
|  已创建 类 UserServiceImpl

jshell> UserService us = new U
UserServiceImpl()   

<再次按 Tab 可查看所有可能的输入提示>
jshell> UserService us = new UserServiceImpl();
us ==> UserServiceImpl@6f539caf

jshell> us.t
test()       toString()   
jshell> us.test();
interface java
```



8. 退出`JShell`模式

```shell
jshell> /exit
|  再见
```

**常用命令**:

| 命令       | 短命令 | 描述                                                     |
| ---------- | ------ | -------------------------------------------------------- |
| /help      | /?     | 查看命令帮助信息或者获取特定命令的详细信息               |
| /list      | /l     | 列出当前已经输入的代码片段                               |
| /edit      | /e     | 编辑以前输入的代码片段                                   |
| /drop      | /d     | 删除以前输入的代码片段                                   |
| /save      | /s     | 将当前会话中的代码保存到文件中                           |
| /open      | /o     | 从文件中加载代码以继续会话                               |
| /reset     | -      | 重置JShell环境，清楚所有已输入的代码片段。               |
| /vars      | /v     | 查看当前定义的所有变量。                                 |
| /methods   | /m     | 查看当前定义的所有方法。                                 |
| /types     | /t     | 当前定义色所有类型。                                     |
| /imports   | /i     | 查看当前导入的包和类                                     |
| /set       | /s     | 设置`JShell`的个人中选项，如类路径、编辑器、输出格式等。 |
| /classpath | /cp    | 查看或设置类路径，一边加载外部类和库                     |
| /history   | /h     | 查看和搜获`JShell`历史记录                               |
| /save      | /s     | 将`JShell`历史记录保存到文件中，以便将其用于以后的会话。 |

### 3.接口私有方法

接口支持私有方法的好处：

- 接口更好演化。
- 代码复用。
- 防止子类滥用。

接口私有方法的若干限制：

- 私有方法不能是抽象的。
- 私有方法只在接口内部使用，无法被接口的实现类或者外部类访问。
- 私有方法不会继承给接口的子接口，每个接口都必须自己定义自己的私有方法。
- 私有静态方法可以再其他静态和非静态方法中使用。
-  私有非静态方法不能再私有静态方法内部使用

代码示例：

a. 接口

```java
public interface MyInterface {
    
   /**
    * 抽象方法
    */
    void publicMethod();
    
    /**
     * 默认方法
     */
    default void defaultMethod() {
        System.out.println("defaultMethod");
        //私有非静态
        privateMethod();
        //私有静态
        privateStaticMethod();
    }
    
    /**
     * 私有方法
     */
    private void privateMethod() {
         System.out.println("privateMethod");
    }
    
    /**
     * 静态方法
     */
    static void staticMethod() {
        System.out.println("staticMethod");
        //私有静态方法
        privateStaticMethod();
    }
    
     /**
     * 私有静态方法
     */
    private static void privateStaticMethod() {
        System.out.println("privateStaticMethod");
    }
}
```

b. 实现类

```java
public class MyInterfaceImpl implements MyInterface {
    @Override
    public void publicMethod() {
        System.out.println("publicMethod");
    }
}
```

c. 测试类

```java
public class Test {
    public static void main(String[] args) {
        // 调用实例方法
        MyInterface myInterface = new MyInterfaceImpl();
        myInterface.publicMethod();
        
        // 调用默认方法
        System.out.println("");
        myInterface.defaultMethod();

        // 调用静态方法
        System.out.println("");
        MyInterface.staticMethod();
    }
}

// 结果......
publicMethod

defaultMethod
privateMethod
privateStaticMethod

staticMethod
privateStaticMethod
```

### 4.String底层存储结构修改

**Java 8之前**:

```java
public final class String
     implements java.io.Serializable, Comparable<String>, CharSequence {

  //The value is used for character storage.
  private final char value[];

}
```

**优化原因**：

每个 `char` 都以 2 个字节存储在内存中。然而 Oracle 的 JDK 开发人员调研了成千上万个应用程序的 `heap dump` 信息，他们注意到大多数字符串都是以 `Latin-1` 字符编码表示的，它只需要一个字节存储就够了，两个字节完全是浪费，这比 `char` 数据类型存储少 50%（1 个字节）。

**Java 9优化**:

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence,
               Constable, ConstantDesc {

    @Stable
    private final byte[] value;

    private final byte coder;
}
```

**优势**:

- 节省内存：对于包含大量`ASCII`字符的字符串，内存占用大幅减少，因为每个字符只占用一个字节而不是两个字节。
- 提高性能。由于字符串的存储结构与编码方式更加紧凑，字符串操作的性能也有所提高。

**注意事项**：

**a.编码格式**：

`isLatin1()`方法用于判断编码格式是否为`Lation-1`字符编码，其使用可以查看`charAt()`方法源码：

```java
public char charAt(int index) {
    if (isLatin1()) {
        return StringLatin1.charAt(value, index);
    } else {
        return StringUTF16.charAt(value, index); 
    }
}
```

`isLatin1()` 用于判断编码格式是否为 `Latin-1` 字符编码，如果是则调用 `StringLatin1.charAt()`，否则调用 `StringUTF16.charAt()`。

这里为什么要判断字符编码呢？`Latin-1` 字符编码也称 `ISO 8859-1`，它包括了拉丁字母（包括西欧、北欧和南欧语言的字母）以及一些常见的符号和特殊字符，但是它并不支持其他非拉丁字母的语言，例如希腊语、俄语或中文，对于这些我们只能使用其他字符编码了。

在 Java 9 中，`String` 支持的字符编码格式有两种：

1. `Latin-1`：`Latin-1` 编码用于存储只包含拉丁字符的字符串。它采用了一字节编码，每个字符占用一个字节（8位）。
2. `UTF-16`：`UTF-16` 编码用于存储包含非拉丁字符的字符串，以及当字符串包含不适合 `Latin-1` 编码的字符时。

在 Java 9 中，`String` 多了一个成员变量 `coder`，它代表编码的格式，0 表示 `Latin-1` ，1 表示 `UTF-16`。JVM 会根据字符串的内容来选择。

也可以强制手动指定使用`UTF-16`

```shell
#强制使用UTF-16
-XX:-CompactStrings
```

### 5.Optional的增强

增加三个方法

- `or()`

```java
public Optional<T> or(Supplier<? extends Optional<? extends T>> supplier);
```

当Optional为空时，调用Supplier获取另一个Optional对象，示例如下：

```java
    @Test
    public void orTest() {
        Optional<String> optional1 = Optional.of("死磕 Java 新特性");
        Optional<String> optional2 = Optional.empty();
        System.out.println("optional1 value:" + optional1.or(()->Optional.of("死磕 Java")).get());
        System.out.println("optional2 value:" + optional2.or(()->Optional.of("死磕 Java")).get());
    }
// 结果......
optional1 value:死磕 Java 新特性
optional2 value:死磕 Java
```

- `ifPresentOrElse()`

`ifPresent(Consumer<? super T> consumer)`意为Optional非空时，将值传递给consumer执行。

`isPresentOrElse()`的方法签名如下:

```java
public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction);
```

意为非空时，将值传递给action执行；为空时执行runnable方法。示例如下：

```java
    @Test
    public void ifPresentOrElseTest() {
        Optional<String> optional = Optional.empty();
        optional.ifPresentOrElse(value -> System.out.println("optional value:" + optional.get()),
                () -> System.out.println("optional value is null"));
    }
// 结果......
optional value is null
```

