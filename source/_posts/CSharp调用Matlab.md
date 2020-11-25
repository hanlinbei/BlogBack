---
title: 'CSharp调用Matlab'
categories:
  - 环境搭建
tags:
  - Matlab
  - 算法
toc: true
abbrlink: 1269
date: 2020-07-06 11:11:57
---

在matlab里仿真实现了，如果不想重新在C#代码里写程序的话，就可以用matlab生成dll程序集直接调用。
<!--more-->

## 配置

拿Matlab自带函数Linqprog(线性规划的函数)举例，首先新建函数，输入如下函数，并保存。
{% asset_img 2020-07-06-18-58-24.png %}
其次在matlab命令行中输入deploytool，跳出如下页面
{% asset_img 2020-07-06-18-58-40.png %}
选择Library Compiler 进入如下页面
{% asset_img 2020-07-06-18-59-12.png %}

第一个箭头选择.net环境，第二个箭头添加你刚才保存的函数。后面其他的配置不需要更改，如果想要对生成的dll文件里面的类名以及命名空间进行修改可以在此图片最后一部分进行修改，此处就不修改。点击右上角Package，保存并等待。此处如果生成失败，说明matlab安装存在问题，自己在网上找资源，或者可以留言找我要资源（免费哟）。这里生成dll文件就已经完成了，就剩下最后一步编写C#程序了。

## 编写C#文件

首先创建项目，并导入两个依赖项，(1)刚才Package生成的dll文件(MyLinprog.dll)。(2)MWArray.dll文件，在Matlab安装目录下%matlabroot%\\toolbox\dotnetbuilder\bin\win64\v4.0\MWArray.dll。导入依赖项成功之后编写函数。

```c#
//导入两个命名空间
//using MathWorks.MATLAB.NET.Arrays;
//using MyLinprog;
static void Main(string[] args)
        {
            //输入参数
            MWArray A = (MWNumericArray)new double[,] { { 1, -1, 1 }, { 3, 2, 4 }, { 3, 2, 0 } };
            MWArray f = (MWNumericArray)new double[] { -5, -4, -6 };
            MWArray b = (MWNumericArray)new double[] { 20, 42, 30 };
            MWArray lb = (MWNumericArray)new double[] { 0, 0, 0 };
            MWArray ub = (MWNumericArray)new double[] { };
            MWArray Aeq = (MWNumericArray)new int[3] {0,0,0};
            MWArray beq = (MWNumericArray)new int[1] {0};
            MWArray x0 = (MWNumericArray)new int[0] ;

            MWArray[] agrsIn = new MWArray[] { (MWNumericArray)f, (MWNumericArray)A,     (MWNumericArray)b, (MWNumericArray)Aeq, (MWNumericArray)beq,(MWNumericArray)lb, (MWNumericArray)ub, (MWNumericArray)x0 };//输入参数

            MWArray[] agrsOut = new MWArray[2]; //输出存放的数组

            MyLinprog.Class1 mu = new MyLinprog.Class1(); //实例化对象
            mu.MyLinprog(2, ref agrsOut, agrsIn); //计算
            Console.WriteLine("x最优值为 : \t");
            Console.WriteLine(agrsOut[0]);
            Console.WriteLine("得到的y值为 : \t" + agrsOut[1]);
            Console.ReadKey();

        }
```

此时已经完成了所有工作，接下来总结在这个过程中常出现的问题
问题1:
{% asset_img 2020-07-06-19-01-47.png %}
在调用MWArray类时出现问题，此时应注意你所用的MWArray的环境需要和你的项目平台保持一致。即，引用的win64下的MWArray时，c#的项目平台应该是x64。

问题2:
在实例化对象处报错（因为我的环境配置好的，修改环境需要重启电脑，所以就没有去把错误调处来）。报错依然是类型初始化异常。
此时就应该查看MCR(Matlab Runtime)与项目平台以及MATLAB\R2017b\bin\win64三者是否保持一致。
