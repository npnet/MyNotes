## Go基础

### 1. go语法

#### 1.1 main函数

```go
package main  //程序的包名
import "fmt"
func main() {
	/* 简单的程序 万能的hello world */
	fmt.Println("Hello Go")
}
```

* golang中的表达式，加`;`和不加；都可以，一般不加

* 函数的`{`一定是和函数名在同一行，否则编译错误

  

#### 1.2 变量

* 局部变量声明

  ```go
  //方法一: 声明一个变量，默认值是0
  var a int
  
  //方法二: 声明一个变量，初始化一个值
  var b int = 100
  
  //方法三: 在初始化时省去数据类型，通过值自动匹配变量数据类型
  var c = 100
  
  //方法四: (常用)省去var关键字，直接自动匹配
  a := 100
  ```

* 全局变量的声明

  **方法四不支持全局**

* 多变量的声明

  ```go
  //单行写法
  var xx,yy int =100,200
  var kk,ll = 100,"Acdef"
  
  //多行写法
  var(
      vv int = 100
      jj bool = true
  )
  ```

  

#### 1.3 常量和iota

* 常量

  ```go
  const a int = 10
  const(
  	a = 10
      b = 20
  )
  ```

* iota

  ```go
  //const来定义枚举类型
  const(
      //可以在const()添加一个关键字，每行的iota都会累加1，第一行的iota默认值是0
      BEIJING = 10*iota //iota=0
      SHANGHAI		 //iota=1 SHANGHAI=10
      SHENZHEN		 //iota=2 SHENZHEN=20
  )
  ```



#### 1.4 函数

* 多返回值

  ```go
  //返回多个值，匿名
  func foo2(a string, b int)(int,int){
      fmt.Println("a=",a)
      fmt.Println("b=",b)
      
      return 666,777
  }
  ```

  ```go
  //返回多个返回值，有形参名称的
  func foo3(a string, b int)(r1 int, r2 int){
      //r1 r2属于foo3形参，初始化默认的值为0
      //r1 r2作用域空间是foo3整个函数体的{}空间
      fmt.Println("r1=",r1)
      fmt.Println("r2=",r2)
      
      //给有名称的返回值赋值
      r1 = 1000
      r2 = 2000
      
      return
  }
  ```

  ```go
  func foo4(a string, b int)(r1, r2 int){
      .......
  }
  ```

* init函数和import导包

  1. init函数

     ![image-20220615151751390](https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220615151751390.png)

     制作包的时候，项目路径如下：

     ```shell
     $GOPATH/GolangStudy/5-init/
     ├── lib1/
     │ └── lib1.go
     ├── lib2/
     │ └── lib2.go
     └── main.go
     ```

     `main.go`

     ```go
     package main
     import (
     	"GolangStudy/init/lib1"
         "GolangStudy/init/lib2"
     )
     
     func main(){
         lib1.Lib1Test()
         lib2.Lib2Test()
     }
     ```

     `lib1.go`

     ```go
     package lib1
     import "fmt"
     
     //当前lib1包提供的API
     func Lib1Test(){
         fmt.Println("lib1 Lib1Test()...")
     }
     
     func init(){
         fmt.Println("lib1 init()...")
     }
     ```

     `lib2.go`

     ```go
     package lib2
     import lib2
     
     //当前lib2包提供的API
     func Lib2Test(){
         fmt.Println("lib2 Lib2Test()...")
     }
     
     func init(){
         fmt.Println("lib2 init()...")
     }
     ```

  2. import导包

     ```go
     //给fmt包起一个别名，匿名，无法使用当前包的方法，但会执行当前包内部的init()方法
     import _ "fmt"
     
     //给fmt包起一个别名aa，aa.Printlb()直接调用
     import aa "fmt"
     
     //将当前fmt包中的全部方法，导入到当前本包的作用域，fmt包中的全部方法可以直接使用API来调用，不需要fmt.API来调用
     import . "fmt"
     ```



#### 1.4 指针

**示例1**：

```go
package main
import "fmt"
func changeValue(p int){
    p = 10
}
func main(){
    var a int = 1
    changeValue(a)
    fmt.Println("a=",a)
}
```

![image-20220615163841637](https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220615163841637.png)



**示例2**：

