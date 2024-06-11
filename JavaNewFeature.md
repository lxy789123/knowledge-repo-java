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

