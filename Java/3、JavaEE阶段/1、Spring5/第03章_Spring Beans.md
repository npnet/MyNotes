# 第03章_Spring Beans

---

## 一、概念与定义

### 1、什么是Spring beans?

**Bean:** 在Spring中，构成应用程序主干并由Spring [IoC](https://so.csdn.net/so/search?q=IoC&spm=1001.2101.3001.7020)容器管理的对象称为bean。Bean是由Spring IoC容器实例化，组装和以其他方式管理的对象。否则，bean仅仅是应用程序中许多对象之一。Bean及其之间的依赖关系反映在容器使用的配置元数据中。

* Spring Beans是构成Spring应用核心的Java对象，这些对象由Spring IoC容器实例化、组装、管理。这些对象通过容器中配置的原数据创建，例如使用XML文件中定义的创建
* 在Spring中创建的beans都是单例的beans。在bean标签中有一个属性为“singleton”，如果设为true