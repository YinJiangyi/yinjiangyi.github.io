---
title: 【转载】java：java对象POJO和JavaBean的区别
typora-root-url: ../../source
date: 2021-01-29 14:40:27
tags: Java
categories: Java开发
---

原文： https://www.jianshu.com/p/224489dfdec8

# POJO

"Plain Ordinary Java Object"，简单普通的java对象。主要用来指代那些没有遵循特定的java对象模型，约定或者框架的对象。

POJO的内在含义是指那些:
 有一些private的参数作为对象的属性，然后针对每一个参数定义get和set方法访问的接口。
 没有从任何类继承、也没有实现任何接口，更没有被其它框架侵入的java对象。



```cpp
public class BasicInfoVo {

private String orderId;

private Integer uid;


public String getOrderId() {
    return orderId;
}

public void setOrderId(String orderId) {
    this.orderId = orderId;
}

public Integer getUid() {
    return uid;
}

public void setUid(Integer uid) {
    this.uid = uid;
}}
```

# JavaBean

JavaBean 是一种JAVA语言写成的可重用组件。JavaBean符合一定规范编写的Java类，不是一种技术，而是一种规范。大家针对这种规范，总结了很多开发技巧、工具函数。符合这种规范的类，可以被其它的程序员或者框架使用。它的方法命名，构造及行为必须符合特定的约定：

1. 所有属性为private。
2. 这个类必须有一个公共的缺省构造函数。即是提供无参数的构造器。
3. 这个类的属性使用getter和setter来访问，其他方法遵从标准命名规范。
4. 这个类应是可序列化的。实现serializable接口。

因为这些要求主要是靠约定而不是靠实现接口，所以许多开发者把JavaBean看作遵从特定命名约定的POJO。



```java
public class UserInfo implements java.io.Serializable{  
  
//实现serializable接口。  
private static final long serialVersionUID = 1L;  
  
private String name;  
private int age;  
  
//无参构造器  
public UserInfo() {  
      
}  

public String getName() {  
    return name;  
}  

public void setName(String name) {  
    this.name = name;  
}  

public int getAge() {  
    return age;  
}  

public void setAge(int age) {  
    this.age = age;  
}  

//javabean当中可以有其它的方法  
public void userInfoPrint(){  
    System.out.println("");  
}  }  
```

# 两者有什么区别

1. POJO其实是比javabean更纯净的简单类或接口。POJO严格地遵守简单对象的概念，而一些JavaBean中往往会封装一些简单逻辑。
2. POJO主要用于数据的临时传递，它只能装载数据， 作为数据存储的载体，而不具有业务逻辑处理的能力。
3. Javabean虽然数据的获取与POJO一样，但是javabean当中可以有其它的方法。