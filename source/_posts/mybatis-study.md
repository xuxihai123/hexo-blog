---
title:  mybatis使用入门
date: 2016-10-24 10:10:45
categories:
- 后端
tags:
- java
- javaee
---

### mybatis使用入门
1.编写全局配置文件configuration.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <!--给实体类起一个别名 user -->
        <typeAlias type="cn.sky.bookshop.beans.User" alias="User" />
    </typeAliases>
    <!--数据源配置  这块用 mysql数据库 -->
    <environments default="development">
        <environment id="development">
            <transactionManager type="jdbc" />
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/ibatis" />
                <property name="username" value="root" />
                <property name="password" value="215890" />
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!--userMapper.xml装载进来  同等于把“dao”的实现装载进来 -->
        <mapper resource="User.xml" />
        <mapper resource="UserMapper.xml" />
    </mappers>
</configuration>
```
2.这里编写一个MyBatis工具类

```java
package cn.sky.bookshop.utils;
import java.io.IOException;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

public final class MyBatisUtil {

    private static SqlSessionFactory sessionFactory;
    private static String resource = "configuration.xml";

    static {
    try {
        sessionFactory = new SqlSessionFactoryBuilder()//
                .build(Resources//
            .getResourceAsReader(resource));
        // sessionFactory.getConfiguration().addMapper(UserDao.class);
    } catch (IOException e) {
        e.printStackTrace();
    }
    }

    public static SqlSession getSqlSession() {
    return sessionFactory.openSession();
    }
}
```
然后接可以先建立一个测试类加载测试一番了。。

3.按Javabean规范写的一个User类

```java
package cn.sky.bookshop.beans;

public class User {
    private int id;
    private String userName;
    private String password;
    private String email;

    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }

    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    public String getEmail() {
        return email;
    }
    public void setEmail(String email) {
        this.email = email;
    }
    public String getUserName() {
        return userName;
    }
    public void setUserName(String userName) {
        this.userName = userName;
    }
    @Override
    public String toString() {
    return "User [email=" + email + ", id=" + id + ", password=" + password
        + ", username=" + userName + "]";
    }
}
```

4.这里将UserDao接口命名为UserMapper接口

```java
package cn.sky.bookshop.dao;

import cn.sky.bookshop.beans.User;

public interface UserMapper {


    public User findUserByName(String username)throws Exception;

    public void insertUser(User user)throws Exception;

    public void updateUser(User user)throws Exception;

    public void deleteUserById(int id)throws Exception;
}
```

5.关键是这里的UserMapper.xml

```java
<?xml version="1.0" encoding="UTF-8" ?>
 <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- namespace必须是定义的接口全限定名 -->
<mapper namespace="cn.sky.bookshop.dao.UserMapper">
    <!-- 里面id是接口的方法名 parameterType是接口里方法的传入参数 -->
    <insert id="insertUser" parameterType="User">
        insert into
        t_user(id,username,password,email)values(#{id},#{userName},#{password},#{email});
    </insert>
    <!-- resultType这里是接口里方法返回值类型：当方法返回集合时，这里还是设为集合元素类型 -->
    <select id="findUserByName" parameterType="string" resultType="User">
        select * from t_user where username=#{userName};
    </select>

    <update id="updateUser" parameterType="User">
        update t_user set
        username=#{userName},password=#{password},email=#{email} where
        id=#{id};
    </update>

    <delete id="deleteUserById" parameterType="int">
        delete from t_user
        where id=#{id};
    </delete>
</mapper>

```

6.注意：上面的步骤配置有些严格。。因为下面的接口动态代理对象（就是个接口的实现）是根据UserMapper接口和UserMapper.xml文件生成的! 下面是我的DAO测试类

```java
UserMapperTest
package cn.sky.bookshop.test;

import org.apache.ibatis.session.SqlSession;

import cn.sky.bookshop.beans.User;
import cn.sky.bookshop.dao.UserMapper;
import cn.sky.bookshop.utils.MyBatisUtil;

public class UserMapperTest {

    /**
     * @param args
     */
    public static void main(String[] args) {
//      addUserMapper(); 添加
//  findUserMapper(); 查找
//  updateUserMapper();更新
    deleteUserMapper();
    }
    //添加记录
    public static void addUserMapper(){
    SqlSession sqlSession = MyBatisUtil.getSqlSession();
    //获取接口的动态代理对象
    UserMapper um = sqlSession.getMapper(UserMapper.class);
    User user = new User();
    user.setUserName("大侠");
    user.setPassword("daxia");
    user.setEmail("fdasf@qq.com2");
    try {
        um.insertUser(user);
    } catch (Exception e) {
        e.printStackTrace();
    }
    sqlSession.commit();
    sqlSession.close();

    }
    //查找记录
    public static void findUserMapper() {
    SqlSession sqlSession = MyBatisUtil.getSqlSession();
    UserMapper um = sqlSession.getMapper(UserMapper.class);
    User user = null;
    try {
        user = um.findUserByName("大侠");
    } catch (Exception e) {
        e.printStackTrace();
    }
    System.out.println(user);
    sqlSession.close();
    }
    //跟新纪录
    public static void updateUserMapper(){
    SqlSession sqlSession = MyBatisUtil.getSqlSession();
    UserMapper um = sqlSession.getMapper(UserMapper.class);
    User user = new User();
    user.setId(4);
    user.setUserName("小虾");
    user.setPassword("xiaoxia");
    user.setEmail("xiaoxiao@qq.com2");
    try {
        um.updateUser(user);
    } catch (Exception e) {
        e.printStackTrace();
    }
    sqlSession.commit();
    sqlSession.close();
    }
    //删除记录
    public static void deleteUserMapper(){
    SqlSession sqlSession = MyBatisUtil.getSqlSession();
    UserMapper um = sqlSession.getMapper(UserMapper.class);
    try {
        um.deleteUserById(4);
    } catch (Exception e) {
        e.printStackTrace();
    }
    sqlSession.commit();
    sqlSession.close();
    }
}
```