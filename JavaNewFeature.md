

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

#### **特点**：

- 即时反馈。
- 代码片段。
- 自动导入。`JShell`自动导入常用Java包和类。
- 脚本化。可以保存多个命令在脚本文件中，通过`JShell`命令执行。

#### **JShell的使用**：

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

#### **常用命令**:

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

#### 接口支持私有方法的好处：

- 接口更好演化。
- 代码复用。
- 防止子类滥用。

#### 接口私有方法的若干限制：

- 私有方法不能是抽象的。
- 私有方法只在接口内部使用，无法被接口的实现类或者外部类访问。
- 私有方法不会继承给接口的子接口，每个接口都必须自己定义自己的私有方法。
- 私有静态方法可以再其他静态和非静态方法中使用。
-  私有非静态方法不能再私有静态方法内部使用

#### 代码示例：

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

#### **Java 8之前**:

```java
public final class String
     implements java.io.Serializable, Comparable<String>, CharSequence {

  //The value is used for character storage.
  private final char value[];

}
```

#### **优化原因**：

每个 `char` 都以 2 个字节存储在内存中。然而 Oracle 的 JDK 开发人员调研了成千上万个应用程序的 `heap dump` 信息，他们注意到大多数字符串都是以 `Latin-1` 字符编码表示的，它只需要一个字节存储就够了，两个字节完全是浪费，这比 `char` 数据类型存储少 50%（1 个字节）。

#### **Java 9优化**:

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

#### **注意事项**：

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

- `stream`

`stream()`允许我们将 `Optional` 对象转化为一个流 （`Stream`），以便能够更灵活地处理包含或不包含值的情况。方法定义如下：

```java
Stream<T> stream()
```

当Optional不为空，stream会返回一个包含这个值的流。

当Optional为空，stream返回空流。

示例：

```java
    @Test
    public void streamTest1() {
        List<Optional<String>> list = List.of(
                Optional.of("死磕 java 并发"),
                Optional.of("死磕 java 新特性"),
                Optional.empty(),
                Optional.of("死磕 Netty"),
                Optional.empty()
        );
        list.stream()
                .flatMap(Optional::stream)
                .filter(o -> o.startsWith("死磕"))
                .map(val -> val + " —— https://skjava.com/")
                .forEach(System.out::println);
    }
// 结果......
死磕 java 并发 —— https://skjava.com/
死磕 java 新特性 —— https://skjava.com/
死磕 Netty —— https://skjava.com/
```

### 6.try-with-resource

#### **前情提要**

java7中`try-with-resource`的使用

1. 资源必须实现`AutoCloseable`或`Closeable`接口
2. 语法。`try(Resource resourceTest = new Resource())`
3. 自动关闭，即时发生了异常。
4. 多个资源。

```java
try (Resource1 r1 = new Resource1();
    Resource2 r2 = new Resource2();
    Resource3 r3 = new Resource3()) {
  //业务代码
} catch (Exception e) {
  
}
```

注意，资源的关闭顺序和它们在try中声明的顺序相反。

#### **Java9中的改进**

之前的资源，必须在try语句中定义和初始化。像下面这样的代码在8及之前版本会报错：

```java
Resource1 r = new Resource1();
try (r) {
  //try后括号中的r会报错
}
...
```

9之后的版本，只要资源是final或等效于final的变量，就可以突破以上约束，仅在try后括号中写出，无需定义声明。

### 7.Stream增强

#### **新增`ofNullable()`**

用于创建一个 `Stream`，其中包含一个非空元素或者为空。该方法的主要目的是简化处理可能包含 `null` 值的集合时的代码，以便避免显式地检查和过滤 `null` 值。其定义如下：

```java
public static<T> Stream<T> ofNullable(T t) {
    return t == null ? Stream.empty()
                     : StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
}
```

- 如果参数 `t` 不为 null，则返回一个包含该非空元素的单元素 Stream。

- 如果参数 `t` 为 null，则返回一个空的 Stream。

这就意味着我们可以使用 `Stream.ofNullable()` 方法来快速创建一个只包含非空元素的 Stream，且无需担心处理 null 值的边界情况。比如，我们有一个 List 包含一些可能为 null 的字符串，我们可以使用 `Stream.ofNullable()` 来创建一个只包含非空字符串的 Stream：

```java
@Test
public void ofNullableTest() {
    List<String> list = Arrays.asList("死磕 Java",null,"死磕 Java 新特性","死磕 Netty",null,null);
    list.stream()
        .flatMap(Stream::ofNullable)
        .forEach(System.out::println);

}
// 结果
"死磕 Java"
"死磕 Java 新特性"
"死磕 Netty"
```

#### **重载`iterate()`**

Java 8 中的 `iterate()` 用于创建一个无限流，其元素由给定的初始值和一个生成下一个元素的函数产生，为了终止流我们需要使用一些限制性的函数来操作，例如 `limit()`：

