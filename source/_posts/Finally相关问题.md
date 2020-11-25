---
title: Finally相关问题
categories:
  - 面试
tags:
  - Java
toc: true
abbrlink: 42536
date: 2020-08-18 21:51:57
---

问题描述：try{}里有一个return语句，那么紧跟在这个try{}后面的finally{}中的代码是否会被执行？如果会的话，什么时候被执行，在return之前还是return之后？
<!--more-->
在Java语言的异常处理中，finally块的作用就是为了保证无论出现什么情况，finally块里的代码一定会被执行。由于程序执行return就意味着结束对当前函数的调用并跳出这个函数体，因此任何语句要执行都只能在return前执行（除非碰到exit函数），因此finally块里的代码也是在return之前执行的。此外，如果try-finally或者catch-finally中都有return，那么finally块中的return将会覆盖别处的return语句，最终返回到调用者那里的是finally中return的值。下面通过一个例子来说明这个问题：

```java
package com.js;
/**
 * try-catch中有return语句，finally中代码运行时机问题
 * @author hanlinbei
 */
 
public class Test{
	public static int testFinally(){
		try {
			return 1;
		} catch (Exception e) {
			return 0;
		}finally{
			System.out.println("execute finally");
		}
	}
	public static void main(String[] args){
		int result = testFinally();
		System.out.println(result);
	}
}
```

运行结果：

```console
execute finally
1
```

从上面这个例子中可以看出，在执行return语句前确实执行了finally块中的代码。紧接着，在finally块里放置个return语句，来看看到底最终返回的是哪个return语句的值，例子如下图所示：

```java
package com.js;
/**
 * try-catch中有多个return语句，研究return的是哪一个
 * @author hanlinbei
 */
 
public class Test{
	public static int testFinally(){
		try {
			return 1;
		} catch (Exception e) {
			return 0;
		}finally{
			System.out.println("execute finally");
			return 3;
		}
	}
	public static void main(String[] args){
		int result = testFinally();
		System.out.println(result);
	}
}
```

运行结果：

```console
execute finally
3
```

从以上运行结果可以看出，当finally块中有return语句时，将会覆盖函数中其他return语句。此外，由于在一个方法内部定义的变量都存储在栈中，当这个函数结束后，其对应的栈就会被回收，此时在其方法体中定义的变量将不存在了，因此，对基本类型的数据，在finally块中改变return的值对返回值没有任何影响，而对引用类型的数据会有影响。下面通过一个例子来说明这个问题：

```java
package com.js;
/**
 * 在finally块中改变基本数据类型、引用类型对比
 * @author hanlinbei
 */
 
public class Test{
	public static int testFinally1(){
		int result = 1;
		try {
			result = 2;
			return result;
		} catch (Exception e) {
			return 0;
		}finally{
			result = 3;
			System.out.println("execute finally1");
		}
	}
	public static StringBuffer testFinally2(){
		StringBuffer s = new StringBuffer("Hello");
		try {
			return s;
		} catch (Exception e) {
			return null;
		}finally{
			s.append(" World");
			System.out.println("execute finally2");
		}
	}
	public static void main(String[] args){
		int result = testFinally1();
		System.out.println(result);
		StringBuffer resultRef = testFinally2();
		System.out.println(resultRef);
	}
}
```

运行结果：

```console
execute finally1
2
execute finally2
Hello World
```

程序在执行到return时会首先将返回值存储在一个指定的位置，其次去执行finally块，最后再返回。在方法testFinally1中调用return前，先把result的值1存储在一个指定的位置，然后再去执行finally块中的代码，此时修改result的值将不会影响到程序的返回结果。testFinally2中，在调用return前先把s存储到一个指定的位置，由于s为引用类型，因此在finally中修改s将会修改程序的返回结果。

引申：出现在Java程序中的finally块是不是一定会被执行？
答案：不一定。

下面给出两个finally块不会被执行的例子：
1）、当程序进入try块之前就出现异常时，会直接结束，不会执行finally块中的代码，示例如下：

```java
package com.js;
/**
 * 在try之前发生异常
 * @author hanlinbei
 */
 
public class Test{
	public static void testFinally1(){
		int result = 1/0;
		try {
			System.out.println("try block");
		} catch (Exception e) {
			System.out.println("catch block");
		}finally{
			System.out.println("finally block");
		}
	}
	public static void main(String[] args){
		testFinally1();
	}
}
```

运行结果：

```console
Exception in thread "main" java.lang.ArithmeticException: / by zero
at com.js.Test.testFinally1(Test.java:9)
at com.js.Test.main(Test.java:19)
```

程序在执行1/0时会抛出异常，导致没有执行try块，因此finally块也就不会被执行。

2）、当程序在try块中强制退出时也不会去执行finally块中的代码，示例如下：

```java
package com.js;
/**
 * 在try之前发生异常
 * @author hanlinbei
 */
 
public class Test{
	public static void testFinally1(){
		try {
			System.out.println("try block");
			System.exit(0);
		} catch (Exception e) {
			System.out.println("catch block");
		}finally{
			System.out.println("finally block");
		}
	}
	public static void main(String[] args){
		testFinally1();
	}
}
```

运行结果：

```console
try block
```

上例在try块中通过调用System.exit(0)强制退出了程序，因此导致finally块中的代码没有被执行