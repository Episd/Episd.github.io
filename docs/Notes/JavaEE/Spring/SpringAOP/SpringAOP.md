AOP(Aspect Oriented Programming) 即面向切面编程，他的核心思想是：**将那些与业务无关，但对很多模块都重要的功能（比如日志、安全、事务等）抽取出来单独处理**，不要在业务代码里到处写重复的逻辑。

## AOP 术语

| 术语 | 含义 |
| :-: | :-: |
| 连接点（Join Point） | 	程序中可以被“切入”的点（比如方法的执行） |
| 切入点 （Pointcut） | 被选择的那些“连接点”，用表达式定义 |
| 通知/增强（Advice） | 要插入的具体代码，比如日志、事务等 |
| 切面（Aspect） | 通知 + 切入点的组合，形成一个“完整的横切功能” |
| 织入（Weaving） | 把切面逻辑“织入”目标对象的过程（运行时完成） |
| 目标对象（Target Object） | 	被代理、被增强的原始对象 |
| 代理对象（Proxy） | 包装目标对象，实际执行通知 + 原始方法 |

## AOP 的作用

场景：你写了一个订单系统，有三个服务方法
=== "订单系统服务类"
    ```Java
    public class OrderService {
        public void createOrder() {
            System.out.println("下单成功！");
        }

        public void cancelOrder() {
            System.out.println("订单取消！");
        }

        public void refundOrder() {
            System.out.println("退款成功！");
        }
    }
    ```

=== "订单系统的具体使用"
    ```Java
    ...
    OrderService oderService = new OrderService();
    ...
    orderService.createOrder();
    ...

    orderService.cancelOrder();
    ...

    orderService.refundOrder();
    ```

现在，订单系统需要在**所有服务**之前增加日志功能：

```
[日志] 正在执行 xxx 方法
```

那么最初、最暴力的做法就是修改订单系统服务类的源代码，在每个方法里加一行日志输出：

```Java
public void createOrder() {
    System.out.println("[日志] 正在执行订单服务！");
    System.out.println("下单成功！");
}
public void cancelOrder() {
    System.out.println("[日志] 正在执行订单服务！");
    System.out.println("订单取消！");
}
public void refundOrder() {
    System.out.println("[日志] 正在执行订单服务！");
    System.out.println("退款成功！");
}
```

这样太麻烦，每加一种功能，就要大刀阔斧的修改源代码，而且写的内容量大（如果有几百个方法呢？），而且很难维护（未来日志格式万一更改，就要改所有方法）。

此时可以引入 AOP 部分思想，将日志这个所有方法都有的业务抽离出来，形成一个切面

```Java title="切面类"
public class LogAspect {
    public void logBefore() {
        System.out.println("[日志] 正在执行订单服务！");
    }
}
```

然后就可以在这个切面的所有切点（用到该切面的通知逻辑的所有方法）中书写对应的输出

```Java
public class OrderService {
    public void createOrder() {
        LogAspect logAspect = new LogAspect();
        logAspect.logBefore();
        System.out.println("下单成功！");
    }

    public void cancelOrder() {
        LogAspect logAspect = new LogAspect();
        logAspect.logBefore();
        System.out.println("订单取消！");
    }

    public void refundOrder() {
        LogAspect logAspect = new LogAspect();
        logAspect.logBefore();
        System.out.println("退款成功！");
    }
}
```

这样虽然在维护时时不用修改所有的方法，耦合度还是太高了，有新功能时还是要修改OrderService 这个类。要使这两者尽可能解耦，那么就可以引入代理这个思想，通过一个代理对象将代码中的 `logAspect.logBefore();` 这一行代码注入进原来对象中

我们先实现一个代理类，用于“增强”原始的 OrderService：