```java
@Test
public void iterate() {
    Stream.iterate(1,v -> v + 2)
            .limit(10)
            .forEach(System.out::println);
}
```

在 Java 9 中为了限制该无序流的长度，增加了一个谓词，方法定义如下：

```java
static <T> Stream<T> iterate(T seed, Predicate<? super T> hasNext, UnaryOperator<T> next)
```

- `seed`：初始元素，作为序列的第一个元素。
- `hasNext`：用于在生成元素时限制序列的长度。如果 `hasNext` 返回 `true`，则`next` 函数就会继续生成下一个元素；一旦 `hasNext` 返回 `false`，序列生成将停止。

例如

```java
@Test
public void iterateTest() {
    Stream.iterate(1,n -> n < 20,v -> v + 2)
            .forEach(System.out::println);
}
```

当生成的值大于等于 20 时，序列生成就停止。

该重载方法允许我们更加方便地生成元素序列，并在需要时限制序列的长度，这在处理无限序列的情况下非常有用。

#### **新增`dropWhile()`和`takeWhile()`**

Java 9 引入 `dropWhile()` 和 `takeWhile()`，这两个方法允许我们根据谓词条件从流中选择或删除元素，直到遇到第一个不满足条件的元素。

`dropWhile()` 和 `takeWhile()`，用于处理流中的前缀元素，而不必处理整个流，为处理 Stream 流提供了更多的灵活性。

### 8.新增只读集合和工厂方法

#### **Java 8 创建不可变集合**

`Collections.unmodifiableXXX()`，其中`XXX`可以是`List`、`Set`或`Map`，例如要创建一个只读 List：

```java
@Test
public void test() {
    List<String> list = new ArrayList<>();
    list.add("死磕 Java 新特性");
    list.add("死磕 Netty");
    // 转换为只读集合
    list = Collections.unmodifiableList(list);
}
```

使用 `Collections.unmodifiableList()` 将 List 转换为不可变集合，如果我们使用 `add()` 来添加元素，会报 `UnsupportedOperationException` 异常信息。

这种方式虽然有效，但是比较麻烦，它需要额外的步骤来处理，而且容易出错。

#### **Java 9 创建不可变集合**

为了解决 Java 8 的问题，Java 9 引入不可变集合和对应的工厂方法，目的就在于提供更安全、更高效的方式来创建不可变集合，同时确保原始集合无法被修改。

`List`、`Set`、`Map` 都提供了对应的工厂方法来创建不可变集合，我们这里列出 `List` 的：

```java
static <E> List<E>  of()
static <E> List<E>  of(E e1)
static <E> List<E>  of(E e1, E e2)
static <E> List<E>  of(E e1, E e2, E e3)
static <E> List<E>  of(E e1, E e2, E e3, E e4)
static <E> List<E>  of(E e1, E e2, E e3, E e4, E e5)
static <E> List<E>  of(E e1, E e2, E e3, E e4, E e5, E e6)
static <E> List<E>  of(E e1, E e2, E e3, E e4, E e5, E e6, E e7)
static <E> List<E>  of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8)
static <E> List<E>  of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9)
static <E> List<E>  of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9, E e10)
//varargs
static <E> List<E>  of(E... elements)
```

看到这么多重载方法是不是有点儿懵逼，感觉是不是只需要有 `of()` 和 `of(E... elements)` 这两个方法就可以了，从实现的效果上来说，确实是可以。`of(E... elements)` 确实是一个非常灵活的方法，可以适用于多种情况，但是Java 9 引入多个重载的 `List.of()` 方法并不是为了提供不同数量的参数选择的灵活性，而是为了性能和可读性的考虑。每个 `List.of()`方法的重载版本都是针对特定的参数数量进行了优化，以提高性能和代码清晰度。当使用 `of(E... elements)` 时，Java运行时需要创建一个数组以容纳传递的元素（**Java 的可变参数，会被编译器转型为一个数组**），这可能会引入一些额外的开销，尤其是在创建小型 `List` 时。而多个重载的 `List.of()` 方法避免了这种开销，因为它们直接接受参数，而无需创建数组。所以，看着懵逼，但是性能杠杠的。

不可变的 List 具有如下几个特征：

1. 这些列表是不可变的。调用任何改变 List 的方法（如`add()`、`remove()`、`replaceAll()`、`clear()`），都会抛出 `UnsupportedOperationException`。
2. 它们不允许 `null` 元素。 尝试添加 `null` 元素将导致 `NullPointerException`。
3. 列表中元素的顺序与提供的参数或提供的数组中的元素的顺序相同。

### 9.改进的CompletableFuture

#### **前情提要**——**Java8中的CompletableFuture**

- Future的局限

