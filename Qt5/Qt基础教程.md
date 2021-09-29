## Qt基础教程

### 1. 第一个Qt小程序

#### 1.1 按钮的创建

在Qt中最常用的控件就是按钮.

**头文件 ： #include\<QPushButton\> **

**创建按钮示例**：

```c++
#include <QPushButton> //头文件
QPushButton * btn = new QPushButton; 

//设置父亲
btn->setParent(this);
//设置文字
btn->setText("德玛西亚");
//移动位置
btn->move(100,100);

//第二种创建
QPushButton * btn2 = new QPushButton("孙悟空",this);
//重新指定窗口大小
this->resize(600,400);

//设置窗口标题
this->setWindowTitle("第一个项目");

//限制窗口大小
this->setFixedSize(600,400);

```

一个按钮其实就是 `QPushButton`类下的对象，如果知识创建出对象是无法显示到窗口中的，所以我们需要依赖一个父窗口。

* 指定一个父亲利用 `setParent`函数
* 设置按钮上显示的文字利用 `setText`函数
* 移动按钮位置利用 `move` 函数



#### 1.2 对象模型（对象树）

在Qt中创建对象时会提供一个Parent对象指针。

* QObject是以对象树的形式组织起来的

  * 当创建一个QObject对象时，会看到QObject的构造函数接收一个QObject指针作为参数，

    这个参数就是parent，也就是父对象指针。

    相当于，在创建QObject对象时，可以提供一个其父对象，我们创建的这个QObject对象会自动添加到其父对象的children()列表

  * 当父对象析构时，这个列表中所有的对象也会被析构。

    (注意，这里的父对象不是继承意义上的父类)

* QWidget是能够在屏幕上显示的一切组件的父类

  * QWidget继承自QObject，因此也继承了这种对象树关系。一个孩子自动地成为一个父组件的子组件。
  * 我们也可以自己删除子对象，他们会自动从父对象列表中删除。

Qt 引入对象树的概念，在一定程度上解决了内存问题。

* 当一个QObject对象在堆上创建的时候，Qt 会同时为其创建一个对象树。不过，对象树中对象的顺序是没有定义的。这意味着，销毁这些对象的顺序也是未定义的。

* 任何对象树中的 QObject对象 delete 的时候，如果这个对象有 parent，则自动将其从 parent 的children()列表中删除；如果有孩子，

  则自动 delete 每一个孩子。Qt 保证没有QObject会被 delete 两次，这是由析构顺序决定的。

如果QObject在栈上创建，Qt 保持同样的行为。正常情况下，这也不会发生什么问题。

```c++
{
    QWidget window;
    QPushButton quit("Quit", &window);
}

```

这段代码是正确的，quit 的析构函数不会被调用两次，因为标准 C++要求，**局部对象的析构顺序应该按照其创建顺序的相反过程**。

会先调用 quit 的析构函数，将其从父对象 window 的子对象列表中删除，然后才会再调用 window 的析构函数。



```c++
{
    QPushButton quit("Quit");
    QWidget window;
    quit.setParent(&window);
}
```

程序会崩溃，作为父对象的 window 会首先被析构，因为它是最后一个创建的对象。在析构过程中，它会调用子对象列表中每一个对象的

析构函数，也就是说， quit 此时就被析构了。在 window 析构之后，quit 也会被析构，这时候已经是第二次调用 quit 的析构函数了，C++ 

不允许调用两次析构函数。



#### 1.3 Qt的窗口坐标系

坐标系：以左上角为原点（0,0），X向右增加，Y向下增加。

对于嵌套窗口，其坐标是**相对于父窗口**来说的。





### 2.信号和槽机制

当某个事件发生之后，它就会发出一个信号（signal）。这种发出是没有目的的，类似广播。如果有对象对这个信号感兴趣，它就会使用连接（connect）函数，将想要处理的信号和自己的一个函数（称为槽（slot））绑定来处理这个信号。

也就是说，**当信号发出时，被连接的槽函数会自动被回调**。

#### 2.1 系统自带的信号和槽

connect函数最常用的一般形式：

```c++
connect(sender, signal, receiver, slot);
```

参数解释：

* sender：发出信号的对象

* signal：发送对象发出的信号

* receiver：接收信号的对象

* slot：接收对象在接收到信号之后所需要调用的函数（槽函数）



**关闭窗口示例**：

```c++
QPushButton * quitBtn = new QPushButton("关闭窗口",this);
connect(quitBtn,&QPushButton::clicked,this,&MyWidget::close);
```



