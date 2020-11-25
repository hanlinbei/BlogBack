---
title: JAVA代理
categories:
  - 面试
tags:
  - JAVA
toc: true
abbrlink: 39750
date: 2020-06-28 21:51:57
---

代理模式（Proxy）是通过代理对象访问目标对象，这样可以在目标对象基础上增强额外的功能，如添加权限，访问控制和审计等功能。
<!--more-->
{% asset_img 2020-06-28-19-53-13.png %}
{% asset_img 2020-06-28-19-53-23.png %}

Java代理分为静态代理和动态代理和Cglib代理，下面进行逐个说明。

## 静态代理

接口类AdminService.java接口

```java
package com.lance.proxy.demo.service;

public interface AdminService {
    void update();
    Object find();
}
```

实现类AdminServiceImpl.java

```java
package com.lance.proxy.demo.service;

public class AdminServiceImpl implements AdminService{
    public void update() {
        System.out.println("修改管理系统数据");
    }

    public Object find() {
        System.out.println("查看管理系统数据");
        return new Object();
    }
}
```

代理类AdminServiceProxy.java

```java
package com.lance.proxy.demo.service;

public class AdminServiceProxy implements AdminService {

    private AdminService adminService;

    public AdminServiceProxy(AdminService adminService) {
        this.adminService = adminService;
    }

    public void update() {
        System.out.println("判断用户是否有权限进行update操作");
        adminService.update();
        System.out.println("记录用户执行update操作的用户信息、更改内容和时间等");
    }

    public Object find() {
        System.out.println("判断用户是否有权限进行find操作");
        System.out.println("记录用户执行find操作的用户信息、查看内容和时间等");
        return adminService.find();
    }
}
```

测试类StaticProxyTest.java

```java
package com.lance.proxy.demo.service;

public class StaticProxyTest {

    public static void main(String[] args) {
        AdminService adminService = new AdminServiceImpl();
        AdminServiceProxy proxy = new AdminServiceProxy(adminService);
        proxy.update();
        System.out.println("=============================");
        proxy.find();
    }
}
```

输出：

```java
判断用户是否有权限进行update操作
修改管理系统数据
记录用户执行update操作的用户信息、更改内容和时间等
=============================
判断用户是否有权限进行find操作
记录用户执行find操作的用户信息、查看内容和时间等
查看管理系统数据
```

总结：
静态代理模式在不改变目标对象的前提下，实现了对目标对象的功能扩展。
不足：静态代理实现了目标对象的所有方法，一旦目标接口增加方法，代理对象和目标对象都要进行相应的修改，增加维护成本。

## JDK动态代理

为解决静态代理对象必须实现接口的所有方法的问题，Java给出了动态代理，动态代理具有如下特点：
1.Proxy对象不需要implements接口；
2.Proxy对象的生成利用JDK的Api，在JVM内存中动态的构建Proxy对象。需要使用java.lang.reflect.Proxy类的

```java
  /**
     * Returns an instance of a proxy class for the specified interfaces
     * that dispatches method invocations to the specified invocation
     * handler.
 
     * @param   loader the class loader to define the proxy class
     * @param   interfaces the list of interfaces for the proxy class
     *          to implement
     * @param   h the invocation handler to dispatch method invocations to
     * @return  a proxy instance with the specified invocation handler of a
     *          proxy class that is defined by the specified class loader
     *          and that implements the specified interfaces

     */
static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,InvocationHandler invocationHandler );
```

方法，方法参数说明：
a.ClassLoader loader：指定当前target对象使用类加载器，获取加载器的方法是固定的；
b.Class<?>[] interfaces：target对象实现的接口的类型，使用泛型方式确认类型
c.InvocationHandler invocationHandler:事件处理,执行target对象的方法时，会触发事件处理器的方法，会把当前执行target对象的方法作为参数传入。

### 实战代码

AdminServiceImpl.java和AdminService.java和原来一样，这里不再赘述。
AdminServiceInvocation.java

```java
package com.lance.proxy.demo.service;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class AdminServiceInvocation  implements InvocationHandler {

    private Object target;

    public AdminServiceInvocation(Object target) {
        this.target = target;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("判断用户是否有权限进行操作");
       Object obj = method.invoke(target);
        System.out.println("记录用户执行操作的用户信息、更改内容和时间等");
        return obj;
    }
}
```

AdminServiceDynamicProxy.java

```java
package com.lance.proxy.demo.service;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

public class AdminServiceDynamicProxy {
    private Object target;
    private InvocationHandler invocationHandler;
    public AdminServiceDynamicProxy(Object target,InvocationHandler invocationHandler){
        this.target = target;
        this.invocationHandler = invocationHandler;
    }

    public Object getPersonProxy() {
        Object obj = Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), invocationHandler);
        return obj;
    }
}
```

DynamicProxyTest.java

