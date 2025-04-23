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

```Java
public class OrderService {
    public void printLog() {
        System.out.println("[日志] 正在执行订单服务！");
    }
    public void createOrder() {
        printLog();
        System.out.println("下单成功！");
    }

    public void cancelOrder() {
        printLog();
        System.out.println("订单取消！");
    }

    public void refundOrder() {
        printLog();
        System.out.println("退款成功！");
    }
}
```

这样虽然在维护时时不用修改所有的方法，耦合度还是太高了，有新功能时还是要修改 OrderService 这个类。那么就可以引入代理这个思想，通过一个代理对象将类似**日志**这种业务在代码执行的时候**注入**进原来的对象中，实现原来对象的**增强**

我们先实现一个代理类，用于“增强”原始的 OrderService：

```Java
public class OrderServiceProxy {
    private OrderService target;

    public OrderServiceProxy(OrderService target) {
        this.target = target;
    }

    public void createOrder() {
        System.out.println("[日志] 正在执行 createOrder 方法");
        target.createOrder();
    }

    public void cancelOrder() {
        System.out.println("[日志] 正在执行 cancelOrder 方法");
        target.cancelOrder();
    }

    public void refundOrder() {
        System.out.println("[日志] 正在执行 refundOrder 方法");
        target.refundOrder();
    }
}
```



## JDK 动态代理

