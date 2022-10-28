# Spring5框架

---

## 一、Spring框架概述

1、spring框架是一个轻量级的开源的java EE开发框架。

2、spring框架解决企业应用开发的复杂性。

3、spring有两个核心：IOC与AOP

* IOC：控制反转，把创建对象过程交给Spring管理。

* AOP：面向切面编程，不修改源代码情况下，进行功能增强。

### 1、Spring特点

1. 方便解耦，简化开发
2. aop支持
3. 方便程序测试
4. 方便集成各种框架
5. 降低Java api使用难度
6. 方便进行事务处理

### 2、Spring版本选择

<img src="images/image-20221027173138359.png" alt="image-20221027173138359" style="zoom:80%;" />

### 3、入门案例

1. 下载Spring
2. idea新建普通Java工程
3. 导入Spring的jar包（bean、core、context、expresstion + commons-logging）
4. 写代码

![](images/image-20221027173550670.png)

* **创建普通类，类里面创建普通方法**

  ```java
  public class User{
      public void add(){
           system.out.println("add....");
       }
  }
  ```

* **创建Spring配置文件，在配置文件中创建对象，新建bean.xml配置文件**

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <!--配置User对象创建-->
      <bean id="user" class="com.spring5.User"></bean>
  </beans>
  ```

* **测试**

  ```java
  public class TestSpring5{
      @Test
      public void testAdd(){
          //1、加载spring配置文件
          ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
          //2、获取配置的创建对象
          User user = context.getBean("user",User.class);
          //3、测试
          System.out.println(user);
          user.add();
      }
  }
  ```



## 二、IOC容器

### 1、IOC底层原理

控制反转，目的：降低耦合度，高内聚，低耦合。把对象创建和对象之间的调用过程，交给Spring进行管理。

xml解析、工厂模式、反射

![image-20221027174035473](images/image-20221027174035473.png)

![image-20221027174059267](images/image-20221027174059267.png)



### 2、IOC接口（BeanFactory）

* IOC思想基于IOC容器完成，IOC容器底层就是对象工厂
* Spring提供了IOC容器实现的两种方式：两个接口
  * BeanFactory：IOC容器基本实现，是Spring内部使用接口，不提供开发人员使用。加载配置文件时不会创建对象，使用对象时才会创建对象（懒汉式加载对象）
  * ApplicationContext：BeanFactory的子接口，提供更多更强大的功能，一般供开发人员进行使用。加载配置文件时就创建对象（饿汉式加载对象）
* ApplicationContext接口实现类
  * FileSystemXmlApplicationContext("盘符路径（绝对路径）")
  * ClassPathXmlApplicationContext("src目录下类路径")



### 3、IOC操作Bean管理

1. 什么是Bean管理
   * Bean管理指的是两个操作
   * Spring创建对象
   * Spring注入属性
2. Bean管理操作的两种方式
   * 基于xml配置文件方式实现
   * 基于注解方式实现



### 4、IOC操作和Bean管理（基于xml）

#### 4.1 基于xml方式创建对象

1. 在Spring配置文件中，使用Bean标签，标签里面添加对应属性，就可以实现对应对象创建

2. 在Bean标签有很多属性，常压的属性：id、class、name

3. 创建对象的时候，默认也是执行无参数构造方法

   ```xml
   <!--配置User对象创建-->
   <bean id="user" class="com.spring5.User"></bean>
   ```

#### 4.2 基于xml方式注入属性

DI：依赖注入，就是注入属性

#### 4.3 第一种注入方式：使用set方法进行注入

1. 创建类，定义属性和对应的set方法

   ```java
   public class Book {
       private String bname;
       private String bauthor;
   
       public void setBname(String bname) {
           this.bname = bname;
       }
       public void setBauthor(String bauthor) {
           this.bauthor = bauthor;
       }
       public static void main(String[] args) {
           Book book = new Book();
           book.setBname("WeiSanJin");
       }
   }
   ```

2. 在Spring配置文件配置对象创建，配置属性注入

   ```xml
   <!--set方法注入属性-->
   <bean id="book" class="com.spring5.Book">
       <!--使用property完成属性注入
           name：类里面属性名称
           value：向属性注入的值
       -->
       <property name="bname" value="WeiSanJin"></property>
       <property name="bauthor" value="WeiSanJin"></property>
   </bean>
   ```



#### 4.4 第二种注入方式：使用有参数构造进行注入

1. 创建类，定义属性，创建属性对应有参数构造方法

   ```java
   public class Orders {
       private String oname;
       private String address;
   
       public Orders(String oname, String address) {
           this.oname = oname;
           this.address = address;
       }
   }
   ```

2. 在Spring配置文件中进行配置

   ```xml
   <bean id="orders" class="com.spring5.Orders">
       <constructor-arg name="oname" value="WeiSanJin"></constructor-arg>
       <constructor-arg name="address" value="WeiSanJin"></constructor-arg>
   </bean>
   ```

3. 测试

   ```java
   @Test
   public void TestOrder(){
       //1.加载Spring配置文件
       ApplicationContext context = new ClassPathXmlApplicationContext("base1.xml");
   
       //2.获取配置创建的对象
       Orders orders = context.getBean("orders",Orders.class);
   
       System.out.println(orders.toString());
   }
   ```



#### 4.5 p名称空间注入（了解）

使用p名称空间注入，可以简化基于xml配置方式

1. 添加p名称空间在配置文件种

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:p="http://www.springframework.org/schema/p"
   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   ```

