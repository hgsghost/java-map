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

##### 1.1.3 uml图和时序图

![uml图](C:\Users\Administrator\Desktop\javamap\java-map\1resource\uml图.jpg)