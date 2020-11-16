# 函数的可变参数
指定参数的函数实现很简单，通过指定的参数名访问就行。但可变参数的函数调用的参数会进行压栈处理，从右到左进行压栈，参数和参数之间的存放是连续的，即知道第一个参数的地址和类型，以及其他参数的类型就可以获得各个参数的地址。

对于不定参数部分用 `...` 表示即可，但第一个的参数必须提供，这个参数用来寻址，实现对所有参数的访问 `void func(int a, ...)`

```cpp
//可变参数标准宏头文件
#include "stdarg.h"
 
//申明va_list数据类型变量pvar，该变量访问变长参数列表中的参数。
va_list pvar;
 
//宏va_start初始化变长参数列表。pvar是va_list型变量，记载列表中的参数信息。
//parmN是省略号"..."前的一个参数名，va_start根据此参数，判断参数列表的起始位置。
va_start(pvar, parmN);
 
//获取变长参数列表中参数的值。pvar是va_list型变量，type为参数值的类型，也是宏va_arg返回数值的类型。
//宏va_arg执行完毕后自动更改对象pvar，将其指向下一个参数。
va_arg(pvar, type);
 
//关闭本次对变长参数列表的访问。
va_end(pvar);
```

```cpp
#include "stdarg.h"
using namespace std;
 
int sum(int count, ...)
{
	int sum_value=0;
 
	va_list args;
	va_start(args,count);
	while(count--)
	{
		sum_value+=va_arg(args,int);
	}
	va_end(args);
 
	return sum_value;
}
 
int _tmain(int argc, _TCHAR* argv[])
{
	cout<<sum(5,1,2,3,4,5);//输出15
}
```