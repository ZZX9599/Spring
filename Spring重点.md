# Spring学习笔记

# 1:Spring核心概念

在Spring核心概念这部分内容中主要包含`IOC/DI`、`IOC容器`和`Bean`，那么问题就来了，这些都是什么呢?

## 1.1:IOC

IOC:控制反转

什么是控制反转呢？

* 使用对象时，由主动new产生对象转换为由==外部==提供对象，此过程中对象创建控制权由程序转移到外部，此思想称为控制反转。

  * 业务层要用数据层的类对象，以前是自己`new`的
  * 现在自己不new了，交给`别人[外部]`来创建对象
  * `别人[外部]`就反转控制了数据层对象的创建权
  * 这种思想就是控制反转
  * 别人[外部]指定是什么呢?继续往下学

  

Spring和IOC之间的关系是什么呢?

* Spring技术对IOC思想进行了实现
* Spring提供了一个容器，称为==IOC容器==，用来充当IOC思想中的"外部"
* IOC思想中的`别人[外部]`指的就是Spring的IOC容器



IOC容器的作用以及内部存放的是什么?

* IOC容器负责对象的创建、初始化等一系列工作，其中包含了数据层和业务层的类对象
* 被创建或被管理的对象在IOC容器中统称为==Bean==
* IOC容器中放的就是一个个的Bean对象



当IOC容器中创建好service和dao对象后，程序能正确执行么?

* 不行，因为service运行需要依赖dao对象
* IOC容器中虽然有service和dao对象
* 但是service对象和dao对象没有任何关系
* 需要把dao对象交给service,也就是说要绑定service和dao对象之间的关系



像这种在容器中建立对象与对象之间的绑定关系就要用到DI



## 1.2:DI

DI:依赖注入

什么是依赖注入呢?

* 在容器中建立bean与bean之间的依赖关系的整个过程，称为依赖注入
  * 业务层要用数据层的类对象，以前是自己`new`的
  * 现在自己不new了，靠`别人[外部其实指的就是IOC容器]`来给注入进来
  * 这种思想就是依赖注入
  * 依赖注入其实就是维护对象之间的关系



IOC容器中哪些bean之间要建立依赖关系呢?

* 这个需要程序员根据业务需求提前建立好关系，如业务层需要依赖数据层，service就要和dao建立依赖关系

介绍完Spring的IOC和DI的概念后，我们会发现这两个概念的最终目标就是:==充分解耦==，具体实现靠:

* 使用IOC容器管理bean（IOC)
* 在IOC容器内将有依赖关系的bean进行关系绑定（DI）
* 最终结果为:使用对象时不仅可以直接从IOC容器中获取，并且获取到的bean已经绑定了所有的依赖关系.

## 1.3:Spring管理什么

Spring管理的其实就是两件事情

1：管理对象的创建

2：管理这些对象之间的关系

## 1.4:小结

什么IOC/DI思想?

* IOC:控制反转，控制反转的是对象的创建权
* DI:依赖注入，绑定对象与对象之间的依赖关系

什么是IOC容器?

- Spring创建了一个容器用来存放所创建的对象，这个容器就叫IOC容器

什么是Bean?

- 容器中所存放的一个个对象就叫Bean或Bean对象
- IOC是一个思想，DI是Spring的具体实现



# 2:IOC使用

## 2.1:核心依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.10.RELEASE</version>
</dependency>
```

## 2.2:创建类

```java
public interface BookDao {
    public void save();
}
public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ...");
    }
}
public interface BookService {
    public void save();
}
public class BookServiceImpl implements BookService {
    private BookDao bookDao = new BookDaoImpl();
    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
}
```

## 2.3:配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
 
    <!--
		bean标签标示配置bean
    	id属性标示给bean起名字
    	class属性表示给bean定义类型
	-->
	<bean id="bookDao" class="com.zzx.dao.impl.BookDaoImpl"/>
    <bean id="bookService" class="com.zzx.service.impl.BookServiceImpl"/>

</beans>
```

注意事项：bean定义时 id 属性在同一个上下文中(配置文件)不能重复

另外一点：底层是反射操作，并且会调用无参数构造方法，所以需要提前准

备好无参数的构造方法，不然会报错！

## 2.4:获取IOC容器

使用Spring提供的接口完成IOC容器的创建，创建App类，编写main方法

```java
public class App {
    public static void main(String[] args) {
        //获取IOC容器【从classpath下获取】
		ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml"); 
    }
}
```

## 2.5:超级父接口

注意：ApplicationContext 是一个容器，它的超级父接口是：BeanFactory

BeanFactory是Spring的一个超级接口，ApplicationContext只是其子接口

```java
public class App {
    public static void main(String[] args) {
        //获取IOC容器【从classpath下获取】
		BeanFactory ctx = new ClassPathXmlApplicationContext("applicationContext.xml"); 
    }
}
```



## 2.6:容器中获取对象

```java
public class App {
    public static void main(String[] args) {
        //获取IOC容器
		ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml"); 
        //根据配置文件bean的id来获取对象
        BookDao bookDao = (BookDao) ctx.getBean("bookDao");
        bookDao.save();
        BookService bookService = (BookService) ctx.getBean("bookService");
        bookService.save();
    }
}
```

IOC只是解决了不用自己new对象，但是对象之间的关系，还是需要我们自己

在类里面 new 对象来指定关系

比如`BookServiceImpl`里面

这段代码：`private BookDao bookDao = new BookDaoImpl()`

为了解决这个问题，就要使用Spring的 `DI`

# 3:DI使用

要想实现依赖注入，必须要基于IOC管理Bean

- DI的入门案例要依赖于前面IOC的入门案例

Service中使用new形式创建的Dao对象是否保留?

- 需要删除掉，最终要使用IOC容器中的bean对象

Service中需要的Dao对象如何进入到Service中?

- 在Service中提供方法，让Spring的IOC容器可以通过该方法传入bean对象

Service与Dao间的关系如何描述?

- 使用配置文件

## 3.1:修改类

```java
public class BookServiceImpl implements BookService {
    //删除业务层中使用new的方式创建的dao对象【关系让Spring的DI来赋值】
    private BookDao bookDao;

    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
}
```

## 3.2:为属性提供setter方法

```java
public class BookServiceImpl implements BookService {
    //删除业务层中使用new的方式创建的dao对象
    private BookDao bookDao;

    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
    //提供对应的set方法
    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }
}
```

## 3.3:修改配置完成注入

在配置文件中添加依赖注入的配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--
		bean标签标示配置bean
    	id属性标示给bean起名字
    	class属性表示给bean定义类型
	-->
    <bean id="bookDao" class="com.zzx.dao.impl.BookDaoImpl"/>

    <bean id="bookService" class="com.zzx.service.impl.BookServiceImpl">
        <!--配置server与dao的关系-->
        <!--
			property标签表示配置当前bean的属性
        	 name属性表示配置哪一个具体的属性
        	 ref属性表示参照哪一个bean
		-->
        <property name="bookDao" ref="bookDao"/>
    </bean>

</beans>
```

这样我们从容器拿出对象【IOC】的时候就已经为`bookDao`赋好值了【DI】

我们这里使用的就是 set 注入，实际上还有其它的方式

我们这里要是不写 set 方法，则这个会报错！

我们在<property>标签里面写了bookDao，则会自动寻找setBookDao

是根据方法名寻找的，并不是根据属性的名字！



# 4:IOC相关

## 4.1:bean的配置

对于bean的配置中，主要会讲解`bean基础配置`,`bean的别名配置`,`bean的作用范围配置`(重点)这三部分

### 1:bean基础配置

```xml
<bean id="" class=""/>
```

其中，bean标签的功能、使用方式以及id和class属性的作用，我们通过一张图来描述下

![image-20221003090632336]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003090632336.png)

着重掌握 bean 标签的 `id`和`class`属性

**思考：**

* class属性能不能写接口如`BookDao`接口的类全名呢?

答案肯定是不行，因为接口是没办法创建对象的。

* 前面提过为bean设置id时，id必须唯一，但是如果由于命名习惯而产生了分歧后，该如何解决?
* 这个时候就要使用到bean的name属性



### 2:bean的name配置

首先来看下别名的配置说明:

![image-20221003090907656]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003090907656.png)

配置别名

打开spring的配置文件applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--name:为bean指定别名，别名可以有多个，使用逗号，分号，空格进行分隔-->
    <bean id="bookService" name="service service4 bookEbi" class="com.zzx.service.impl.BookServiceImpl">
        <property name="bookDao" ref="bookDao"/>
    </bean>

    <!--scope：为bean设置作用范围，可选值为单例singloton，非单例prototype-->
    <bean id="bookDao" name="dao" class="com.zzx.dao.impl.BookDaoImpl"/>
</beans>
```

**说明:Ebi全称Enterprise Business Interface，翻译为企业业务接口**

根据名称容器中获取bean对象

```java
public class AppForName {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        //此处根据bean标签的id属性和name属性的任意一个值来获取bean对象
        BookService bookService1 = (BookService) ctx.getBean("service");
        BookService bookService2 = (BookService) ctx.getBean("service4");
        BookService bookService3 = (BookService) ctx.getBean("bookEbi");
        bookService1.save();
        bookService2.save();
        bookService3.save();
    }
}
```

获取bean无论是通过id还是name获取，如果无法获取到，将抛出异常

### 3: bean的scope配置

关于bean的作用范围是bean属性配置的一个`重点`内容。

看到这个作用范围，我们就得思考bean的作用范围是来控制bean哪块内容的?

我们先来看下`bean作用范围的配置属性`:

![image-20221003091346451]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003091346451.png)



**我们先来验证IOC容器中对象是否为单例**

验证思路

​	同一个bean获取两次，将对象打印到控制台，看打印出的地址值是否一致

- 具体验证不在笔记编码了，很简单的一个配置

- 结论:默认情况下，Spring创建的bean对象都是单例的

获取到结论后，问题就来了，那如果我想创建出来非单例的bean对象，该如何实现呢?



配置bean为非单例

在Spring配置文件中，配置scope属性来实现bean的非单例创建

* 在Spring的配置文件中，修改`<bean>`的scope属性

```xml
<bean id="bookDao" name="dao" class="com.zzx.dao.impl.BookDaoImpl" scope=""/>
```

* 将scope设置为`singleton`---> 单例

```xml
<bean id="bookDao" name="dao" class="com.zzx.dao.impl.BookDaoImpl" scope="singleton"/>
```

- 将scope设置为`prototype`--->不是单例

```xml
<bean id="bookDao" name="dao" class="com.zzx.dao.impl.BookDaoImpl" scope="prototype"/>
```



### 4:scope思考

介绍完`scope`属性以后，我们来思考几个问题:

* 为什么bean默认为单例?
  * bean为单例的意思是在Spring的IOC容器中只会有该类的一个对象
  * bean对象只有一个就避免了对象的频繁创建与销毁，达到了bean对象的复用，性能高
* bean在容器中是单例的，会不会产生线程安全问题?
  * 如果对象是有状态对象，即该对象有成员变量可以用来存储数据的，
  * 因为所有请求线程共用一个bean对象，所以会存在线程安全问题。
  * 如果对象是无状态对象，即该对象没有成员变量没有进行数据存储的，
  * 因方法中的局部变量在方法调用完成后会被销毁，所以不会存在线程安全问题。
* 哪些bean对象适合交给容器进行管理?
  * 表现层对象
  * 业务层对象
  * 数据层对象
  * 工具对象
* 哪些bean对象不适合交给容器进行管理?
  * 封装实例的域对象，因为会引发线程安全问题，所以不适合。



## 4.2:bean实例化

对象已经能交给Spring的IOC容器来创建了，但是容器是如何来创建对象的呢?

就需要研究下`bean的实例化过程`，在这块内容中主要解决两部分内容，分别是

* bean是如何创建的
* 实例化bean的三种方式，`构造方法`,`静态工厂`和`实例工厂`

在讲解这三种创建方式之前，我们需要先确认一件事:

bean本质上就是对象，对象在new的时候会使用构造方法完成，那`创建bean也是使用构造方法完成的`

基于这个知识点出发，我们来验证spring中bean的`三种创建方式`



### 1:构造方法实例化

```java
public class BookDaoImpl implements BookDao {
    public BookDaoImpl() {
        System.out.println("book dao constructor is running ....");
    }
    public void save() {
        System.out.println("book dao save ...");
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	<bean id="bookDao" class="com.zzx.dao.impl.BookDaoImpl"/>
</beans>
```

运行程序拿对象，如控制台有打印构造函数中的输出，说明Spring容器在创建对象的时候也走的是构造函数

结果：执行了构造函数



将构造函数改成private测试

结果：执行了构造函数，能访问到类中的私有构造方法,显而易见Spring底层用的是反射



构造函数中添加一个参数测试

```java
public class BookDaoImpl implements BookDao {
    private BookDaoImpl(int i) {
        System.out.println("book dao constructor is running ....");
    }
    public void save() {
        System.out.println("book dao save ...");
    }
}
```

运行程序

程序会报错，说明Spring底层使用的是类的无参构造方法



### 2:静态工厂实例化

接下来研究Spring中的第二种bean的创建方式`静态工厂实例化`:



工厂方式创建bean

在讲这种方式之前，我们需要先`使用工厂来创建对象`的方式:

准备一个OrderDao和OrderDaoImpl类

```java
public interface OrderDao {
    public void save();
}

public class OrderDaoImpl implements OrderDao {
    public void save() {
        System.out.println("order dao save ...");
    }
}
```

创建一个工厂类`OrderDaoFactory`并提供一个`静态方法`

```java
//静态工厂创建对象
public class OrderDaoFactory {
    public static OrderDao getOrderDao(){
        return new OrderDaoImpl();
    }
}
```

编写AppForInstanceOrder运行类，在类中通过工厂获取对象

