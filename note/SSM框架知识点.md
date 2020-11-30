# SSM框架知识点

### 1.必备知识点

#### 1.1 JDBC连接池之C3P0

```
1.步骤：
    1. 导入jar包 (两个) c3p0-0.9.5.2.jar mchange-commons-java-0.2.12.jar ，
    	* 不要忘记导入数据库驱动jar包 mysql-connector-java-5.1.37-bin.jar
    2. 定义配置文件：
        * 名称： c3p0.properties 或者 c3p0-config.xml。两者选一。
        * 路径：直接将文件放在src目录下即可，会自动加载配置文件。
```

##### c3p0.properties

```c3p0.properties配置文件
#JDBC具备自己的规范，在JDBC4之前是必须要填写驱动名称的，但是之后版本不需要填写
c3p0.driverClass=com.mysql.jdbc.Driver
c3p0.jdbcUrl=jdbc:mysql://localhost:3306/demo
c3p0.user=root
c3p0.password=123456
```

##### c3p0-config.xml

```c3p0-config.xml
<?xml version="1.0" encoding="UTF-8" ?>
<c3p0-config>
    <default-config>
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/demo5</property>
        <property name="user">root</property>
        <property name="password">123456</property>
    </default-config>
</c3p0-config>
```

##### c3p0-config.xml

```c3p0-config.xml
<c3p0-config>
  <!-- 使用默认的配置读取连接池对象 -->
  <default-config>
  	<!--  连接参数 -->
    <property name="driverClass">com.mysql.jdbc.Driver</property>
    <property name="jdbcUrl">jdbc:mysql://localhost:3306/db4</property>
    <property name="user">root</property>
    <property name="password">root</property>
    
    <!-- 连接池参数 -->
    <!--初始化申请的连接数量-->
    <property name="initialPoolSize">5</property>
    <!--最大的连接数量-->
    <property name="maxPoolSize">10</property>
    <!--超时时间-->
    <property name="checkoutTimeout">3000</property>
  </default-config>

  <named-config name="otherc3p0"> 
    <!--  连接参数 -->
    <property name="driverClass">com.mysql.jdbc.Driver</property>
    <property name="jdbcUrl">jdbc:mysql://localhost:3306/db3</property>
    <property name="user">root</property>
    <property name="password">root</property>
    
    <!-- 连接池参数 -->
    <property name="initialPoolSize">5</property>
    <property name="maxPoolSize">8</property>
    <property name="checkoutTimeout">1000</property>
  </named-config>
</c3p0-config>
```

##### 使用、代码执行

```C3P0运行
* 代码：
    //1.创建 c3p0连接池 对象
    ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();
    try {
        //2.获取连接对象
        Connection connection = comboPooledDataSource.getConnection();
        //3.执行sql语句
        String sql = "select * from account";
        PreparedStatement ps = connection.prepareStatement(sql);
        ResultSet rs = ps.executeQuery();
        while (rs.next()){
        System.out.println(rs.getString("name"));

        }
    } catch (SQLException throwables) {
    	throwables.printStackTrace();
    }
```

#### 1.2 JDBC连接池之druid

```
1. 步骤：
    1. 导入jar包 druid-1.0.9.jar
    2. 定义配置文件：
        * 是properties形式的
        * 可以叫任意名称，可以放在任意目录下
    3. 代码执行流程
        1. 加载配置文件：.Properties
        2. 获取数据库连接池对象：通过工厂来来获取  DruidDataSourceFactory
        3. 获取连接：getConnection
```

##### druid.properties

```druid.properties
#驱动名称
driverClassName=com.mysql.jdbc.Driver
#url
url=jdbc:mysql://localhost:3306/demo
#用户名
username=root
#密码
password=123456
#配置初始化大小、最小空闲连接数、最大连接数
initialSize=5
minIdle=10
maxActive=20

#下面的可不需要完全填写
#配置监控系统拦截的filters：监控统计用的filter:stat日志用的filter:log4j防御sql注入的filter:wall
filters=stat
#配置获取连接等待超时的时间
maxWait=60000
#配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
timeBetweenEvictionRunsMillis=60000
#配置一个连接在池中最小的生存的时间，单位是毫秒
minEvictableIdleTimeMillis=600000
maxEvictableIdleTimeMillis=900000
#建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
testWhileIdle=true
#申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
testOnBorrow=false
#归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能
testOnReturn=false
#是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。
poolPreparedStatements=true
#要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100
maxOpenPreparedStatements=20
#asyncInit是1.1.4中新增加的配置，如果有initialSize数量较多时，打开会加快应用启动时间
asyncInit=true
```

##### 使用、代码执行