```go
package main
import "fmt"
func changeValue(p *int){
    *p = 10
}
func main(){
    var a int = 1
    changeValue(&a)
    fmt.Println("a=",a)
}
```

![image-20220615164040419](https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220615164040419.png)



#### 1.5 defer

* defer的执行顺序

  ![image-20220615164436189](https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220615164436189.png)

  执行顺序：**fun3() --> fun2() --> fun1()**

* defer 和 return 语句执行顺序

  ```go
  package main
  import "fmt"
  func deferFunc() int{
      fmt.Println("defer func called...")
      return 0
  }
  
  func returnFunc() int{
      fmt.Println("return func called...")
      return 0
  }
  
  func returnAndDefer() int{
      defer deferFunc()
      return returnFunc()
  }
  
  func main(){
      returnAndDefer()
  }
  ```

  执行顺序：**return之后的语句先执行，defer后的语句后执行**



#### 1.6 切片slice

* 数组

  数组的声明方式

  ```go
  var myArray1 [10]int
  myArray2 := [10]int{1,2,3,4}
  myArray3 := [4]int{11,22,33,44}
  ```

  1. 数组的长度是固件的

  2. 固定长度的数组在传参时，严格匹配数字类型

     

  函数传参方式

  ```go
  func printArray(myArray [4]int){
      //值传递，不会改变原数组
  }
  
  func printArray(myArray *[4]int){
      //引用传递，会改变原数组
  }
  ```



* slice(动态数组)

  声明方式

  ```go
  mArray := []int{1,2,3,4}
  ```

  ```go
  func printArrat(myArray []int){
      //引用传递
  }
  ```

  动态数组在传参上是引用传递，而且不同元素长度的动态数组他们的形参是一致的



* slice

  声明方式

  ```go
  //声明slice1是一个切片并且初始化，默认值是1,2,3，长度len是3
  slice1 := []int{1,2,3}
  
  //声明slice1是一个切片，但没有给slice分配空间
  var slice1 []int
  slice1 = make([]int, 3) //开辟三个空间，默认值是0
  
  //声明slice1是一个切片，同时给slice分配空间，3个空间，初始值是0
  var slice1 []int = make([]int, 3)
  
  //声明slice1是一个切片，同时给slice分配空间，3个空间，初始值是0，通过:=推导出slice是一个切片
  slice1 := make([]int, 3)
  ```

  使用方式

  1. 切片容量的追加

     **len 和 cap**

     切片的长度和容量不同，长度表示左指针到右指针之间的距离，容量表示左指针至底层数组末尾的距离

     ![image-20220616102816272](https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220616102816272.png)

     

     **切片的扩容机制**

     append时，如果长度增加后超过容量，则将容量增加2倍

     

  2. 切片的截取

     ```go
     package main
     import "fmt"
     
     func main() {
        /* 创建切片 */
        numbers := []int{0,1,2,3,4,5,6,7,8}  
        printSlice(numbers)
     
        /* 打印原始切片 */
        fmt.Println("numbers ==", numbers)
     
        /* 打印子切片从索引1(包含) 到索引4(不包含)*/
        fmt.Println("numbers[1:4] ==", numbers[1:4])
     
        /* 默认下限为 0*/
        fmt.Println("numbers[:3] ==", numbers[:3])
     
        /* 默认上限为 len(s)*/
        fmt.Println("numbers[4:] ==", numbers[4:])
     
        numbers1 := make([]int,0,5)
        printSlice(numbers1)
     
        /* 打印子切片从索引  0(包含) 到索引 2(不包含) */
        number2 := numbers[:2]
        printSlice(number2)
     
        /* 打印子切片从索引 2(包含) 到索引 5(不包含) */
        number3 := numbers[2:5]
        printSlice(number3)
     
     }
     
     func printSlice(x []int){
        fmt.Printf("len=%d cap=%d slice=%v\n",len(x),cap(x),x)
     }
     ```

     

#### 1.7 map

声明方式

```go
//第一种声明方式
//声明myMap是一种map类型，key是string，value是string
var myMap1 map[string]string
if myMap == nil{
    fmt.Println("myMap1是一个空map")
}

//在使用map前，需要先用make给map分配数据空间
myMap1 = make(map[string]string, 10)

myMap1["one"] = "Java"
myMap1["two"] = "C++"
myMap1["three"] = "Python"
```

