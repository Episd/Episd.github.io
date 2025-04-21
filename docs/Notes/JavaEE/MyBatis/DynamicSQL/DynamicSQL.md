## 动态SQL中的元素

以下是动态SQL中的常用元素

| 元素                | 说明                                           |
|:---------------------:|:----------------------------------------------:|
| `<if>`              | 判断语句，用于单条件判断|
| `<choose>` (`<when>`、`<otherwise>`) | 相当于Java中的switch...case...default语句，用于多条件判断 |
| `<where>`           | 简化SQL语句中where的条件判断 |
| `<trim>`            | 可以灵活地去除多余的关键字  |
| `<set>`             | 用于SQL语句的动态更新 |
| `<foreach>`         | 循环语句，常用于in语句等列举条件中  |

## `<if>` 元素

我自己的理解，就是将 SQL 的一部分包含在 `<if>` 元素中，如果 test 属性的条件满足，就将该部分拼接到 `<if>` 元素的位置。

具体例子如下：

=== "POJO类"
    ```Java
    package com.yuzhuojun.pojo;

    public class Customer {
        private int id;
        private String username;
        private String jobs;
        private String phone;

        getter&setter

        @Override
        public String toString() {
            ...
        }
    }
    ```

=== "`<if>` 标签"
    ```xml
    <mapper namespace="com.example.mapper.CustomerMapper">
        <!-- 输入数据类型为Customer, 返回类型也为Customer -->
        <select id="findByNameAndJobs"
                parameterType="com.example.pojo.Customer"
                resultType="com.example.pojo.Customer">
            SELECT * FROM t_customer
            WHERE 1=1
            <if test="username != null and username != ''">
                AND username LIKE concat('%', #{username},'%')
            </if>
            <if test="jobs !=null and jobs != ' '">
                AND jobs= #{jobs}
            </if>
        </select>
    </mapper>
    ```
=== "测试方法"
    ```Java
    public void ifTest(){
        SqlSession session = MyBatisUtils.getSession();
        try {
            Customer customer = new Customer();
            customer.setUsername("小方");
            List<Customer> customers = 
                session.selectList("com.yuzhuojun.mapper.CustomerMapper.findByNameAndJobs", customer);
            for (Customer custom : customers) {
                System.out.println(custom);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            session.commit();
            session.close();
        }
    }
    ```

??? note "test属性的内容"
    test 属性中的内容是 OGNL 表达式，只能进行常见的逻辑判断，比如 `==`（等于），`!=`（不等于），`>`（大于），`<`（小于），`&&`（与），`||`（或）等。
    其中的属性名必须和 SQL 映射到的 POJO 定义的属性名完全一致，也可以使用对象的方法。若 POJO 中有一个方法返回数据，也可以用 `()` 来调用该方法。
    例如`"getJobs() != null"`

在代码运行时，如果 test 中的语句为真，那么其包含的字段会拼接在该位置，如：

=== "第一个 `<if>` 中的test语句满足时的SQL"
    ``` SQL
    SELECT * FROM t_customer
    WHERE 1=1
    AND username LIKE '%xxxx%'
    ```

=== "第二个 `<if>` 中的test语句满足时的SQL"
    ``` SQL
    SELECT * FROM t_customer
    WHERE 1=1
    AND jobs= xxx
    ```

=== "两个都满足时的SQL"
    ``` SQL
    SELECT * FROM t_customer
    WHERE 1=1
    AND username LIKE '%xxxx%'
    AND jobs= xxx
    ```

其中的 `1=1` 可以让我们在书写 SQL 片段时不需要考虑 AND 是否加，所有的待拼接 SQL 都可以加 AND

## `<choose><when><otherwise>` 元素

这 3 个元素组合使用，就相当于 `if...else...`，与 `<if>` 不同，他只会选择一条语句。`<if>` 元素会选择所有满足 `test` 的 SQL 语句拼接。这 3 个元素则从上到下选择第一次满足的。

具体例子如下：

=== "`<choose><when><otherwise>` 标签"
    ```xml
    <select id="findByNameOrJobs"
            parameterType="com.yuzhuojun.pojo.Customer"
            resultType="com.yuzhuojun.pojo.Customer">
        SELECT * FROM t_customer
        WHERE 1=1
        <choose>
            <when test="username != null and username != ''">
                AND username LIKE concat('%', #{username}, '%')
            </when>
            <when test="jobs != null and jobs != ''">
                AND jobs = #{jobs}
            </when>
            <otherwise>
                AND phone is NOT NULL
            </otherwise>
        </choose>
    </select>
    ```
