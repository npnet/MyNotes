## C++ 基础入门

### 1.C++初识

#### 1.1 第一个C++程序

创建项目
创建文件
编写代码
运行程序

```c++
#include<iostream>
using namespace std;

int main() {

	cout << "Hello world" << endl;

	system("pause");

	return 0;
}
```



#### 1.2 注释

1. **单行注释**	`//描述信息`

   通常放在一行代码的上方，或者一条语句的末尾，对该行代码说明

2. **多行注释 **  ` /* 描述信息 */`

   通常放在一段代码的上方，对该段代码做整体说明



#### 1.3 变量

**作用**：给一段指定的内存空间起名，方便操作这段内存

**语法**：`数据类型	变量名 = 初始值；`

**示例**：

```c++
#include<iostream>
using namespace std;

int main() {

	//变量的定义
	//语法：数据类型  变量名 = 初始值

	int a = 10;

	cout << "a = " << a << endl;
	
	system("pause");

	return 0;
}
```



#### 1.4 常量

**作用**：用于记录程序中不可更改的数据

C++定义常量的两种形式：

1. **#define** 宏常量：`#define 常量名 常量值`

   通常在文件上方定义，表示一个常量

2. **const**修饰的变量：`const 数据类型 常量名 = 常量值`

   通常在变量定义前加关键字const，修饰该变量为常量，不可修改

**示例**：

```c++
//1、宏常量
#define day 7

int main() {

	cout << "一周里总共有 " << day << " 天" << endl;
	//day = 8;  //报错，宏常量不可以修改

	//2、const修饰变量
	const int month = 12;
	cout << "一年里总共有 " << month << " 个月份" << endl;
	//month = 24; //报错，常量是不可以修改的
	
	
	system("pause");

	return 0;
}
```