```go
//第二种声明方式
myMap2 := make(map[int]string)
myMap2[1] = "Java"
myMap2[2] = "C++"
myMap2[3] = "Python"
```

```go
//第三种声明方式
myMap3 := map[string]string{
    "one":"Java",
    "two":"C++",
    "three":"Python",
}
```



#### 1.8 面向对象特征

* 封装

  ```go
  //如果类名首字母大写，表示其他包也能访问
  type Hero struct{
      //如果类的属性首字母大写，表示该属性是对外能够访问的，否则只能当前包的内部访问
      Name string
      Ad	 int
      level int
  }
  ```

  ```go
  func (h *Hero) Show(){
      fmt.Println("Name=",h.Name)
      fmt.Println("Ad=",h.Ad)
      fmt.Println("Level=",h.level)
  }
  
  func (h *Hero) GetName() string{
      return h.Name
  }
  
  func (h Hero) SetName(newName string){
      //h 是调用该方法的对象的一个副本（拷贝）
      h.Name = newName
  }
  
  func (h *Hero) SetName2(newName string){
      //h 是调用该方法的对象的一个引用
      h.Name = newName
  }
  ```

  类名、属性名、方法名、首字母大写表示对外(其他包)可以访问，否则只能在本包内访问

  

* 继承

  **父类**

  ```go
  type Human struct{
      name string
      sex string
  }
  ```

  ```go
  func (h *Human)Eat(){
      fmt.Println("Human.Eat()...")
  }
  
  func (h *Human)Walk(){
      fmt.Println("Human.Walk()...")
  }
  ```

  

  **子类**

  ```go
  type SuperMan struct{
      Human //SuperMan类继承了Human类的方法
      level int
  }
  ```

  ```go
  //重新定义父类的方法Eat()
  func (s *SuperMan) Eat(){
      fmt.Println("SuperMan.Eat()...")
  }
  
  //子类的新方法
  func (s *SuperMan) Fly(){
      fmt.Println("SuperMan.Fly()...")
  }
  
  func (s *SuperMan) Print(){
      fmt.Println("name=",s.name)
      fmt.Println("sex=",s.sex)
      fmt.Println("level=",s.level)
  }
  ```

  

  **定义子类**

  ```go
  //定义一个子类对象
  //s := SuperMan{Human{"li4","female"}, 88}
  var s SuperMan
  s.name = "li4"
  s.sex = "male"
  s.level = 88
  ```

  

* 多态

  **基本要素**

  1. 有一个父类(有接口)

     ```go
     //本质是一个指针
     type AnimalIF interface{
         Sleep()
         GetColor() string //获取动物的颜色
         GetType() string  //获取动物的种类
     }
     ```

  2. 有子类(实现了父类的全部接口方法)

     ```go
     //具体的类
     type Cat struct{
         color string //猫的颜色
     }
     
     func(c *Cat) Sleep(){
         fmt.Println("Cat is sleep")
     }
     
     func (c *Cat) GetColor() string{
         return c.color
     }
     
     func (c *Cat) GetType() string{
         return "Cat"
     }
     ```

  3. 父类类型的变量(指针)指向(引用)子类的具体数据变量

     ```go
     var animal AnimalIF //接口的数据类型，指针
     animal = &Cat{"Green"}
     animal.Sleep() //调用的就是Cat的Sleep()方法，多态的现象
     ```

  

  **interface**

  1. interface{}

     空接口

     int、string、float32、float64、struct ...都实现了interface{}

     可以用interface{}类型**引用任意的数据类型**

  2. 类型断言

     ```go
     //给interface{}提供"类型断言"的机制
     value,ok := arg.(string)
     if !ok{
         fmt.Println("arg is not string type")
     }else{
         fmt.Println("arg is string type,value = ",value)
         fmt.Printf("value type is %T\n",value)
     }
     ```

  

#### 1.9 反射

* 变量的结构

  ![image-20220617095017684](https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220617095017684.png)

* reflect包

  ```go
  //ValueOf用来获取输入参数接口中的数据的值，如果接口为空则返回0
  func ValueOf(i interface{}) Value{...}
  
  //TypeOf用来动态获取输入参数接口中的值的类型，如果接口为空则返回nil
  func TypeOf(i interface{}) Type{...}
  ```

  

#### 1.10 结构体标签