1. 缺乏回调机制。
2. 无法取消任务。
3. 缺乏异常处理机制。
4. 单一结果。
5. 无法进行链式调用。
6. 无法组合多任务。

- CompletableFuture功能概况

1. 提供回调机制
2. 提供异常处理机制
3. 可以取消任务
4. 提供异步编程能力
5. 支持组合、链式操作

- API

1. 构建异步操作

| 方法              | 说明                                      | 方法返回 |
| ----------------- | ----------------------------------------- | -------- |
| runAsync          | 异步执行任务，默认 ForkJoinPool 线程池    | 无返回值 |
| supplyAsync       | 异步执行任务，默认 ForkJoinPool 线程池    | 有返回值 |
| `completedFuture` | 创建一个已经完成的 CompletableFuture 对象 | 有返回值 |

2. 两个线程依次执行

| 方法            | 说明                                                         | 方法返回值           |
| --------------- | ------------------------------------------------------------ | -------------------- |
| `thenApply`     | 获取前一个线程的执行结果，第二个线程处理该结果，生成一个新的 CompletableFuture 对象 | 有返回值             |
| `thenAccept`    | 获取前一个线程的执行结果，第二个线程消费结果，不会返还给调用端 | 无返回值             |
| `thenRun`       | 第一个线程执行完后，再执行，它忽略第一个线程的执行结果，也不返回结果 | 无返回值             |
| `thenCompose`   | 获取前一个线程的执行结果，对其进行组合，返回新的 CompletableFuture 对象 | 有返回值             |
| `whenComplete`  | 获取前一个线程的结果或异常，消费                             | 不影响上一线程返回值 |
| `exceptionally` | 线程异常执行，配合whenComplete 使用                          | 有返回值             |
| `handle`        | 相当于whenComplete + exceptionally                           | 有返回值             |

3. 等待2个线程都执行完

| 方法             | 说明                                          | 方法返回值 |
| ---------------- | --------------------------------------------- | ---------- |
| `thenCombine`    | 2个线程都要有返回值，等待都结束，结果合并转换 | 有返回值   |
| `thenAcceptBoth` | 2个线程都要有返回值，等待都结束，结果合并消费 | 无返回值   |
| `runAfterBoth`   | 2个线程无需有返回值，等待都结束，执行其他逻辑 | 无返回值   |

4. 等待2个线程任一执行完

| 方法             | 说明                                            | 方法返回值 |
| ---------------- | ----------------------------------------------- | ---------- |
| `applyToEither`  | 2个线程都要有返回值，等待任一结束，转换其结果   | 有返回值   |
| `acceptEither`   | 2个线程都要有返回值，等待任一结束，消费其结果   | 无返回值   |
| `runAfterEither` | 2个线程无需有返回值，等待任一结束，执行其他逻辑 | 无返回值   |

5. 多个线程等待

| 方法    | 说明                   | 方法返回值 |
| ------- | ---------------------- | ---------- |
| `anyOf` | 多个线程任一执行完返回 | 有返回值   |
| `allOf` | 多个线程全部执行完返回 | 无返回值   |

#### **Java9中的改进**

- 新的工厂方法

```java
//该方法允许我们快速创建一个已经有结果的 CompletableFuture，这对于单元测试或需要立即返回结果的场景非常有用。
public static <U> CompletableFuture<U> completedFuture(U value);
//此方法创建一个异常完成（异常终止）的 CompletableFuture 实例
//方法接受的参数为 Throwable，表明为异常终止，它为异步编程提供了一种简洁的方式来表示已知的失败情况，这对于错误处理和异常测试场景非常重要。
public static <U> CompletableFuture<U> failedFuture(Throwable ex);
```

- 支持超时和延迟执行

超时：

```java
//允许为 CompletableFuture 设置一个超时时间。如果在指定的超时时间内未完成，CompletableFuture 将以 TimeoutException 完成
//该方法为 CompletableFuture 提供了超时机制，这对于避免永久挂起的异步操作和保证响应性至关重要
public CompletableFuture<T> orTimeout(long timeout, TimeUnit unit);
/**
 * 示例
 */
@Test
public void orTimeTest() {
   try {
       CompletableFuture completableFuture = CompletableFuture.runAsync(()->{
           System.out.println("异步任务开始执行....");
           try {
               TimeUnit.SECONDS.sleep(5);
           } catch (InterruptedException e) {
               throw new RuntimeException(e);
           }
       }).orTimeout(2,TimeUnit.SECONDS);

       completableFuture.join();
   } catch (Exception e) {
           System.out.println(e);
  }
}
/**
 * 执行结果
 */
异步任务开始执行....
java.util.concurrent.CompletionException: java.util.concurrent.TimeoutException

```

超时返回默认值：