```java
public class AppForInstanceOrder {
    public static void main(String[] args) {
        //通过静态工厂创建对象
        OrderDao orderDao = OrderDaoFactory.getOrderDao();
        orderDao.save();
    }
}
```

如果代码中对象是通过上面的这种方式来创建的，如何将其交给Spring来管理呢?

这就要用到Spring中的静态工厂实例化的知识了，具体实现步骤为:



1：在spring的配置文件中添加以下内容:

```xml
<bean id="orderDao" class="com.zzx.factory.OrderDaoFactory" factorymethod="getOrderDao"/>
```

class:工厂类的类全名

factory-mehod:具体工厂类中创建对象的方法名

对应关系如下图:

![image-20221003093849438]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003093849438.png)

看到这，可能有人会问了，你这种方式在工厂类中不也是直接new对象的，和我自己直接new没什么太大的区别，而且静态工厂的方式反而更复杂，这种方式的意义是什么?

主要的原因是:

* 在工厂的静态方法中，我们除了new对象还可以做其他的一些业务操作，这些操作必不可少，比如:

```java
public class OrderDaoFactory {
    public static OrderDao getOrderDao(){
        //模拟必要的业务操作
        System.out.println("factory setup....");
        return new OrderDaoImpl();
    }
}
```

介绍完静态工厂实例化后，这种方式一般是用来兼容早期的一些老系统，所以了解为主

### 3:实例工厂实例化

接下来继续来研究Spring的第三种bean的创建方式`实例工厂实例化`:

1：准备一个UserDao和UserDaoImpl类

```java
public interface UserDao {
    public void save();
}

public class UserDaoImpl implements UserDao {

    public void save() {
        System.out.println("user dao save ...");
    }
}
```

2：创建一个工厂类OrderDaoFactory并提供一个`普通方法`，注意此处和`静态工厂的工厂类不一样`的地方是方法`不是静态方法`

```java
public class UserDaoFactory {
    public UserDao getUserDao(){
        return new UserDaoImpl();
    }
}
```

3：编写AppForInstanceUser运行类，在类中通过工厂获取对象

```java
public class AppForInstanceUser {
    public static void main(String[] args) {
        //创建实例工厂对象
        UserDaoFactory userDaoFactory = new UserDaoFactory();
        //通过实例工厂对象创建对象
        UserDao userDao = userDaoFactory.getUserDao();
        userDao.save();
}
```



对于上面这种实例工厂的方式如何交给Spring管理呢?

具体实现步骤为:



在spring的配置文件中添加以下内容:

```xml
<bean id="userFactory" class="com.zzx.factory.UserDaoFactory"/>

<bean id="userDao" factory-method="getUserDao" factory-bean="userFactory"/>
```

实例化工厂运行的顺序是:

* 创建实例化工厂对象,对应的是第一行配置

* 调用对象中的方法来创建bean，对应的是第二行配置

  * factory-bean:工厂的实例对象

  * factory-method:工厂对象中的具体创建对象的方法名,对应关系如下:

![image-20221003094356844]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003094356844.png)

factory-mehod:具体工厂类中创建对象的方法名

实例工厂实例化的方式就已经介绍完了，配置的过程还是比较复杂

所以Spring为了简化这种配置方式就提供了一种叫`FactoryBean`的方式来简化开发。



### 4:FactoryBean实例化

FactoryBean的使用

具体的使用步骤为:

1：创建一个UserDaoFactoryBean的类，实现FactoryBean接口，重写接口的方法

```java
public class UserDaoFactoryBean implements FactoryBean<UserDao> {
    //代替原始实例工厂中创建对象的方法
    public UserDao getObject() throws Exception {
        return new UserDaoImpl();
    }
    //返回所创建类的Class对象
    public Class<?> getObjectType() {
        return UserDao.class;
    }
}
```

2：在Spring的配置文件中进行配置

```xml
<bean id="userDao" class="com.zzx.factory.UserDaoFactoryBean"/>
```

这种方式在Spring去整合其他框架的时候会被用到，所以这种方式需要大家理解掌握



查看源码会发现，FactoryBean接口其实会有三个方法，分别是:

```java
T getObject() throws Exception;

Class<?> getObjectType();

default boolean isSingleton() {
		return true;
}
```

方法一：getObject()，被重写后，在方法中进行对象的创建并返回

方法二：getObjectType(),被重写后，主要返回的是被创建类的Class对象

方法三：没有重写，因为它已经给了默认值，从方法名中可以看出其作用是设置对象是否为单例，默认true

那如果想改成单例具体如何实现？只需要将isSingleton()方法进行重写，修改返回为false，即可

但是一般情况下我们都会采用单例，也就是采用默认即可。所以isSingleton()方法一般不需要进行重写



### 5:bean实例化小结

bean是如何创建的呢?

```
构造方法
```

Spring的IOC实例化对象的三种方式分别是:

* 构造方法(常用)
* 静态工厂(了解)
* 实例工厂(了解)
  * FactoryBean(实用)

这些方式中，重点掌握`构造方法`和`FactoryBean`即可。

需要注意的一点是，构造方法在类中默认会提供，但是如果重写了构造方法，默认的就会消失

在使用的过程中需要注意，如果需要重写构造方法，最好把默认的构造方法也重写下。



## 4.3:bean的生命周期

关于bean的相关知识还有最后一个是`bean的生命周期`,对于生命周期，我们主要围绕着`bean生命周期控制`来讲解:

* 首先理解下什么是生命周期?
  * 从创建到消亡的完整过程,例如人从出生到死亡的整个过程就是一个生命周期。
* bean生命周期是什么?
  * bean对象从创建到销毁的整体过程。
* bean生命周期控制是什么?
  * 在bean创建后到销毁前做一些事情。

现在我们面临的问题是如何在bean的创建之后和销毁之前把我们需要添加的内容添加进去



### 1:准备

```java
public interface BookDao {
    public void save();
}

public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ...");
    }
}

public interface BookService {
    public void save();
}

public class BookServiceImpl implements BookService{
    private BookDao bookDao;

    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }

    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bookDao" class="com.zzx.dao.impl.BookDaoImpl"/>
</beans>
```

```java
public class AppForLifeCycle {
    public static void main( String[] args ) {
        ApplicationContext ctx = new 
        	ClassPathXmlApplicationContext("applicationContext.xml");
        BookDao bookDao = (BookDao) ctx.getBean("bookDao");
        bookDao.save();
    }
}
```

### 2:生命周期设置

接下来，在上面这个环境中来为BookDao添加生命周期的控制方法，具体的控制有两个阶段:

* bean创建之后，想要添加内容，比如用来初始化需要用到资源
* bean销毁之前，想要添加内容，比如用来释放用到的资源



**步骤1:添加初始化和销毁方法**

针对这两个阶段，我们在BooDaoImpl类中分别添加两个方法，==方法名任意==

```java
public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ...");
    }
    //表示bean初始化对应的操作
    public void init(){
        System.out.println("init...");
    }
    //表示bean销毁前对应的操作
    public void destory(){
        System.out.println("destory...");
    }
}
```

**步骤2:配置生命周期**

在配置文件添加配置，如下:

```xml
<bean id="bookDao" class="com.zzx.dao.impl.BookDaoImpl" init-method="init" destroy-method="destory"/>
```

**步骤3:运行程序，发现初始方法执行了，销毁方法没有执行**



原因：

* Spring的IOC容器是运行在JVM中
* 运行main方法后,JVM启动,Spring加载配置文件生成IOC容器,从容器获取bean对象，然后调方法执行
* main方法执行完后，JVM退出，这个时候IOC容器中的bean还没有来得及销毁就已经结束了
* 所以没有调用对应的destroy方法

知道了出现问题的原因，具体该如何解决呢?



### 3:close关闭容器

* ApplicationContext中没有close方法

* 需要将ApplicationContext更换成ClassPathXmlApplicationContext

  ```java
  ClassPathXmlApplicationContext ctx = new 
      ClassPathXmlApplicationContext("applicationContext.xml");
  ```

* 调用ctx的close()方法

  ```java
  ctx.close();
  ```

* 运行程序，就能执行destroy方法的内容



### 4:注册钩子关闭容器

* 在容器未关闭之前，提前设置好回调函数，让JVM在退出之前回调此函数来关闭容器

* 调用ctx的`registerShutdownHook()`方法

  ```java
  ctx.registerShutdownHook();
  ```


**注意:**`registerShutdownHook在ApplicationContext中也没有`



两种方式介绍完后，`close` 和`registerShutdownHook`选哪个?

相同点:这两种都能用来关闭容器

不同点：

- close()是在调用的时候关闭，类似于我们自己手动关闭
- registerShutdownHook()是在JVM退出前调用关闭，是设置回调函数，等待JVM自己调度

分析上面的实现过程

- 会发现添加初始化和销毁方法，即需要编码也需要配置，实现起来步骤比较多也比较乱。

Spring提供了两个接口来完成生命周期的控制，好处是可以不用再进行配置`init-method`和`destroy-method`

接下来在BookServiceImpl完成这两个接口的使用:

修改BookServiceImpl类，添加两个接口`InitializingBean`， `DisposableBean`并实现接口中的两个方法

`afterPropertiesSet`和`destroy`

```java
public class BookServiceImpl implements BookService, InitializingBean, DisposableBean {
    private BookDao bookDao;
    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }
    public void save() {
        System.out.println("book service save ...");
        bookDao.save(); 
    }
    public void destroy() throws Exception {
        System.out.println("service destroy");
    }
    public void afterPropertiesSet() throws Exception {
        System.out.println("service init");
    }
}
```

**小细节**

* 对于InitializingBean接口中的afterPropertiesSet方法，翻译过来为`属性设置之后`。
* 对于BookServiceImpl来说，bookDao是它的一个属性
* setBookDao方法是Spring的IOC容器为其注入属性的方法
* 思考:afterPropertiesSet和setBookDao谁先执行?
* 答案：先set，赋值结束完成后才会执行`afterPropertiesSet`



### 5:bean生命周期小结

1：关于Spring中对bean生命周期控制提供了两种方式:

* 在配置文件中的bean标签中添加`init-method`和`destroy-method`属性
* 类实现`InitializingBean`与`DisposableBean`接口，这种方式了解下即可。

2：对于bean的生命周期控制在bean的整个生命周期中所处的位置如下:

* 初始化容器
  * 1.创建对象(内存分配)
  * 2.执行构造方法
  * 3.执行属性注入(set操作)
  * 4.执行bean初始化方法
* 使用bean
  * 1.执行业务操作
* 关闭/销毁容器
  * 1.执行bean销毁方法

3：关闭容器的两种方式:

* ConfigurableApplicationContext是ApplicationContext的子类
  * close()方法
  * registerShutdownHook()方法



# 5:DI相关

## 5.1:setter注入



### 1:注入引用类型

在bean中定义引用类型属性，并提供可访问的 `set`方法

```java
public class BookServiceImpl implements BookService {
    private BookDao bookDao;
    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }
}
```

配置中使用`property`标签的`ref`属性注入引用类型对象

```xml
<bean id="bookService" class="com.zzx.service.impl.BookServiceImpl">
	<property name="bookDao" ref="bookDao"/>
</bean>

<bean id="bookDao" class="com.zzx.dao.imipl.BookDaoImpl"/>
```



### 2:注入简单类型

**思考:**

引用类型使用的是`<property name="" ref=""/>`,简单数据类型还是使用ref么?

ref是指向Spring的IOC容器中的另一个bean对象的，对于简单数据类型，没有对应的bean对象，该如何配置?



1：在BookDaoImpl类中声明对应的简单数据类型的属性,并提供对应的setter方法

```java
public class BookDaoImpl implements BookDao {

    private String databaseName;
    private int connectionNum;

    public void setConnectionNum(int connectionNum) {
        this.connectionNum = connectionNum;
    }

    public void setDatabaseName(String databaseName) {
        this.databaseName = databaseName;
    }

    public void save() {
        System.out.println("book dao save ..."+databaseName+","+connectionNum);
    }
}
```

2：配置文件中进行注入配置

在applicationContext.xml配置文件中使用property标签注入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="bookDao" class="com.zzx.dao.impl.BookDaoImpl">
        <property name="databaseName" value="mysql"/>
     	<property name="connectionNum" value="10"/>
    </bean>
</beans>
```

**说明:**

value:后面跟的是简单数据类型，对于参数类型，Spring在注入的时候会自动转换，但是不能写成

```xml
<property name="connectionNum" value="abc"/>
```

这样的话，spring在将`abc`转换成int类型的时候就会报错



**注意:**两个property注入标签的顺序可以任意。

* 对于引用数据类型使用的是`<property name="" ref=""/>`
* 对于简单数据类型使用的是`<property name="" value=""/>`



## 5.2:构造注入

### 1:注入引用类型

```java
public class BookServiceImpl implements BookService{
    private BookDao bookDao;

    public BookServiceImpl(BookDao bookDao) {
        this.bookDao = bookDao;
    }

    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bookDao" class="com.zzx.dao.impl.BookDaoImpl"/>
    <bean id="bookService" class="com.zzx.service.impl.BookServiceImpl">
        <constructor-arg name="bookDao" ref="bookDao"/>
    </bean>
</beans>
```

**说明:**

标签<constructor-arg>中

* name属性对应的值为构造函数中方法形参的参数名，必须要保持一致

* ref属性指向的是spring的IOC容器中其他bean对象。



### 2:注入多个引用类型

```java
public class BookServiceImpl implements BookService{
    private BookDao bookDao;
    private UserDao userDao;

