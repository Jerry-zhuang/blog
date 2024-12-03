# 命令注入

## 一、无废话总结

**关键词**：Runtime/ProcessBuilder/ProcessImpl

白盒搜索关键函数，倒推数据流，查看命令执行相关函数的参数是否可控。

**参数可控+命令执行函数=命令注入**

## 二、相关函数

在 Java 中，执行系统命令主要函数包括：**Runtime**、**ProcessBuilder** 和 **ProcessImpl**。

### Runtime 类

Runtime 类是 Java 中用于执行系统命令的主要类之一。它的构造器是私有的，无法通过 new Runtime() 创建对象，只能通过静态方法 Runtime.getRuntime() 获取。

Runtime 类提供了 exec() 方法，用于执行外部命令或程序。例如：

```java
Runtime.getRuntime().exec("command");
```

### ProcessBuilder 类

ProcessBuilder 类提供了更灵活的方式来执行系统命令。它允许用户设置环境变量、工作目录等，并可以处理多个命令。ProcessBuilder 通常通过 new ProcessBuilder(String\[] cmdarray) 来创建对象。例如：

```java
ProcessBuilder processBuilder = new ProcessBuilder("command");
processBuilder.start();
```

### ProcessImpl 类

ProcessImpl 类是 Process 类的实现之一，用于创建和管理子进程。它的构造器也是私有的，无法通过 new ProcessImpl() 创建对象。通常通过反射机制来调用其方法。例如：

```java
Class<?> clazz = Class.forName("java.lang.ProcessImpl");
Method method = clazz.getDeclaredMethod("start", String.class);
method.invoke(null, "command");
```

## 三、注意事项

在使用 Runtime.exec() 进行命令执行时，若参数可控，可以通过拼接命令的方式进行命令注入。例如：

```java
String userInput = "ping www.baidu.com; cat /etc/passwd"; // 假设为命令拼接
Runtime.getRuntime().exec(userInput);
```

但是在使用 ProcessBuilder 的场景中，由于使用的是命令数组的形式作为输入，无法通过拼接的方式进行命令注入，除了在命令数组中使用了 /bin/sh 或者 /bin/bash 为开头的方式进行命令执行则存在问题。例如：

```java
String[] arrCmd = {"/bin/sh", "-c", cmd}; // 其中 cmd 为外部输入
ProcessBuilder processBuilder = new ProcessBuilder(arrCmd);
processBuilder.start();
```