=== "测试方法"
    ```Java
    public void chooseTest(){
        SqlSession session = MyBatisUtils.getSession();
        try {
            Customer customer = new Customer();
    //      customer.setUsername("小方");
            customer.setJobs("纪律委员");
            List<Customer> customers =
                    session.selectList("com.yuzhuojun.mapper.CustomerMapper.findByNameOrJobs", customer);
            printResult(customers);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            session.commit();
            session.close();
        }
    }
    ```

## `<where><trim>` 元素

在不使用这两个元素时，要使用 `1=1` 来规避一些会导致 SQL 语句出错的情况（比如拼接导致多余的 AND，或者所有条件都不满足导致的空 WHERE 子句）。`<where>` 会在其中存在 SQL 时才添加 WHERE 关键字，并且删除多余的 AND、OR，而 `<trim>` 可以为 SQL 添加指定前缀（例如 WHERE）或者删除前缀（例如 AND、OR）

具体例子如下：

=== "`<where>` 标签"
    ```xml
    <select id="findByNameOrJobsUseWhere"
            parameterType="com.yuzhuojun.pojo.Customer"
            resultType="com.yuzhuojun.pojo.Customer">
        SELECT * FROM t_customer
        <where>
            <if test="username != null and username != ''">
                AND username LIKE concat('%', #{username},'%')
            </if>
            <if test="jobs != null and jobs != ''">
                AND jobs= #{jobs}
            </if>
        </where>
    </select>
    ```
=== "`<trim>` 标签"
    ```xml
    <select id="findByNameOrJobsUseTrim"
            parameterType="com.yuzhuojun.pojo.Customer"
            resultType="com.yuzhuojun.pojo.Customer">
        SELECT * FROM t_customer
        <trim prefix="WHERE" prefixOverrides="AND">
            <if test="username != null and username != ''">
                AND username LIKE concat('%', #{username},'%')
            </if>
            <if test="jobs != null and jobs != ''">
                AND jobs= #{jobs}
            </if>
        </trim>
    </select>
    ```
=== "测试方法"
    ```Java
    public void whereOrTrimTest(){
        SqlSession session = MyBatisUtils.getSession();
        try {
            Customer customer = new Customer();
            customer.setUsername("小方");
            customer.setJobs("心理委员");
    //      List<Customer> customers =
    //              session.selectList("com.yuzhuojun.mapper.CustomerMapper.findByNameOrJobsUseWhere", customer);
            List<Customer> customers =
                    session.selectList("com.yuzhuojun.mapper.CustomerMapper.findByNameOrJobsUseTrim", customer);
            printResult(customers);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            session.commit();
            session.close();
        }
    }
    ```

`<trim>` 元素的属性：

| 属性                  | 说明                           |
|:---------------------:|:------------------------------:|
| prefix                | 指定给 SQL 语句增加的前缀       |
| prefixOverrides       | 指定 SQL 语句中要去掉的前缀字符串|
| suffix                | 指定给 SQL 语句增加的后缀       |
| suffixOverrides       | 指定 SQL 语句中要去掉的后缀字符串 |

## `<set>` 元素

`<set>` 用于 `<update>`元素，作用与前面的 `<where>` 类似，可以结合 `<if>` 条件来动态选择需要更新的属性并为 SQL 添加 `SET` 关键字。

具体例子如下：

=== "`<set>` 标签"
    ```xml
    <update id="updateBySet" parameterType="com.yuzhuojun.pojo.Customer">
        UPDATE t_customer
        <set>
            <if test="username != null and username != ''">
                username = #{username},
            </if>
            <if test="jobs != null and jobs != ''">
                jobs = #{jobs},
            </if>
            <if test="phone != null and phone != ''">
                phone = #{phone},
            </if>
        </set>
        WHERE id = #{id}
    </update>
    ```
=== "测试方法"
    ```Java
    public void whereTest(){
        SqlSession session = MyBatisUtils.getSession();
        try {
            Customer customer = new Customer();
            customer.setId(1);
            customer.setUsername("方哥");
            customer.setJobs("班长");
            customer.setPhone("16666666666");
            int effectedRows = session.update("com.yuzhuojun.mapper.CustomerMapper.updateBySet", customer);
            System.out.println(effectedRows);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            session.commit();
            session.close();
        }
    }
    ```

上述例子中，具体修改的列会根据对应属性是否存在来决定。同样地，也可以使用 `<trim>` 元素添加前缀 `SET`，删除后缀 `,` 来实现 `<set>` 的功能

???+ warning "注意"
    `<set>` 与 `<if>` 结合使用时，所有传入的更新字段不能同时为空

## `<foreach>` 元素

### `<foreacht>` 元素更新数组