```
	//1.加载配置文件
    Properties properties = new Properties();
    InputStream resourceAsStream = DruidTest2.class.getClassLoader().getResourceAsStream("druid.properties");
    properties.load(resourceAsStream);

    //2.获取数据库连接对象
    try {
        DataSource dataSource = DruidDataSourceFactory.createDataSource(properties);
        //3.获取连接
        Connection connection = dataSource.getConnection();
        //4.执行sql语句
        String sql = "select * from account";
        PreparedStatement ps = connection.prepareStatement(sql);
        ResultSet rs = ps.executeQuery();
        while (rs.next()){
        	System.out.println(rs.getString("name"));
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
```

#### 1.3 JdbcTemplate常用方法执行CRUD

##### update()：增、删、改

```
* update():执行DML语句。增、删、改语句
	增：
		String sql = "insert into emp(id,ename,dept_id) values(?,?,?)";
		int count = template.update(sql, 1015, "郭靖", 10);
	删：
        String sql = "delete from emp where id = ?";
        int count = template.update(sql, 1015);
    改：
		String sql = "update emp set salary = 10000 where id = 1001";
		int count = template.update(sql);
```

##### queryForMap()：查询结果封装成 Map 集合

```
* queryForMap():查询结果将结果集封装为map集合。
	* 将列名作为key，将值作为value 将这条记录封装为一个map集合。
	* 注意：这个方法查询的结果集长度只能是1。
	
	String sql = "select * from emp where id = ? or id = ?";
	Map<String, Object> map = template.queryForMap(sql, 1001,1002);
```

##### queryForList()：查询结果封装成 list 集合

```
* queryForList():查询结果将结果集封装为list集合
	* 注意：将每一条记录封装为一个Map集合，再将Map集合装载到List集合中

	将记录封装成List:
        String sql = "select * from emp";
        List<Map<String, Object>> list = template.queryForList(sql);

        for (Map<String, Object> stringObjectMap : list) {
            System.out.println(stringObjectMap);
        }
        
	将记录封装为Emp对象的List集合：
        String sql = "select * from emp";
        List<Emp> list = template.query(sql, new RowMapper<Emp>() {

            @Override
            public Emp mapRow(ResultSet rs, int i) throws SQLException {
                Emp emp = new Emp();
                int id = rs.getInt("id");
                String ename = rs.getString("ename");
                int job_id = rs.getInt("job_id");
                int mgr = rs.getInt("mgr");
                Date joindate = rs.getDate("joindate");
                double salary = rs.getDouble("salary");
                double bonus = rs.getDouble("bonus");
                int dept_id = rs.getInt("dept_id");

                emp.setId(id);
                emp.setEname(ename);
                emp.setJob_id(job_id);
                emp.setMgr(mgr);
                emp.setJoindate(joindate);
                emp.setSalary(salary);
                emp.setBonus(bonus);
                emp.setDept_id(dept_id);

                return emp;
            }
		});
		
		for (Emp emp : list) {
			System.out.println(emp);
		}
```

##### query()：查询结果封装成 JavaBean 对象

```
* query():查询结果，将结果封装为JavaBean对象
	* query的参数：RowMapper
		* 一般我们使用BeanPropertyRowMapper实现类。可以完成数据到JavaBean的自动封装。
		* new BeanPropertyRowMapper<类型>(类型.class)
	
    String sql = "select * from emp";
    List<Emp> list = template.query(sql, new BeanPropertyRowMapper<Emp>(Emp.class));
    for (Emp emp : list) {
    	System.out.println(emp);
    }    	
```

##### queryForObject：查询结果封装成对象

```
* queryForObject：查询结果，将结果封装为对象
	* 一般用于聚合函数的查询	
    
    String sql = "select count(id) from emp";
    Long total = template.queryForObject(sql, Long.class);
```

### 2.Spring 入门

##### 2.1：概念

```
1.JAR包的分类
    以RELEASE.jar结尾的是Spring框架class文件的JAR包；
    以RELEASE-javadoc.jar结尾的是Spring框架API文档的压缩包；
    以RELEASE-sources.jar结尾的是Spring框架源文件的压缩包。

2.Spring的4个基础包
	spring-core-5.2.3.RELEASE.jar：
		包含Spring框架基本的核心工具类，Spring其他组件都要用到这个包里的类，是其他组件的基本核心。
	spring-beans-5.2.3.RELEASE.jar：
		所有应用都要用到的JAR包，它包含访问配置文件、创建和管理Bean以及进行 IoC 或者 DI 操作相关的所有类。	
	spring-context-5.2.3.RELEASE.jar：
		Spring提供了在基础IoC功能上的扩展服务，还提供许多企业级服务的支持，如邮件服务、任务调度、JNDI定位、EJB集成、远程访问、缓存以及各种视图层框架的封装等。
	spring-expression-5.2.3.RELEASE.jar：
		定义了Spring的表达式语言。

3.Spring的第三方依赖包
	Spring的核心容器还需要依赖commons.logging的JAR包。
	
4.总结
	初学者学习Spring框架时，只需将Spring的4个基础包以及commons.logging-1.2jar赋值到项目的lib目录，并发布到类路径中即可。


```