* 结构体标签的定义

  ```go
  type resume struct{
      Name string `info:"name" doc:"我的名字"`
      Sex string `info:"sex"`
  }
  ```

  ```go
  func findTag(str interface{}){
      t := reflect.TypeOf(str).Elem()
      
      for i:=0; i<t.NumField(); i++{
          taginfo := t.Field(i).Tag.Get("info")
          tagdoc := t.Field(i).Tag.Get("doc")
          fmt.Println("info:", taginfo, " doc:",tagdoc)
      }
  }
  ```

* 结构体标签应用

  json编解码

  orm映射关系



### 2. go高阶

#### 2.1 goroutine

```go
go func(){
    defer fmt.Println("A.defer")
    
    func(){
        defer fmt.Println("B.defer")
        //退出当前goroutine
        runtime.Goexit() //终止当前的goroutine
        fmt.Println("B")
    }()
    fmt.Println("A")
}()

go func(a int, b int) bool{
    fmt.Println("a=",a," b=",b)
    return true
}(10,10)
```

runtime.Goexit() 退出当前的goroutine



#### 2.2 channel

* channel的定义

  ![image-20220617110710427](https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220617110710427.png)

  ```go
  make(chan Type) //等价于make(chan Type, 0)
  make(chan Type, capacity)
  ```

* channel的使用

  ```go
  package main
  import "fmt"
  
  func main(){
      //定义一个channel
      c := make(chan int)
      
      go func(){
          defer fmt.Println("goroutine结束")
          fmt.Println("goroutine正在运行...")
          c<-666 //将666发送给c
      }()
      
      num := <-c
      fmt.Println("nnum=",num)
      fmt.Println("main foroutine结束")
  }
  ```

  

#### 2.3 无缓冲的channel

<img src="https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220617111859493.png" alt="image-20220617111859493" style="zoom:67%;" />

1. 第一步，两个goroutine都到达通道，但哪个都没有开始执行发送或者接收
2. 第二步，左侧goroutine将手伸进通道，模拟了向通道发送数据的行为。这时这个goroutine会在通道中被锁住，直到交换完成
3. 第三步，右侧goroutine将手放进通道，模拟了从通道里接收数据。这个goroutine一样会在通道中被锁住，直到交换完成
4. 第四、第五步，进行交换，并最终在第六步，两个goroutine都将手从通道拿出来，模拟了被锁住的goroutine得到释放。



#### 2.4 有缓冲的channel

<img src="https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220617112847939.png" alt="image-20220617112847939" style="zoom:67%;" />

1. 第一步，右侧的goroutine正在从通道接收一个值
2. 第二步，右侧的goroutine独立完成了接收值的动作，而左侧的goroutine正在发送一个新值到通道里
3. 第三步，左侧的goroutine还在向通道发送新值，右侧的goroutine正在从通道接收另外一个值。这个步骤的两个操作既不是同步的也不会互相阻塞
4. 第四步，所有的发送和接收都完成，而通道里还有几个值，也有一些空间可以存更多的值

**特点**

当channel已经满，再向里面写数据，就会阻塞

当channel为空，从里面取数据也会阻塞



#### 2.5 关闭channel

```go
func main(){
    c := make(chan int)
    
    go func(){
        for i:=0; i<5; i++{
            c<-i
        }
        
        //close可以关闭一个channel
        close(c)
    }
    
    for{
        //ok如果为true表示channel没有关闭，如果false表示channel已经关闭
        if data,ok := <-c;ok{
            fmt.Println(data)
        }else{
            break
        }
    }
    fmt.Println("Main Finished...")
}
```

* channel不像文件一样需要经常关闭，只有在确认没有任何发送数据或者显示结束range循环之类的，才去关闭channel

* 关闭channel后，无法向channel再发送数据（引发panic错误后导致接收立即返回0值）

* 关闭channel后，可以继续从channel接收数据
* 对于nil channel，无论收发都会被阻塞



#### 2.6 channel与range

```go
func main(){
    c := make(chan int)
    
    go func(){
        for i:=0; i<5; i++{
            c <-i
        }
        //close可以关闭一个channel
        close(c)
    }()
    
    //可以使用range来迭代不断操作channel
    for data := range c{
        fmt.Println(data)
    }
    fmt.Println("Main Finished...")
}
```



#### 2.7 channel与select