    public BookServiceImpl(BookDao bookDao,UserDao userDao) {
        this.bookDao = bookDao;
        this.userDao = userDao;
    }

    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
        userDao.save();
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bookDao" class="com.zzx.dao.impl.BookDaoImpl"/>
    <bean id="userDao" class="com.zzx.dao.impl.UserDaoImpl"/>
    <bean id="bookService" class="com.zzx.service.impl.BookServiceImpl">
        <constructor-arg name="bookDao" ref="bookDao"/>
        <constructor-arg name="userDao" ref="userDao"/>
    </bean>
</beans>
```

**说明:**这两个`<contructor-arg>`的配置顺序可以任意

### 3:注入多个简单类型

```java
public class BookDaoImpl implements BookDao {
    private String databaseName;
    private int connectionNum;

    public BookDaoImpl(String databaseName, int connectionNum) {
        this.databaseName = databaseName;
        this.connectionNum = connectionNum;
    }

    public void save() {
        System.out.println("book dao save ..."+databaseName+","+connectionNum);
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bookDao" class="com.zzx.dao.impl.BookDaoImpl">
        <constructor-arg name="databaseName" value="mysql"/>
        <constructor-arg name="connectionNum" value="666"/>
    </bean>
    <bean id="userDao" class="com.zzx.dao.impl.UserDaoImpl"/>
    <bean id="bookService" class="com.zzx.service.impl.BookServiceImpl">
        <constructor-arg name="bookDao" ref="bookDao"/>
        <constructor-arg name="userDao" ref="userDao"/>
    </bean>
</beans>
```

### 4:注意点

实际上注入这个构造类型的数据存在很多的方式

- 使用下标
- 使用属性的名字
- 不使用也可以，高版本的会自行推断

## 5.3:自动配置

### 1:依赖自动装配

IoC容器根据bean所依赖的资源在容器中自动查找并注入到bean中的过程称为自动装配



### 2:自动装配方式

* 按类型（常用）
* 按名称
* 按构造方法
* 不启用自动装配



### 3:自动装配的配置

自动装配只需要修改applicationContext.xml配置文件即可:

- 将`<property>`标签删除

- 在`<bean>`标签中添加autowire属性

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="com.zzx.dao.impl.BookDaoImpl"/>
    <!--autowire属性：开启自动装配，通常使用按类型装配-->
    <bean id="bookService" class="com.zzx.service.impl.BookServiceImpl" autowire="byType"/>

</beans>
```

* 需要注入属性的类中对应属性的setter方法不能省略
* 因为底层就是靠这个set方法来实现赋值的
* 被注入的对象必须要被Spring的IOC容器管理
* 按照类型在Spring的IOC容器中如果找到多个对象
* 会报`NoUniqueBeanDefinitionException`

一个类型在IOC中有多个对象，还想要注入成功，这个时候就需要按照名称注入，配置方式为:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="com.zzx.dao.impl.BookDaoImpl"/>
    <!--autowire属性：开启自动装配，通常使用按类型装配-->
    <bean id="bookService" class="com.zzx.service.impl.BookServiceImpl" autowire="byName"/>

</beans>
```

按照名称注入中的名称指的是什么?

![image-20221003101928708]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003101928708.png)

* bookDao是private修饰的，外部类无法直接方法
* 外部类只能通过属性的set方法进行访问
* 对外部类来说，setBookDao方法名，去掉set后首字母小写是其属性名
* 为什么是去掉set首字母小写?
* 这个规则是set方法生成的默认规则，set方法的生成是把属性名首字母大写前面加set形成的方法名
* 所以按照名称注入，其实是和对应的set方法有关，但是如果按照标准起名称，属性名和set对应的名是一致的
* 如果按照名称去找对应的bean对象，找不到则注入Null
* 当某一个类型在IOC容器中有多个对象，按照名称注入只找其指定名称对应的bean对象，不会报错 



两种方式介绍完后，以后用的更多的是`按照类型`注入

无论是按照类型注入还是按照名称注入，内部都是使用的set方法【注意】



最后对于依赖注入，需要注意一些其他的配置特征:

1. 自动装配用于引用类型依赖注入，不能对简单类型进行操作
2. 使用按类型装配时（byType）必须保障容器中相同类型的bean唯一，推荐使用
3. 使用按名称装配时（byName）必须保障容器中具有指定名称的bean，因变量名与配置耦合，不推荐
4. 自动装配优先级低于setter注入与构造器注入，同时出现时自动装配配置失效



## 5.4:集合注入

前面我们已经能完成引入数据类型和简单数据类型的注入，但是还有一种数据类型集合，集合中既可以装简

单数据类型也可以装引用数据类型，对于集合，在Spring中该如何注入呢?

先来回顾下，常见的集合类型有哪些?

* 数组
* List
* Set
* Map
* Properties

针对不同的集合类型，该如何实现注入呢?



### 1:准备

```java
public interface BookDao {
    public void save();
}
    
public class BookDaoImpl implements BookDao {

    private int[] array;

    private List<String> list;

    private Set<String> set;

    private Map<String,String> map;

    private Properties properties;

     public void save() {
        System.out.println("book dao save ...");

        System.out.println("遍历数组:" + Arrays.toString(array));

        System.out.println("遍历List" + list);

        System.out.println("遍历Set" + set);

        System.out.println("遍历Map" + map);

        System.out.println("遍历Properties" + properties);
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bookDao" class="com.zzx.dao.impl.BookDaoImpl"/>
</beans>
```

```java
public class AppForDICollection {
    public static void main( String[] args ) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        BookDao bookDao = (BookDao) ctx.getBean("bookDao");
        bookDao.save();
    }
}
```

接下来，在上面这个环境中来完成`集合注入`的学习:

下面的所有配置方式，都是在bookDao的bean标签中使用`<property>`进行注入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bookDao" class="com.zzx.dao.impl.BookDaoImpl">
        
    </bean>
</beans>
```

### 2:注入数组类型

```xml
<property name="array">
    <array>
        <value>100</value>
        <value>200</value>
        <value>300</value>
    </array>
</property>
```

如果是引用类型的数组，则直接使用<ref>即可

### 3:注入List类型

```xml
<property name="list">
    <list>
        <value>itcast</value>
        <value>zzx</value>
        <value>boxuegu</value>
        <value>chuanzhihui</value>
    </list>
</property>
```

如果是引用类型的数组，则直接使用<ref>即可

### 4:注入Set类型

```xml
<property name="set">
    <set>
        <value>itcast</value>
        <value>zzx</value>
        <value>boxuegu</value>
        <value>boxuegu</value>
    </set>
</property>
```

如果是引用类型的数组，则直接使用<ref>即可

效果是去查【我们都知道Set集合附带去查的功能】

### 5:注入Map类型

```xml
<property name="map">
    <map>
        <entry key="country" value="china"/>
        <entry key="province" value="henan"/>
        <entry key="city" value="kaifeng"/>
    </map>
</property>
```

如果key是引用类型的数组，则直接使用 `key-ref` 属性即可

如果value是引用类型的数组，则直接使用 `value-ref` 属性即可



### 6:注入Properties类型

Properties继承自HashTable，所以也是一个Map集合

HashMap：线程不安全，底层是哈希表/散列表

HashTable：线程安全，底层是哈希表/散列表

Properties：被称为属性类，key和value都只能是String类型

```xml
<property name="properties">
    <props>
        <prop key="country">china</prop>
        <prop key="province">henan</prop>
        <prop key="city">kaifeng</prop>
    </props>
</property>
```

**说明：**

* property标签表示setter方式注入，构造方式注入constructor-arg标签内部也可以写`<array>`、`<list>`、`<set>`、`<map>`、`<props>`标签
* List的底层也是通过数组实现的，所以`<list>`和`<array>`标签是可以混用
* 集合中要添加引用类型，只需要把`<value>`标签改成`<ref>`标签，这种方式用的比较少

## 5.5:注入特殊情况

### 1:注入空串

**方式一：写了value但是不赋值**

```xml
<properties name="email" value=""/>
```

**方式二：使用<value/>**

```xml
<properties name="email">
	<value/>
</properties>	
```

### 2:注入null

**方式一：不赋值**

```xml
<properties name="email"/>
```

**方式二：使用<null/>**

```xml
<properties name="email">
	<null/>
</properties>	
```



# 6:管理第三方bean

前面都是基于我们自己写的类，现在如果有需求让我们去管理第三方jar包中的类，该如何管理?

## 6.1:德鲁伊对象管理

在这一节中，我们将通过一个案例来学习下对于第三方bean该如何进行配置管理。

以后我们会用到很多第三方的bean

本次案例将使用咱们前面提到过的数据源`Druid(德鲁伊)`和`C3P0`来配置学习下



需求:使用Spring的IOC容器来管理Druid连接池对象

1.使用第三方的技术，需要在pom.xml添加依赖

2.在配置文件中将【第三方的类】制作成一个bean，让IOC容器进行管理

3.数据库连接需要基础的四要素`驱动`、`连接`、`用户名`和`密码`，【如何注入】到对应的bean中

4.从IOC容器中获取对应的bean对象，将其打印到控制台查看结果



1：导入依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.16</version>
</dependency>
```

2：配置第三方的bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">
	<!--管理DruidDataSource对象-->
    <bean class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring_db"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>
</beans>
```

3：获取连接池对象

```java
public class App {
    public static void main(String[] args) {
       ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
       DataSource dataSource = (DataSource) ctx.getBean("dataSource");
       System.out.println(dataSource);
    }
}
```



## 6.2:实现C3P0管理

完成了DruidDataSource的管理，接下来我们再来加深下练习，这次我们来管理`C3P0`数据源，具体的实现步骤是什么呢?

>需求:使用Spring的IOC容器来管理C3P0连接池对象
>
>实现方案和上面基本一致，重点要关注管理的是哪个bean对象`?

1：导入C3P0依赖

```xml
<dependency>
    <groupId>c3p0</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.1.2</version>
</dependency>
```

2：配置第三方bean，在applicationContext.xml配置文件中添加配置

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/spring_db"/>
    <property name="user" value="root"/>
    <property name="password" value="root"/>
    <property name="maxPoolSize" value="1000"/>
</bean>
```

注意：C3P0的四个属性和Druid的四个属性是不一样的，因为是不同厂商实现，百度查一下配置就好了



3：测试，发现找不到Mysql驱动，添加依赖即可

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.30</version>
</dependency>
```



**注意：**

* 数据连接池在配置属性的时候，除了可以注入数据库连接四要素外还可以配置很多其他的属性，具体都有哪些属性用到的时候再去查，一般配置基础的四个，其他都有自己的默认值
* Druid和C3P0在没有导入mysql驱动包的前提下，一个没报错一个报错，说明Druid在初始化的时候没有去加载驱动，而C3P0刚好相反
* Druid程序运行虽然没有报错，但是当调用DruidDataSource的getConnection()方法获取连接的时候，也会报找不到驱动类的错误



## 6.3:加载properties文件

上节中我们已经完成两个数据源`druid`和`C3P0`的配置，但是其中包含了一些问题，我们来分析下:

* 这两个数据源中都使用到了一些固定的常量如数据库连接四要素，把这些值写在Spring的配置文件中不利于后期维护
* 需要将这些值提取到一个外部的properties配置文件中
* Spring框架如何从配置文件中读取属性值来配置就是接下来要解决的问题。

```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:3306/spring_db
jdbc.username=root
jdbc.password=root
```

在applicationContext.xml中开`context`命名空间

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
</beans>
```

加载properties配置文件

在配置文件中使用`context`命名空间下的标签来加载properties配置文件

```xml
<context:property-placeholder location="jdbc.properties"/>
```

完成属性注入

使用`${key}`来读取properties配置文件中的内容并完成属性注入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
    
    <context:property-placeholder location="jdbc.properties"/>
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

### 1:读取单个属性

```java
public interface BookDao {
    public void save();
}

public class BookDaoImpl implements BookDao {
    private String name;

    public void setName(String name) {
        this.name = name;
    }

    public void save() {
        System.out.println("book dao save ..." + name);
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
    
    <context:property-placeholder location="jdbc.properties"/>
    
    <bean id="bookDao" class="com.zzx.dao.impl.BookDaoImpl">
        <property name="name" value="${jdbc.driver}"/>
    </bean>
</beans>
```

至此，读取properties配置文件中的内容就已经完成，但是在使用的时候，有些注意事项:

* 问题一:键值对的key为`username`引发的问题

  在properties中配置键值对的时候，如果key设置为`username`

```properties
username=root666
```

运行后，在控制台打印的却不是`root666`，而是自己电脑的用户名

解决方式：

```xml
<context:property-placeholder location="jdbc.properties" system-properties-mode="NEVER"/>
```

- 问题二:当有多个properties配置文件需要被加载，该如何配置?

```xml
<!--方式一 -->
<context:property-placeholder location="jdbc.properties,jdbc2.properties"/>
<!--方式二-->
<context:property-placeholder location="*.properties"/>
<!--方式三 -->
<context:property-placeholder location="classpath:*.properties"/>
<!--方式四-->
<context:property-placeholder location="classpath*:*.properties"/>
```

**说明:**

* 方式一:可以实现，如果配置文件多的话，每个都需要配置
* 方式二:`*.properties`代表所有以properties结尾的文件都会被加载，可以解决方式一的问题，但是不标准
* 方式三:标准的写法，`classpath:`代表的是从根路径下开始查找，但是只能查询当前项目的根路径
* 方式四:不仅可以加载当前项目还可以加载当前项目所依赖的所有项目的根路径下的properties配置文件



# 7:核心容器

这里所说的核心容器，大家可以把它简单的理解为`ApplicationContext`，前面虽然已经用到过，但是并没有系统的学习，接下来咱们从以下几个问题入手来学习下容器的相关知识

* 如何创建容器?
* 创建好容器后，如何从容器中获取bean对象?
* 容器类的层次结构是什么?
* BeanFactory是什么?



## 7.1:创建方式

在resources下添加applicationContext.xml

创建`ApplicationContext`的方式为:

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
```

这种方式翻译为：类路径下的XML配置文件

除了上面这种方式，Spring还提供了另外一种创建方式为:

```java
ApplicationContext ctx = new FileSystemXmlApplicationContext("applicationContext.xml");
```

这种方式翻译为：文件系统下的XML配置文件

使用这种方式，运行，会出现错误：找不到这个配置文件

从错误信息中能发现，这种方式是从项目路径下开始查找`applicationContext.xml`配置文件的，所以需要

将其修改为绝对路径

**说明:**这种方式虽能实现，但是当项目的位置发生变化后,代码也需要跟着改,耦合度较高,不推荐使用



## 7.2:Bean的三种获取方式

方式一，就是目前案例中获取的方式:

```java
BookDao bookDao = (BookDao) ctx.getBean("bookDao");
```

这种方式存在的问题是每次获取的时候都需要进行类型转换，有没有更简单的方式呢



方式二：

```java
BookDao bookDao = ctx.getBean("bookDao"，BookDao.class);
```

这种方式可以解决类型强转问题，但是参数又多加了一个，相对来说没有简化多少



方式三:

```java
BookDao bookDao = ctx.getBean(BookDao.class);
```

这种方式就类似我们之前所学习依赖注入中的按类型注入。必须要确保IOC容器中该类型对应的bean对象只能有一个



## 7.3:容器类层次结构

BeanFactory接口的结构的层次关系

![image-20221003104902839]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003104902839.png)

![image-20221003105241132]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003105241132.png)

## 7.4:BeanFactory的使用

使用BeanFactory来创建IOC容器的具体实现方式为:

```java
public class AppForBeanFactory {
    public static void main(String[] args) {
        Resource resources = new ClassPathResource("applicationContext.xml");
        BeanFactory bf = new XmlBeanFactory(resources);
        BookDao bookDao = bf.getBean(BookDao.class);
        bookDao.save();
    }
}
```

为了更好的看出`BeanFactory`和`ApplicationContext`之间的区别，在BookDaoImpl添加如下构造函数:

```java
public class BookDaoImpl implements BookDao {
    public BookDaoImpl() {
        System.out.println("constructor");
    }
    public void save() {
        System.out.println("book dao save ..." );
    }
}
```

如果不去获取bean对象，打印会发现：

* BeanFactory是延迟加载，只有在获取bean对象的时候才会去创建

* ApplicationContext是立即加载，容器加载的时候就会创建bean对象

* ApplicationContext要想成为延迟加载，只需要按照如下方式进行配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="bookDao" class="com.zzx.dao.impl.BookDaoImpl"  lazy-init="true"/>
</beans>
```

* 容器类层次结构

  * 只需要知晓容器的最上级的父接口为 BeanFactory即可

* BeanFactory

  * 使用BeanFactory创建的容器是延迟加载
  * 使用ApplicationContext创建的容器是立即加载
  * 具体BeanFactory如何创建只需要了解即可



## 7.5:总结

这节中没有新的知识点，只是对前面知识的一个大总结，共包含如下内容:

### 1:容器相关

- BeanFactory是IoC容器的顶层接口，初始化BeanFactory对象时，加载的bean延迟加载
- ApplicationContext接口是Spring容器的核心接口，初始化时bean立即加载
- ApplicationContext接口提供基础的bean操作相关方法，通过其他接口扩展其功能
- ApplicationContext接口常用初始化类
  - ClassPathXmlApplicationContext(常用)
  - FileSystemXmlApplicationContext

### 2:bean相关

![image-20221003105534204]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003105534204.png)

### 3:依赖注入相关

![image-20221003105622858]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003105622858.png)



# 8:IOC/DI注解开发

## 8.1:定义bean

步骤1：删除原XML配置

将配置文件中的`<bean>`标签删除掉

```xml
<bean id="bookDao" class="com.zzx.dao.impl.BookDaoImpl"/>
```



步骤2：Dao上添加注解

在BookDaoImpl类上添加`@Component`注解

```java
@Component("bookDao")
public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ..." );
    }
}
```

注意:@Component注解不可以添加在接口上，因为接口是无法创建对象的

一个@Component就相当于是一个`<bean>`



步骤3：配置Spring的注解包扫描

为了让Spring框架能够扫描到写在类上的注解，需要在配置文件上进行包扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <context:component-scan base-package="com.zzx"/>
</beans>
```

* **说明:**

  component-scan

  * component:组件,Spring将管理的bean视作自己的一个组件
  * scan:扫描

  base-package指定Spring框架扫描的包路径，它会扫描指定包及其子包中的所有类上的注解。

  * 包路径越多[如:com.zzx.dao.impl]，扫描的范围越小速度越快
  * 包路径越少[如:com.zzx],扫描的范围越大速度越慢
  * 一般扫描到项目的组织名称即Maven的groupId下[如:com.zzx]即可
  * @Component注解如果不起名称，会有一个默认值就是`当前类名首字母小写`
  * 所以也可以按照名称获取



对于@Component注解，还衍生出了其他三个注解`@Controller`、`@Service`、`@Repository`

这三个注解和@Component注解的作用是一样的，为什么要衍生出这三个呢?

方便我们后期在编写类的时候能很好的区分出这个类是属于`表现层`、`业务层`还是`数据层`的类。

| 名称 | @Component/@Controller/@Service/@Repository |
| ---- | ------------------------------------------- |
| 类型 | 类注解                                      |
| 位置 | 类定义上方                                  |
| 作用 | 设置该类为spring管理的bean                  |
| 属性 | value（默认）：定义bean的id                 |



## 8.2:纯注解开发

上面已经可以使用注解来配置bean,但是依然有用到配置文件，在配置文件中对包进行了扫描

Spring3.0开启了纯注解开发模式，使用Java类替代配置文件，开启了Spring快速开发赛道



1：将配置文件applicationContext.xml删除掉，创建配置类来替换

```java
public class SpringConfig {
}
```

2：标识该类为配置类

在配置类上添加`@Configuration`注解，将其标识为一个配置类,替换`applicationContext.xml`

```java
@Configuration
public class SpringConfig {
}
```

3：用注解替换包扫描配置

在配置类上添加包扫描注解`@ComponentScan`替换`<context:component-scan base-package=""/>`

```java
@Configuration
@ComponentScan("com.zzx")
public class SpringConfig {
}
```

4：创建运行类并执行

```java
public class AppForAnnotation {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        BookDao bookDao = (BookDao) ctx.getBean("bookDao");
        System.out.println(bookDao);
        BookService bookService = ctx.getBean(BookService.class);
        System.out.println(bookService);
    }
}
```

至此，纯注解开发的方式就已经完成了，主要内容包括:

* Java类替换Spring核心配置文件

* @Configuration注解用于设定当前类为配置类

* @ComponentScan注解用于设定扫描路径，此注解只能添加一次，多个数据请用数组格式

  ```java
  @ComponentScan({com.zzx.service","com.zzx.dao"})
  ```

* 读取Spring核心配置文件初始化容器对象切换为读取Java配置类初始化容器对象

  ```java
  //加载配置文件初始化容器
  ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
  //加载配置类初始化容器
  ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
  ```

### 1:@Configuration

| 名称 | @Configuration                                     |
| ---- | -------------------------------------------------- |
| 类型 | 类注解                                             |
| 位置 | 类定义上方                                         |
| 作用 | 设置该类为spring配置类，替换applicationContext.xml |
| 属性 | value（默认）：定义bean的id                        |

### 2:@ComponentScan

| 名称 | @ComponentScan                                           |
| ---- | -------------------------------------------------------- |
| 类型 | 类注解                                                   |
| 位置 | 类定义上方                                               |
| 作用 | 设置spring配置类扫描路径，用于加载使用注解格式定义的bean |
| 属性 | value（默认）：扫描路径，此路径可以逐层向下扫描          |

## 8.3:小结

这一节重点掌握的是使用注解完成Spring的bean管理，需要掌握的内容为:

* 记住@Component、@Controller、@Service、@Repository这四个注解
* applicationContext.xml中`<context:component-san/>`的作用是指定扫描包路径，注解为@ComponentScan
* @Configuration标识该类为配置类，使用类替换applicationContext.xml文件
* ClassPathXmlApplicationContext是加载XML配置文件
* AnnotationConfigApplicationContext是加载配置类





## 8.4: 注解Bean作用范围

默认情况下注解生成的bean也是单例

要想将BookDaoImpl变成非单例，只需要在其类上添加`@scope`注解

```java
@Repository
@Scope("prototype")
public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ...");
    }
}
```

| 名称 | @Scope                                                       |
| ---- | ------------------------------------------------------------ |
| 类型 | 类注解                                                       |
| 位置 | 类定义上方                                                   |
| 作用 | 设置该类创建对象的作用范围<br/>可用于设置创建出的bean是否为单例对象 |
| 属性 | value（默认）：定义bean作用范围，<br/>默认值singleton（单例），可选值prototype（非单例） |



## 8.5:注解Bean生命周期

在BookDaoImpl中添加两个方法，`init`和`destroy`，方法名可以任意

```java
@Repository
public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ...");
    }
    public void init() {
        System.out.println("init ...");
    }
    public void destroy() {
        System.out.println("destroy ...");
    }
}

