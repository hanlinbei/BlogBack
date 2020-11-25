---
title: 重载和重写
categories:
  - 面试
tags:
  - 面经
toc: true
abbrlink: 30931
date: 2020-08-17 11:11:57
---

Java的重载和重写
<!--more-->
之前刷选择题刷了挺多，碰到过重载和重写区别的题挺多的，当时感觉记住了，可回过头来看结果又有些东西忘记了，所以还是写篇文章记录下吧。

## 重载

1. 重载是多态在编译器时期的表现形式
2. 重载的判定只有两个条件，方法名一致、形参列表不同，返回值不同不能作为判断条件

```java
public class Father {

    public static void main(String[] args) {
        // TODO Auto-generated method stub
        Father s = new Father();
        s.sayHello();
        s.sayHello("wintershii");

    }

    public void sayHello() {
        System.out.println("Hello");
    }

    public void sayHello(String name) {
        System.out.println("Hello" + " " + name);
    }
}
```

## 重写

1. 重写是在方法运行时，通过调用者的实际类型来确定方法的调用版本
2. 重写只发生在可见的实例方法中。静态方法不存在重写，形式上的重写只能说是隐藏。私有方法不存在重写，父类中private方法子类就算定义了，相当于一个新方法。静态方法和实例方法不存在相互重写。
3. 重写满足一个原则：两同两小一大。两同是方法名和形参列表相同。两小指的是重写方法的返回值和抛出异常要和被重写方法的返回值和抛出的异常相同或者是其子类，注意，一旦返回值是基本数据类型，那么重写方法和被重写方法必须相同，且不存在自动拆装箱问题。一大指的是重写方法的访问修饰符要大于等于被重写方法的访问修饰符。

```java
public class Father {

    public static void main(String[] args) {
        // TODO Auto-generated method stub
        Son s = new Son();
        s.sayHello();
    }

    public void sayHello() {
        System.out.println("Hello");
    }
}

class Son extends Father{

    @Override
    public void sayHello() {
        // TODO Auto-generated method stub
        System.out.println("hello by ");
    }

}
```

### 面试时，问：重载（Overload）和重写（Override）的区别

> 答：方法的重载和重写都是实现多态的方式，区别在于前者实现的是编译时的多台性，后者实现的是运行时的多态性。重载发生在一个类中，同名的方法如果有不同的参数列表(参数类型不同，参数个数不同或者两者都不同)则被视为重载；重写发生在子类与父类之间，重写要求子类被重写方法与父类被重写方法有相同的参数列表，有兼容的返回类型，有比父类被重写方法更好访问，不能比父类被重写方法声明更多的异常(里氏替换原则)。重载对返回值类型没有特殊的要求，不能根据返回类型进行区分。