```java
//该允许在指定的超时时间内如果未完成，则用一个默认值来完成 CompletableFuture
//为默认默认值。该方法提供了一种优雅的回退机制，确保即使在超时的情况下也能保持异步流的连续性和完整性。
public CompletableFuture<T> completeOnTimeout(T value, long timeout, TimeUnit unit);

/*
 * 示例
 */
@Test
public void completeOnTimeoutTest() {
    CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(()->{
        System.out.println("异步任务开始执行....");
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return "死磕 Java 新特性";
    }).completeOnTimeout("死磕 Java",2,TimeUnit.SECONDS);
    System.out.println("执行结果为：" + completableFuture.join());
}
/*
* 执行结果
*/
异步任务开始执行....
执行结果为：死磕 Java
```

- 支持延迟执行

```java
//CompletableFuture 提供了delayedExecutor() 来支持延迟执行，该方法创建一个延迟执行的 Executor，可以将任务的执行推迟到未来某个时间点。方法定义如下：
public static Executor delayedExecutor(long delay, TimeUnit unit, Executor executor);

/**
 * 示例
 */
@Test
public void completeOnTimeoutTest() {
    // 创建一个延迟执行的Executor
    Executor delayedExecutor = CompletableFuture.delayedExecutor(3, TimeUnit.SECONDS);

    // 使用延迟的Executor执行一个简单任务
    CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
        System.out.println("任务延迟后执行...");
    }, delayedExecutor);

    // 等待异步任务完成
    future.join();
}
```

- 支持子类化

| 方法                  | 功能                                                         | 备注 |
| --------------------- | ------------------------------------------------------------ | ---- |
| newIncompleteFuture() | 创建一个新的、不完整的 `CompletableFuture` 实例， 子类可以重写这个方法来返回 `CompletableFuture` 的子类实例，允许在整个 `CompletableFuture` API 中自定义实例的行为。 |      |
| defaultExecutor()     | 该方法提供 `CompletableFuture` 操作的默认执行器，子类可以重写此方法以提供不同的默认执行器，这对于定制任务执行策略特别有用。 |      |
| copy()                | 创建一个与当前 `CompletableFuture` 状态相同的新实例。子类可以重写此方法以确保复制的实例是特定子类的实例，而不仅仅是 `CompletableFuture`。 |      |

## 二、Java10

### 0.概览

1. 局部变量类型推断
2. 统一垃圾回收接口
3. 并行全垃圾回收器G1
4. 应用程序类数据共享(Application Class-Data String, `AppCDS`)
5. 线程-局部变量管控
6. 一处Native-Header自动生成工具
7. 额外的Unicode语言标签扩展
8. 备用存储装置上的堆分配
9. 基于Java的实验性JIT编译器
10. 根证书认证
11. 基于时间的版本发布模式
12. 新增API

### 1.局部变量类型推断

#### 好处

```
1.减少样板代码
2.增强代码可读性
3.提高开发效率
```

#### 使用场景

1. 局部变量声明

```java
var i = 10;
var str = "局部变量声明";
```

2. 初始化集合和范型表达式

```java
//注意，这里的<String>不可写成<>
var List = new ArrayList<String>();

var map = new Hash<String,List<String>>();
//此处如果省略尖括号<>(angle brackets)中的内容，编译器会认为范型类型为Object
var List = new ArrayList<>();
```

3. 遍历操作

```java
//预定义 List<String> list;

for (var s : list){
  System.out.print(s);
}
```

4. 在`try-with-resource`语句中

```java
String path = new String("");
try (var reader = new BufferedReader(new FileReader(path))){
  //
}
```

#### 不适用场景

1. 使用`null`初始化变量时

```java
var object = null;
```

2. 需要使用具体类特定方法时

如果需要调用一个特定类的方法，而这个方法不是由其所有可能的子类共有的，那么最好显示声明这个类类型。比如：

```java
//定义三个类和一个工厂方法
		public static Father getOne() {
        return new GrandSon();
    }

    static class Father {
        public void fa() {
            System.out.println("fa");
        }
    }

    static class Son extends Father {
        public void son() {
            System.out.println("son");
        }
    }

    static class GrandSon extends Son {
        public void gSon() {
            System.out.println("gSon");
        }
    }
//此处如果调用f的方法，其实还是会当醉Father类对待
    var f = getOne();
```



### 2.不可变集合的增强



### 3.部分新增api

|                             API                              |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|           `List.copyOf`,`Set.copyOf`,`Map.copyOf`            | 这些方法提供了一种创建`不可修改集合`的快捷方式，<br/>参数是现有的集合 |
| `Collectors.toUnmodifiableList`<br/>`Collectors.toUnmodifiableSet`<br/>`Collectors.toUnmodifiableMap` |       这些方法允许将Stream的元素收集到不可修改的集合中       |
|                    `Optinal.orElseThrow`                     | `Optional.get()`的一个更易读的替代<br/>它在值不存在时抛出`NoSuchElementException` |

