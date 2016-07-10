环境：vs2013 32位版，Matlab2015b 64位
内容：在Matlab 编写 *.m文件 然后编译生成动态链接库 *.dll 在 vs2013中使用
####1. Matlab中编写 程序 Add.m
```
function  c=Add(a,b)
c=a+b;
end
```
保存后
#####1).设置编译环境
 在CommandWindow中输入以下命令：
```
>> mex -setup
```
>MEX configured to use 'Microsoft Visual C++ 2013 Professional (C)' for C language compilation.
Warning: The MATLAB C and Fortran API has changed to support MATLAB
	 variables with more than 2^32-1 elements. In the near future
	 you will be required to update your code to utilize the
	 new API. You can find more information about this at:
	 http://www.mathworks.com/help/matlab/matlab_external/upgrading-mex-files-to-use-64-bit-api.html.
To choose a different language, select one from the following:
 mex -setup C++ 
 mex -setup FORTRAN

选择： mex -setup C++ （单击）
>MEX configured to use 'Microsoft Visual C++ 2013 Professional' for C++ language compilation.
Warning: The MATLAB C and Fortran API has changed to support MATLAB
	 variables with more than 2^32-1 elements. In the near future
	 you will be required to update your code to utilize the
	 new API. You can find more information about this at:
	 http://www.mathworks.com/help/matlab/matlab_external/upgrading-mex-files-to-use-64-bit-api.html.
	 
	 
 在CommandWindow中输入以下命令：

```
 >> mbuild -setup       //
```
 >MBUILD configured to use 'Microsoft Visual C++ 2013 Professional (C)' for C language compilation.
To choose a different language, select one from the following:
 mex -setup C++ -client MBUILD 
 mex -setup FORTRAN -client MBUILD
 
 
选择：  mex -setup C++ -client MBUILD  （单击）

>MBUILD configured to use 'Microsoft Visual C++ 2013 Professional' for C++ language compilation.

如果出现以上说明，则证明Matlab编译器设置成功了。
#####2).接下来是生成M文件的DLL文件。
在Matlab主窗口中键入如下代码：
```c
>> mcc -W cpplib:mydllAdd  -T link:lib Add.m
/*其中cpplib:后面是 生成文件的文件名，是自己取的，
link:lib后面的Add.m 是 *.M文件的文件名。 
-W   -T 是参数，具体含义可以通过mcc –help命令查看，注意参数的大小写。
*/
```
>使用 'Microsoft Visual C++ 2013 Professional' 编译。 

完成后：在当前文件夹下生成若干文件 
如图: 
￼

####2.VS2013中调用 dll文件
#####1） 创建c++ 项目   testDll （注意选择空项目）

￼
#####2）将生成的 mydllAdd.dll ；mydllAdd.h  mydllAdd.lib 文件拷贝到 testDll 项目目录下
将mydllAdd.h 添加到头文件里
右击->头文件->添加->添加现有项-找到mydllAdd.h 
￼

网上看到说好像 mydllAdd.dll 需要拷贝到debug 目录下 我没拷  程序也能运行
#####3）设置附加包含目录目录
右击项目->属性->C/C++ 添加路径：
```c
D:\home\study\MATLAB\R2015b\extern\include      //根据自己matlab安装目录调整
```
￼

#####4）设置附加 库目录
右击项目->属性->连接器->附加库目录
```c
D:\WorkStation\matlab\Add;
D:\home\study\MATLAB\R2015b\extern\lib\win64\microsoft;   //根据自己matlab安装目录调整
```

￼
#####5）设置附加依赖项
右击项目->属性->连接器->输入->附加依赖项
```c
mclmcrrt.lib
mclmcr.lib
libmx.lib
libmat.lib
libeng.lib
mydllAdd.lib        //这个是由 *.m 文件生成的    根据自己matlab安装目录调整
```
设置如图：
￼

￼


#####6）更改配置管理器  用64位编译
如图：单击配置管理器->活动解决方案平台->新建->选择64位
￼

#####7）编写主程序 源.c 代码


```c    
//源.c
#include "mclmcrrt.h"
#include "mclmcr.h"
#include "matrix.h"
#include "mclcppclass.h"
#include "mydllAdd.h"               //*.m文件生成
#include <stdio.h>   
#include <iostream>   

using namespace std;
int main()
{
	double a;
	double b;
	double c;
		// initialize lib，这里必须做初始化！   
	if (!mydllAddInitialize())
	{//若失败执行
		cout << "Could not initialize mydllAdd!" << endl;
	}
	cout << "请输入a:" << endl;
	cin >> a;
	cout << "请输入b:" << endl;
	cin >> b;
	
	// 为变量分配内存空间，可以查帮助mwArray   文章附件有链接
	mwArray mwA(1, 1, mxDOUBLE_CLASS); // 1，1表示矩阵的大小（所有maltab只有一种变量，就是矩阵，为了和Cpp变量接轨，设置成1*1的矩阵，mxDOUBLE_CLASS表示变量的精度）   
	mwArray mwB(1, 1, mxDOUBLE_CLASS);
	mwArray mwC(1, 1, mxDOUBLE_CLASS);

	// set data 调用类里面的SetData函数给类赋值   
	mwA.SetData(&a, 1);
	mwB.SetData(&b, 1);
	// using my add，调用动态库里 写的函数   
	Add(1, mwC, mwA, mwB);
	// get data 调用类里面的Get函数获取取函数返回值   
	c = mwC.Get(1, 1);
	cout << "c=a+b=" << c << endl;

	// 后面是一些终止调用的程序   
	// terminate the lib   
	mydllAddTerminate();
	// terminate MCR   
	mclTerminateApplication();
	getchar();
	return 0;
 }
 ```
#####7)执行程序 结果如下：
 
￼
注：虽然程序功能简单 ，但是是一个完整的通过VS调用Matlab动态库的流程
按此方法，借助Matlab强大的数学计算能力，通过在VS中调用Matlab生成的 动态链接库的方法 实现信号处理和图形图像的处理
####3. mwArray 类使用方法
链接：