```

如何对方法进行标识，哪个是初始化方法，哪个是销毁方法?

只需要在对应的方法上添加`@PostConstruct`和`@PreDestroy`注解即可

```java
@Repository
public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ...");
    }
    @PostConstruct //在构造方法之后执行，替换 init-method
    public void init() {
        System.out.println("init ...");
    }
    @PreDestroy //在销毁方法之前执行,替换 destroy-method
    public void destroy() {
        System.out.println("destroy ...");
    }
}
```

需要注意的是`destroy`只有在容器关闭的时候，才会执行，所以需要修改App的类

```java
public class App {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        BookDao bookDao1 = ctx.getBean(BookDao.class);
        BookDao bookDao2 = ctx.getBean(BookDao.class);
        System.out.println(bookDao1);
        System.out.println(bookDao2);
        ctx.close(); //关闭容器
    }
}
```

**注意:**@PostConstruct和@PreDestroy注解如果找不到，需要导入下面的jar包

```java
<dependency>
  <groupId>javax.annotation</groupId>
  <artifactId>javax.annotation-api</artifactId>
  <version>1.3.2</version>
</dependency>
```

找不到的原因是，从JDK9以后jdk中的javax.annotation包被移除了，这两个注解刚好就在这个包中。

| 名称 | @PostConstruct         |
| ---- | ---------------------- |
| 类型 | 方法注解               |
| 位置 | 方法上                 |
| 作用 | 设置该方法为初始化方法 |
| 属性 | 无                     |

| 名称 | @PreDestroy          |
| ---- | -------------------- |
| 类型 | 方法注解             |
| 位置 | 方法上               |
| 作用 | 设置该方法为销毁方法 |
| 属性 | 无                   |

**小结**

![image-20221003111538933]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003111538933.png)

## 8.6:注解开发依赖注入

Spring为了使用注解简化开发，并没有提供`构造函数注入`、`setter注入`对应的注解

只提供了自动装配的注解实现

### 1:按照类型注入

```java
@Service
public class BookServiceImpl implements BookService {
    @Autowired
    private BookDao bookDao;

    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
}
```

**注意:**

* @Autowired可以写在属性上，也可也写在setter方法上，最简单的处理方式是`写在属性上并将setter方法删除掉`
* 为什么setter方法可以删除呢?
  * 自动装配基于反射设计创建对象并通过暴力反射为私有属性进行设值
  * 普通反射只能获取public修饰的内容
  * 暴力反射除了获取public修饰的内容还可以获取private修改的内容
  * 所以此处无需提供setter方法

@Autowired是按照类型注入，那么对应BookDao接口如果有多个实现类，比如添加BookDaoImpl2

这个时候再次运行App，就会报错

此时，按照类型注入就无法区分到底注入哪个对象，解决方案:`按照名称注入`

* 先给两个Dao类分别起个名称

```java
@Repository("bookDao")
public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ..." );
    }
}
@Repository("bookDao2")
public class BookDaoImpl2 implements BookDao {
    public void save() {
        System.out.println("book dao save ...2" );
    }
}
```

此时就可以注入成功，但是得思考个问题: 

* @Autowired是按照类型注入的，给BookDao的两个实现起了名称，它还是有两个bean对象，为什么不报错?

* @Autowired默认按照类型自动装配，如果IOC容器中同类的Bean找到多个，就按照变量名和Bean的名称匹配。因为变量名叫`bookDao`而容器中也有一个`booDao`，所以可以成功注入。

* 分析下面这种情况是否能完成注入呢?

```java
@Repository("bookDao1")
public class BookDaoImpl1 implements BookDao {
    public void save() {
        System.out.println("book dao save ..." );
    }
}
@Repository("bookDao2")
public class BookDaoImpl2 implements BookDao {
    public void save() {
        System.out.println("book dao save ...2" );
    }
}
```

不行，因为按照类型会找到多个bean对象，此时会按照`bookDao`名称去找，因为IOC容器只有

`bookDao1`和`bookDao2`，所以找不到，会报`NoUniqueBeanDefinitionException`

### 2:按照名称注入

当根据类型在容器中找到多个bean,注入参数的属性名又和容器中bean的名称不一致，这个时候该如何解决，就需要使用到`@Qualifier`来指定注入哪个名称的bean对象。

```java
@Service
public class BookServiceImpl implements BookService {
    @Autowired
    @Qualifier("bookDao1")
    private BookDao bookDao;
    
    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
}
```

@Qualifier注解后的值就是需要注入的bean的名称。

注意:@Qualifier不能独立使用，必须和@Autowired一起使用



### 3:简单类型注入

```java
@Repository("bookDao")
public class BookDaoImpl implements BookDao {
    @Value("zzx")
    private String name;
    public void save() {
        System.out.println("book dao save ..." + name);
    }
}
```



### 4:读取配置文件

`@Value`一般会被用在从properties配置文件中读取内容进行使用，具体如何实现读取配置文件呢?

```java
@Configuration
@ComponentScan("com.zzx")
@PropertySource("jdbc.properties")
public class SpringConfig {
}
```

使用@Value读取配置文件中的内容

```java
@Repository("bookDao")
public class BookDaoImpl implements BookDao {
    @Value("${name}")
    private String name;
    public void save() {
        System.out.println("book dao save ..." + name);
    }
}
```

**注意:**

* 如果读取的properties配置文件有多个，可以使用`@PropertySource`的属性来指定多个

  ```java
  @PropertySource({"jdbc.properties","xxx.properties"})
  ```

* `@PropertySource`注解属性中不支持使用通配符`*`,运行会报错

  ```java
  @PropertySource({"*.properties"})
  ```

* `@PropertySource`注解属性中可以把`classpath:`加上,代表从当前项目的根路径找文件

  ```java
  @PropertySource({"classpath:jdbc.properties"})
  ```



### 5:注解小结

| 名称 | @Autowired                                                   |
| ---- | ------------------------------------------------------------ |
| 类型 | 属性注解  或  方法注解（了解）  或  方法形参注解（了解）     |
| 位置 | 属性定义上方  或  标准set方法上方  或  类set方法上方  或  方法形参前面 |
| 作用 | 为引用类型属性设置值                                         |
| 属性 | required：true/false，定义该属性是否允许为null               |



| 名称 | @Qualifier                                           |
| ---- | ---------------------------------------------------- |
| 类型 | 属性注解  或  方法注解（了解）                       |
| 位置 | 属性定义上方  或  标准set方法上方  或  类set方法上方 |
| 作用 | 为引用类型属性指定注入的beanId                       |
| 属性 | value（默认）：设置注入的beanId                      |



| 名称 | @Value                                               |
| ---- | ---------------------------------------------------- |
| 类型 | 属性注解  或  方法注解（了解）                       |
| 位置 | 属性定义上方  或  标准set方法上方  或  类set方法上方 |
| 作用 | 为  基本数据类型  或  字符串类型  属性设置值         |
| 属性 | value（默认）：要注入的属性值                        |



| 名称 | @PropertySource                                              |
| ---- | ------------------------------------------------------------ |
| 类型 | 类注解                                                       |
| 位置 | 类定义上方                                                   |
| 作用 | 加载properties文件中的属性值                                 |
| 属性 | value（默认）：设置加载的properties文件对应的文件名或文件名组成的数组 |



## 8.7:管理第三方bean

前面定义bean的时候都是在自己开发的类上面写个注解就完成了，但如果是第三方的类，这些类都是在jar包中，我们没有办法在别人框架写好的类上面添加注解，这个时候该怎么办?

遇到上述问题，我们就需要有一种更加灵活的方式来定义bean，这种方式不能在原始代码上面书写注解的类，一样能定义bean,这就用到了一个全新的注解`@Bean`

### 1:管理Druid数据源

1：添加依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.16</version>
</dependency>
```