#### 2.2 自定义信号和槽

**示例**：

```c++
//首先定义一个学生类和老师类：
//老师类中声明信号 饿了 hungry
signals:
       void hungury();

//学生类中声明槽   请客 treat
	public slots:
       void treat();

//在窗口中声明一个公共方法下课，这个方法的调用会触发老师饿了这个信号，而响应槽函数学生请客
void MyWidget::ClassIsOver()
{
    //发送信号
    emit teacher->hungury();
}

//学生响应了槽函数，并且打印信息
//自定义槽函数 实现
void Student::eat()
{
	qDebug() << "该吃饭了！";
}

//在窗口中连接信号槽
teacher = new Teacher(this);
student = new Student(this);

connect(teacher,&Teacher::hungury,student,&Student::treat);

//并且调用下课函数，测试打印出 “该吃饭了”
//自定义的信号 hungry带参数，需要提供重载的自定义信号和 自定义槽
void hungury(QString name);  自定义信号
void treat(QString name );    自定义槽
    
//但是由于有两个重名的自定义信号和自定义的槽，直接连接会报错，所以需要利用函数指针来指向函数地址， 然后在做连接
void (Teacher:: * teacherSingal)(QString) = &Teacher::hungury;
void (Student:: * studentSlot)(QString) = &Student::treat;
connect(teacher,teacherSingal,student,studentSlot);

```



自定义信号和槽注意事项：

* 发送者和接收者都需要是QObject子类（槽函数为全局函数、Lambda表达式等无需接受者的时候除外）
* 信号和槽返回值都是void
* 信号只需要声明不需要实现，槽函数需要声明和实现
* 槽函数作为普通成员函数时，会受到public、protected、private影响
* 任何成员函数，static函数，全局函数和Lambda表达式都可以作为槽函数
* 信号和槽要求其参数类型一致，若不一致，允许槽函数的参数比信号的参数少



#### 2.3 信号槽的拓展

* 一个信号可以和多个槽相连

  槽会一个接一个被调用，但**调用顺序不确定**

* 多个信号可以连接到同一个槽

  任意一个信号发出，该槽就会被调用

* 一个信号可以连接到另一个信号

* 槽可以被取消连接

  当一个对象被delete后，Qt自动取消连接到该对象上的槽，disconnect关键字也可以取消连接槽



#### 2.4 Qt4版本的信号槽写法

**示例**：

```c++
connect(zt,SIGNAL(hungry(QString)),st,SLOT(treat(QString)));
```

**SIGNAL和SLOT这两个宏，将两个函数名转换成了字符串**

缺点：编译器不检查字符串是否匹配，而在运行时报错，增加程序不稳定性



#### 2.5 Lambda表达式

**用于定义并创建匿名的函数对象**

基本结构：

```c++
[capture](parameters) mutable -> return-type
{
	statement
}
```

```c++
[函数对象参数](操作符重载函数参数)mutable ->返回值{函数体}
```

1. 函数对象参数

   [ ]标识一个Lambda的开始，函数对象参数只能使用那些到定义Lambda为止时Lambda所在作用范围内可见的局部变量

   包括Lambda所在类的this

   * 空，没有使用任何函数对象参数
   * =，函数体内可以使用Lambda所在作用范围内所有可见的局部变量（**值传递**）
   * &，函数体内可以使用Lambda所在作用范围内所有可见的局部变量（**引用传递方式**）
   * this，函数体内可以使用Lambda所在类中的成员变量。
   * a，将a按值进行传递。要修改传递进来的a的拷贝，可以添加 `mutable` 修饰符
   * &a，将a按引用进行传递
   * a，&b，将a按值进行传递，b按引用进行传递
   * =，&a，&b，除a和b按引用进行传递，其他参数按值进行传递

2. 操作符重载函数参数

   ​	标识重载的()操作符的参数，参数按值传递（a,b）和按引用传递（&a,&b）

3. 可修改标识符

   ​	mutable，修改按值拷贝进来的参数的值，不是修改值本体

**示例**：

```c++
QPushButton * myBtn = new QPushButton (this);
QPushButton * myBtn2 = new QPushButton (this);
myBtn2->move(100,100);
int m = 10;

connect(myBtn,&QPushButton::clicked,this,[m] ()mutable { m = 100 + 10; qDebug() << m; });

connect(myBtn2,&QPushButton::clicked,this,[=] ()  { qDebug() << m; });

qDebug() << m;

```

4. 函数返回值

   -> 返回值类型

5. 函数体

   { }标识函数体的实现



