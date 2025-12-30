---
layout: post
title: Lombok
subtitle: Lombok simple usage
date: 2022-04-12
author: BF
thumbnail: /img/bf/java.jpg
catalog: true
toc: true
categories: Diary
header-img: /img/bf/java.jpg
tags:
    - java
    - Lombok
---

## Overview

项目中有用到`Lombok`。在查资料的时候，也发现有人分析使用它的好处与坏处。

- [《迷茫了，我们到底该不该用 lombok？》](https://baijiahao.baidu.com/s?id=1687106698282169215&wfr=spider&for=pc)。
- [《作为一名资深后端开发，为什么从不推荐别人使用 Lombok，谈谈我的看法...》](https://jiming.blog.csdn.net/article/details/104897398?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ETopBlog-1.topblog&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ETopBlog-1.topblog&utm_relevant_index=1)

不过我们一般使用的话，就不用考虑那么多了，怎么方便怎么来。

[https://projectlombok.org/](https://projectlombok.org/)

<!--more-->

## 基本介绍

> Project Lombok is a java library that automatically plugs into your editor and build tools, spicing up your java.
> Never write another getter or equals method again, with one annotation your class has a fully featured builder, Automate your logging variables, and much more.

lombok 可以通过简单的注解的形式来帮助我们简化和消除一些必须有但显得很臃肿的 Java 代码，比如常见的`Getter&Setter`、`toString()`、`构造函数`等等。lombok 不仅方便编写，同时也让我们的代码更简洁。
lombok 提供了一个功能完整的 jar 包，可以很方便的与我们的项目进行集成。

### 常用注解

![Lombok-Annotation](/img/post/2022/04/2022-04-12-lombok-usage/Lombok.png)

| Annotation               | Usage                                                                                                                         |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| @Setter                  | 注解在类或字段，注解在类时为所有字段生成 setter 方法，注解在字段上时只为该字段生成 setter 方法。                              |
| @Getter                  | 使用方法同上，区别在于生成的是 getter 方法。                                                                                  |
| @ToString                | 注解在类，添加 toString 方法。                                                                                                |
| @EqualsAndHashCode       | 注解在类，生成 hashCode 和 equals 方法。                                                                                      |
| @NoArgsConstructor       | 注解在类，生成无参的构造方法。                                                                                                |
| @RequiredArgsConstructor | 注解在类，为类中需要特殊处理的字段生成构造方法，比如 final 和被@NonNull 注解的字段。                                          |
| @AllArgsConstructor      | 注解在类，生成包含类中所有字段的构造方法。                                                                                    |
| @Data                    | 注解在类，生成 setter/getter、equals、canEqual、hashCode、toString 方法，如为 final 属性，则不会为该属性生成 setter 方法。    |
| @Slf4j                   | 注解在类，生成 log 变量，严格意义来说是常量。private static final Logger log = LoggerFactory.getLogger(UserController.class); |

### 基本原理

在Lombok使用的过程中，只需要添加相应的注解，无需再为此写任何代码。自动生成的代码到底是如何产生的呢？

核心之处就是对于注解的解析上。JDK5引入了注解的同时，也提供了两种解析方式。

#### 运行时解析

运行时能够解析的注解，必须将@Retention设置为RUNTIME，这样就可以通过反射拿到该注解。java.lang.reflect反射包中提供了一个接口AnnotatedElement，该接口定义了获取注解信息的几个方法，Class、Constructor、Field、Method、Package等都实现了该接口，对反射熟悉的朋友应该都会很熟悉这种解析方式。

#### 编译时解析

编译时解析有两种机制，分别简单描述下：

##### 1) Annotation Processing Tool

apt自JDK5产生，JDK7已标记为过期，不推荐使用，JDK8中已彻底删除，自JDK6开始，可以使用Pluggable Annotation Processing API来替换它，apt被替换主要有2点原因：

1. api都在com.sun.mirror非标准包下
2. 没有集成到javac中，需要额外运行

##### 2) Pluggable Annotation Processing API

JSR 269自JDK6加入，作为apt的替代方案，它解决了apt的两个问题，javac在执行的时候会调用实现了该API的程序，这样我们就可以对编译器做一些增强，javac执行的过程如下：
![javac-process](/img/post/2022/04/2022-04-12-lombok-usage/javac-process.png)

Lombok 就是一个实现了"JSR 269 API"的程序。

`Lombok`的底层具体实现流程如下：

1. `javac`对源代码进行分析，生成了一棵抽象语法树（AST）
2. 编译过程中调用实现了"JSR 269 API"的 Lombok 程序
3. 此时 Lombok 就对第一步骤得到的 AST 进行处理，找到@Data 注解所在类对应的语法树（AST），然后修改该语法树（AST），增加 getter 和 setter 方法定义的相应树节点
4. `javac`使用修改后的抽象语法树（AST）生成字节码文件，即给 class 增加新的节点（代码块）

## 在 IDE 中安装 Lombok

### Eclipse

下载来`lombok.jar`之后，双击运行，选择 Eclipse 的安装目录点击`Install/Update`。
![install-lombok-for-eclipse](/img/post/2022/04/2022-04-12-lombok-usage/install-lombok-for-eclipse.gif)

安装完成后，会在`eclipse`的目录下生成`lombok.jar`并在`eclipse.ini`中生成参数指向这个`jar`.

![eclipse-lombok-installed](/img/post/2022/04/2022-04-12-lombok-usage/eclipse-lombok-installed.jpg)

重启 Eclipse。

### IDEA

`File → Settings → Plugins`，输入`lombok`搜索，选中插件后`install`进行安装即可，安装后需重启 IDEA 才能运行。

## 使用示例

项目中添加依赖：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.22</version>
</dependency>
```

创建一个类使用`@Data`：

```java
package org.bearfly.page;

import lombok.Data;

@Data
public class Person {
 private int id;
 private String name;
 private int age;
 private String job;
}
```

反编译查看编译后的`Person.class`文件，可以看到`lombok`自动帮助添加了对应的方法。

```java
package org.bearfly.page;

public class Person {
  private int id;
  
  private String name;
  
  private int age;
  
  private String job;
  
  public void setId(int id) {
    this.id = id;
  }
  
  public void setName(String name) {
    this.name = name;
  }
  
  public void setAge(int age) {
    this.age = age;
  }
  
  public void setJob(String job) {
    this.job = job;
  }
  
  public boolean equals(Object o) {
    if (o == this)
      return true; 
    if (!(o instanceof Person))
      return false; 
    Person other = (Person)o;
    if (!other.canEqual(this))
      return false; 
    if (getId() != other.getId())
      return false; 
    if (getAge() != other.getAge())
      return false; 
    Object this$name = getName(), other$name = other.getName();
    if ((this$name == null) ? (other$name != null) : !this$name.equals(other$name))
      return false; 
    Object this$job = getJob(), other$job = other.getJob();
    return !((this$job == null) ? (other$job != null) : !this$job.equals(other$job));
  }
  
  protected boolean canEqual(Object other) {
    return other instanceof Person;
  }
  
  public int hashCode() {
    int PRIME = 59;
    result = 1;
    result = result * 59 + getId();
    result = result * 59 + getAge();
    Object $name = getName();
    result = result * 59 + (($name == null) ? 43 : $name.hashCode());
    Object $job = getJob();
    return result * 59 + (($job == null) ? 43 : $job.hashCode());
  }
  
  public String toString() {
    return "Person(id=" + getId() + ", name=" + getName() + ", age=" + getAge() + ", job=" + getJob() + ")";
  }
  
  public int getId() {
    return this.id;
  }
  
  public String getName() {
    return this.name;
  }
  
  public int getAge() {
    return this.age;
  }
  
  public String getJob() {
    return this.job;
  }
}

```

在别的地方直接可以使用：

```java
package org.bearfly;

import org.bearfly.page.Person;

public class Application {

    public static void main(String[] args) {
        Person person = new Person();
        person.setId(0);
        person.getName();
    }

}
```

## 优缺点

### 优点

1. 能通过注解的形式自动生成构造器、getter/setter、equals、hashcode、toString等方法，提高了一定的开发效率
2. 让代码变得简洁，不用过多的去关注相应的方法
3. 属性做修改时，也简化了维护为这些属性所生成的getter/setter方法等

### 　缺点

1. 强制要求队友安装idea插件
   如果使用lombok注解编写代码，就要求参与开发的所有人都必须安装lombok插件，否则代码编译出错。
2. 代码可读性变差
   使用lombok注解之后，最后生成的代码你其实是看不到的，你能看到的是代码被修改之前的样子。如果要想查看某个getter或setter方法的引用过程，是非常困难的。
3. 升级JDK对功能有影响
4. 有一些坑
   使用@Data时会默认使用@EqualsAndHashCode(callSuper=false)，这时候生成的equals()方法只会比较子类的属性，不会考虑从父类继承的属性，无论父类属性访问权限是否开放。
   使用@Builder时要加上@AllArgsConstructor，否则可能会报错。
5. 使用lombok生成的代码不太好调试。

## 参考

- [迷茫了，我们到底该不该用 lombok?](https://baijiahao.baidu.com/s?id=1687106698282169215&wfr=spider&for=pc)。
- [作为一名资深后端开发，为什么从不推荐别人使用 Lombok，谈谈我的看法...](https://jiming.blog.csdn.net/article/details/104897398?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ETopBlog-1.topblog&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ETopBlog-1.topblog&utm_relevant_index=1)
- [Lombok 的基本使用](https://www.jianshu.com/p/2543c71a8e45)
- [Lombok 简介、使用、工作原理、优缺点](https://blog.csdn.net/ThinkWon/article/details/101392808)
- [Lombok 详解](https://blog.csdn.net/qq3399013670/article/details/81948737)