2：在配置类中添加一个方法，注意该方法的返回值就是要创建的Bean对象类型

```java
@Configuration
public class SpringConfig {
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName("com.mysql.jdbc.cj.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/spring_db");
        ds.setUsername("root");
        ds.setPassword("root");
        return ds;
    }
}
```

3：在方法上添加`@Bean`注解

@Bean注解的作用是将方法的返回值制作为Spring管理的一个bean对象

```java
@Configuration
public class SpringConfig {
	@Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/spring_db");
        ds.setUsername("root");
        ds.setPassword("root");
        return ds;
    }
}
```

4：使用

```java
public class App {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        DataSource dataSource = ctx.getBean(DataSource.class);
        System.out.println(dataSource);
    }
}
```

至此使用@Bean来管理第三方bean的案例就已经完成

如果有多个bean要被Spring管理，直接在配置类中多些几个方法，方法上添加@Bean注解即可



### 2:引入外部配置类

如果把所有的第三方bean都配置到Spring的配置类`SpringConfig`中，虽然可以，但是不利于代码阅读和分

类管理，所有我们就想能不能按照类别将这些bean配置到不同的配置类中?

对于数据源的bean，我们新建一个`JdbcConfig`配置类，并把数据源配置到该类下。

```java
@Configuration
public class JdbcConfig {
	@Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/spring_db");
        ds.setUsername("root");
        ds.setPassword("root");
        return ds;
    }
}
```

现在的问题是，这个配置类如何能被Spring配置类加载到，并创建DataSource

对象在IOC容器中？针对这个问题，有两个解决方案:



### 3:包扫描引入外部配置

步骤1:在Spring的配置类上添加包扫描

```java
@Configuration
@ComponentScan("com.zzx.config")
public class SpringConfig {
	
}
```

步骤2:在JdbcConfig上添加配置注解

JdbcConfig类要放入到`com.zzx.config`包下，需要被Spring的配置类扫描到即可

```java
@Configuration
public class JdbcConfig {
	@Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName("com.mysql.jdbc.cj.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/spring_db");
        ds.setUsername("root");
        ds.setPassword("root");
        return ds;
    }
}
```

步骤3:运行

依然能获取到bean对象并打印控制台。

这种方式虽然能够扫描到，但是不能很快的知晓都引入了哪些配置类，所有这种方式不推荐使用



### 4:@Import引入外部配置

方案一实现起来有点小复杂，Spring早就想到了这一点，于是又给我们提供了第二种方案。

这种方案可以不用加`@Configuration`注解，但是必须在Spring配置类上使用`@Import`注解手动引入需要加

载的配置类



步骤1:去除JdbcConfig类上的注解

```java
public class JdbcConfig {
	@Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/spring_db");
        ds.setUsername("root");
        ds.setPassword("root");
        return ds;
    }
}
```

步骤2:在Spring配置类中引入

```java
@Configuration
@Import({JdbcConfig.class})
public class SpringConfig {
	
}
```

**注意:**

* 扫描注解可以移除

* @Import参数需要的是一个数组，可以引入多个配置类。

* @Import注解在配置类中只能写一次，下面的方式是==不允许的==

```java
@Configuration
@Import(JdbcConfig.class)
@Import(Xxx.class)
public class SpringConfig {
	
}
```

| 名称 | @Bean                                  |
| ---- | -------------------------------------- |
| 类型 | 方法注解                               |
| 位置 | 方法定义上方                           |
| 作用 | 设置该方法的返回值作为spring管理的bean |
| 属性 | value（默认）：定义bean的id            |



| 名称 | @Import                                                      |
| ---- | ------------------------------------------------------------ |
| 类型 | 类注解                                                       |
| 位置 | 类定义上方                                                   |
| 作用 | 导入配置类                                                   |
| 属性 | value（默认）：定义导入的配置类类名，<br/>当配置类有多个时使用数组格式一次性导入多个配置类 |



## 8.8:给第三方bean注入资源

在使用@Bean创建bean对象的时候，如果方法在创建的过程中需要其他资源该怎么办?

这些资源会有两大类，分别是`简单数据类型` 和`引用数据类型`

对于下面代码关于数据库的四要素不应该写死在代码中，应该是从properties配置文件中读取。如何来优化下面的代码?

```java
public class JdbcConfig {
	@Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/spring_db");
        ds.setUsername("root");
        ds.setPassword("root");
        return ds;
    }
}
```

### 1:注入简单数据类型

```java
public class JdbcConfig {
    @Value("com.mysql.jdbc.Driver")
    private String driver;
    @Value("jdbc:mysql://localhost:3306/spring_db")
    private String url;
    @Value("root")
    private String userName;
    @Value("password")
    private String password;
	@Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(userName);
        ds.setPassword(password);
        return ds;
    }
}
```

现在的数据库连接四要素还是写在代码中，需要做的是将这些内容提

取到jdbc.properties配置文件，大家思考下该如何实现?

```java
@PropertySource("classpath:jdbc.properties")
public class JdbcConfig {
    @Value("jdbc.driverClassName")
    private String driverClassName;
    @Value("jdbc.url")
    private String url;
    @Value("jdbc.username")
    private String username;
    @Value("jdbc.password")
    private String password;

    @Bean
    public DruidDataSource druidDataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driverClassName);
        ds.setUrl(url);
        ds.setUsername(username);
        ds.setPassword(password);
        return ds;
    }
}
```

### 2:注入引用数据类型

假设在构建DataSource对象的时候，需要用到BookDao对象，该如何把BookDao对象注入进方法内让其使用呢?

1：扫描到BookDao对象并存入容器

```java
@Configuration
@ComponentScan("com.zzx.dao")
@Import({JdbcConfig.class})
public class SpringConfig {
}
```

2：在JdbcConfig类的方法上添加参数

```java
@Bean
public DataSource dataSource(BookDao bookDao){
    System.out.println(bookDao);
    DruidDataSource ds = new DruidDataSource();
    ds.setDriverClassName(driver);
    ds.setUrl(url);
    ds.setUsername(userName);
    ds.setPassword(password);
    return ds;
}
```

引用类型注入只需要为bean定义方法设置形参即可，容器会根据类型自动装配对象

## 8.9:注解开发总结

前面我们已经完成了XML配置和注解的开发实现，至于两者之间的差异，咱们放在一块去对比回顾下:

![image-20221003120228711]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003120228711.png)



# 9:Spring整合

## 9.1:整合Mybatis

### 1:整合思路分析

哪些对象可以交给Spring来管理?

Mybatis程序核心对象分析

真正需要交给Spring管理的是SqlSessionFactory

整合Mybatis，就是将Mybatis用到的内容交给Spring管理，分析下配置文件

![image-20221003144252760]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003144252760.png)

**说明:**

* 第一行读取外部properties配置文件，Spring有提供具体的解决方案`@PropertySource`，需要交给Spring
* 第二行起别名包扫描，为SqlSessionFactory服务的，需要交给Spring
* 第三行主要用于做连接池，Spring之前我们已经整合了Druid连接池，这块也需要交给Spring
* 前面三行一起都是为了创建SqlSession对象用的，那么用Spring管理SqlSession对象吗?回忆下SqlSession是由SqlSessionFactory创建出来的，所以只需要将SqlSessionFactory交给Spring管理即可。
* 第四行是Mapper接口和映射文件[如果使用注解就没有该映射文件]，这个是在获取到SqlSession以后执行具体操作的时候用，所以它和SqlSessionFactory创建的时机都不在同一个时间，可能需要单独管理



### 2:具体实现

前面我们已经分析了Spring与Mybatis的整合，大体需要做两件事，

第一件事是:Spring要管理MyBatis中的SqlSessionFactory

第二件事是:Spring要管理Mapper接口的扫描

具体该如何实现，具体的步骤为:



1：加入依赖

```xml
<dependency>
    <!--Spring操作数据库需要该jar包-->
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.10.RELEASE</version>
</dependency>

<dependency>
    <!--
		Spring与Mybatis整合的jar包
		这个jar包mybatis在前面，是Mybatis提供的
	-->
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.0</version>
</dependency>
```

2：创建Spring的主配置类

```java
//配置类注解
@Configuration
@ComponentScan("com.zzx")
public class SpringConfig {
}
```

3：创建数据源的配置类

```java
public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String userName;
    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(userName);
        ds.setPassword(password);
        return ds;
    }
}
```

4：主配置类中读properties并引入数据源配置类

```java
@Configuration
@ComponentScan("com.zzx")
@PropertySource("classpath:jdbc.properties")
@Import(JdbcConfig.class)
public class SpringConfig {
}
```

5：创建Mybatis配置类并配置`SqlSessionFactoryBean`和`MapperScannerConfigurer`

```java
public class MybatisConfig {
    //定义bean，SqlSessionFactoryBean，用于产生SqlSessionFactory对象
    @Bean
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource){
        SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
        //设置模型类的别名扫描
        ssfb.setTypeAliasesPackage("com.zzx.domain");
        //设置数据源
        ssfb.setDataSource(dataSource);
        return ssfb;
    }
    //定义bean，返回MapperScannerConfigurer对象
    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer(){
        MapperScannerConfigurer msc = new MapperScannerConfigurer();
        msc.setBasePackage("com.zzx.dao");
        return msc;
    }
}
```

6：主配置类中引入Mybatis配置类

```java
@Configuration
@ComponentScan("com.zzx")
@PropertySource("classpath:jdbc.properties")
@Import({JdbcConfig.class,MybatisConfig.class})
public class SpringConfig {
}
```



### 3:重要对象解析

**SqlSessionFactoryBean：**

使用SqlSessionFactoryBean封装SqlSessionFactory需要的环境信息

![image-20221003145353268]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003145353268.png)SqlSessionFactoryBean是前面我们讲解FactoryBean【实例化工厂】的一个子类，在该类中将

SqlSessionFactory的创建进行了封装，简化对象的创建，我们只需要将其需要的内容设置即可

方法中有一个参数为dataSource,当前Spring容器中已经创建了Druid数据源，类型刚好是DataSource类型，此

时在初始化SqlSessionFactoryBean这个对象的时候，发现需要使用DataSource对象，而容器中刚好有这么一

个对象，就自动加载了DruidDataSource对象



**MapperScannerConfigurer：**

使用MapperScannerConfigurer加载Dao接口，创建代理对象保存到IOC容器中

![image-20221003145447655]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003145447655.png)

这个MapperScannerConfigurer对象也是MyBatis提供的专用于整合的jar包中的类，用来处理原始配置文件中

的mappers相关配置，加载数据层的Mapper接口类

MapperScannerConfigurer有一个核心属性basePackage，就是用来设置所扫描的包路径

# 10:AOP介绍

AOP是在不改原有代码的前提下对其进行增强

我们主要就是围绕着这一句话进行展开学习，主要学习两方面内容`AOP核心概念`,`AOP作用`:

## 10.1:什么是AOP

* AOP(Aspect Oriented Programming)面向切面编程，一种编程范式，指导开发者如何组织程序结构。
* OOP(Object Oriented Programming)面向对象编程

