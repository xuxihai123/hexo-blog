---
title: Hibernate学习笔记--环境搭建及运行
---

## hibernate开发包下载
http://sourceforge.net/projects/hibernate/files/

## 添加jar包
将hibernate-release-4.3.7.Final\lib\required目录下的jar导入

## 在src目录下创建Student.java
(按Javabean规范来,这些属性跟数据库中的字段对应起来)
```java
package cn.sky.bookshop.bean;

public class Student {
    private int id;
    private String uname;
    private String upass;
    
    
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getUname() {
        return uname;
    }
    public void setUname(String uname) {
        this.uname = uname;
    }
    public String getUpass() {
        return upass;
    }
    public void setUpass(String upass) {
        this.upass = upass;
    }
    @Override
    public String toString() {
	return "Student [id=" + id + ", uname=" + uname + ", upass=" + upass
		+ "]";
    }
}
```

## 建立一个对象关系映射文件

建立一个对象关系映射文件Student.hbm.xml
```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="cn.sky.bookshop.bean">
	<class name="Student" table="student">
		<id name="id" column="id">
			<generator class="native"></generator>
		</id>
		<property name="uname" column="uname"></property>
		<property name="upass" column="upass"></property>
	</class>
</hibernate-mapping>
```
## 建立一个hibernate.cfg.xml配置文件

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>

    <session-factory>

        <!-- 数据库连接设置 -->
        <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
        <property name="connection.url">jdbc:mysql://localhost:3306/hibernate</property>
        <property name="connection.username">root</property>
        <property name="connection.password">215890</property>

        <!-- JDBC connection pool (use the built-in) -->
        <property name="connection.pool_size">1</property>

        <!-- SQL方言 -->
        <property name="dialect">org.hibernate.dialect.MySQLDialect</property>

        <!-- Enable Hibernate's automatic session context management -->
        <property name="current_session_context_class">thread</property>


        <!-- 是否显示执行的SQL语句 -->
        <property name="show_sql">true</property>
        <!-- 对象关系映射文件 -->
        <mapping resource="cn/sky/bookshop/bean/Student.hbm.xml"/>

    </session-factory>

</hibernate-configuration>
```

## 建立一个HibernateUtil.java类
主要是获取session时使用

```java
package cn.sky.bookshop.util;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.hibernate.cfg.Configuration;

public final class HibernateUtil {
    private static SessionFactory factory;
    private static Configuration conf;
    static {
	conf = new Configuration().configure();
	factory = conf.buildSessionFactory();
    }

    public static Session getSession() {
	return factory.openSession();
    }

    public static void closeSession(Session session) {
	if (session != null && session.isOpen()) {
	    session.close();
	}
    }
}
```
注意：这里色SessionFactory对象只有一个，这是按饿汉式单例设计的。

## 设计DAO
然后可以设计DAO了，下面是我的DAO测试类

```java
package cn.sky.bookshop.test;


import java.util.List;

import org.hibernate.Criteria;
import org.hibernate.Query;
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.criterion.Restrictions;

import cn.sky.bookshop.bean.Student;
import cn.sky.bookshop.util.HibernateUtil;

public class TestStudent {

    public static void main(String[] args) {
		// TODO Auto-generated method stub
		Student stu = new Student();
		stu.setUname("李四2");
		stu.setUpass("123456");
	//	addStudent(stu); //添加数据
	//	updateStudent(stu, 1); //修改数据
	//	Student stu2 = selectStudentById(1); 查询数据
	//	System.out.println(stu2.toString());
	//	deleteStudentById(1);

		List list = selectStudentIf();
		for (Object tem : list) {
			Student stu3 = (Student)tem;
			System.out.println(stu3.toString());
		}
    }
    
    //添加学生记录
    public static void addStudent(Student stu){
		Session session1 = HibernateUtil.getSession();
		session1.beginTransaction();//开启事物(即创建了一个事物对象
		session1.save(stu);	    //保存stu对象
		session1.getTransaction().commit(); //提交事务
		HibernateUtil.closeSession(session1);
    }
    //更新学生记录
    public static void updateStudent(Student stu,int id){
		Session session1 = HibernateUtil.getSession();
		Transaction tc = session1.beginTransaction(); //创建事物对象
		stu.setId(id);
		session1.update(stu);			//更新对象
		tc.commit();				//提交事务
		HibernateUtil.closeSession(session1);
    }
    //查询学生记录
    public static Student selectStudentById(int id){
		Session session1 = HibernateUtil.getSession();
		Transaction tc = session1.beginTransaction(); //创建事物对象
		Query query = session1.createQuery("from Student s where s.id=?");
		query.setInteger(0, id);  //这里下标从0开始替换?
		Student stu = (Student)query.uniqueResult();
		tc.commit();	
		HibernateUtil.closeSession(session1);
		return stu;
    }
    //删除学生记录
    public static void deleteStudentById(int id){
		Session session1 = HibernateUtil.getSession();
		Transaction tc = session1.beginTransaction(); //创建事物对象
		Student stu = new Student();
		stu.setId(id);
		session1.delete(stu);
		tc.commit();	
		HibernateUtil.closeSession(session1);
    }
    //条件查询
    public static List<Student> selectStudentIf(){
		Session s1 = HibernateUtil.getSession();
		Transaction tc = s1.beginTransaction();
		Criteria criteria = s1.createCriteria(Student.class); //创建条件查询对象
		criteria.add(Restrictions.gt("id",1)); //添加查询条件
		criteria.add(Restrictions.lt("id",5));
		criteria.add(Restrictions.eq("id",2));
		List<Student> list = criteria.list();
		tc.commit();
		s1.close(); 
		return list;
    }

}
```