2. 进行属性注入，在bean标签里面进行操作

   ```xml
   <bean id="book" class="com.spring5.Book" p:bname="WeiSanJin" p:bauthor="WeiSanJin"</bean>
   ```



### 5、IOC操作Bean管理（xml注入其他类型属性）

#### 5.1 字面量

1. null值

   ```xml
   <bean id="book" class="com.spring5.Book">
       <property name="bname" value="WeiSanJin"></property>
       <property name="bauthor" value="WeiSanJin"></property>
       <property name="address">
           <null></null>
       </property>
   </bean>
   ```

2. 属性值包含特殊符号

   ```xml
   //方法一：转义字符
   <property name="address" value="&lt;北京&dt;"></property>
   //方法二：CDATA
   <property name="address">
       <value>
       <![CDATA[<北京>]]>
       </value>
   </property>
   ```



#### 5.2 注入外部bean（使用引用，注入其他类的对象）

1. 创建两个类service类和dao类

2. 在service调用dao类的方法

3. 在spring配置文件中进行配置

   ```java
   public interface UserDao {
       public void updata();
   }
   
   public class UserDaoImpl implements UserDao{
   
       @Override
       public void updata() {
           System.out.println("dao updata.......");
       }
   }
   
   public class UserService {
       // 创建UserDao类型属性，生成set方法
       private UserDao userDao;
       public void setUserDao(UserDao userDao) {
           this.userDao = userDao;
       }
   
       public void add(){
           System.out.println("service add......");
           //原理注入方式
           //UserDao userDao = new UserDaoImpl();
           //userDao.updata();
       }
   }
   
   // 第二步写配置文件xml文件
   // service和dao对象创建
   <bean id="userService" class="com.spring5.service.UserService">
       <!-- 注入UserDao对象 
       name属性：类里面属性名称
       ref属性：创建userDao对象bean标签id值
       -->
       <property name="userDao" ref="userDaoImpl"></property>
       </bean>
       // 配置dao对象
       <bean id="userDaoImpl" class="com.spring5.dao.UserDaoImpl"></bean>
   ```



#### 5.3 注入属性  -  内部bean和级联赋值

```java
public class Dept {
    private String dname;

    public void setDname(String dname) {
        this.dname = dname;
    }

    @Override
    public String toString() {
        return "Dept{" +
                "dname='" + dname + '\'' +
                '}';
    }
}
```

```java
public class Emp {
    private String ename;
    private String genfer;
    private Dept dept;

    public void setEname(String ename) {
        this.ename = ename;
    }

    public void setGenfer(String genfer) {
        this.genfer = genfer;
    }

    public void setDept(Dept dept) {
        this.dept = dept;
    }

    @Override
    public String toString() {
        return "Emp{" +
                "ename='" + ename + '\'' +
                ", genfer='" + genfer + '\'' +
                ", dept=" + dept +
                '}';
    }
}
```