我们都知道OOP是一种编程思想，那么AOP也是一种编程思想，编程思想主要的内容就是指导程序员该如何编写程序，所以它们两个是不同的`编程范式`。

## 10.2:AOP作用

- 作用:在不改动的基础上为其进行功能增强，前面咱们有技术就可以实现这样的功能即代理模式。

前面咱们有技术就可以实现这样的功能即`代理模式`。

## 10.3:AOP核心概念

为了能更好的理解AOP的相关概念，我们准备了一个环境，整个环境的内容我们暂时可以不用关注，最主要的类为:`BookDaoImpl`，先来看看相关概念

```java
@Repository
public class BookDaoImpl implements BookDao {
    public void save() {
        //记录程序当前执行执行（开始时间）
        Long startTime = System.currentTimeMillis();
        //业务执行万次
        for (int i = 0;i<10000;i++) {
            System.out.println("book dao save ...");
        }
        //记录程序当前执行时间（结束时间）
        Long endTime = System.currentTimeMillis();
        //计算时间差
        Long totalTime = endTime-startTime;
        //输出信息
        System.out.println("执行万次消耗时间：" + totalTime + "ms");
    }
    public void update(){
        System.out.println("book dao update ...");
    }
    public void delete(){
        System.out.println("book dao delete ...");
    }
    public void select(){
        System.out.println("book dao select ...");
    }
}
```

四个方法，save具有计算时间的功能，但是`uodate`,`delete`,`select`都没有计算时间的功能

Spring的AOP是对一个类的方法在不进行任何修改的前提下实现增强，涉及的概念如下

### 1:连接点

对于上面的案例中BookServiceImpl中有`save`,`update`,`delete`和`select`方法

都不存在统计耗时的方法，这些方法我们给起了一个名字叫`连接点`



### 2:切入点

`update`和`delete`和`select`只有打印没有计算万次执行消耗时间，但是在运行的时候我们使用AOP使得`update`和`delete`已经有该功能，那也就是说`update`和`delete`方法都已经被增强，所以对于需要增强的方法我们给起了一个名字叫`切入点`



### 3:通知

执行BookServiceImpl的update和delete方法的时候都被添加了一个计算万次执行消耗时间的功能，将这个功能抽取到一个方法中，换句话说就是存放共性功能的方法，我们给起了个名字叫`通知`



### 4:切面

通知是要增强的内容，会有多个，切入点是需要被增强的方法，也会有多个，那哪个切入点需要添加哪个通知，就需要提前将它们之间的关系描述清楚，那么对于通知和切入点之间的关系描述，我们给起了个名字叫`切面`，切面就是来描述切入点需要绑定哪些通知的



### 5:通知类

通知是一个方法，方法不能独立存在需要被写在一个类中，这个类我们也给起了个名字叫`通知类`

至此AOP中的核心概念就已经介绍完了



# 11:AOP入门案例

## 11.1:准备

需求：在方法执行前输出当前系统时间

```java
@Repository
public class BookDaoImpl implements BookDao {

    @Override
    public void save() {
        System.out.println(System.currentTimeMillis());
        System.out.println("book dao save ...");
    }

    @Override
    public void update() {
        System.out.println("book dao update ...");
    }
}
```

**说明:**

* 目前打印save方法的时候，因为方法中有打印系统时间，所以运行的时候是可以看到系统时间
* 对于update方法来说，就没有该功能
* 我们要使用SpringAOP的方式在不改变update方法的前提下让其具有打印系统时间的功能。



## 11.2:实现

1：添加依赖

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
```

* 因为`spring-context`中已经导入了`spring-aop`,所以不需要再单独导入`spring-aop`
* 导入AspectJ的jar包是因为AspectJ是AOP思想的一个具体实现，Spring有自己的AOP实现，但是相比于AspectJ来说比较麻烦，所以我们直接采用Spring整合ApsectJ的方式进行AOP开发。



2：定义通知类和通知

通知就是将共性功能抽取出来后形成的方法，共性功能指的就是当前系统时间的打印。

```java
public class MyAdvice {
    public void method(){
        System.out.println(System.currentTimeMillis());
    }
}
```

类名和方法名没有要求，可以任意。



3：定义切入点

BookDaoImpl中有两个方法，分别是save和update，我们要增强的是update方法，该如何定义呢?

```java
public class MyAdvice {
    //切入点
    @Pointcut("execution(void com.zzx.dao.BookDao.update())")
    private void pt(){}
    
    //通知
    public void method(){
        System.out.println(System.currentTimeMillis());
    }
}
```

**说明:**

* 切入点定义依托一个不具有实际意义的方法进行，即无参数、无返回值、方法体无实际逻辑。
* execution及后面编写的内容，后面会有章节专门去学习。



4：制作切面

切面是用来描述通知和切入点之间的关系，如何进行关系的绑定?

```java
public class MyAdvice {
    //切入点
    @Pointcut("execution(void com.zzx.dao.BookDao.update())")
    private void pt(){}
    
    // 绑定切入点和通知
    @Before("pt()")
    public void method(){
        System.out.println(System.currentTimeMillis());
    }
}
```

绑定切入点与通知关系，并指定通知添加到原始连接点的具体执行位置

**说明:**@Before翻译过来是之前，也就是说通知会在切入点方法执行之前执行，除此之前还有其他四种类型



5：将通知类配给容器并标识其为切面类

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.zzx.dao.BookDao.update())")
    private void pt(){}
    
    @Before("pt()")
    public void method(){
        System.out.println(System.currentTimeMillis());
    }
}
```



6：开启注解格式AOP功能

```java
@Configuration
@ComponentScan("com.zzx")
@EnableAspectJAutoProxy
public class SpringConfig {
}
```



## 11.2:总结注解

| 名称 | @EnableAspectJAutoProxy |
| ---- | ----------------------- |
| 类型 | 配置类注解              |
| 位置 | 配置类定义上方          |
| 作用 | 开启注解格式AOP功能     |



| 名称 | @Aspect               |
| ---- | --------------------- |
| 类型 | 类注解                |
| 位置 | 切面类定义上方        |
| 作用 | 设置当前类为AOP切面类 |



| 名称 | @Pointcut                   |
| ---- | --------------------------- |
| 类型 | 方法注解                    |
| 位置 | 切入点方法定义上方          |
| 作用 | 设置切入点方法              |
| 属性 | value（默认）：切入点表达式 |



| 名称 | @Before                                                      |
| ---- | ------------------------------------------------------------ |
| 类型 | 方法注解                                                     |
| 位置 | 通知方法定义上方                                             |
| 作用 | 设置当前通知方法与切入点之间的绑定关系，当前通知方法在原始切入点方法前运行 |



# 12:AOP工作流程

AOP的入门案例已经完成，对于刚才案例的执行过程，我们就得来分析分析，这一节我们主要讲解两个知识点:`AOP工作流程`和`AOP核心概念`。其中核心概念是对前面核心概念的补充。

由于AOP是基于Spring容器管理的bean做的增强，所以整个工作过程需要从Spring加载bean说起

## 12.1:Spring容器启动

* 容器启动就需要去加载bean,哪些类需要被加载呢?
* 需要被增强的类，如:BookServiceImpl
* 通知类，如:MyAdvice
* 注意此时bean对象还没有创建成功

## 12.2:读取所有切入点

![image-20221003161947409]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003161947409.png)

上面这个例子中有两个切入点的配置，但是第一个`ptx()`并没有被使用，所以不会被读取。

## 12.3:初始化bean

判定bean对应的类中的方法是否匹配到任意切入点

* 注意第1步在容器启动的时候，bean对象还没有被创建成功。

* 要被实例化bean对象的类中的方法和切入点进行匹配

![image-20221003162221395]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003162221395.png)

* 匹配失败，创建原始对象,如`UserDao`
  * 匹配失败说明不需要增强，直接调用原始对象的方法即可。
* 匹配成功，创建原始对象（目标对象）的代理对象，如:`BookDao`
  * 匹配成功说明需要对其进行增强
  * 对哪个类做增强，这个类对应的对象就叫做目标对象
  * 因为要对目标对象进行功能增强，而采用的技术是动态代理，所以会为其创建一个代理对象
  * 最终运行的是代理对象的方法，在该方法中会对原始方法进行功能增强



## 12.4:获取bean执行方法

* 获取的bean是原始对象时，调用方法并执行，完成操作
* 获取的bean是代理对象时，根据代理对象的运行模式运行原始方法与增强的内容，完成操作

## 12.5:验证容器中是否为代理对象

为了验证IOC容器中创建的对象和我们刚才所说的结论是否一致，首先先把结论理出来:

* 如果目标对象中的方法会被增强，那么容器中将存入的是目标对象的代理对象【带有proxy】
* 如果目标对象中的方法不被增强，那么容器中将存入的是目标对象本身

## 12.6:AOP核心概念

在上面介绍AOP的工作流程中，我们提到了两个核心概念，分别是:

* 目标对象(Target)：原始功能去掉共性功能对应的类产生的对象，这种对象是无法直接完成最终工作的
* 代理(Proxy)：目标对象无法直接完成工作，需要对其进行功能回填，通过原始对象的代理对象实现

上面这两个概念比较抽象，简单来说，

目标对象就是要增强的类[如:BookServiceImpl类]对应的对象，也叫原始对象，不能说它不能运行，只能说它在运行的过程中对于要增强的内容是缺失的。

SpringAOP是在不改变原有设计(代码)的前提下对其进行增强的，它的底层采用的是代理模式实现的，所以要对原始对象进行增强，就需要对原始对象创建代理对象，在代理对象中的方法把通知[如:MyAdvice中的method方法]内容加进去，就实现了增强,这就是我们所说的代理(Proxy)



# 13:AOP配置管理

## 13.1:切入点配置

### 1:切入点表达式

首先我们先要明确两个概念:

* 切入点:要进行增强的方法
* 切入点表达式:要进行增强的方法的描述方式，怎么找到这个方法

对于切入点的描述，我们其实是有两种方式的，先来看下前面的例子

描述方式一：执行com.zzx.dao包下的BookDao接口中的无参数update方法

```java
execution(void com.zzx.dao.BookDao.update())
```

描述方式二：执行com.zzx.dao.impl包下的BookDaoImpl类中的无参数update方法

```java
execution(void com.zzx.dao.impl.BookDaoImpl.update())
```

因为调用接口方法的时候最终运行的还是其实现类的方法，所以上面两种描述方式都是可以的。

对于切入点表达式的语法为:

* 切入点表达式标准格式：动作关键字(访问修饰符  返回值  包名.类/接口名.方法名(参数) 异常名）

对于这个格式，我们不需要硬记，通过一个例子，理解它:

```java
execution(public User com.zzx.service.UserService.findById(int))
```

* execution：动作关键字，描述切入点的行为动作，例如execution表示执行到指定切入点
* public:访问修饰符,还可以是public，private等，可以省略
* User：返回值，写返回值类型
* com.zzx.service：包名，多级包使用点连接
* UserService:类/接口名称
* findById：方法名
* int:参数，直接写参数的类型，多个类型用逗号隔开
* 异常名：方法定义中抛出指定异常，可以省略

切入点表达式就是要找到需要增强的方法，所以它就是对一个具体方法的描述，但是方法的定义会有很多，所以如果每一个方法对应一个切入点表达式，想想这块就会觉得将来编写起来会比较麻烦，有没有更简单的方式呢?

就需要用到下面所学习的通配符。

### 2:通配符

我们使用通配符描述切入点，主要的目的就是简化之前的配置，具体都有哪些通配符可以使用?

* `*`:单个独立的任意符号，可以独立出现，也可以作为前缀或者后缀的匹配符出现

  ```
  execution（public * com.zzx.*.UserService.find*(*))
  ```

  匹配com.zzx包下的任意包中的UserService类或接口中所有find开头的带有一个参数的方法

* `..`：多个连续的任意符号，可以独立出现，常用于简化包名与参数的书写

  ```
  execution（public User com..UserService.findById(..))
  ```

  匹配com包下的任意包中的UserService类或接口中所有名称为findById的方法

* `+`：专用于匹配子类类型

  ```
  execution(* *..*Service+.*(..))
  ```

  这个使用率较低，描述子类的，咱们做JavaEE开发，继承机会就一次，使用都很慎重，所以很少用它。*Service+，表示所有以Service结尾的接口的子类。

接下来，我们把案例中使用到的切入点表达式来分析下:



![image-20221003163222937]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003163222937.png)

```java
execution(void com.zzx.dao.BookDao.update())
匹配接口，能匹配到
execution(void com.zzx.dao.impl.BookDaoImpl.update())
匹配实现类，能匹配到
execution(* com.zzx.dao.impl.BookDaoImpl.update())
返回值任意，能匹配到
execution(* com.zzx.dao.impl.BookDaoImpl.update(*))
返回值任意，但是update方法必须要有一个参数，无法匹配，要想匹配需要在update接口和实现类添加参数
execution(void com.*.*.*.*.update())
返回值为void,com包下的任意包三层包下的任意类的update方法，匹配到的是实现类，能匹配
execution(void com.*.*.*.update())
返回值为void,com包下的任意两层包下的任意类的update方法，匹配到的是接口，能匹配
execution(void *..update())
返回值为void，方法名是update的任意包下的任意类，能匹配
execution(* *..*(..))
匹配项目中任意类的任意方法，能匹配，但是不建议使用这种方式，影响范围广
execution(* *..u*(..))
匹配项目中任意包任意类下只要以u开头的方法，update方法能满足，能匹配
execution(* *..*e(..))
匹配项目中任意包任意类下只要以e结尾的方法，update和save方法能满足，能匹配
execution(void com..*())
返回值为void，com包下的任意包任意类任意方法，能匹配，*代表的是方法
execution(* com.zzx.*.*Service.find*(..))
将项目中所有业务层方法的以find开头的方法匹配
execution(* com.zzx.*.*Service.save*(..))
将项目中所有业务层方法的以save开头的方法匹配
```