### 3.QMainWindow

![QMainWindow.jpg](https://i.loli.net/2021/09/27/MXrp4asgUFY2JWc.jpg)

#### 3.1 菜单栏

一个主窗口最多只有一个菜单栏，位于主窗口顶部，主窗口标题栏下面

**头文件 ： #include\<QMenuBar\> **

* 创建菜单栏，通过 `QMainWindow`类的 `menubar()`函数获取主窗口菜单栏指针

  ```c++
  QMenuBar* bar = menuBar();
  ```
  
* 将菜单栏放置在窗口中

  ```c++
  setMenuBar(bar);
  ```

* 创建菜单，调用`QMenuBar`的成员函数`addMenu`来添加菜单

  ```c++
  QMenu* fileMenu = bar->addMenu("文件");
  ```
  
* 创建菜单项，调用 `QMenu`的成员函数`addAction`来添加菜单项

    ```c++
    QAction* newAction = fileMenu->addAction("新建");
    ```



#### 3.2 工具栏

主窗口上可以有多个工具栏。

**头文件 ： #include\<QToolBar\> **

* 直接调用`QMainWindow`类中的`addToolBar`获取主窗口的工具条对象，每增加一条工具条都需要调用一次该函数

  ```c++
  QToolBar* toolbar = new QToolBar(this);
  addToolBar(Qt::LeftToolBarArea, toolbar);
  ```

* 设置只允许左右停靠

  ```c++
  toolbar->setAllowedAreas(Qt::LeftToolBarArea | Qt::RightToolBarArea); //Qt::AllToolBarAreas	以上四个位置都可停靠
  ```

* 设置浮动

  ```c++
  toolbar->setFloatable(false); //工具条不可以悬浮
  ```

* 设置移动（总开关）

  ```c++
  toolbar->setMovable(false); //工具条不可移动, 只能停靠在初始化的位置上
  ```

* 插入属于工具条的动作，即在工具条上上添加操作，通过`QToolBar`类的`addAction`函数添加

  ```c++
  toolbar->addAction(newAction)
  ```

* 添加分割线

  ```c++
  toolbar->addSeparator();
  ```

* 工具栏中添加控件

  ```c++
  QPushButton* btn = new QPushButton("Button",this);
  toolbar->addWidget(btn);
  ```



#### 3.3 状态栏

状态栏最多只能有一个。

**头文件 ： #include\<QStatusBar\> **

派生自QWidget类，使用方法与QWidget类似，QStatusBar类常用成员函数:

```c++
//添加小部件
void addWidget(QWidget * widget, int stretch = 0)
//插入小部件
int	insertWidget(int index, QWidget * widget, int stretch = 0)
//删除小部件
void removeWidget(QWidget * widget)
```

* 创建状态栏

  ```c++
  QStatusBar* stbar = statusBar();
  ```

* 设置到窗口中

  ```c++
  setStatusBar(stbar);
  ```

* 放标签控件

  **头文件 ： #include\<QLabel\> **

  ```c++
  QLabel* label1 = new QLabel("提示信息",this);
  stbar->addWidget(label1);
  
  QLabel* label2 = new QLabel("右侧提示信息",this);
  stbar->addPermanentWidget(label2);
  ```



#### 3.4 铆接部件

铆接部件 QDockWidget，也称浮动窗口，可以有多个。

**头文件 ： #include\<QDockWidget\> **

* 创建并添加铆接部件

  ```c++
  QDockWidget* dockWidget = new QDockWidget("浮动",this);
  addDockWidget(Qt::RightDockWidgetArea,dockWidget);
  ```

* 设置后期停靠区域，只允许左右停靠

  ```c++
  dockWidget->setAllowedAreas(Qt::LeftDockWidgetArea|Qt::RightDockWidgetArea);
  ```



#### 3.5 核心部件（中心部件）

中心显示的部件都可以作为核心部件，例如一个记事本文件，可以利用QTextEdit做核心部件

**头文件 ： #include\<QTextEdit\> **

* 设置中心部件，只能一个

  ```c++
  QTextEdit* edit = new QTextEdit(this);
  setCentralWidget(edit);
  ```

  

#### 5.6 资源文件

Qt 资源系统是一个跨平台的资源机制，用于将程序运行时所需要的资源以二进制的形式存储于可执行文件内部。

将资源以资源文件形式存储，它是会编译到可执行文件内部

**添加**：

工程文件右键 -->  添加新文件... --> Qt --> Qt Resource File -->  选择 --> 设置文件名

添加前缀，添加文件