```xml
<!--级联赋值-->
<bean id="emp" class="com.spring5.bean.Emp">
    <!--设置两个普通属性-->
    <property name="ename" value="WeiSanJin"></property>
    <property name="genfer" value="WeiSanJin"></property>
    <property name="dept">
        <bean id="dept" class="com.spring5.bean.Dept">
            <property name="dname" value="保安部"></property>
        </bean>
    </property>
</bean>
```



#### 5.4 注入属性  -  级联赋值

```xml
<!--级联赋值-->
<bean id="emp" class="com.spring5.bean.Emp">
    <!--设置两个普通属性-->
    <property name="ename" value="WeiSanJin"></property>
    <property name="genfer" value="WeiSanJin"></property>
    <!--级联赋值-->
    <property name="dept" ref="dept"></property>
</bean>
<bean id="dept" class="com.spring5.bean.Dept">
    <property name="dname" value="财务部"></property>
</bean>
```



### 6、IOC操作Bean管理（xml注入集合属性）

* 注入数据类型属性
* 注入List集合类型属性
* 注入Map集合类型属性

1. 创建类，定义数据，list，map，set类型属性，生成对应set方法

   ```java
   public class Stu {
       // 1. 数组类型属性
       private String[] courses;
   
       // 2. list集合类型属性
       private List<String> list;
   
       // 3. map集合类型属性
       private Map<String,String> maps;
   
       // 4. set集合类型属性
       private Set<String> sets;
   
       public void setCourses(String[] courses) {
           this.courses = courses;
       }
   
       public void setList(List<String> list) {
           this.list = list;
       }
   
       public void setMaps(Map<String, String> maps) {
           this.maps = maps;
       }
   
       public void setSets(Set<String> sets) {
           this.sets = sets;
       }
       @Override
       public String toString() {
           return "Stu{" +
                   "courses=" + Arrays.toString(courses) +
                   ", list=" + list +
                   ", maps=" + maps +
                   ", sets=" + sets +
                   '}';
       }
   }
   ```

2. 在Spring配置文件进行配置

   ```xml
   <!--1. 集合类型属性注入-->
   <bean id="stu" class="com.spring5.collectionytpe.Stu">
       <!--数组类型属性注入 -->
       <property name="courses">
           <array>
               <value>Java课程</value>
               <value>数据库课程</value>
           </array>
       </property>
       <!--list类型属性注入 -->
       <property name="list">
           <list>
               <value>张三</value>
               <value>小三</value>
           </list>
       </property>
       <!--map类型属性注入 -->
       <property name="maps">
           <map>
               <entry key="Java" value="java"></entry>
               <entry key="PHP" value="php"></entry>
           </map>
       </property>
       <!--set类型属性注入 -->
       <property name="sets">
           <set>
               <value>Mysql</value>
               <value>Redis</value>
           </set>
       </property>
   </bean>
   ```

3. 测试

   ```java
   @Test
   public void TestStu(){
       //1.加载Spring配置文件
       ApplicationContext context = new ClassPathXmlApplicationContext("base5.xml");
   
       //2.获取配置创建的对象
       Stu stu = context.getBean("stu", Stu.class);
   
       System.out.println(stu.toString());
   }
   ```

4. 在集合里设置对象类型值

   ```java
   // 1.创建课程类
   public class Course {
       private String cname;
   
       public void setCname(String cname) {
           this.cname = cname;
       }
   }
   // 2.创建学生类
   public class Stu {
       // 学生所学多门课程
       private List<Course> courseList;
           public void setCourseList(List<Course> courseList) {
           this.courseList = courseList;
       }
   }
   ```

   ```xml
   <!-- xml配置-->
   <!-- 注入list集合类型，值是对象-->
   <bean>
       <property name="courseList">
           <list>
               <ref bean="course1"></ref>
               <ref bean="course2"></ref>
           </list>
       </property>
   </bean>
   
   <!-- 创建多个course对象-->
   <bean id="course1" class="com.spring5.collectionytpe.Course">
       <property name="cname" value="String"></property>
   </bean>
   <bean id="course2" class="com.spring5.collectionytpe.Course">
       <property name="cname" value="String"></property>
   </bean>
   ```