单流程下go只能监控一个channel的状态，select可以完成监控多个channel的状态

```go
select{
case <-chan1:
    //如果chan1成功读取到数据，则进行该case处理语句
case chan2<- 1:
    //如果成功向chan2写入数据，则进行该case处理语句
default:
    //如果上面都没有成功，则进入default处理流程
}
```

```go
func fibonacii(c , quit chan int){
    x,y := 1,1
    for{
        select{
        case c<-x:
            //如果c可写，则该case就会进来
            x = y
            y = x + y
        case <-quit:
            fmt.Println("quit")
            return
        }
    }
}

func main(){
    c := make(chan int)
    quit := make(chan int)
    
    //sub go
    go func(){
        for i:=0; i<10; i++{
            fmt.Println(<-c)
        }
        quit<-0
    }()
    
    //main go
    fibonacii(c,quit)
}
```

```shell
1
1
2
4
8
16
32
64
128
256
quit
```

select具备多路channel的监控状态功能



### 3. go modules模块管理

#### 3.1 Go Modules模式

* go mod命令

  ```go
  go mod init //生成go.mod文件
  go mod download //下载go.mod文件中指明的所有依赖
  go mod tidy //整理现有的依赖
  go mod graph //查看现有的依赖结构
  go mod edit //编辑go.mod文件
  go mod vendor //导出项目所有依赖到vendor目录
  go mod verify //校验一个模块是否被篡改过
  go mod why //查看为什么需要依赖某模块
  ```

* go mod环境变量

  * `GO111MODULE` : 是否开启go modules模式

  * `GOPROXY` : 项目的第三方依赖库的下载源地址

    建议设置国内地址：

    阿里云：<https://mirrors.aliyun.com/goproxy/>

    七牛云：<https://goproxy.cn,direct>

    direct: 用于指示Go回源到模块版本的源地址去抓取（如Github）

  * `GOSUMDB` : 用来校验拉取的第三方库是否是完整的

    默认国外网站，如已设置`GOPROXY`，该处不用设置

  * `GOPRIVATE` : 通过设置GOPRIVATE即可

    ```go
    go env -w GOPRIVATE="git.example.com,github.com/aceld/zinx
    //表示git.example.com和github.com/aceld/zinx是私有仓库，不会进行GOPROXY下载和校验
    
    go evn -w GOPRIVATE="*.example.com"
    //表示所有模块路径为example.com的子域名，比如git.example.com或者hello.example.com都不进行GOPROXY下载和校验
    ```

* 通过 go env来查看环境变量

  ```go
  go env -w GO111MODULE=on
  //或通过Linux export环境方式也可以
  ```



#### 3.2 使用Go Modules初始化项目

1. 开启Go Modules模块

   保证GO111MODULES=on

   ```go
   go env -w GO111MODULE=on
   
   export GO111MODULE=on
   //设置在用户启动脚本中
   //需要重新打开终端或执行source ~/.bashrc
   ```

2. 初始化项目

   * 任雯文件夹创建一个项目

     ```shell
     mkdir -p $HOME/modules_test
     ```

   * 创建go.mod文件，同时起当前项目的模块名称

     ```go
     go mod init  github.com/module_test
     ```

   * 生成go.mod文件

   * 在该项目中编写源代码

     如果源代码中依赖某个库（github.com/aceld/zinx/znet）

     手动down : 

     ```go
     go get github.com/aceld/zinx/znet
     ```

     自动down

   * go mod文件会添加一行新代码

     ```go
     require github.com/aceld/zinx v0.0.0-20200315073925-f09df55dc746 // indirect
     ```

     //indirect表示简介依赖

     直接依赖是znet包，间接依赖是zinx包

   * 会生成一个go.sum文件

     罗列当前项目直接或间接依赖的所有模块版本，保证今后项目依赖版本不会被篡改



#### 3.3 修改项目模块的版本依赖关系

```go
go mod edit -replace=zinx@v0.0.0-20200306023939bc416543ae24=zinx@v0.0.0-20200221135252-8a8954e75100
```

go mod文件就会被修改

```go
module github.com/aceld/modules_test
go 1.14
require github.com/aceld/zinx v0.0.0-20200306023939-bc416543ae24 // indirect
replace zinx v0.0.0-20200306023939-bc416543ae24 => zinx v0.0.0-20200221135252-8a8954e75100
```





