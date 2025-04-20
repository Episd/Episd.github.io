## MyBatis 映射文件中的常用元素

|属性|说明|
|:-:|:-:|
|mapper|映射文件的根元素，该元素只有一个namespace属性（命名空间）|
|cache|配置给定命名空间的缓存|
|cache-ref|从其他命名空间引用缓存配置|
|resultMap|描述数据库结果集和对象的对应关系|
|sql|可以重用的SQL块，也可以被其他语句使用。|
|insert|用于映射插入语句|
|delete|用于映射删除语句|
|update|用于映射更新语句|
|select|用于映射查询语句|

namespace 属性的两个作用：

- 区分不同的 mapper，全局唯一
- 绑定DAO接口

绑定 DAO 接口的具体用法：

=== "步骤1 UserDao.java"

    ```Java
    package com.example.dao;

    public interface UserDao {
        User getUserById(int id);
    }
    ```

=== "步骤2 UserMapper.xml"

    ```xml 
    <mapper namespace="com.example.dao.UserDao">
        <select id="getUserById" resultType="com.example.model.User"> <!-- (1)! -->
            SELECT * FROM user WHERE id = #{id}
        </select>
    </mapper>
    ```

    1. id 需与接口方法名一致，namespace必须与接口全限定名完全一致。

=== "步骤3 mybatis-config.xml"

    ```xml
    <mappers>
        <!-- 方式1：直接指定Mapper XML路径 -->
        <mapper resource="com/example/dao/UserDao.xml"/>
        <!-- 方式2：扫描包（需接口与XML同名且同包） -->
        <package name="com.example.dao"/>
        <!-- 方式3：指定接口类（需XML与接口同包） -->
        <mapper class="com.example.dao.UserDao"/>
    </mappers>
    ```

=== "步骤4 UserTest.java"

    ```java
    // 通过SqlSession获取动态代理对象
    UserDao userDao = sqlSession.getMapper(UserDao.class);
    User user = userDao.getUserById(1); // 自动执行对应SQL
    ```

??? note "`<mapper>`元素如何却别不同的 XML 文件"
    在不同的映射文件中，<mapper>元素的子元素的id可以相同，MyBatis通过<mapper>元素的namespace属性值和子元素的id联合区分不同的Mapper.xml文件。接口中的方法与映射文件中SQL语句id应一一对应

### `<select>` 元素

```xml title="查询"
<select id="findUserById" parameterType="Integer"
    resultType="com.itheima.pojo.User">
    select * from users where id = #{id}
</select>
```

`<select>` 元素的常用属性

|属性|说明|
|:-:|:-:|
|id|`<select>` 元素的唯一标识|
|parameterType|可选属性，指定SQL语句所需参数的类型，其默认值是unset|
|resultType|用于指定执行这条SQL语句返回的是哪个类实体|
|resultMap|表示外部resultMap的命名引用, resultMap和resultType不能同时使用|
|flushCache|用于指定是否需要MyBatis清空本地缓存和二级缓存|
|useCache|用于控制二级缓存的开启和关闭。|
|timeout|用于设置超时时间，单位为秒|
|fetchSize|获取记录的总条数设定，默认值是unset|
|statementType|用于设置MyBatis预处理类|
|resultSetType|表示结果集的类型，它的默认值是unset|

### `<insert>` 元素

```xml title="插入操作"
<insert id="addUser" parameterType="com.itheima.pojo.User">
    insert into users(uid,uname,uage)values(#{uid},#{uname},#{uage})
</insert>
```

很多时候，执行插入操作后，需要获取插入成功的数据生成的主键值。

如果使用的数据库支持主键自动增长（如MySQL和SQL Server），那么可以通过keyProperty属性指定POJO类的某个属性接收主键返回值（通常会设置到id属性上），然后将useGeneratedKeys的属性值设置为true。

```xml title="示例代码"
<insert id="addUser" parameterType="com.itheima.pojo.User"
    keyProperty="uid" useGeneratedKeys="true" >
    insert into users(uid,uname,uage) values(#{uid},#{uname},#{uage})
</insert>
```

如果使用的数据库不支持主键自动增长，那么使用MyBatis提供的<selectKey>元素来自定义主键。

```xml title="示例代码"
<selectKey
    keyProperty="id" resultType="Integer"
    order="BEFORE"	statementType="PREPARED">
```

### `<update>` 元素

在执行完`<update>`元素中定义的SQL语句后，会返回更新的记录数量

```xml title="更新操作"
<update id="updateUser" parameterType="com.itheima.pojo.User">
    update users set uname= #{uname},uage = #{uage} where uid = #{uid}	
</update>
```

### `<delete>` 元素

在执行完 `<delete>` 元素中的SQL语句之后，会返回删除的记录数量

```xml title="删除操作"
<delete id="deleteUser" parameterType="Integer">
    delete from users where uid=#{uid}
</delete>
```

### `<sql>` 元素

`<sql>` 元素用于定义可重用的SQL代码片段

```xml title="根据客户id查询客户信息的SQL片段 "
<!--定义要查询的表 -->
<sql id=“someinclude">from <include refid="${include_target}" /></sql>
<!--定义查询列 --><sql id=“userColumns">	uid,uname,uage 	</sql>
<!--根据客户id查询客户信息 -->
<select id="findUserById" parameterType="Integer" 
         resultType="com.itheima.pojo.User">	select 
    <include refid="userColumns"/>
    <include refid="someinclude">
        <property name="include_target" value="users" /></include>
    where uid = #{uid}
</select> 
```

### `<resultMap>` 元素

数据表中的列和需要返回的对象的属性可能不会完全一致，这种情况下MyBatis不会自动赋值，这时就需要使用 `<resultMap>` 元素进行结果集映射

以下为具体案例

=== "xx.sql"
    ```sql
    USE mybatis;
    CREATE TABLE t_student(
    sid INT PRIMARY KEY AUTO_INCREMENT,
    sname VARCHAR(50),
    sage INT
    );
    INSERT INTO t_student(sname,sage) VALUES('Lucy',25);
    INSERT INTO t_student(sname,sage) VALUES('Lili',20);
    INSERT INTO t_student(sname,sage) VALUES('Jim',20);
    ```

=== "Student.java"
    ```Java
    package com.itheima.pojo;
    public class Student {
        private Integer id;            // 主键id
        private String name;        // 学生姓名
        private Integer age;         // 学生年龄
        // 省略getter/setter方法
        @Override
        public String toString() {
    return "User [id=" + id + ", name=" + name + ", age=" + age + "]";}
    }
    ```

=== "StudentMapper.xml"
    ```xml
    <!-- 只显示mapper元素的内容-->
    <mapper namespace="com.itheima.mapper.StudentMapper">
        <resultMap type="com.itheima.pojo.Student" id="studentMap">
            <id property="id" column="sid"/>
            <result property="name" column="sname"/>
            <result property="age" column="sage"/>    
        </resultMap>
        <select id="findAllStudent" resultMap="studentMap">
        select * from t_student
        </select>
    </mapper>
    ```

=== "mybatis-config.xml"
    ```xml
    <mapper resource="com/itheima/mapper/StudentMapper.xml">
    </mapper>
    ```

=== "MyBatisTest.java"
    ```Java
    public class MyBatisTest {
        private SqlSessionFactory sqlSessionFactory;
        private SqlSession sqlSession;
        // init()方法省略
        @Test
        public void findAllStudentTest() {
            List<Student> list = 
            sqlSession.selectList("com.itheima.mapper.StudentMapper.
        findAllStudent");
            for (Student student : list) {	System.out.println(student);}}
        // destory()方法省略
    }
    ```