##### 2.2：maven方式搭建Spring框架步骤（xml方式）

###### 1.Maven工程的pom.xml文件中导入 Spring 开发的基本包坐标

```
1.Maven工程的pom.xml文件中导入 Spring 开发的基本包坐标

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>SpringTest</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.3.RELEASE</version>
        </dependency>
    </dependencies>

</project>
    
```

###### 2.编写 Dao 接口和实现类

```
2.编写 Dao 接口和实现类
```

###### 3.创建 Spring 核心配置文件：applicationContext.xml

```
3.创建 Spring 核心配置文件：applicationContext.xml
  配置文件要添加到类路径中；使用idea创建项目时要放在 resource 目录下。
	
    <?xml version="1.0" encoding="UTF-8" ?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    </beans>
```

###### 4.在 Spring 配置文件中配置 自动装配对象

```
4.在 Spring 配置文件中配置 自动装配对象

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--注册一个对象，spring回自动创建这个对象-->
    <!--
    	一个bean标签就表示一个对象
    	id:这个对象的唯一标识
    	class:注册对象的完全限定名
    -->
    <bean id="person" class="com.mashibing.bean.Person">
        <!--使用property标签给对象的属性赋值
        	name:表示属性的名称
        	value：表示属性的值
        -->
        <property name="id" value="1"></property>
        <property name="name" value="zhangsan"></property>
        <property name="age" value="18"></property>
        <property name="gender" value="男"></property>
    </bean>
</beans>
```

###### 5.使用 Spring 的 API 获得 Bean 实例

```
5.使用 Spring 的 API 获得 Bean 实例

package com.mashibing.test;

import com.mashibing.bean.Person;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringDemoTest {
    public static void main(String[] args) {
        //ApplicationContext:表示ioc容器
        //ClassPathXmlApplicationContext:表示从当前classpath路径中获取xml文件的配置
        //根据spring的配置文件来获取ioc容器对象
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        Person person = (Person) context.getBean("person");
        System.out.println(person);
    }
}
```

### 3.Spring 的配置文件

##### 3.1 Bean对象的获取

###### 1.通过 bean 的 id 获取对象

```java
    <!--注册一个对象，spring回自动创建这个对象-->
    <!--
    	一个bean标签就表示一个对象
    	id:这个对象的唯一标识
    	class:注册对象的完全限定名
    -->
    <bean id="person" class="com.mashibing.bean.Person">
        <!--使用property标签给对象的属性赋值
        	name:表示属性的名称
        	value：表示属性的值
        -->
        <property name="id" value="1"></property>
        <property name="name" value="zhangsan"></property>
        <property name="age" value="18"></property>
        <property name="gender" value="男"></property>
    </bean>
```

```
bean对象获取：
	Person person = (Person) context.getBean("person");
```

###### 2.通过 bean 的 类型 获取对象

```
bean对象获取：
	Person person = context.getBean(Person.class);
```

```
注意：
	通过bean的类型在查找对象的时候，在配置文件中不能存在多个类型一致的bean对象。
	如果有的话，可以通过如下方法：
		Person person = context.getBean("person", Person.class);
```

##### 3.2 Spring对象的属性赋值方式

###### 3.2.1 通过 构造器 给 bean对象 赋值

```java
	<!--给person类添加构造方法-->
	<bean id="person2" class="com.mashibing.bean.Person">
        <constructor-arg name="id" value="1"></constructor-arg>
        <constructor-arg name="name" value="lisi"></constructor-arg>
        <constructor-arg name="age" value="20"></constructor-arg>
        <constructor-arg name="gender" value="女"></constructor-arg>
    </bean>

	<!--在使用构造器赋值的时候可以省略name属性，但是此时就要求必须严格按照构造器参数的顺序来填写了-->
	<bean id="person3" class="com.mashibing.bean.Person">
        <constructor-arg value="1"></constructor-arg>
        <constructor-arg value="lisi"></constructor-arg>
        <constructor-arg value="20"></constructor-arg>
        <constructor-arg value="女"></constructor-arg>
    </bean>

	<!--如果想不按照顺序来添加参数值，那么可以搭配index属性来使用-->
    <bean id="person4" class="com.mashibing.bean.Person">
        <constructor-arg value="lisi" index="1"></constructor-arg>
        <constructor-arg value="1" index="0"></constructor-arg>
        <constructor-arg value="女" index="3"></constructor-arg>
        <constructor-arg value="20" index="2"></constructor-arg>
    </bean>
        
	<!--当有多个参数个数相同，不同类型的构造器的时候，可以通过type来强制类型-->
	将person的age类型设置为Integer类型
	public Person(int id, String name, Integer age) {
        this.id = id;
        this.name = name;
        this.age = age;
        System.out.println("Age");
    }

    public Person(int id, String name, String gender) {
        this.id = id;
        this.name = name;
        this.gender = gender;
        System.out.println("gender");
    }

	<bean id="person5" class="com.mashibing.bean.Person">
        <constructor-arg value="1"></constructor-arg>
        <constructor-arg value="lisi"></constructor-arg>
        <constructor-arg value="20" type="java.lang.Integer"></constructor-arg>
    </bean>
        
	<!--如果不修改为integer类型，那么需要type跟index组合使用-->
	 <bean id="person5" class="com.mashibing.bean.Person">
        <constructor-arg value="1"></constructor-arg>
        <constructor-arg value="lisi"></constructor-arg>
        <constructor-arg value="20" type="int" index="2"></constructor-arg>
    </bean>        
```

