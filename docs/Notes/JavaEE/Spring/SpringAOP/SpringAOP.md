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
        logAspect.logBefore(method.geName());

        // 执行目标方法
        Object result = method.invoke(target, args);

        // 在目标方法执行后加日志(后增强)
        logAspect.logAfter(method.geName());

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