### 3:书写技巧

对于切入点表达式的编写其实是很灵活的，那么在编写的时候，有没有什么好的技巧让我们用用:

- 所有代码按照标准规范开发，否则以下技巧全部失效
- 描述切入点通常描述接口，而不描述实现类,如果描述到实现类，就出现紧耦合了
- 访问控制修饰符针对接口开发均采用public描述（**==可省略访问控制修饰符描述==**）
- 返回值类型对于增删改类使用精准类型加速匹配，对于查询类使用\*通配快速描述
- **==包名==**书写**==尽量不使用..匹配==**，效率过低，常用\*做单个包描述匹配，或精准匹配
- **==接口名/类名==**书写名称与模块相关的**==采用\*匹配==**，例如UserService书写成\*Service，绑定业务层接口名
- **==方法名==**书写以**==动词==**进行**==精准匹配==**，名词采用*匹配，例如getById书写成getBy*,selectAll书写成selectAll
- 参数规则较为复杂，根据业务方法灵活调整
- 通常**==不使用异常==**作为**==匹配==**规则



## 13.2:AOP通知类型

```java
@Before("pt()")
public void method(){
    System.out.println(System.currentTimeMillis());
}
```

它所代表的含义是将`通知`添加到`切入点`方法执行的==前面==。

除了这个注解外，还有没有其他的注解，换个问题就是除了可以在前面加，能不能在其他的地方加?

### 1:类型介绍

我们先来回顾下AOP通知:

* AOP通知描述了抽取的共性功能，根据共性功能抽取的位置不同，最终运行代码时要将其加入到合理的位置

通知具体要添加到切入点的哪里?

共提供了5种通知类型:

- 前置通知
- 后置通知
- **==环绕通知(重点)==**
- 返回后通知(了解)
- 抛出异常后通知(了解)

为了更好的理解这几种通知类型，我们来看一张图

![image-20221003163633232]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003163633232.png)

(1)前置通知，追加功能到方法执行前,类似于在代码1或者代码2添加内容

(2)后置通知,追加功能到方法执行后,不管方法执行的过程中有没有抛出异常都会执行，类似于在代码5添加内容

(3)返回后通知,追加功能到方法执行后，只有方法正常执行结束后才进行,类似于在代码3添加内容，如果方法执行抛出异常，返回后通知将不会被添加

(4)抛出异常后通知,追加功能到方法抛出异常后，只有方法执行出异常才进行,类似于在代码4添加内容，只有方法抛出异常后才会被添加

(5)环绕通知,环绕通知功能比较强大，它可以追加功能到方法执行的前后，这也是比较常用的方式，它可以实现其他四种通知类型的功能，具体是如何实现的，需要我们往下学习。



### 2:准备

1：准备依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.9.4</version>
    </dependency>
</dependencies>
```

2：接口和实现类

```java
public interface BookDao {
    public void update();
    public int select();
}

@Repository
public class BookDaoImpl implements BookDao {
    public void update(){
        System.out.println("book dao update ...");
    }
    public int select() {
        System.out.println("book dao select is running ...");
        return 100;
    }
}
```

3：配置

```java
@Configuration
@ComponentScan("com.zzx")
@EnableAspectJAutoProxy
public class SpringConfig {
}
```

4：通知类

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.zzx.dao.BookDao.update())")
    private void pt(){}

    public void before() {
        System.out.println("before advice ...");
    }

    public void after() {
        System.out.println("after advice ...");
    }

    public void around(){
        System.out.println("around before advice ...");
        System.out.println("around after advice ...");
    }

    public void afterReturning() {
        System.out.println("afterReturning advice ...");
    }
    
    public void afterThrowing() {
        System.out.println("afterThrowing advice ...");
    }
}
```

### 3:前置通知

修改MyAdvice，在before方法上添加`@Before注解`

```java
@Pointcut("execution(void com.zzx.dao.BookDao.update())")
private void pt(){}
@Before("pt()")
public void before() {
    System.out.println("before advice ...");
}
```

### 4:后置通知

```java
@Pointcut("execution(void com.zzx.dao.BookDao.update())")
private void pt(){}
@After("pt()")
public void after() {
    System.out.println("after advice ...");
}
```

### 5:环绕通知

```java
@Pointcut("execution(void com.zzx.dao.BookDao.update())")
private void pt(){}
@Around("pt()")
public void around(){
    System.out.println("around before advice ...");
    System.out.println("around after advice ...");
}
```

运行结果中，通知的内容打印出来，但是原始方法的内容却没有被执行

因为环绕通知需要在原始方法的前后进行增强，所以环绕通知就必须要能对原始操作进行调用，它才能知道

到底哪里在前面执行，哪里在后面执行，具体如何实现?

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.zzx.dao.BookDao.update())")
    private void pt(){}
    
    @Around("pt()")
    public void around(ProceedingJoinPoint pjp) throws Throwable{
        System.out.println("around before advice ...");
        //表示对原始操作的调用
        pjp.proceed();
        System.out.println("around after advice ...");
    }
}
```

**说明:**proceed()为什么要抛出异常?

原因很简单，看下源码就知道了

```java
Object proceed() throws Throwable;
```

为什么这样设计？

因为我们也不知道我们代理的方法到底有没有异常，所有设计为必须进行异常处理



### 6:环绕通知返回值

所以如果我们使用环绕通知的话，要根据原始方法的返回值来设置环绕通知的返回值

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.zzx.dao.BookDao.update())")
    private void pt(){}
    
    @Pointcut("execution(int com.zzx.dao.BookDao.select())")
    private void pt2(){}
    
    @Around("pt2()")
    public Object aroundSelect(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("around before advice ...");
        //表示对原始操作的调用
        Object ret = pjp.proceed();
        System.out.println("around after advice ...");
        return ret;
    }
}
```

**说明:**

​	为什么返回的是Object而不是int的主要原因是Object类型更通用

​	在环绕通知中是可以对原始方法返回值就行修改的



### 7:返回后通知

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.zzx.dao.BookDao.update())")
    private void pt(){}
    
    @Pointcut("execution(int com.zzx.dao.BookDao.select())")
    private void pt2(){}
    
    @AfterReturning("pt2()")
    public void afterReturning() {
        System.out.println("afterReturning advice ...");
    }
}
```

**注意：**返回后通知是需要在原始方法`select`正常执行后才会被执行，如果`select()`方法执行的过程中出现了异常，那么返回后通知是不会被执行。后置通知是不管原始方法有没有抛出异常都会被执行

### 8:异常后通知

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.zzx.dao.BookDao.update())")
    private void pt(){}
    
    @Pointcut("execution(int com.zzx.dao.BookDao.select())")
    private void pt2(){}
    
    @AfterReturning("pt2()")
    public void afterThrowing() {
        System.out.println("afterThrowing advice ...");
    }
}
```

### 9:通知类型总结

| 名称 | @After                                                       |
| ---- | ------------------------------------------------------------ |
| 类型 | 方法注解                                                     |
| 位置 | 通知方法定义上方                                             |
| 作用 | 设置当前通知方法与切入点之间的绑定关系，当前通知方法在原始切入点方法后运行 |



| 名称 | @AfterReturning                                              |
| ---- | ------------------------------------------------------------ |
| 类型 | 方法注解                                                     |
| 位置 | 通知方法定义上方                                             |
| 作用 | 设置当前通知方法与切入点之间绑定关系，当前通知方法在原始切入点方法正常执行完毕后执行 |



| 名称 | @AfterThrowing                                               |
| ---- | ------------------------------------------------------------ |
| 类型 | 方法注解                                                     |
| 位置 | 通知方法定义上方                                             |
| 作用 | 设置当前通知方法与切入点之间绑定关系，当前通知方法在原始切入点方法运行抛出异常后执行 |



| 名称 | @Around                                                      |
| ---- | ------------------------------------------------------------ |
| 类型 | 方法注解                                                     |
| 位置 | 通知方法定义上方                                             |
| 作用 | 设置当前通知方法与切入点之间的绑定关系，当前通知方法在原始切入点方法前后运行 |



**环绕通知注意事项**

1. 环绕通知必须依赖形参`ProceedingJoinPoint`才能实现对原始方法的调用，进而实现原始方法调用前后同时添加通知
2. 通知中如果未使用ProceedingJoinPoint对原始方法进行调用将跳过原始方法的执行
3. 对原始方法的调用可以不接收返回值，通知方法设置成void即可，如果接收返回值，最好设定为Object类型
4. 原始方法的返回值如果是void类型，通知方法的返回值类型可以设置成void,也可以设置成Object
5. 由于无法预知原始方法运行后是否会抛出异常，因此环绕通知方法必须要处理Throwable异常



## 13.3:AOP通知获取数据

### 1:非环绕通知获取方式

在方法上添加JoinPoint,通过JoinPoint来获取参数

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.zzx.dao.BookDao.findName(..))")
    private void pt(){}

    @Before("pt()")
    public void before(JoinPoint jp) 
        Object[] args = jp.getArgs();
        System.out.println(Arrays.toString(args));
        System.out.println("before advice ..." );
    }
}
```

使用JoinPoint的方式获取参数适用于`前置`、`后置`、`返回后`、`抛出异常后`通知。剩下的大家自行去验证



### 2:环绕通知获取方式

环绕通知使用的是ProceedingJoinPoint，因为ProceedingJoinPoint是JoinPoint类的子类，所以对于ProceedingJoinPoint类中应该也会有对应的`getArgs()`方法

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.zzx.dao.BookDao.findName(..))")
    private void pt(){}

    @Around("pt()")
    public Object around(ProceedingJoinPoint pjp)throws Throwable {
        Object[] args = pjp.getArgs();
        System.out.println(Arrays.toString(args));
        Object ret = pjp.proceed();
        return ret;
    }
}
```

### 3:ProceedingJoinPoin

ProceedingJoinPoin的proceed()方法是有两个构造方法，分别是:

```java
joinPoint.proceed();
joinPoint.proceed(Object[] objects)
```

调用无参数的proceed，当原始方法有参数，会在调用的过程中自动传入参数

所以调用这两个方法的任意一个都可以完成功能

但是当需要修改原始方法的参数时，就只能采用带有参数的方法,如下:

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.zzx.dao.BookDao.findName(..))")
    private void pt(){}

    @Around("pt()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable{
        Object[] args = pjp.getArgs();
        System.out.println(Arrays.toString(args));
        args[0] = 666;
        Object ret = pjp.proceed(args);
        return ret;
    }
}
```

有了这个特性后，我们就可以在环绕通知中对原始方法的参数进行拦截过滤，避免由于参数的问题导致程序无法正确运行，保证代码的健壮性。



### 4:环绕通知获取返回值

返回值，只有返回后`AfterReturing`和环绕`Around`这两个通知类型可以获取，具体环绕通知如何获取?

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.zzx.dao.BookDao.findName(..))")
    private void pt(){}

    @Around("pt()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable{
        Object[] args = pjp.getArgs();
        System.out.println(Arrays.toString(args));
        args[0] = 666;
        Object ret = pjp.proceed(args);
        return ret;
    }
}
```

上述代码中，`ret`就是方法的返回值，我们是可以直接获取，不但可以获取，如果需要还可以进行修改。

### 5:返回后通知获取返回值

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.zzx.dao.BookDao.findName(..))")
    private void pt(){}

    @AfterReturning(value = "pt()",returning = "ret")
    public void afterReturning(Object ret) {
        System.out.println("afterReturning advice ..."+ret);
    }
}
```

![image-20221003171114805]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003171114805.png)

afterReturning方法参数类型的问题

参数类型可以写成String，但是为了能匹配更多的参数类型，建议写成Object类型



afterReturning方法参数的顺序问题

![image-20221003171151939]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003171151939.png)



### 6:环绕通知获取异常

这块比较简单，以前我们是抛出异常，现在只需要将异常捕获，就可以获取到原始方法的异常信息了

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.zzx.dao.BookDao.findName(..))")
    private void pt(){}

    @Around("pt()")
    public Object around(ProceedingJoinPoint pjp){
        Object[] args = pjp.getArgs();
        System.out.println(Arrays.toString(args));
        args[0] = 666;
        Object ret = null;
        try{
            ret = pjp.proceed(args);
        }catch(Throwable throwable){
            t.printStackTrace();
        }
        return ret;
    }
}
```

在catch方法中就可以获取到异常，至于获取到异常以后该如何处理，这个就和你的业务需求有关了。

### 7:抛出异常后通知获取异常

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.zzx.dao.BookDao.findName(..))")
    private void pt(){}

    @AfterThrowing(value = "pt()",throwing = "t")
    public void afterThrowing(Throwable t) {
        System.out.println("afterThrowing advice ..."+t);
    }
}
```

![image-20221003171321473]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003171321473.png)

# 13:AOP事务管理

## 13.1:概念介绍

- 事务作用：在数据层保障一系列的数据库操作同成功同失败
- Spring事务作用：在数据层或**==业务层==**保障一系列的数据库操作同成功同失败

数据层有事务我们可以理解，为什么业务层也需要处理事务呢?

举个简单的例子：

* 转账业务会有两次数据层的调用，一次是加钱一次是减钱
* 把事务放在数据层，加钱和减钱就有两个事务
* 没办法保证加钱和减钱同时成功或者同时失败
* 这个时候就需要将事务放在业务层进行处理。

Spring为了管理事务，提供了一个平台事务管理器`PlatformTransactionManager`接口

![image-20221003171729967]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003171729967.png)

commit是用来提交事务，rollback是用来回滚事务。

PlatformTransactionManager只是一个接口

Spring还为其提供了一个具体的实现`DataSourceTransactionManager`和`HibernateTransactionManager`



从名称上可以看出，我们只需要给它一个DataSource对象，它就可以帮你去在业务层管理事务。其内部采用

的是JDBC的事务。所以说如果你持久层采用的是JDBC相关的技术，就可以采用这个事务管理器来管理你的

事务。而Mybatis内部采用的就是JDBC的事务，所以后期我们Spring整合Mybatis就采用的这个

DataSourceTransactionManager事务管理器



## 13.2:准备环境

步骤1:准备数据库表

之前我们在整合Mybatis的时候已经创建了这个表,可以直接使用

```sql
create database spring_db character set utf8;
use spring_db;
create table tbl_account(
    id int primary key auto_increment,
    name varchar(35),
    money double
);
insert into tbl_account values(1,'Tom',1000);
insert into tbl_account values(2,'Jerry',1000);
```

步骤2:创建项目导入jar包

项目的pom.xml添加相关依赖

```xml
<dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.2.10.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.16</version>
    </dependency>

    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.6</version>
    </dependency>

    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.47</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.2.10.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.0</version>
    </dependency>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>5.2.10.RELEASE</version>
    </dependency>

  </dependencies>