```java
package com.lance.proxy.demo.service;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class DynamicProxyTest {
    public static void main(String[] args) {

        // 方法一
        System.out.println("============ 方法一 ==============");
        AdminService adminService = new AdminServiceImpl();
        System.out.println("代理的目标对象：" + adminService.getClass());

        AdminServiceInvocation adminServiceInvocation = new AdminServiceInvocation(adminService);

        AdminService proxy = (AdminService) new AdminServiceDynamicProxy(adminService, adminServiceInvocation).getPersonProxy();

        System.out.println("代理对象：" + proxy.getClass());

        Object obj = proxy.find();
        System.out.println("find 返回对象：" + obj.getClass());
        System.out.println("----------------------------------");
        proxy.update();

        //方法二
        System.out.println("============ 方法二 ==============");
        AdminService target = new AdminServiceImpl();
        AdminServiceInvocation invocation = new AdminServiceInvocation(adminService);
        AdminService proxy2 = (AdminService) Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), invocation);

        Object obj2 = proxy2.find();
        System.out.println("find 返回对象：" + obj2.getClass());
        System.out.println("----------------------------------");
        proxy2.update();

        //方法三
        System.out.println("============ 方法三 ==============");
        final AdminService target3 = new AdminServiceImpl();
        AdminService proxy3 = (AdminService) Proxy.newProxyInstance(target3.getClass().getClassLoader(), target3.getClass().getInterfaces(), new InvocationHandler() {

            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("判断用户是否有权限进行操作");
                Object obj = method.invoke(target3, args);
                System.out.println("记录用户执行操作的用户信息、更改内容和时间等");
                return obj;
            }
        });

        Object obj3 = proxy3.find();
        System.out.println("find 返回对象：" + obj3.getClass());
        System.out.println("----------------------------------");
        proxy3.update();


    }
}
```

输出结果：

```java
============ 方法一 ==============
代理的目标对象：class com.lance.proxy.demo.service.AdminServiceImpl
代理对象：class com.sun.proxy.$Proxy0
判断用户是否有权限进行操作
查看管理系统数据
记录用户执行操作的用户信息、更改内容和时间等
find 返回对象：class java.lang.Object
----------------------------------
判断用户是否有权限进行操作
修改管理系统数据
记录用户执行操作的用户信息、更改内容和时间等
============ 方法二 ==============
判断用户是否有权限进行操作
查看管理系统数据
记录用户执行操作的用户信息、更改内容和时间等
find 返回对象：class java.lang.Object
----------------------------------
判断用户是否有权限进行操作
修改管理系统数据
记录用户执行操作的用户信息、更改内容和时间等
============ 方法三 ==============
判断用户是否有权限进行操作
查看管理系统数据
记录用户执行操作的用户信息、更改内容和时间等
find 返回对象：class java.lang.Object
----------------------------------
判断用户是否有权限进行操作
修改管理系统数据
记录用户执行操作的用户信息、更改内容和时间等
```

## Cglib代理

DK动态代理要求target对象是一个接口的实现对象，假如target对象只是一个单独的对象，并没有实现任何接口，这时候就会用到Cglib代理(Code Generation Library)，即通过构建一个子类对象，从而实现对target对象的代理，因此目标对象不能是final类(报错)，且目标对象的方法不能是final或static（不执行代理功能）。
Cglib依赖的jar包

```yml
  <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>3.2.10</version>
  </dependency>
```

### 实战

目标对象类AdminCglibService.java

```java
package com.lance.proxy.demo.service;

public class AdminCglibService {
    public void update() {
        System.out.println("修改管理系统数据");
    }

    public Object find() {
        System.out.println("查看管理系统数据");
        return new Object();
    }
}
```

代理类AdminServiceCglibProxy.java

```java
package com.lance.proxy.demo.service;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class AdminServiceCglibProxy implements MethodInterceptor {

    private Object target;

    public AdminServiceCglibProxy(Object target) {
        this.target = target;
    }

    //给目标对象创建一个代理对象
    public Object getProxyInstance() {
        //工具类
        Enhancer en = new Enhancer();
        //设置父类
        en.setSuperclass(target.getClass());
        //设置回调函数
        en.setCallback(this);
        //创建子类代理对象
        return en.create();
    }

    public Object intercept(Object object, Method method, Object[] arg2, MethodProxy proxy) throws Throwable {

        System.out.println("判断用户是否有权限进行操作");
        Object obj = method.invoke(target);
        System.out.println("记录用户执行操作的用户信息、更改内容和时间等");
        return obj;
    }


}
```

Cglib代理测试类CglibProxyTest.java

```java
package com.lance.proxy.demo.service;

public class CglibProxyTest {
    public static void main(String[] args) {

        AdminCglibService target = new AdminCglibService();
        AdminServiceCglibProxy proxyFactory = new AdminServiceCglibProxy(target);
        AdminCglibService proxy = (AdminCglibService)proxyFactory.getProxyInstance();

        System.out.println("代理对象：" + proxy.getClass());

        Object obj = proxy.find();
        System.out.println("find 返回对象：" + obj.getClass());
        System.out.println("----------------------------------");
        proxy.update();
    }
}
```

输出结果：

```java
代理对象：class com.lance.proxy.demo.service.AdminCglibService$$EnhancerByCGLIB$$41b156f9
判断用户是否有权限进行操作
查看管理系统数据
记录用户执行操作的用户信息、更改内容和时间等
find 返回对象：class java.lang.Object
----------------------------------
判断用户是否有权限进行操作
修改管理系统数据
记录用户执行操作的用户信息、更改内容和时间等
```

## 总结

理解上述Java代理后，也就明白Spring AOP的代理实现模式，即加入Spring中的target是接口的实现时，就使用JDK动态代理，否是就使用Cglib代理。Spring也可以通过<aop:config proxy-target-class="true">强制使用Cglib代理，使用Java字节码编辑类库ASM操作字节码来实现，直接以二进制形式动态地生成 stub 类或其他代理类，性能比JDK更强。
