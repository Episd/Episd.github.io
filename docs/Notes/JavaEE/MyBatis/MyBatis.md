## MyBatis 核心对象

### SqlSessionFactoryBuilder

SqlSessionFactoryBuilder 是一个工厂对象，有三种构造方法，主要是用：字节流(InputStream)，字符流(Reader)，配置对象(Configuration)来构造。

``` java
<!-- 字节流 -->
build(InputStream inputStream,String environment,Properties properties)
<!-- 字符流 -->
build(Reader reader,String environment,Properties properties)
<!-- 类（对象） -->
build(Configuration config)
```

SqlSessionFactoryBuilder 是线程安全的，有一个就行，建议使用单例模式设计。

### SqlSessionFactory

SqlSessionFactory 的方法：

|    方法名称    |    