```Java
public class OrderServiceProxy {
    // 要增强的目标对象
    private OrderService target;
    public OrderServiceProxy(OrderService target) {
        this.target = target;
    }

    public void createOrder() {
        LogAspect logAspect = new LogAspect();
        logAspect.logBefore();
        target.createOrder();
    }

    public void cancelOrder() {
        LogAspect logAspect = new LogAspect();
        logAspect.logBefore();
        target.cancelOrder();
    }

    public void refundOrder() {
        LogAspect logAspect = new LogAspect();
        logAspect.logBefore();
        target.refundOrder();
    }
}
```

在使用时，不再使用 `OrderService` 类，而是使用代理：

```Java
OrderService orderService = new OrderService();
OrderServiceProxy proxy = new OrderServiceProxy(orderService);

proxy.createOrder();
proxy.cancelOrder();
proxy.refundOrder();
```

这样，业务类 `OrderService` 保持干净，与日志这一类事务可以完全分离开。但是还存在一些问题，如果有一百个方法，这样还是得写一百次方法增强代码，于是，可以引入接口+JDK动态代理来解决这一问题。

## JDK 动态代理

先为 OrderService 抽象出一个接口，再让业务类实现它

```Java title="OrderService的接口"
public interface IOrderService {
    void createOrder();
    void cancelOrder();
    void refundOrder();
}
```

```Java title="OrderService的实现"
public class OrderService implements IOrderService {
    public void createOrder() {
        System.out.println("下单成功！");
    }
    public void cancelOrder() {
        System.out.println("订单取消！");
    }
    public void refundOrder() {
        System.out.println("退款成功！");
    }
}
```

接着，使用 Java 的动态代理统一处理日志增强，定义代理类时需要实现 `InvocationHandle` 接口

```Java title="切面类"
// 自定义一个切面类用于通知处理
public class LogAspect {
    public void logBefore(String methodName) {
        System.out.println("[日志] 正在执行 " + methodName + " 方法");
    }

    public void logAfter(String methodName) {
        System.out.println("[日志] 执行完 " + methodName + " 方法");
    }
}
```

```Java title="代理类"
package com.example.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

// 定义代理类
public class MyProxy implements InvocationHandler {
    private Object target; // 被代理对象

    // 构造方法接收目标对象
    public MyProxy(Object target) {
        this.target = target;
    }

    // 创建代理对象的方法
    public Object createProxy() {
        // 获取目标对象的类加载器
        ClassLoader classLoader = target.getClass().getClassLoader();
        // 获取目标对象实现的接口
        Class[] interfaces = target.getClass().getInterfaces();
        // 返回代理对象
        return Proxy.newProxyInstance(classLoader, interfaces, this);
    }

    // 处理方法调用时的增强逻辑
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) {
        LogAspect logAspect = new LogAspect();
        // 在目标方法执行前加日志(前增强)
        logAspect.logBefore(method.getName());

        // 执行目标方法
        Object result = method.invoke(target, args);

        // 在目标方法执行后加日志(后增强)
        logAspect.logAfter(method.getName());

        return result;
    }
}
```

在使用时，构造代理对象，为其注入目标对象之后，只需要用代理对象 + **原对象接口方法**即可增强原对象的方法。

使用方法如下：

```Java
public class Main {
    public static void main(String[] args) {
        // 创建订单服务实例
        OrderService orderService = new OrderService();

        // 创建代理类实例，并生成代理对象
        MyProxy proxy = new MyProxy(orderService);
        IOrderService orderServiceProxy = (IOrderService) proxy.createProxy();

        // 使用代理对象调用方法
        orderServiceProxy.createOrder();
        orderServiceProxy.cancelOrder();
        orderServiceProxy.refundOrder();
    }
}
```

??? question "为什么代理对象没有实现 `IOrderService` 接口，却可以调用其方法？"
    在 JDK 动态代理机制下，所有实现了 `InvocationHandler` 接口的类（例如此处的 `MyProxy` 类）会拦截对代理对象的方法调用（如此处的 `createOrder()`），并转发到 `invoke()` 方法。

    在 `invoke()` 方法中，首先执行前增强

    然后通过 `method.invoke(target)` 调用目标对象的 `target` 的实际方法（即实际目标对象的 `createOrder()` ）

    在