###### 3.2.2 通过 set 方法

```java
public class UserServiceImpl implements UserService {
    private UserDao userDao;
    
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;  
        } 
        
    @Override    
    public void save() {      
   		 userDao.save();
	}
}

<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"/>
<bean id="userService" class="com.itheima.service.impl.UserServiceImpl">
	<property name="userDao" ref="userDao"/>
</bean>
```

###### 3.2.3 P命名空间注入（本质是 set 方法）

```
首先，需要引入P命名空间：
	xmlns:p="http://www.springframework.org/schema/p"
```

```
<bean id="userService" class="com.itheima.service.impl.UserServiceImpl" p:userDao-ref="userDao"/>

注：userDao-ref="userDao" 相当于 name="userDao" ref="userDao"。
   carsDao-ref="cars" 相当于 name="carsDao" ref="cars"。
```

###### 3.2.4 复杂类型注入 

（1）赋空值

```
<bean id="person" class="com.mashibing.bean.Person">
    <property name="name">
        <!--赋空值-->
        <null></null>
    </property>
</bean>
```

（2）引用其他对象

```
<bean id="person" class="com.mashibing.bean.Person">
	<!--通过ref引用其他对象，方法一：引用外部bean-->
    <property name="address" ref="address"></property>
       
     <!--方法二：引用内部bean-->
     <property name="address">
            <bean class="com.mashibing.bean.Address">
                <property name="province" value="北京"></property>
                <property name="city" value="北京"></property>
                <property name="town" value="西城区"></property>
            </bean>
     </property>
</bean>     
```

（3）集合数据类型（List<String>）的注入

```
<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl">
    <property name="strList">
        <list>
            <value>aaa</value>
            <value>bbb</value>
            <value>ccc</value>
        </list>
    </property>
</bean>
```

（4）集合数据类型（List<User>）的注入

```
<bean id="u1" class="com.itheima.domain.User"/>
<bean id="u2" class="com.itheima.domain.User"/>

<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl">
    <property name="userList">
        <list>
        	<!--方法一：引用内部bean-->
            <bean class="com.itheima.domain.User"/>
            
            <bean class="com.itheima.domain.User">
            	<property name="name" value="多线程与高并发"></property>
                <property name="author" value="yurb"></property>
                <property name="price" value="1000"></property>
            </bean>
            
            <!--方法二：引用外部bean-->
            <ref bean="u1"/>
            <ref bean="u2"/>       
        </list>
    </property>
</bean>
```

（5）集合数据类型（ Map<String,User> ）的注入

```
<bean id="u1" class="com.itheima.domain.User"/>
<bean id="u2" class="com.itheima.domain.User"/>

<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl">
    <property name="userMap">
        <map>            
            <entry key="user1" value-ref="u1"/>
            <entry key="user2" value-ref="u2"/>
        </map>
    </property>
</bean>
```

（6）集合数据类型（Properties）的注入

```
<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl">
    <property name="properties">
        <props>
            <prop key="p1">aaa</prop>
            <prop key="p2">bbb</prop> 
            <prop key="p3">ccc</prop>
        </props>
    </property>
</bean>
```

（7）数组数据类型（Array）的注入

```
<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl">
    <property name="properties">
    	<array>
            <value>book</value>
            <value>movie</value>
            <value>game</value>
       	</array>
    </property>
</bean>
```

（8）set 赋值

```
<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl">
    <property name="properties">
    	<set>
            <value>111</value>
            <value>222</value>
            <value>222</value>
    	</set>
    </property>
</bean>
```

（9）级联属性 赋值

```
<bean id="person2" class="com.mashibing.bean.Person">
    <property name="address" ref="address"></property>
    <property name="address.province" value="北京"></property>
</bean>
```

