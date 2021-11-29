# 第一章  java基础

## 1.1 面向对象

[参考链接](https://pdai.tech/md/java/basic/java-basic-oop.html)

##### 1.1.1 三大特性:封装,继承,多态

多态分为编译时多态和运行时多态

1. 编译时多态:方法重载
2. 运行时多态:程序中定义的对象引用所指向的类型在运行期间确定(一个父类,多个子类)

运行时多态需要3个条件:继承,重写,向上转型

##### 1.1.2 类之间的关系

+ 泛化关系generalization(继承)
+ 实现关系realization(实现接口)
+ 聚合关系aggregation(部分组成整体,整体和部分不是强依赖,整体不存在部分依然存在,例如 电脑和键盘鼠标屏幕)
+ 组合关系composition(整体和部分强依赖,例如公司和部门)
+ 关联关系association(不同类对象的静态关联关系例如 学校和学生)
+ 依赖关系dependency(依赖关系主要作用在运行过程中)
  + a类b类的(某个方法中)局部变量
  + a类是b类方法的参数
  + a类想b类发信息,从而影响b类

##### 1.1.3 uml图

![uml图](.\1resource\uml图.jpg)

##### 1.1.4 时序图

时序图主要有以下7种元素

+ 角色
+ 对象
+ 生命线
+ 控制焦点
+ 消息
+ 自关联消息
+ 组合片段

![时序图](.\1resource\时序图.jpg)

##### 1.1.5 缓冲池

new Integer(123) 与 Integer.valueOf(123) 的区别在于(java8 integer缓冲池从-128~127)

* new Integer(123) 每次都会新建一个对象
* Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。

##### 1.1.6 如何拷贝一个对象

使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。例如:

```java
public class CloneConstructorExample {
    private int[] arr;

    public CloneConstructorExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }
    public CloneConstructorExample(CloneConstructorExample original) {
        arr = new int[original.arr.length];
        for (int i = 0; i < original.arr.length; i++) {
            arr[i] = original.arr[i];
        }
    }
    public void set(int index, int value) {
        arr[index] = value;
    }
    public int get(int index) {
        return arr[index];
    }
}
```

## 1.2泛型的桥接

[泛型的桥接](https://pdai.tech/md/java/basic/java-basic-x-generic.html)

## 1.3 注解

元注解:

+ @Target (被注解类可修饰的对象范围(对象,方法,属性等等) 取值范围定义在ElementType枚举中)
+ @Retention&@RetentionTarget(被注解类保留的时间范围(source,class,runtime)取值范围在RetentionPolicy中)
+ @Documented(描述在使用 javadoc 工具为类生成帮助文档时是否要保留其注解信息。)
+ @Inherited (被它修饰的Annotation将具有继承性。如果某个类使用了被@Inherited修饰的Annotation，则其子类将自动具有该注解。)
+ @Repeatable(重复注解 允许多个相同注解指向相同对象)

自定义注解:

```java
package com.pdai.java.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyMethodAnnotation {
    public String title() default "";
    public String description() default "";
}
```

注解无法被继承,但是如果某个类使用了被@inherited修饰的注解 那么它的子类也自动有这个注解

## 1.4 异常

![异常](.\1resource\异常.png)

只针对不正常的情况才使用异常:

1. 异常机制的设计初衷是用于不正常的情况，所以很少会会JVM实现试图对它们的性能进行优化。所以，创建、抛出和捕获异常的开销是很昂贵的。
2. 把代码放在try-catch中返回阻止了JVM实现本来可能要执行的某些特定的优化。
3. 对数组进行遍历的标准模式并不会导致冗余的检查，有些现代的JVM实现会将它们优化掉。

## 1.5 反射

1. 反射类及反射方法的获取，都是通过从列表中搜寻查找匹配的方法，所以查找性能会随类的大小方法多少而变化；

2. 每个类都会有一个与之对应的Class实例，从而每个类都可以获取method反射方法，并作用到其他实例身上；

3. 反射也是考虑了线程安全的，放心使用；

4. 反射使用软引用relectionData缓存class信息，避免每次重新从jvm获取带来的开销；

5. 反射调用多次生成新代理Accessor, 而通过字节码生存的则考虑了卸载功能，所以会使用独立的类加载器；

6. 当找到需要的方法，都会copy一份出来，而不是使用原来的实例，从而保证数据隔离；

7. 调度反射方法，最终是由jvm执行invoke0()执行；

## 1.6 SPI（Service Provider Interface）

[SPI](https://pdai.tech/md/java/advanced/java-advanced-spi.html#spi%e6%9c%ba%e5%88%b6%e7%9a%84%e7%ae%80%e5%8d%95%e7%a4%ba%e4%be%8b)