这样，只需为不同的切面书写不同的通知逻辑即可。

## CGLib 动态代理

与JDK动态代理依靠反射不同，CGLib动态代理依靠字节码操作，创造一个增强后的子类继承自原有父类，通过这个子类对象来实现对原对象的增强。

```Java title="CGLib动态代理的代理类"
package com.example.proxy;

import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

// 定义代理类
public class CGLibProxy implements MethodInterceptor {
    public Object createProxy(Object target) {
        Enhancer enhancer = new Enhancer();         // 构造CGLib的Enhancer工具类
        enhancer.setSuperclass(target.getClass());  // 设置增强的父类
        enhancer.setCallback(this);                 // 设置带有拦截方法的对象，这里就是这个代理类本身，他拥有intercept方法
        return enhancer.create();                   // 生成一个带有增强方法的新类对象
    }

    // (1)
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable{
        // 这里的切面类还是之前那个
        LogAspect logAspect = new LogAspect();

        // 前增强
        logAspect.logBefore(method.getName());

        // 执行目标方法
        Object result = methodProxy.invokeSuper(o, objects);

        // 在目标方法执行后加日志(后增强)
        logAspect.logAfter(method.getName());

        return result;
    }
}
```

1. | 参数名             | 类型                          | 功能说明                                                                                           |
|------------------|-----------------------------|--------------------------------------------------------------------------------------------------|
| `o`              | `Object`                    | **代理对象本身**。生成的子类对象实例（比如`UserDao2 userDao2Plus`那个代理对象）。                              |
| `method`         | `Method`（反射API）          | 当前被拦截的方法对象，提供**方法元信息**（方法名、参数类型、返回类型等）。                                          |
| `objects`        | `Object[]`                  | 方法的参数列表（如果方法带参数，这里就是调用时传入的参数数组；无参数时是空数组）。                                     |
| `methodProxy`    | `MethodProxy`（CGLIB专用）   | 用于高效调用被代理类的原始方法，推荐使用`methodProxy.invokeSuper()`来避免传统反射的性能损失。                         |

## 基于 XML 的 AOP 实现

上述内容还没有涉及到 Spring 的声明式 AOP。接下来做有关 Spring 的 AOP 笔记

``` xml title="声明式AOP用到的依赖包"
<!-- aspectjrt包的依赖 -->
<dependency>
     <groupId>org.aspectj</groupId>
     <artifactId>aspectjrt</artifactId>
     <version>1.9.1</version>	
</dependency>
<!-- aspectjweaver包的依赖 -->
<dependency>
     <groupId>org.aspectj</groupId>
     <artifactId>aspectjweaver</artifactId>
     <version>1.9.6</version>	
</dependency>
```

在使用基于 XML 的 AOP 时，不必再写上面提到的复杂的代理类，只需要定义好切面类，在 spring 的 XML 配置文件中用标签声明好切面、切入点、通知与切入点的关系（具体是前/后增强还是环绕）即可。当然，切面类和具体的服务类都要作为 bean 来管理，一样要写在 spring 的配置中。

具体地，有关 SpringAOP 的标签如下：

| 元素                | 描述                                           |
|:---------------------:|:----------------------------------------------:|
| `<aop:config>`      | Spring AOP配置的根元素                            |
| `<aop:aspect>`      | 配置切面                                      |
| `<aop:advisor>`     | 配置通知器                                     |
| `<aop:pointcut>`    | 配置切点                                      |
| `<aop:before>`      | 配置前置通知,在目标方法执行前实施增强,可以应用于权限管理等功能               |
| `<aop:after>`       | 配置后置通知,在目标方法执行后实施增强,可以应用于关闭流、上传文件、删除临时文件等功能 |
| `<aop:around>`      | 配置环绕方式,在目标方法执行前后实施增强,可以应用于日志、事务管理等功能            |
| `<aop:after-returning>` | 配置返回通知,在目标方法成功执行之后调用通知                     |
| `<aop:after-throwing>`  | 配置异常通知,在方法抛出异常后实施增强,可以应用于处理异常记录日志等功能            |

