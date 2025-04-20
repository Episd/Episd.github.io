## SqlSessionFactoryBuilder

SqlSessionFactoryBuilder 是一个工厂对象，用三种方法构造 SqlSessionFactory 对象，主要是用：字节流(InputStream)，字符流 (Reader)，配置对象 (Configuration) 来构造。

```java title="SqlSessionFactoryBuilder构造SqlSessionFactory对象的方法"
<!-- 字节流 -->
build(InputStream inputStream,String environment,Properties properties)
<!-- 字符流 -->
build(Reader reader,String environment,Properties properties)
<!-- 类(对象) -->
build(Configuration config)
```

通过过读取XML配置文件的方式构造SqlSessionFactory对象的关键代码如下所示。 

```java title="读取XML文件的方式构造SqlSessionFactory对象 "
// 读取配置文件
InputStream inputStream = Resources.getResourceAsStream("配置文件位置");
// 根据配置文件构建SqlSessionFactory
SqlSessionFactory sqlSessionFactory = 
new SqlSessionFactoryBuilder().build(inputStream);
```

SqlSessionFactory 是线程安全的，有一个就行，建议使用单例模式设计。

## SqlSessionFactory

SqlSessionFactory 用于开启 SqlSession 事务。

SqlSessionFactory 的方法：

|    方法名称    |    描述    |
| -------------- | ---------- |
|SqlSession openSeeeion()|开启一个事务|
|...|用到了再补充|

## SqlSession

SqlSession 是应用程序与持久层之间执行交互的一个单线程对象，就是用来操作数据库的。其中主要包含SQL操作的方法。

SqlSession对象常用方法

|    方法名称    |    描述    |
| -------------- | ---------- |
|<T> T selectOne(String statement)|查询一条(Statement为包.select元素的id)|
|<T> T selectOne(String statement, Object parameter)|查询一条(parameter是查询用的参数)|
|<E> List<E> selectList(String statement)|查询多条|
|<E> List<E> selectList(String statement, Object parameter)|查询多条|
|int insert(String statement)|插入|
|int insert(String statement, Object parameter)||
|int update(String statement)|更新|
|int update(String statement, Object parameter)||
|int delete(String statement)|删除|
|int delete(String statement, Object parameter)||
|void commit()|提交事务|
|void rollback()|回滚事务的方法。|
|void close()|关闭SqlSession对象。|
|...|用到了再补充|

SqlSession对象的是用范围

每一个线程都应该有自己的 SqlSession 对象，该对象不能共享。

```Java title="SqlSession对象的用法"
SqlSession sqlSession = sqlSessionFactory.openSession();
try {	
    // 此处执行持久化操作
} finally {		sqlSession.close();		}
```