5. 把集合注入部分提取出来

   1. 在spring配置文件中引入名称空间util

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:util="http://www.springframework.org/schema/util"
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                                 http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
      ```

   2. 使用util标签完成list集合提取

      ```xml
      <!--1 提取list集合类型属性注入-->
      <util:list id="bookList">
          <value>三国演义</value>
          <value>水浒传</value>
          <value>西游记</value>
          <value>红楼梦</value>
      </util:list>
      
      <!--2 提取list集合类型属性使用-->
      <bean id="book" class="com.spring5.collectionytpe.Book">
          <property name="list" ref="bookList"></property>
      </bean>
      ```

      

### 7、IOC操作Bean管理（FactoryBean）

Spring有两种类型bean，一种普通bean，另一种工厂bean(FactoryBean)

1. 普通bean在配置文件中，定义bean类型就是返回类型
2. 工厂bean在配置文件中定义bean类型可以和返回类型不一样

* 第一步创建类，让这个类作为工厂Bean，实现接口FactoryBean

* 第二步实现接口里的方法，在实现方法中定义返回的bean类型

  ```java
  //实现接口类
  public class MyBean implements FactoryBean<Course>{
      @override
      public Course getObject() throws Exceptions{
              Course  course = new Course();
              course.setCourse("java");
              return course;
      }
      @override
      public Class<?> getObjectType(){
              return NULL;
      }
      @override
      public Boolean isSingleton(){
              return false;
      }
  }
  //配置类
  <bean id = "mybean" class="com.zhh.entity.MyBean"></bean>
  //测试类
  @test
  public void test2(){
      ApplicationContext application  = new 	classPathXpthXmlApplicationContext("bean4.xml");
  	Course course =  context.getBean("MyBean",Course.class);
  }
  ```



### 8、IOC操作Bean管理（bean作用域）

1. 在Spring里，设置创建Bean实例是单实例还是多实例
2. 在Spring里，默认设置创建Bean实例是单实例
3. 如何设置单实例还是多实例

Spring配置文件bean标签里scope属性用于设置单实例还是多实例

scope属性值：

* 默认值：singleton，表示单实例对象
* prototype，表示多实例对象

```xml
//配置
<bean id="book" class="com.zhh.entity.Book" scope="prototype">
    <property name="list" ref = "booklist"></property>
</bean>
```

4. singleton与prototype区别

   1. singleton表示单实例，prototype表示多实例

   2. 设置scope是singleton时，加载Spring配置文件时会创建单实例对象

      设置scope是prototype时，不是加载Spring配置文件时就创建对象，而是在调用getBean方法时创建多实例对象



### 9、IOC操作Bean管理（生命周期）

#### 9.1 生命周期

从对象创建到对象销毁的过程

#### 9.2 bean生命周期

1. 通过构造器创建bean实例（无参数构造）
2. 为bean的属性设置值和对其他bean引用（调用set方法）
3. 调用bean的初始化方法（需要进行配置）
4. bean可以使用了（对象获取到了）
5. 当容器关闭时，调用bean的销毁方法（需要进行配置销毁方法）

```java
public class Orders {
    private String oname;

    public Orders() {
        System.out.println("第一步 执行无参构造创建bean实例");
    }

    public void setOname(String oname) {
        this.oname = oname;
        System.out.println("第二步 调用set方法设置属性");
    }

    public void initMethod(){
        System.out.println("第三步 执行初始化的方法");
    }

    public void destroyMethod(){
        System.out.println("第五步 执行销毁的方法");
    }

    @Override
    public String toString() {
        return "Orders{" +
                "oname='" + oname + '\'' +
                '}';
    }
}
```

```xml
<bean id="orders" class="com.spring5.bean.Orders" init-method="initMethod" destroy-method="destroyMethod">
    <property name="oname" value="手机"></property>
