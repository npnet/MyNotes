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

  





### 2. go高阶





### 3. go modules模块管理