一个具体的 XML 配置可能类似以下结构：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--UserDAO对象-->
    <bean id="userDao" class="com.example.XMLProxy.DAO.UserDaoImpl" />
    <!--切面类对象-->
    <bean id="xmlAspect" class="com.example.XMLProxy.Aspect.XmlAspect"/>
    <!--AOP-->
    <aop:config>
        <!--全局切入点-->
        <aop:pointcut id="pointcut" expression="execution(* com.example.XMLProxy.DAO.UserDaoImpl.*(..))"/>
        <!--配置切面-->
        <aop:aspect ref="xmlAspect">
            <!--上面的全局切入点也可以写在这里，写在这里只对当前切面生效-->
            <!--配置关系-->
            <aop:before method="before" pointcut-ref="pointcut"/>
        </aop:aspect>
    </aop:config>
</beans>
```

关于切入点的 `<aop:pointcut>` 元素，通常会指定指定id、expression 属性。其中 expression 属性用于指定切入点关联的**切入点表达式**。其基本格式如下：

```XML
execution(modifiers-pattern?ret-type-pattern declaring-type-pattern?
name-pattern(param-pattern) throws-pattern?)
```

其中，各个部分的参数说明如下：

- modifiers-pattern：表示定义的目标方法的访问修饰符，如public、private等。
- ret-type-pattern：表示定义的目标方法的返回值类型，如void、String等。
- declaring-type-pattern：表示定义的目标方法的类路径，如com.itheima.jdk.UserDaoImpl。
- name-pattern：表示具体需要被代理的目标方法，如add()方法。
- param-pattern：表示需要被代理的目标方法包含的参数，本章示例中目标方法参数都为空。
- throws-pattern：表示需要被代理的目标方法抛出的异常类型。

也有一个比较简单的记法：

```xml
execution(返回值 包名.类名.方法名(参数))
<!--完整的形式-->
execution([修饰符] 返回值类型 全限定类名.方法名(参数列表) throws 异常类型)
```

用 `*` 做通配，用 `..` 匹配任意层级、任意个参数。

配置完成后，在测试类中，读取 XML 配置，定义 DAO 对象，直接调用方法即可。 

## 基于注解的 AOP 实现

有关的注解如下：

| 元素                | 描述               |
|:---------------------:|:--------------------:|
| @Aspect             | 配置切面           |
| @Pointcut           | 配置切点           |
| @Before             | 配置前置通知       |
| @After              | 配置后置通知       |
| @Around             | 配置环绕方式       |
| @AfterReturning     | 配置返回通知       |
| @AfterThrowing      | 配置异常通知       |

用注解实现 AOP，只需要在基于 XML 的 AOP 实现中，在切面类中定义一个空方法pointCut()，这个方法也可以叫其他名字，仅仅只是起一个标记作用。为这个空方法添加@pointcut注解，在各个具体的通知方法前加上@Before、@After等注解，并在配置文件中启用自动代理即可

xml 配置文件类似以下结构：

```xml
<!--以上部分省略-->
    <!--配置bean-->
    <bean id="userDao" class="com.example.AnnoAOP.DAO.UserDaoImpl"/>
    <bean id="annoAspect" class="com.example.AnnoAOP.Aspect.AnnoAspect"/>
    <aop:aspectj-autoproxy/>
```

Aspect 类可能类似以下结构：

```Java
@Aspect
public class AnnoAspect {
    @Pointcut("execution(* com.example.AnnoAOP.DAO.UserDaoImpl.* (..))")
    public void pointCut() {
    }
    @Before("pointCut()")
    public void before() {
        System.out.println("前置通知");
    }
}
```