```

步骤3:根据表创建模型类

```java
public class Account implements Serializable {

    private Integer id;
    private String name;
    private Double money;
	//setter...getter...toString...方法略    
}
```

步骤4:创建Dao接口

```java
public interface AccountDao {

    @Update("update tbl_account set money = money + #{money} where name = #{name}")
    void inMoney(@Param("name") String name, @Param("money") Double money);

    @Update("update tbl_account set money = money - #{money} where name = #{name}")
    void outMoney(@Param("name") String name, @Param("money") Double money);
}
```

步骤5:创建Service接口和实现类

```java
public interface AccountService {
    /**
     * 转账操作
     * @param out 传出方
     * @param in 转入方
     * @param money 金额
     */
    public void transfer(String out,String in ,Double money) ;
}

@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;

    public void transfer(String out,String in ,Double money) {
        accountDao.outMoney(out,money);
        accountDao.inMoney(in,money);
    }

}
```

步骤6:添加jdbc.properties文件

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring_db?useSSL=false
jdbc.username=root
jdbc.password=root
```

步骤7:创建JdbcConfig配置类

```java
public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String userName;
    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(userName);
        ds.setPassword(password);
        return ds;
    }
}
```

步骤8:创建MybatisConfig配置类

```java
public class MybatisConfig {

    @Bean
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource){
        SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
        ssfb.setTypeAliasesPackage("com.zzx.domain");
        ssfb.setDataSource(dataSource);
        return ssfb;
    }

    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer(){
        MapperScannerConfigurer msc = new MapperScannerConfigurer();
        msc.setBasePackage("com.zzx.dao");
        return msc;
    }
}
```

步骤9:创建SpringConfig配置类

```java
@Configuration
@ComponentScan("com.zzx")
@PropertySource("classpath:jdbc.properties")
@Import({JdbcConfig.class,MybatisConfig.class})
public class SpringConfig {
}

```

步骤10:编写测试类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfig.class)
public class AccountServiceTest {

    @Autowired
    private AccountService accountService;

    @Test
    public void testTransfer() throws IOException {
        accountService.transfer("Tom","Jerry",100D);
    }

}
```



## 13.3:问题演示

```java
@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;

    public void transfer(String out,String in ,Double money) {
        accountDao.outMoney(out,money);
        int i = 1/0;
        accountDao.inMoney(in,money);
    }

}
```

这个时候就模拟了转账过程中出现异常的情况，正确的操作应该是转账出问题了，`Tom`应该还是900，`Jerry`应该还是1100，但是真正运行后会发现，并没有像我们想象的那样，`Tom`账户为800而`Jerry`还是1100,100块钱凭空消失了

当程序出问题后，我们需要让事务进行回滚，而且这个事务应该是加在业务层上，而Spring的事务管理就是用来解决这类问题的。



## 13.4:解决问题

Spring事务管理具体的实现步骤为:

步骤1:在需要被事务管理的方法上添加注解`@Transactional`

```java
public interface AccountService {
    /**
     * 转账操作
     * @param out 传出方
     * @param in 转入方
     * @param money 金额
     */
    //配置当前接口方法具有事务
    public void transfer(String out,String in ,Double money) ;
}

@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;
    
	@Transactional
    public void transfer(String out,String in ,Double money) {
        accountDao.outMoney(out,money);
        int i = 1/0;
        accountDao.inMoney(in,money);
    }
}
```

注意:

@Transactional可以写在接口类上、接口方法上、实现类上和实现类方法上

* 写在接口类上，该接口的所有实现类的所有方法都会有事务
* 写在接口方法上，该接口的所有实现类的该方法都会有事务
* 写在实现类上，该类中的所有方法都会有事务
* 写在实现类方法上，该方法上有事务
* 建议写在实现类或实现类的方法上



步骤2:在JdbcConfig类中配置事务管理器

```java
public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String userName;
    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(userName);
        ds.setPassword(password);
        return ds;
    }

    //配置事务管理器，mybatis使用的是jdbc事务
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource){
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }
}
```

**注意：**事务管理器要根据使用技术进行选择，Mybatis框架使用的是JDBC事务，可以直接使用`DataSourceTransactionManager`



步骤3：开启事务注解

在SpringConfig的配置类中开启

```java
@Configuration
@ComponentScan("com.zzx")
@PropertySource("classpath:jdbc.properties")
@Import({JdbcConfig.class,MybatisConfig.class})
//开启注解式事务驱动
@EnableTransactionManagement
public class SpringConfig {
}
```



| 名称 | @EnableTransactionManagement           |
| ---- | -------------------------------------- |
| 类型 | 配置类注解                             |
| 位置 | 配置类定义上方                         |
| 作用 | 设置当前Spring环境中开启注解式事务支持 |



| 名称 | @Transactional                                               |
| ---- | ------------------------------------------------------------ |
| 类型 | 接口注解  类注解  方法注解                                   |
| 位置 | 业务层接口上方  业务层实现类上方  业务方法上方               |
| 作用 | 为当前业务层方法添加事务（如果设置在类或接口上方则类或接口中所有方法均添加事务） |



## 13.5:Spring事务角色

这节中我们重点要理解两个概念，分别是`事务管理员`和`事务协调员`。

未开启Spring的事务之前:

![image-20221003175212956]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003175212956.png)

* AccountDao的outMoney因为是修改操作，会开启一个事务T1
* AccountDao的inMoney因为是修改操作，会开启一个事务T2
* AccountService的transfer没有事务，
  * 运行过程中如果没有抛出异常，则T1和T2都正常提交，数据正确
  * 如果在两个方法中间抛出异常，T1因为执行成功提交事务，T2因为抛异常不会被执行
  * 就会导致数据出现错误



开启Spring的事务管理后：

![image-20221003175303778]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003175303778.png)

* transfer上添加了@Transactional注解，在该方法上就会有一个事务T
* AccountDao的outMoney方法的事务T1加入到transfer的事务T中
* AccountDao的inMoney方法的事务T2加入到transfer的事务T中
* 这样就保证他们在同一个事务中，当业务层中出现异常，整个事务就会回滚，保证数据的准确性



通过上面例子的分析，我们就可以得到如下概念:

- 事务管理员：发起事务方，在Spring中通常指代业务层开启事务的方法
- 事务协调员：加入事务方，在Spring中通常指代数据层方法，也可以是业务层方法



## 13.5:Spring事务属性

上一节我们介绍了两个概念，事务的管理员和事务的协同员，对于这两个概念具体做什么的，我们待会通过

案例来使用下。除了这两个概念，还有就是事务的其他相关配置都有哪些，就是我们接下来要学习的内容。

在这一节中，我们主要学习三部分内容`事务配置`、`转账业务追加日志`、`事务传播行为`

### 1:事务配置之回滚

![image-20221003203749998]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003203749998.png)

上面这些属性都可以在`@Transactional`注解的参数上进行设置。

* readOnly：true只读事务，false读写事务，增删改要设为false,查询设为true。
* timeout:设置超时时间单位秒，多长时间之内事务没有提交成功就自动回滚，-1表示不设置超时时间。
* rollbackFor:当出现指定异常进行事务回滚
* noRollbackFor:当出现指定异常不进行事务回滚

  * 思考:出现异常事务会自动回滚，这个是我们之前就已经知道的
  * noRollbackFor是设定对于指定的异常不回滚，这个好理解
  * rollbackFor是指定回滚异常，对于异常事务不应该都回滚么，为什么还要指定?
  * 这块需要更正一个知识点，并不是所有的异常都会回滚事务，比如下面的代码就不会回滚

```java
public interface AccountService {
    /**
     * 转账操作
     * @param out 传出方
     * @param in 转入方
     * @param money 金额
     */
    //配置当前接口方法具有事务
    public void transfer(String out,String in ,Double money) throws IOException;
}

@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;
	@Transactional
    public void transfer(String out,String in ,Double money) throws IOException{
        accountDao.outMoney(out,money);
        //int i = 1/0; //这个异常事务会回滚
        if(true){
            throw new IOException(); //这个异常事务就不会回滚
        }
        accountDao.inMoney(in,money);
    }

}
```

出现这个问题的原因是，Spring的事务只会对`Error异常`和`RuntimeException异常`及其子类进行事务回

顾，其他的异常类型是不会回滚的，对应IOException不符合上述条件所以不回滚

此时就可以使用rollbackFor属性来设置出现IOException异常不回滚

```java
@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;
	 @Transactional(rollbackFor = {IOException.class})
    public void transfer(String out,String in ,Double money) throws IOException{
        accountDao.outMoney(out,money);
        //int i = 1/0; //这个异常事务会回滚
        if(true){
            throw new IOException(); //这个异常事务就不会回滚
        }
        accountDao.inMoney(in,money);
    }

}
```

上面代码的意思是：出现IOException的时候，也会回滚，而不仅仅是Error 和 RuntimeException 及其子类

* rollbackForClassName等同于rollbackFor,只不过属性为异常的类全名字符串
* noRollbackForClassName等同于noRollbackFor，只不过属性为异常的类全名字符串

### 2:事务配置之隔离级别

Isolation设置事务的隔离级别

* DEFAULT   :默认隔离级别, 会采用数据库的隔离级别
* READ_UNCOMMITTED : 读未提交
* READ_COMMITTED : 读已提交
* REPEATABLE_READ : 重复读取
* SERIALIZABLE: 串行化

### 3:事务配置之传播行为

 需求：转账业务追加日志案例

在前面的转案例的基础上添加新的需求，完成转账后记录日志。

- 需求：实现任意两个账户间转账操作，并对每次转账操作在数据库进行留痕
- 需求微缩：A账户减钱，B账户加钱，数据库记录日志

基于上述的业务需求，我们来分析下该如何实现:

①：基于转账操作案例添加日志模块，实现数据库中记录日志

②：业务层转账操作（transfer），调用减钱、加钱与记录日志功能

需要注意一点就是，我们这个案例的预期效果为:

`无论转账操作是否成功，均进行转账操作的日志留痕`

具体的实现先不写了

就是一个方法，能够添加日志的方法：`logService.log(out,in,money)`

```java
public interface AccountService {
    /**
     * 转账操作
     * @param out 传出方
     * @param in 转入方
     * @param money 金额
     */
    //配置当前接口方法具有事务
    public void transfer(String out,String in ,Double money)throws IOException ;
}
@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;
    @Autowired
    private LogService logService;
	@Transactional
    public void transfer(String out,String in ,Double money) {
        try{
            accountDao.outMoney(out,money);
            accountDao.inMoney(in,money);
        }finally {
            logService.log(out,in,money);
        }
    }
}
```

也就是在 finally 里加上 记录日志

* 当程序正常运行，tbl_account表中转账成功，tbl_log表中日志记录成功

* 当转账业务之间出现异常(int i =1/0),转账失败，tbl_account成功回滚，但是tbl_log表未添加数据
* 这个结果和我们想要的不一样，什么原因?该如何解决?
* 失败原因:日志的记录与转账操作隶属同一个事务，同成功同失败
* 最终效果:无论转账操作是否成功，日志必须保留

![image-20221003210629669]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003210629669.png)

对于上述案例的分析:

* log方法、inMoney方法和outMoney方法都属于增删改，分别持有事务 `T1,T2,T3`【事务协调员】
* transfer因为加了`@Transactional`注解，也开启了总体的事务T【事务管理员】
* 前面我们讲过Spring事务会把`T1,T2,T3`都加入到事务T中
* 所以当转账失败后，`所有的事务都回滚`，导致日志没有记录下来
* 这和我们的需求不符，这个时候`我们就想能不能让log方法单独是一个事务呢`

要想解决这个问题，就需要用到事务传播行为，所谓的事务传播行为指的是：



事务传播行为：事务协调员【各自的事务】对事务管理员【@Transcation】所携带事务的处理态度

具体如何解决，就需要用到之前我们没有说的`propagation属性`。



修改logService改变事务的传播行为

```java
@Service
public class LogServiceImpl implements LogService {

    @Autowired
    private LogDao logDao;
	//propagation设置事务属性：传播行为设置为当前操作需要新事务
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void log(String out,String in,Double money ) {
        logDao.log("转账操作由"+out+"到"+in+",金额："+money);
    }
}
```

运行后，就能实现我们想要的结果，不管转账是否成功，都会记录日志

propagation = Propagation.REQUIRES_NEW【不管业务层有没有大的事务，我这个方法都新开一个事务】



事务传播行为的可选值：

![image-20221003211423284]( https://zzx-note.oss-cn-beijing.aliyuncs.com/spring/image-20221003211423284.png)

对于我们开发实际中使用的话，因为默认值需要事务是常态的。根据开发过程选择其他的就可以了，例如案例中需要新事务就需要手工配置。其实入账和出账操作上也有事务，采用的就是默认值