</bean>
```

```java
@Test
public void test1(){
    //1.加载Spring配置文件
    ApplicationContext context = new ClassPathXmlApplicationContext("bean7.xml");

    //2.获取配置创建的对象
    Orders orders = context.getBean("orders", Orders.class);

    System.out.println(orders.toString());

    //手动销毁bean实例
    ((ClassPathXmlApplicationContext) context).close();
}
```

#### 9.3 bean的后置处理器，bean生命周期有七步

1. 通过构造器创建Bean实例（无参数构造）
2. 为bean的属性设置值和对其他bean引用（调用set方法）
3. 把bean实例传递bean后置处理器的方法postProcessBeforeInitialization
4. 调用bean的初始化的方法（需要进行配置）
5. 把bean实例传递bean后置处理器的方法postProcessAfterInitialization
6. bean可以使用了（对象获取到了）
7. 当容器关闭时，调用bean的销毁方法（需要进行配置销毁的方法）

```java
//MyBean.java
public class MyBeanPost implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之前执行的方法");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之后执行的方法");
        return bean;
    }
}
```

```xml
<!-- 配置后置处理器-->
<!--在该配置文件下的bean都会引用该后置处理器-->
<bean id="myBean" class="com.java.spring5.period.MyBeanPost"></bean>
```



### 10、IOC操作Bean管理（xml自动装配）

* 什么是自动装配

  根据指定装配规则（属性名称或者属性类型），Spring自动将匹配的属性值进行注入

* 演示自动装配过程

  ```xml
  <!--实现自动装配
      bean标签属性autowire，配置自动装配
      autowire属性常用两个值：
          byName根据属性名称注入，注入值bean的id值和类属性名称一样
          byType根据属性类型注入
  -->
  <bean id="emp" class="com.spring5.autowire.Emp" autowire="byName"></bean>
  <bean id="dept" class="com.spring5.autowire.Dept"></bean>
  ```




### 11、IOC操作Bean管理（外部属性文件）

#### 11.1 直接配置数据库信息

1. 配置德鲁伊连接池
2. 引入德鲁伊连接池依赖jar包

```xml
<!--直接配置连接池-->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql://localhost:3306/test"></property>
    <property name="username" value="root"></property>
    <property name="password" value="123456"></property>
</bean>
```

#### 11.2 引入外部属性文件配置数据库连接池

1. 创建外部属性文件,properties格式文件，写数据库信息

   ```properties
   prop.driverClass=com.mysql.jdbc.Driver
   prop.url=jdbc:mysql://localhost:3306/test
   prop.userName=root
   prop.password=123456
   ```

2. 把外部properties属性文件引入到Spring配置文件中

   引入context名称空间

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
                              http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
   ```

3. 在Soring配置文件中使用标签引入外部属性文件

   ```xml
   <!--    引入外部属性文件-->
   <context:property-placeholder location="classpath:jdbc.properties"></context:property-placeholder>
   
   <!--    配置连接池-->
   <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
       <property name="driverClassName" value="${prop.driverClass}"></property>
       <property name="url" value="${prop.url}"></property>
       <property name="username" value="${prop.userName}"></property>
       <property name="password" value="${prop.password}"></property>
   </bean>
   ```



### 10、IOC操作Bean管理（基于注解方式）

#### 10.1 什么是注解

* 格式：@注解名称(属性名=属性值，属性名=属性值)
* 使用注解：注解作用在类，方法，属性上
* 使用目的：简化xml配置

#### 10.2 Spring针对Bean管理中创建对象提供注解

* @Component：普通用法
* @Service：用于service业务逻辑层
* @Controller：用于web层
* @Repository：用于DAO持久层

#### 10.3 基于注解方式实现对象创建例子

1. 引入依赖

   **apring-aop-5.2.6.RELEASE**

2. 引入context名称空间

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
                              http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
   ```

3. 开启组件扫描

   1、如果扫描多个包，多个包使用逗号隔开

   2、扫描包的上层目录

   ```xml
   <context:component-scan base-package="com.atguigu"></context:component-sacn>
   ```

4. 创建类，在类上面添加创建对象注解

   ```java
   @Component(value = "User")
   public class User {
       public void add(){
           System.out.println("service add......");
       }
   }
   ```

5. 测试

   ```java
   @Test
   public void testService(){
       //1.加载Spring配置文件
       ApplicationContext context = new ClassPathXmlApplicationContext("base9.xml");
   
       //2.获取配置创建的对象
       User user = context.getBean("User", User.class);
   
       user.add();
   }
   ```



#### 10.4 开启组件扫描细节配置

```xml
<!--示例1
        use-default-filters="false" 表示现在不使用默认filter，自己配置fillter
        context:include-filter,设置扫描哪些内容
    -->
<context:component-scan base-package="com.spring5" use-default-filters="false">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>

<!--示例2
        下面配置扫描包所有内容
        context:exclude-filter：设置哪些内容不进行扫描
    -->
<context:component-scan base-package="com.spring5">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```







