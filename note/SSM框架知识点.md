# SSM框架知识点

## 1.必备知识点

### 1.1 JDBC连接池之C3P0

```
1.步骤：
    1. 导入jar包 (两个) c3p0-0.9.5.2.jar mchange-commons-java-0.2.12.jar ，
    	* 不要忘记导入数据库驱动jar包 mysql-connector-java-5.1.37-bin.jar
    2. 定义配置文件：
        * 名称： c3p0.properties 或者 c3p0-config.xml。两者选一。
        * 路径：直接将文件放在src目录下即可，会自动加载配置文件。
```

##### c3p0.properties

```properties
#JDBC具备自己的规范，在JDBC4之前是必须要填写驱动名称的，但是之后版本不需要填写
c3p0.driverClass=com.mysql.jdbc.Driver
c3p0.jdbcUrl=jdbc:mysql://localhost:3306/demo
c3p0.user=root
c3p0.password=123456
```

##### c3p0-config.xml

```xml
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

```xml
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

```java
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

### 1.2 JDBC连接池之druid

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

```properties
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

```java
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

### 1.3 JdbcTemplate常用方法执行CRUD

##### update()：增、删、改

```java
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

```java
* queryForMap():查询结果将结果集封装为map集合。
	* 将列名作为key，将值作为value 将这条记录封装为一个map集合。
	* 注意：这个方法查询的结果集长度只能是1。
	
	String sql = "select * from emp where id = ? or id = ?";
	Map<String, Object> map = template.queryForMap(sql, 1001,1002);
```

##### queryForList()：查询结果封装成 list 集合

```java
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

```java
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

```java
* queryForObject：查询结果，将结果封装为对象
	* 一般用于聚合函数的查询	
    //聚合查询
    String sql = "select count(id) from emp";
    Long total = template.queryForObject(sql, Long.class);

	//查询一个对象
	Account account = jdbcTemplate.queryForObject("select * from account where name=?", new BeanPropertyRowMapper<Account>(Account.class), "tom");
```

## 2.Spring 

### 1.Spring入门

#### 1.1：概念

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

#### 1.2：maven方式搭建Spring框架步骤（xml方式）

###### 1.Maven工程的pom.xml文件中导入 Spring 开发的基本包坐标

```xml
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

```xml
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

```xml
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

```java
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

### 2.Spring 的配置文件

#### 2.1 Bean 对象的创建（实例化）

###### 1.使用无参构造方法实例化

​	它会根据默认无参构造方法来创建类对象，如果bean中没有默认无参构造函数，将会创建失败。

```xml
<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"/>
```

###### 2.工厂静态方法实例化

​	静态工厂：工厂本身不需要创建对象，但是可以通过静态方法调用，对象=工厂类.静态工厂方法名（）；

```java
public class StaticFactoryBean {
    public static UserDao createUserDao(){    
    return new UserDaoImpl();
    }
}
```

```xml
<!--
	静态工厂的使用：
		class:指定静态工厂类
		factory-method:指定哪个方法是工厂方法
-->
<bean id="userDao" class="com.itheima.factory.StaticFactoryBean" factory-method="createUserDao" />
```

###### 3.工厂实例方法实例化

​	实例工厂：工厂本身需要创建对象，工厂类 工厂对象=new 工厂类；工厂对象.get对象名（）；

```java
public class DynamicFactoryBean {  
	public UserDao createUserDao(){        
		return new UserDaoImpl(); 
	}
}
```

```xml
<!--
    factory-bean:指定使用哪个工厂实例
    factory-method:指定使用哪个工厂实例的方法
-->
<bean id="factoryBean" class="com.itheima.factory.DynamicFactoryBean"/>
<bean id="userDao" factory-bean="factoryBean" factory-method="createUserDao"/>
```

###### 4.继承FactoryBean来创建对象

​	FactoryBean是Spring规定的一个接口，当前接口的实现类，Spring都会将其作为一个工厂，但是在ioc容器启动的时候不会创建实例，只有在使用的时候才会创建对象。

```java
package com.mashibing.factory;

import com.mashibing.bean.Person;
import org.springframework.beans.factory.FactoryBean;

/**
 * 实现了FactoryBean接口的类是Spring中可以识别的工厂类，spring会自动调用工厂方法创建实例
 */
public class MyFactoryBean implements FactoryBean<Person> {

    /**
     * 工厂方法，返回需要创建的对象
     * @return
     * @throws Exception
     */
    @Override
    public Person getObject() throws Exception {
        Person person = new Person();
        person.setName("maliu");
        return person;
    }

    /**
     * 返回创建对象的类型,spring会自动调用该方法返回对象的类型
     * @return
     */
    @Override
    public Class<?> getObjectType() {
        return Person.class;
    }

    /**
     * 创建的对象是否是单例对象
     * @return
     */
    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

```xml
<bean id="myfactorybean" class="com.mashibing.factory.MyFactoryBean"></bean>
```

#### 2.2 Bean 对象的获取

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

```java
bean对象获取：
	Person person = (Person) context.getBean("person");
```

###### 2.通过 bean 的 类型 获取对象

```java
bean对象获取：
	Person person = context.getBean(Person.class);
```

```java
注意：
	通过bean的类型在查找对象的时候，在配置文件中不能存在多个类型一致的bean对象。
	如果有的话，可以通过如下方法：
		Person person = context.getBean("person", Person.class);
```

#### 2.3 Bean 对象的属性赋值方式

###### 1.通过 构造器 给 bean对象 赋值

```xml
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

###### 2.通过 set 方法

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

###### 3.P命名空间注入（本质是 set 方法）

```xml
首先，需要引入P命名空间：
	xmlns:p="http://www.springframework.org/schema/p"
```

```xml
<bean id="userService" class="com.itheima.service.impl.UserServiceImpl" p:userDao-ref="userDao"/>

注：userDao-ref="userDao" 相当于 name="userDao" ref="userDao"。
   carsDao-ref="cars" 相当于 name="carsDao" ref="cars"。
```

###### 4.复杂类型注入 

（1）赋空值

```xml
<bean id="person" class="com.mashibing.bean.Person">
    <property name="name">
        <!--赋空值-->
        <null></null>
    </property>
</bean>
```

（2）引用其他对象

```xml
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

```xml
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

```xml
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

```xml
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

```xml
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

```xml
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

```xml
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

```xml
<bean id="person2" class="com.mashibing.bean.Person">
    <property name="address" ref="address"></property>
    <property name="address.province" value="北京"></property>
</bean>
```

###### 5.Bean 对象的继承

```xml
<bean id="person" class="com.mashibing.bean.Person">
    <property name="id" value="1"></property>
    <property name="name" value="zhangsan"></property>
    <property name="age" value="21"></property>
    <property name="gender" value="男"></property>
</bean>

<!--parent:指定bean的配置信息继承于哪个bean-->
<bean id="person2" class="com.mashibing.bean.Person" parent="person">
    <property name="name" value="lisi"></property>
</bean>
```

如果想实现Java文件的抽象类，不需要将当前bean实例化的话，可以使用abstract属性

```xml
注：添加 abstract="true" 后，不会再实例化 person 对象。
<bean id="person" class="com.mashibing.bean.Person" abstract="true">
    <property name="id" value="1"></property>
    <property name="name" value="zhangsan"></property>
    <property name="age" value="21"></property>
    <property name="gender" value="男"></property>
</bean>

<!--parent:指定bean的配置信息继承于哪个bean-->
<bean id="person2" class="com.mashibing.bean.Person" parent="person">
    <property name="name" value="lisi"></property>
</bean>
```

###### 6.Bean 对象创建的依赖关系

bean对象在创建的时候是按照bean在配置文件的顺序决定的，也可以使用depend-on标签来决定顺序。

```xml
<bean id="book" class="com.mashibing.bean.Book" depends-on="person,address"></bean>
<bean id="address" class="com.mashibing.bean.Address"></bean>
<bean id="person" class="com.mashibing.bean.Person"></bean>
```

#### 2.4 Bean对象的作用范围

scope:指对象的作用范围，取值如下： 

| 取值范围         | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| singleton        | 默认值，单例的                                               |
| prototype        | 多例的                                                       |
| request          | WEB   项目中，Spring   创建一个   Bean   的对象，将对象存入到   request   域中 |
| session          | WEB   项目中，Spring   创建一个   Bean   的对象，将对象存入到   session   域中 |
| global   session | WEB   项目中，应用在   Portlet   环境，如果没有   Portlet   环境那么globalSession   相当于   session |

1）当scope的取值为singleton时

​      Bean的实例化个数：1个

​      Bean的实例化时机：当Spring核心文件被加载时，实例化配置的Bean实例

​      Bean的生命周期：

对象创建：当应用加载，创建容器时，对象就被创建了

对象运行：只要容器在，对象一直活着

对象销毁：当应用卸载，销毁容器时，对象就被销毁了

2）当scope的取值为prototype时

​      Bean的实例化个数：多个

​      Bean的实例化时机：当调用getBean()方法时实例化Bean

对象创建：当使用对象时，创建新的对象实例

对象运行：只要对象在使用中，就一直活着

对象销毁：当对象长时间不用时，被 Java 的垃圾回收器回收了

```xml
<bean id="person4" class="com.mashibing.bean.Person" scope="prototype"></bean>
```

#### 2.5 Bean 对象的初始化和销毁方法

```xml
<!--bean生命周期表示bean的创建到销毁
        如果bean是单例，容器在启动的时候会创建好，关闭的时候会销毁创建的bean
        如果bean是多例，获取的时候创建对象，销毁的时候不会有任何的调用

    init-method：指定类中的初始化方法名称
    destroy-method：指定类中销毁方法名称
-->
    <bean id="address" class="com.mashibing.bean.Address" init-method="init" destroy-method="destory"></bean>
```

#### 2.6  Bean 对象初始化方法 的 前后处理方法

​	spring中包含一个BeanPostProcessor的接口，可以在bean的初始化方法的前后调用该方法，如果配置了初始化方法的前置和后置处理器，无论是否包含初始化方法，都会进行调用。

​	实现 BeanPostProcessor 接口，重写 postProcessBeforeInitialization() 和 postProcessAfterInitialization()。

```java
package com.mashibing.bean;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class MyBeanPostProcessor implements BeanPostProcessor {
    /**
     * 在初始化方法调用之前执行
     * @param bean  初始化的bean对象
     * @param beanName  xml配置文件中的bean的id属性
     * @return
     * @throws BeansException
     */
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization:"+beanName+"调用初始化前置方法");
        return bean;
    }

    /**
     * 在初始化方法调用之后执行
     * @param bean
     * @param beanName
     * @return
     * @throws BeansException
     */
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization:"+beanName+"调用初始化后缀方法");
        return bean;
    }
}
```

#### 2.7 引用外部配置文件

###### 0.添加 context 命名空间

```xml
<!-- 原 -->

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
```

```xml
<!-- 添加 context 命名空间 和 约束路径 -->
<!--
	命名空间：xmlns:context="http://www.springframework.org/schema/context"

	约束路径：http://www.springframework.org/schema/context        
        	http://www.springframework.org/schema/context/spring-context.xsd
-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

       xmlns:context="http://www.springframework.org/schema/context"
       
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd    
                           
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd"> 
```

###### 1.引入 xml 文件

```xml
<!-- Spring中引入其他配置文件 -->
<import resource="applicationContext-xxx.xml"/>
```

###### 2.引入 properties 文件

dbconfig.properties

```properties
username=root
password=123456
url=jdbc:mysql://localhost:3306/demo
driverClassName=com.mysql.jdbc.Driver
```

```xml
<!--加载外部的properties文件（方式一），用于解析${}形式的变量。-->
<!--如果需要加载多个properties文件，就写在一起，之间使用逗号隔开。-->
<context:property-placeholder location="classpath:dbconfig.properties"/>

<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="username" value="${username}"></property>
    <property name="password" value="${password}"></property>
    <property name="url" value="${url}"></property>
    <property name="driverClassName" value="${driverClassName}"></property>
</bean>

```

#### 2.8 Spring 基于 xml 文件的自动装配

​	当一个对象中需要引用另外一个对象的时候，在之前的配置中我们都是通过property标签来进行手动配置的，其实在spring中还提供了一个非常强大的功能就是自动装配，可以按照我们指定的规则进行配置，配置的方式有以下几种：

​		default/no：不自动装配

​		byName：按照名字进行装配，以属性名作为id去容器中查找组件，进行赋值，如果找不到则装配null

​		byType：按照类型进行装配,以属性的类型作为查找依据去容器中找到这个组件，如果有多个类型相同的bean对象，那么会报异常，如果找不到则装配null

​		constructor：按照构造器进行装配，先按照有参构造器参数的类型进行装配，没有就直接装配null；如果按照类型找到了多个，那么就使用参数名作为id继续匹配，找到就装配，找不到就装配null

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="address" class="com.mashibing.bean.Address">
        <property name="province" value="河北"></property>
        <property name="city" value="邯郸"></property>
        <property name="town" value="武安"></property>
    </bean>
    <bean id="person" class="com.mashibing.bean.Person" autowire="byName"></bean>
    <bean id="person2" class="com.mashibing.bean.Person" autowire="byType"></bean>
    <bean id="person3" class="com.mashibing.bean.Person" autowire="constructor"></bean>
</beans>
```

#### 2.9 SpEL的使用

SpEL:Spring Expression Language,spring的表达式语言，支持运行时查询操作对象

使用#{...}作为语法规则，所有的大括号中的字符都认为是SpEL.

```xml
    <bean id="person4" class="com.mashibing.bean.Person">
        <!--支持任何运算符-->
        <property name="age" value="#{12*2}"></property>
        <!--可以引用其他bean的某个属性值-->
        <property name="name" value="#{address.province}"></property>
        <!--引用其他bean-->
        <property name="address" value="#{address}"></property>
        <!--调用静态方法-->
        <property name="hobbies" value="#{T(java.util.UUID).randomUUID().toString().substring(0,4)}"></property>
        <!--调用非静态方法-->
        <property name="gender" value="#{address.getCity()}"></property>
    </bean>
```

### 3. Spring 注解开发

#### 3.1 原始注解

Spring原始注解主要是替代<Bean>的配置

| 注解           | 说明                                           |
| -------------- | ---------------------------------------------- |
| @Component     | 使用在类上用于实例化Bean                       |
| @Controller    | 使用在web层类上用于实例化Bean                  |
| @Service       | 使用在service层类上用于实例化Bean              |
| @Repository    | 使用在dao层类上用于实例化Bean                  |
| @Autowired     | 使用在字段上用于根据类型依赖注入               |
| @Qualifier     | 结合@Autowired一起使用用于根据名称进行依赖注入 |
| @Resource      | 相当于@Autowired+@Qualifier，按照名称进行注入  |
| @Value         | 注入普通属性                                   |
| @Scope         | 标注Bean的作用范围                             |
| @PostConstruct | 使用在方法上标注该方法是Bean的初始化方法       |
| @PreDestroy    | 使用在方法上标注该方法是Bean的销毁方法         |

#### 3.2 使用注解的步骤

```xml
<!--
    使用注解需要如下步骤：
    1、在需要实例化的类上添加四个注解(@Component、@Controller、@Service、@Repository)中的任意一个
    2、xml文件添加自动扫描注解的组件，此操作需要依赖context命名空间
    3、xml文件添加自动扫描的标签context:component-scan

	注意：当使用注解注册组件和使用配置文件注册组件是一样的，但是要注意：
		1、组件的id默认就是组件的类名首字符小写，如果非要改名字的话，直接在注解中添加即可
		2、组件默认情况下都是单例的,如果需要配置多例模式的话，可以在注解下添加@Scope注解
-->
<!--
    定义自动扫描的基础包:
    base-package:指定扫描的基础包，spring在启动的时候会将基础包及子包下所有加了注解的类都自动扫描进IOC容器。
-->
```

application-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
    
    <!--注解的组件扫描-->
    <context:component-scan base-package="com.mashibing"></context:component-scan>

</beans>    
```

#### 3.3 原始注解的使用

使用 @Component 或 @Repository 标识UserDaoImpl需要Spring进行实例化。

```java
//@Component("userDao")
@Repository("userDao")
public class UserDaoImpl implements UserDao {
    @Override
    public void save() {
    	System.out.println("save running... ...");
    }
}
```

使用 @Component 或 @Service 标识UserServiceImpl需要Spring进行实例化

使用 @Autowired 或 @Autowired+@Qulifier 或 @Resource 进行userDao的注入

```java
//@Component("userService")
@Service("userService")
public class UserServiceImpl implements UserService {
    /*
    	@Autowired
    	@Qualifier("userDao")
    */  
    @Resource(name="userDao") 
    private UserDao userDao;
    
    @Override
    public void save() {       
   	  userDao.save();
    }
}
```

使用@Value进行字符串的注入

```java
@Repository("userDao")
public class UserDaoImpl implements UserDao {
    @Value("注入普通数据")
    private String str;
    
    @Value("${jdbc.driver}")
    private String driver;
    
    @Override
    public void save() {
        System.out.println(str);
        System.out.println(driver);
        System.out.println("save running... ...");
    }
}
```

使用@Scope标注Bean的范围

```java
//@Scope("prototype")
@Scope("singleton")
public class UserDaoImpl implements UserDao {
   //此处省略代码
}
```

使用@PostConstruct标注初始化方法，使用@PreDestroy标注销毁方法

```java
@PostConstruct
public void init(){
	System.out.println("初始化方法....");
}

@PreDestroy
public void destroy(){
	System.out.println("销毁方法.....");
}
```

#### 3.4 定义扫描包时要包含的类和不要包含的类

###### 1.定义要包含的类

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
  
    
    <context:component-scan base-package="com.mashibing" use-default-filters="false">
        <!--
        当定义好基础扫描的包之后，可以排除包中的某些类，使用如下的方式:
        type:表示指定过滤的规则
            annotation：按照注解进行排除，标注了指定注解的组件不要,expression表示要过滤的注解
            assignable：指定排除某个具体的类，按照类排除，expression表示不注册的具体类名
            aspectj：后面讲aop的时候说明要使用的aspectj表达式，不用
            custom：定义一个typeFilter,自己写代码决定哪些类被过滤掉，不用
            regex：使用正则表达式过滤，不用
        -->

        <!--
			指定只扫描哪些组件，默认情况下是全部扫描的，
			所以此时要配置的话需要在component-scan标签中添加 use-default-filters="false" 
			这样就不是扫描所有的类,而是扫描 include 的类。
		-->
        <!-- type 以 assignable 举例 -->
        <context:include-filter type="assignable" expression="com.mashibing.service.PersonService"/>
        
    </context:component-scan>
    
</beans>
```

###### 2.定义不要包含的类

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
  
    
    <context:component-scan base-package="com.mashibing" use-default-filters="true">
        <!--
        当定义好基础扫描的包之后，可以排除包中的某些类，使用如下的方式:
        type:表示指定过滤的规则
            annotation：按照注解进行排除，标注了指定注解的组件不要,expression表示要过滤的注解
            assignable：指定排除某个具体的类，按照类排除，expression表示不注册的具体类名
            aspectj：后面讲aop的时候说明要使用的aspectj表达式，不用
            custom：定义一个typeFilter,自己写代码决定哪些类被过滤掉，不用
            regex：使用正则表达式过滤，不用
        -->

        <!--
			这个是扫描组件的时候不包含什么,这里需要注意的是 use-default-filters="true"。
			这个要么设置 成true,要么就不设置,因为默认是true,这个为true就是先扫描所有的类。
			然后根据自己定义的扫描规则排除一些类。
		-->
        <!-- type 以 annotation 举例 -->        
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        
    </context:component-scan>
    
</beans>
```

#### 3.5 @AutoWired进行自动注入的注意点：

```
注意：当使用AutoWired注解的时候，自动装配的时候是根据 类型 实现的。
1、如果只找到一个，则直接进行赋值。
2、如果没有找到，则直接抛出异常。
3、如果找到多个，那么会按照 变量名作为id 继续匹配。
    1、匹配上直接进行装配
    2、如果匹配不上则直接报异常
还可以使用 @Qualifier注解来指定id的名称 ，让spring不要使用变量名,当使用@Qualifier注解的时候也会有两种情况：
	1、找到，则直接装配
	2、找不到，就会报错    
	
	
使用 @AutoWired 注入到方法上：
	1、此方法在bean创建的时候会自动调用。
	2、这个方法的每一个参数都会自动注入值。
```

#### 3.6 自动装配注解：@AutoWired 和 @Resoure 的区别：

```
1、@AutoWired:是spring中提供的注解，@Resource:是jdk中定义的注解，依靠的是java的标准。
2、@AutoWired默认是按照类型进行装配，默认情况下要求依赖的对象必须存在，@Resource默认是按照名字进行匹配的，同时可以指定name属性。
3、@AutoWired只适合spring框架，而@Resource扩展性更好。
```

#### 3.7 Spring的新注解

| 注解            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| @Configuration  | 用于指定当前类是一个 Spring   配置类，当创建容器时会从该类上加载注解 |
| @ComponentScan  | 用于指定 Spring   在初始化容器时要扫描的包。   作用和在 Spring   的 xml 配置文件中的   <context:component-scan   base-package="com.itheima"/>一样 |
| @Bean           | 使用在方法上，标注将该方法的返回值存储到   Spring   容器中   |
| @PropertySource | 用于加载.properties   文件中的配置                           |
| @Import         | 用于导入其他配置类                                           |

```java
@Configuration
@ComponentScan("com.itheima")
@Import({DataSourceConfiguration.class})
public class SpringConfiguration {
}
```

```java
@PropertySource("classpath:jdbc.properties")
public class DataSourceConfiguration {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;
```

```java
@Bean(name="dataSource")
public DataSource getDataSource() throws PropertyVetoException { 
    ComboPooledDataSource dataSource = new ComboPooledDataSource(); 
    dataSource.setDriverClass(driver);
    dataSource.setJdbcUrl(url);
    dataSource.setUser(username);
    dataSource.setPassword(password);
    return dataSource;
} 
```

### 4.Spring整合Junit

#### 4.1 原始Junit测试Spring的问题

在测试类中，每个测试方法都有以下两行代码：

```java
 ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
 IAccountService as = ac.getBean("accountService",IAccountService.class);
```

这两行代码的作用是获取容器，如果不写的话，直接会提示空指针异常。所以又不能轻易删掉。

#### 4.2 上述问题解决思路

让SpringJunit负责创建Spring容器，但是需要将配置文件的名称告诉它。

将需要进行测试Bean直接在测试类中进行注入。

#### 4.3 Spring集成Junit步骤

①导入spring集成Junit的坐标

②使用@Runwith注解替换原来的运行期

③使用@ContextConfiguration指定配置文件或配置类

④使用@Autowired注入需要测试的对象

⑤创建测试方法进行测试

#### 4.4 Spring集成Junit代码实现

①导入spring集成Junit的坐标

```xml
<!-- 使用@RunWith、@ContextConfiguration 注解需要此包 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>
<!-- 使用@Test注解需要此包 -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

②使用@Runwith注解替换原来的运行期

```java
@RunWith(SpringJUnit4ClassRunner.class)
public class SpringJunitTest {
}
```

③使用@ContextConfiguration指定配置文件或配置类

```java
@RunWith(SpringJUnit4ClassRunner.class)

//加载 核心配置文件 和 加载核心配置类 二选一
//加载spring核心配置文件
//@ContextConfiguration(value = {"classpath:applicationContext.xml"})

//加载spring核心配置类
@ContextConfiguration(classes = {SpringConfiguration.class})
public class SpringJunitTest {
}
```

④使用@Autowired注入需要测试的对象

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {SpringConfiguration.class})
public class SpringJunitTest {
    @Autowired
    private UserService userService;
}
```

⑤创建测试方法进行测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {SpringConfiguration.class})
public class SpringJunitTest {
    @Autowired
    private UserService userService;
    @Test
    public void testUserService(){
   	 userService.save();
    }
}
```

### 5.Spring AOP

#### 5.1 概念

```
AOP 为 Aspect Oriented Programming 的缩写，意思为面向切面编程。
	是通过预编译方式和运行期 动态代理 实现程序功能的统一维护的一种技术。
	
作用和优势：
	作用：在程序运行期间，在不修改源码的情况下对方法进行功能增强。
	优势：减少重复代码，提高开发效率，并且便于维护。
	
底层实现：
	AOP 的底层是通过 Spring 提供的的 动态代理技术 实现的。
	在运行期间，Spring通过 动态代理技术 动态的生成代理对象，代理对象方法执行时进行增强功能的介入，在去调用目标对象的方法，从而完成功能的增强。
```

#### 5.2 AOP的动态代理技术

```
常用的动态代理技术：
	JDK 代理 : 基于接口的动态代理技术。
	cglib 代理：基于父类的动态代理技术。
```

#### 5.3 JDK代理

1.目标类接口

```java
public interface TargetInterface {
    public void method();
}
```

2.目标类

```java
public class Target implements TargetInterface {
    @Override
    public void method() {
        System.out.println("Target running....");
    }
}
```

3.动态代理代码及运行

​	方法一，直接运行：

```java
//动态代理代码
Target target = new Target(); //创建目标对象
//创建代理对象
TargetInterface proxy = (TargetInterface) Proxy.newProxyInstance(target.getClass()
.getClassLoader(),target.getClass().getInterfaces(),new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) 
            throws Throwable {
                System.out.println("前置增强代码...");
                Object invoke = method.invoke(target, args);
                System.out.println("后置增强代码...");
                return invoke;
            }
        }
);
 
//运行
// 测试,当调用接口的任何方法时，代理对象的代码都无序修改
proxy.method();
```

​	方法二:创建代理对象的类：

```java
public class TargetProxy {

    /**
     *
     *  为传入的参数对象创建一个动态代理对象
     * @param targetInterface 被代理对象
     * @return
     */
    public static TargetInterface getProxy(final TargetInterface targetInterface){


        //被代理对象的类加载器
        ClassLoader loader = targetInterface.getClass().getClassLoader();
        //被代理对象的接口
        Class<?>[] interfaces = targetInterface.getClass().getInterfaces();
        //方法执行器，执行被代理对象的目标方法
        InvocationHandler h = new InvocationHandler() {
            /**
             *  执行目标方法
             * @param proxy 代理对象，给jdk使用，任何时候都不要操作此对象
             * @param method 当前将要执行的目标对象的方法
             * @param args 这个方法调用时外界传入的参数值
             * @return
             * @throws Throwable
             */
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                //利用反射执行目标方法,目标方法执行后的返回值
//                System.out.println("这是动态代理执行的方法");
                Object result = null;
                try {
                    System.out.println(method.getName()+"方法开始执行，参数是："+ Arrays.asList(args));
                    result = method.invoke(calculator, args);
                    System.out.println(method.getName()+"方法执行完成，结果是："+ result);
                } catch (Exception e) {
                    System.out.println(method.getName()+"方法出现异常："+ e.getMessage());
                } finally {
                    System.out.println(method.getName()+"方法执行结束了......");
                }
                //将结果返回回去
                return result;
            }
        };
        Object proxy = Proxy.newProxyInstance(loader, interfaces, h);
        return (TargetInterface) proxy;
    }
}
```

```java
//运行
TargetInterface proxy = TargetProxy.getProxy(new Target());
proxy.method();
```

#### 5.4 CGLIB代理

1.目标类

```java
public class Target {
    public void method() {
        System.out.println("Target running....");
    }
}
```

2.动态代理代码及运行

```java
Target target = new Target();//创建目标对象
Enhancer enhancer = new Enhancer();   //创建增强器
enhancer.setSuperclass(Target.class); //设置父类
enhancer.setCallback(new MethodInterceptor() { //设置回调
    @Override
    public Object intercept(Object o, Method method, Object[] objects,
                            MethodProxy methodProxy) throws Throwable {
        System.out.println("前置代码增强1....");
        Object invoke = method.invoke(target, objects);
        System.out.println("后置代码增强1....");
        return invoke;
    }
});
//运行
Target proxy1 = (Target) enhancer.create(); //创建代理对象
proxy1.method();
```

#### 5.5 AOP相关术语

###### 术语：

```
-Aspect（切面）：是 切入点 和 通知 的结合。
	在 Aspect 中会包含着一些 Pointcut 以及相应的 Advice。

-Joinpoint（连接点）：所谓连接点是指那些被拦截到的点。
	在spring中,这些点指的是方法，因为spring只支持方法类型的连接点。

-Advice（通知/ 增强）：所谓通知是指拦截到 Joinpoint 之后所要做的事情就是通知。
	通知有多种类型，包括“around”, “before” and “after”等等。

-Pointcut（切入点）：所谓切入点是指我们要对哪些 Joinpoint 进行拦截的定义。

-Target（目标对象）：代理的目标对象。
	织入 Advice 的目标对象。

-Proxy （代理）：一个类被 AOP 织入增强后，就产生一个结果代理类。

-Weaving（织入）：是指把增强应用到目标对象来创建新的代理对象的过程。
	将 Aspect 和其他对象连接起来, 并创建 Advice object 的过程。
	spring采用动态代理织入，而AspectJ采用编译期织入和类装载期织入。

个人总结：
    Pointcut（切入点）：被增强的方法。
    Advice（通知/ 增强）：封装增强业务逻辑的方法。
    Aspect（切面）：切点+通知。
    Weaving（织入）：将切点与通知结合的过程。
	在 Spring AOP 中 Jointpoint 指代的是所有方法的执行点, 而 pointcut 是一个描述信息, 它修饰的是 Jointpoint, 通过 pointcut, 我们就可以确定哪些 Jointpoint 可以被织入 Advice，而这一整套动作，可以被认为是一个 Aspect。
```

###### 通知的分类：

```
-前置通知（Before advice）: 
	在 joinpoint 前被执行的 advice。它并不能够阻止 joinpoint 的执行, 除非发生了异常。
	在连接点之前运行，但无法阻止执行流程进入连接点的通知（除非它引发异常）。
	
-后置返回通知（After returning advice）:
	在一个 joinpoint 正常返回后执行的 advice。
	在连接点正常完成后执行的通知（例如，当方法没有抛出任何异常并正常返回时）。
	
-后置异常通知（After throwing advice）: 
	当一个 joinpoint 抛出异常后执行的 advice。
	在方法抛出异常退出时执行的通知。
	
-后置通知（总会执行）（After (finally) advice）: 
	无论一个 joinpoint 是正常退出还是发生了异常, 都会被执行的 advice。
	当连接点退出的时候执行的通知（无论是正常返回还是异常退出）。
	
-环绕通知（Around Advice）:
	在 joinpoint 前和 jointpoint 退出后都执行的 advice。 这个是最常用的 advice。
	环绕通知可以在方法调用前后完成自定义的行为。
	它可以选择是否 继续执行连接点 或 直接返回自定义的返回值 又或 抛出异常将执行结束。
```

#### 5.6 应用场景

```
- 日志管理
- 权限认证
- 安全检查
- 事务控制
```

#### 5.7 基于Spring的AOP开发

##### 0.注意点

```
在spring容器中：
	如果有接口，会使用jdk自带的动态代理；容器中保存的组件是代理对象com.sun.proxy.$Proxy。
	如果没有接口，会使用cglib的动态代理；代理对象是 $$EnhancerBySpringCGLIB$$ 类型。
```

##### 1.导入 AOP 相关坐标

```xml
<!--导入spring的context坐标，context依赖aop-->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>5.3.2</version>
</dependency>
<!-- aspectj的织入 -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.6</version>
    <scope>runtime</scope>
</dependency>
```

##### 2.创建目标接口和目标类（内部有切点）

**2.1:有接口，使用JDK代理**

2.1.1:目标类接口

```java
public interface TargetInterface {
    public void method();
}
```

2.1.2:目标类

```java
public class Target implements TargetInterface {
    @Override
    public void method() {
        System.out.println("Target running....");
    }
}
```

**2.2:无接口，使用CGLIB代理**

2.2.1:目标类

```java
public class Target {
    @Override
    public void method() {
        System.out.println("Target running....");
    }
}
```

##### 3.创建切面类（内部有增强方法）

```java
public class MyAspect {
    //前置增强方法
    public void before(){
        System.out.println("前置代码增强.....");
    }
}
```

##### 4.将目标类和切面类的对象创建权交给 spring

**4.1:XML方式**

```xml
<!--
	在spring的核心配置文件applicationContext下:
	class路径根据自己实际的全限定类名进行调整
-->
<!--配置目标类-->
<bean id="target" class="cn.yurb.aop.Target"></bean>
<!--配置切面类-->
<bean id="myAspect" class="cn.yurb.aop.MyAspect"></bean>
```

**4.2:注解方式**

```xml
<!--
	在spring的核心配置文件applicationContext.xml下:
    1.需要添加AOP命名空间
	2.开启注解扫描 
-->
<context:component-scan base-package="cn.yurb"></context:component-scan>

3.在Target 和 MyAspect类上添加 @Repository 注解
```

##### 5.在 applicationContext.xml 中配置织入关系

**5.1:XML方式**

```xml
<aop:config>
    <!--引用myAspect的Bean为切面对象-->
    <aop:aspect ref="myAspect">
        <!--配置Target的method方法执行时要进行myAspect的before方法前置增强-->
        <aop:before method="before" pointcut="execution(public void cn.yurb.aop.Target.method())"></aop:before>
    </aop:aspect>
</aop:config>
```

**5.2:注解方式**

```xml
<!-- 1.在applicationContext.xml下开启扫描 -->
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

```java
//2.在切面类上添加 @Aspect 注解
//3.在切面类上的具体方法上添加AOP通知
@Repository
@Aspect
public class MyAspect {

    @Before("execution(public void cn.yurb.aop.impl.TargetImpl.method())")
    public void before(){
        System.out.println("spring aop 的前置通知,注解方式");
    }
}
```

##### 6.测试代码

**6.1:XML方式**

6.1.1:JDK代理

```java
public class test_aop2 {
    @Test
    public void test(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        //注意这里的类型是接口类型，而不是实体类类型
        TargetInterface target = (TargetInterface)context.getBean("target");
        target.method();
        System.out.println(target.getClass());
        System.out.println("~~~~~~~~~~~~~~~~~~");
	}
}
```

6.1.2:CGLIB代理

```java
public class test_aop2 {
    @Test
    public void test(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
		
		//没有接口，所以直接是实体类类型
        Target target1 = (Target)context.getBean("target");
        target1.method();
        System.out.println(target1.getClass());
        System.out.println("~~~~~~~~~~~~~~~~~~");
	}
}
```

**6.2:注解方式**

**6.2.1:JDK代理**

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class test_anno {

    @Autowired
    private TargetInterface target;

    @Test
    public void test(){
        target.method();
        System.out.println(target.getClass());
    }
}

```

**6.2.2:CGLIB代理**

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class test_anno {

    @Autowired
    private Target target;

    @Test
    public void test(){
        target.method();
        System.out.println(target.getClass());
    }
}
```

##### 7.切入点表达式：通配符

###### 表达式格式：

```xml
execution（modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern（param-pattern）throws-pattern?）

除了返回类型模式（上面代码片断中的ret-type-pattern），名字模式和参数模式以外， 所有的部分都是可选的。
```

###### *

```
1、匹配一个或者多个字符
	execution( public int com.mashibing.inter.My*alculator.*(int,int))
2、匹配任意一个参数，
	execution( public int com.mashibing.inter.MyCalculator.*(int,*))
3、只能匹配一层路径，如果项目路径下有多层目录，那么*只能匹配一层路径
4、权限位置不能使用*，如果想表示全部权限，那么不写即可
	execution( * com.mashibing.inter.MyCalculator.*(int,*))
```

###### ..

```
1、匹配多个参数，任意类型参数
execution( * com.mashibing.inter.MyCalculator.*(..))
2、匹配任意多层路径
execution( * com.mashibing..MyCalculator.*(..))
```

###### 最偷懒的写法：

```
最偷懒的方式：execution(*  *(..))    或者   execution(*  *.*(..))
```

###### 最精确的写法：

```
最精确的方式：execution( public int com.mashibing.inter.MyCalculator.add(int,int))
```

###### && 、||、 ！

```xml
&&：两个表达式同时满足
execution( public int com.mashibing.inter.MyCalculator.*(..)) && execution(* *.*(int,int) )

||：任意满足一个表达式即可
execution( public int com.mashibing.inter.MyCalculator.*(..)) && execution(* *.*(int,int) )

!：只要不是这个位置都可以进行切入
execution(!public int com.mashibing.inter.MyCalculator.*(..))
```

#### 5.8 通知方法的执行顺序

```
1、正常执行：@Before--->@After--->@AfterReturning
2、异常执行：@Before--->@After--->@AfterThrowing
```

#### 5.9 获取 待增强方法 的详细信息

###### 获取方法的信息

```java
//在切面类方法中添加JoinPoint参数
@Repository
@Aspect
public class MyAspect {

    @Before("execution(public void cn.yurb.aop.impl.TargetImpl.method())")
    public void before(JoinPoint joinPoint){
        System.out.println("spring aop 的前置通知,注解方式");
        //1.获取 待增强方法 的名字
        String name = joinPoint.getSignature().getName();
        //2.获取 待增强方法 的参数
        Object[] args = joinPoint.getArgs();
        //打印参数
        System.out.println(Arrays.asList(args));
        
    }
}
```

###### 获取方法的结果

```java
//在切面类方法中添加JoinPoint参数,还需添加一个参数来进行结果接收,并告诉spring使用哪个参数来接收结果
@Repository
@Aspect
public class MyAspect {

    @AfterReturning("execution(public void cn.yurb.aop.impl.TargetImpl.method())",returning = "result")
    public void stop(JoinPoint joinPoint, Object result){
        String name = joinPoint.getSignature().getName();
        System.out.println(name+"方法执行完成，结果是："+result);   
    }
}
```

###### 获取异常的信息

```java
//在切面类方法中添加JoinPoint参数,还需添加一个参数来进行结果接收,并告诉spring使用哪个参数来接收异常信息
@Repository
@Aspect
public class MyAspect {
    
    @AfterThrowing("execution(public void cn.yurb.aop.impl.TargetImpl.method())",throwing = "exception")
    public void logException(JoinPoint joinPoint, Exception exception){
        String name = joinPoint.getSignature().getName();
        System.out.println(name+"方法出现异常："+exception);   
    }
}
```

#### 5.10 spring对通知方法的要求

```
	spring对于通知方法的要求并不是很高，你可以任意改变方法的返回值和方法的访问修饰符，但是唯一不能修改的就是方法的参数，会出现参数绑定的错误，原因在于通知方法是spring利用反射调用的，每次方法调用得确定这个方法的参数的值。
```

#### 5.11 表达式的抽取

如果在实际使用过程中，多个方法的表达式是一致的话，那么可以考虑将切入点表达式抽取出来：

###### XML方式

```xml
<!-- applicationContext.xml文件下 -->
<!-- 在增强中使用 pointcut-ref 属性代替 pointcut 属性来引用 抽取后的切点表达式(方法名) -->
<aop:config>
    <!--引用myAspect的Bean为切面对象-->
    <aop:aspect ref="myAspect">
        <aop:pointcut id="myPointcut" expression="execution(* cn.yurb.aop.*.*(..))"/>
        <aop:before method="before" pointcut-ref="myPointcut"></aop:before>
    </aop:aspect>
</aop:config>
```

###### 注解方式

```java
/**
	1.在类中声明一个 没有实现 的 返回void 的 空方法
	2.给该方法上标注@Potintcut注解
	3.在需要增强的方法上增加 通知 注解(值是方法名)
*/
@@Component("myAspect")
@Aspect
public class MyAspect {
    @Before("myPoint()")
    public void before(){
        System.out.println("前置代码增强.....");
    }
    
    @Pointcut("execution(* cn.yurb.aop.*.*(..))")
    public void myPoint(){}
}
```

#### 5.12 环绕通知的使用

```java
@Repository
@Aspect
public class MyAspectAround {

    @Around("execution(public void cn.yurb.aop.impl.TargetImpl.method())")
    public void around(ProceedingJoinPoint proceedingJoinPoint){
        Object[] args = proceedingJoinPoint.getArgs();
        Object proceed = null;
        try {
            System.out.println("环绕前置通知:"+name+"方法开始，参数是"+Arrays.asList(args));
            //利用反射调用目标方法，就是method.invoke()
            proceed = proceedingJoinPoint.proceed(args);
            System.out.println("环绕返回通知:"+name+"方法返回，返回值是"+proceed);
        } catch (Throwable e) {
            System.out.println("环绕异常通知"+name+"方法出现异常，异常信息是："+e);
        }finally {
            System.out.println("环绕后置通知"+name+"方法结束");
        }
    }
}

```

#### 5.13 通知的执行顺序

###### 一个切面类：

```
环绕通知的执行顺序是优于普通通知的，具体的执行顺序如下：
	环绕前置-->普通前置-->目标方法执行-->环绕正常结束/出现异常-->环绕后置-->普通后置-->普通返回或者异常。
但是需要注意的是，如果出现了异常，那么环绕通知会处理或者捕获异常，普通异常通知是接收不到的，因此最好的方式是在环绕异常通知中向外抛出异常。
```

###### 多个切面类：

```
	在spring中，默认是按照切面类名称的字典顺序进行执行的，但是如果想自己改变具体的执行顺序的话，可以使用@Order注解来解决，数值越小，优先级越高。
```

### 6. Spring 配置 JdbcTemplate

#### 6.1 导入坐标

```xml
<!-- 用于数据库连接 -->
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.6</version>
</dependency>

<!-- 使用jdbc可以封装一些操作 -->
<!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.3.2</version>
</dependency>

<!-- druid的数据库连接池,不同的数据库连接池导入的包不同-->
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.4</version>
</dependency>

```

#### 6.2 在resource目录下创建 jdbc.properties

和spring的配置文件分离开，有利于后期维护

```properties
jdbc.driver=com.mysql.jdbc.Driver
#前提是已创建好数据库及对应表
jdbc.url=jdbc:mysql://localhost:3306/ssm
jdbc.username=root
jdbc.password=root
```

#### 6.3 在spring核心文件 applicationContext.xml 中添加相应的bean

```xml
<!--加载jdbc.properties-->
<context:property-placeholder location="classpath:jdbc.properties"/>

<!--druid 数据源对象-->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="username" value="${jdbc.username}"></property>
    <property name="password" value="${jdbc.password}"></property>
    <property name="url" value="${jdbc.url}"></property>
    <property name="driverClassName" value="${jdbc.driver}"></property>
</bean>

<!--jdbc模板对象-->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>

<!--** 具备具名函数的jdbcTemplate对象 **-->
<!-- 
	区别：JDBC用法中，SQL参数是用占位符？表示，并且受到位置的限制，一旦参数的位置发生变化，必须改变参数的绑定。
		 SQL具名参数是按照名称绑定，而不是位置绑定。
	具名参数只在 SImpleJdbcTemplate 和 NamedParameterJdbcTemplate 中得到支持。
-->
<bean id="namedParameterJdbcTemplate" class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
        <constructor-arg name="dataSource" ref="dataSource"></constructor-arg>
</bean>
```

#### 6.4 测试运行

###### XML方式

```java
public class test_jdbc {

    @Test
    public void run(){

        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        JdbcTemplate jdbcTemplate = (JdbcTemplate)context.getBean("jdbcTemplate");

        List<Map<String, Object>> maps = jdbcTemplate.queryForList("select * from account");
        for (Map<String, Object> map : maps) {
            System.out.println(map);
        }
        
        //具名jdbcTemplate的使用
        NamedParameterJdbcTemplate namedParameterJdbcTemplate  = (NamedParameterJdbcTemplate)context.getBean("namedParameterJdbcTemplate");
        String sql = "insert into account(name,money) values(:name,:money)";
        //以Map类型
        Map<String,Object> map = new HashMap<>();
        map.put("name","zhang");
        map.put("money",120.2);
        int update = namedParameterJdbcTemplate.update(sql, map);

        //以Bean对象
        Account account = new Account();
        account.setMoney(115);
        account.setName("yu");
        //把类对象转换为参数类型
        SqlParameterSource paramSource = new BeanPropertySqlParameterSource(account);
        update = namedParameterJdbcTemplate.update(sql, paramSource);
        System.out.println(update);
    }
}

```

###### 注解方式

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class test_jdbc {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Test
    public void run(){

        List<Map<String, Object>> maps = jdbcTemplate.queryForList("select * from account");
        for (Map<String, Object> map : maps) {
            System.out.println(map);
        }
    }
}

```

### 7.声明式事务

#### 7.1 事务的分类

###### 编程式事务

```
在代码中直接加入处理事务的逻辑，可能需要在代码中显式调用beginTransaction()、commit()、rollback()等事务管理相关的方法。
```

###### 声明式事务：基于Spring-AOP

```
在方法的外部添加注解或者直接在配置文件中定义，将事务管理代码从业务方法中分离出来，以声明的方式来实现事务管理。spring的AOP恰好可以完成此功能：事务管理代码的固定模式作为一种横切关注点，通过AOP方法模块化，进而实现声明式事务。
```

#### 7.2 声明式事务的配置

###### XML

在spring核心配置文件applicationContext.xml下：

```xml
<!-- 事务配置,需要有dataSource的Bean对象 -->
<!--1.平台事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!--2.事务增强配置-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="*"/>
    </tx:attributes>
</tx:advice>

<!--3.事务的aop增强，将切入点表达式和事务通知联系起来-->
<aop:config>
    <aop:pointcut id="myPointcut" expression="execution(* cn.yurb.aop.*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="myPointcut"></aop:advisor>
</aop:config>
```

###### 注解

```xml
<!-- 1.配置事务管理器，事务依赖于数据源所产生的连接对象，只有连接对象创建成功了才能够处理事务 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!-- 2.开启注解驱动，即对事务相关的注解进行扫描，解析含义并执行功能 -->
<tx:annotation-driven transaction-manager="transactionManager"/>

<!--3.在需要开启事务的方法上加上 @Transactional 注解-->
```

#### 7.3 事务配置的属性

```java
isolation：设置事务的隔离级别
propagation：事务的传播行为
noRollbackFor：那些异常事务可以不回滚
noRollbackForClassName：填写的参数是全类名
rollbackFor：哪些异常事务需要回滚
rollbackForClassName：填写的参数是全类名
readOnly：设置事务是否为只读事务		
timeout：事务超出指定执行时长后自动终止并回滚,单位是秒
```

#### 7.4 超时属性

防止长期运行的事务占用资源，事务在超过超时时间后，会强制回滚到之前。

###### XML

```xml
<!-- 事务配置,需要有dataSource的Bean对象 -->
<!--1.平台事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!--2.事务增强配置-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <!--以check开头的方法，都受到事务控制，这里设置超时时间为3，单位为S-->
		<tx:method name="check*" timeout="3"/>
    </tx:attributes>
</tx:advice>

<!--3.事务的aop增强，将切入点表达式和事务通知联系起来-->
<aop:config>
    <aop:pointcut id="myPointcut" expression="execution(* cn.yurb.aop.*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="myPointcut"></aop:advisor>
</aop:config>
```

```java
@Service
public class Method {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void check(){

        jdbcTemplate.update("update account set money = 666 where id = 1");
        
        //当check()方法里睡眠4S再执行sql语句时，会因为超时而导致全部回滚。设置金额为666也会失败。
        try{
            Thread.sleep(4000);
        }catch (Exception e){

        }
        
        jdbcTemplate.update("update account set money = 999 where id = 1");

    }

}
```

###### 注解

注解方式的 applicationContext.xml 配置和 8.2.注解 一样

```java
//设置超时时间为3S
@Transactional(timeout = 3)
public void check_aop(){

    jdbcTemplate.update("update account set money = 666 where id = 1");
    try{
        Thread.sleep(2000);
    }catch (Exception e){

    }
    jdbcTemplate.update("update account set money = 999 where id = 1");

}
```

#### 7.5 事务只读属性

​		如果你一次执行单条查询语句，则没有必要启用事务支持，数据库默认支持SQL执行期间的读一致性；

​		如果你一次执行多条查询语句，例如统计查询，报表查询，在这种场景下，多条查询SQL必须保证整体的读一致性，否则，在前条SQL查询之后，后条SQL查询之前，数据被其他用户改变，则该次整体的统计查询将会出现读数据不一致的状态，此时，应该启用事务支持。

​		对于只读查询，可以指定事务类型为readonly，即只读事务。由于只读事务不存在数据的修改，因此数据库将会为只读事务提供一些优化手段.

###### XML

```xml
<!--其他配置 或 Check()方法 参考 超时属性 -->
<!--2.事务增强配置-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <!--以check开头的方法，都受到事务控制。
			这里设置超时时间为3，单位为S。
			设置只读属性为true。
		-->
		<tx:method name="check*" timeout="3" read-only="true"/>
    </tx:attributes>
</tx:advice>
```

###### 注解

注解方式的 applicationContext.xml 配置和 8.2.注解 一样

```java
//设置超时时间为3S,只读属性设置为true
@Transactional(timeout = 3,readOnly = true)
```

#### 7.6 设置异常不回滚

注意：运行时异常默认回滚，编译时异常默认不回滚。

###### XML

```xml
<!--其他配置 或 Check()方法 参考 超时属性 -->
<!--2.事务增强配置-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <!--以check开头的方法，都受到事务控制。
			这里设置超时时间为3，单位为S。
			设置只读属性为true。
		-->
		<tx:method name="check*" timeout="3" read-only="true" no-rollback-for="java.lang.ArithmeticException"/>
    </tx:attributes>
</tx:advice>
```

###### 注解

```java
//ArithmeticException.class 除0异常  NullPointerException.class 空指针异常 
@Transactional(timeout = 3,noRollbackFor = {ArithmeticException.class,NullPointerException.class})

//ArithmeticException.class 除0异常 
@Transactional(timeout = 3,noRollbackForClassName = {"java.lang.ArithmeticException"})
```

#### 7.7 设置异常回滚

###### XML

```xml


<!--其他配置 或 Check()方法 参考 超时属性 -->
<!--2.事务增强配置-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <!--以check开头的方法，都受到事务控制。
			这里设置超时时间为3，单位为S。
			设置只读属性为true。
		-->
		<tx:method name="check*" timeout="3" read-only="true" no-rollback-for="java.lang.ArithmeticException" rollback-for="全限定类名"/>
    </tx:attributes>
</tx:advice>
```

###### 注解

```java
@Transactional(timeout = 3,rollbackFor = {FileNotFoundException.class})
```

#### 7.8 设置隔离级别

| 隔离级别 | 异常情况 | 异常情况   | 异常情况 |
| -------- | -------- | ---------- | -------- |
| 读未提交 | 脏读     | 不可重复读 | 幻读     |
| 读已提交 |          | 不可重复读 | 幻读     |
| 可重复读 |          |            | 幻读     |
| 序列化   |          |            |          |

```
读未提交(Read uncommitted)：一个事务可以读取另一个未提交事务的数据，最低级别。
						任何情况都无法保证。
读已提交(Read committed)：一个事务要等另一个事务提交后才能读取数据。
						可避免脏读的发生。
可重复读(Repeatable read)：就是在开始读取数据（事务开启）时，不再允许修改操作。
						可避免脏读、不可重复读的发生。
串行(Serializable)：是最高的事务隔离级别，在该级别下，事务串行化顺序执行。
						可以避免脏读、不可重复读与幻读。
	  串行的特点：效率低下，比较耗数据库性能；
				强制事务排序，使之不可能相互冲突，从而解决幻读问题。
				简言之,它是在每个读的 数据行 上加上共享锁。
                能导致大量的超时现象和锁竞争。	
```

MySQL数据库默认的隔离级别是 可重复读。

###### 脏读

```
原因：读取到了其他事务没有提交的数据。
```

###### 不可重复读

```
原因：有一个事务需要读两次数据，因为一个事务只能读取到已提交的数据。
	 当读完第一次时，同时有另一个事务改变了数据，并提交；
	 当前事务第二次读取时，就会出现数据和上次不一样。
	 即同一个事务中多次读取数据出现不一致的情况。
```

###### 幻读

```
原因：一个事务在读取数据时，另一个事务插入或修改了数据，导致上个事务第二次读取数据时，数据不一致。
```

###### 不可重复读和幻读的区别

```
不可重复读的重点是修改；
幻读的重点是在于新增或删除。
```

#### 7.9 事务的7种传播特性

传播行为：如果在开始当前事务之前，一个事务上下文已经存在，此时可以指定一个事务性方法的执行行为。

![事务的传播特性](G:\GitResp\java\note\SSM\事务的传播特性.png)

###### PROPAGATION_REQUIRED=0

```
如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
	无父事务时：子事务作为独立事务执行。
	有父事务时：子事务中的操作串入父事务中执行，并且一起提交，一个操作失败全部回滚。
```

###### PROPAGATION_SUPPORTS=1

```
如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
	无父事务时：以非事务方式执行
	有父事务时：加入父事务执行，等同于PROPAGATION_REQUIRED
```

###### PROPAGATION_MANDATORY=2

```
如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
	无父事务时：抛出异常
	有父事务时：加入父事务执行，等同于PROPAGATION_REQUIRED
```

###### PROPAGATION_REQUIRES_NEW=3

```
创建一个新的事务，如果当前存在事务，则把当前事务挂起。
	挂起(Suspend)：通知TransactionManager不再检查某事务的状态，直到Resume

	无父事务时：子事务新建事务作为独立事务执行
	有父事务时：子事务新建事务作为独立事务执行，独立提交；
	
注意点：
例：
    T1{

      O(A);

      T2{
        O(B);

        O(C); 
      };

      O(D); 
    };

子事务T2不受父事务T1回滚的影响，但仍作为T1的子逻辑,
O(D)失败，O(A)回滚，T2中的事务不回滚；
T2失败回滚，T1捕获异常后，可以选择提交或回滚，未捕获异常，同T2一起回滚。
```

###### NOT_SUPPORTED=4

```
以非事务方式运行，如果当前存在事务，则把当前事务挂起。
	无父事务时：以非事务方式执行。
	有父事务时：挂起父事务，自己按照无事务方式运行。
	
  子事务自身无回滚，出现异常若向上抛，可能导致父事务回滚。
  父事务回滚时，不会影响子事务。
```

###### NEVER=5

```
以非事务方式运行，如果当前存在事务，则抛出异常。
	无父事务时：以非事务方式执行。
	有父事务时：抛出异常(若不处理会导致父事务回滚)。
```

###### NESTED=6

```
如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行
	无父事务时：创建独立事务，等同于PROPAGATION_REQUIRED
	有父事务时：嵌套在父事务之内
	
子事务依赖父事务：子事务于父事务提交时提交；父事务回滚，子事务也回滚。
Savepoint：子事务回滚时，父事务不回滚
```

###### 图解

![事务的传播特性图解](G:\GitResp\java\note\SSM\事务的传播特性图解.png)

## 3.SpringMVC

### 1.具体执行流程

​	当发起请求时被前置的控制器拦截到请求，根据请求参数生成代理请求，找到请求对应的实际控制器，控制器处理请求，创建数据模型，访问数据库，将模型响应给中心控制器，控制器使用模型与视图渲染视图结果，将结果返回给中心控制器，再将结果返回给请求者。

![springmvc运行流程](G:\GitResp\java\note\SSM\springmvc运行流程.jpg)

```
1.用户发出请求，前端控制器DispatcherServlet接收请求并拦截请求。
2.DispatcherServlet调用 处理器映射器HandlerMapping。HandlerMapping根据请求url查找Handler。
3.处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。
4.DispatcherServlet调用HandlerAdapter处理器适配器，让其按照特定的规则去执行Handler。
5.HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。
6.Controller将具体的执行信息返回给HandlerAdapter,如ModelAndView。
7.HandlerAdapter将Controller执行结果ModelAndView返回给DispatcherServlet。
8.DispatcherServlet调用视图解析器(ViewResolver)来解析HandlerAdapter传递的逻辑视图名(ModelAndView)。
9.视图解析器(ViewResolver)将解析的逻辑视图名(具体的View)传给DispatcherServlet。
10.DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图，进行视图渲染。
11.DispatcherServlet将响应数据返回给客户端。
```

### 2.SpringMVC组建解析

###### 1.**前端控制器：DispatcherServlet**

​    用户请求到达前端控制器，它就相当于 MVC 模式中的 C，DispatcherServlet 是整个流程控制的中心，由

它调用其它组件处理用户的请求，DispatcherServlet 的存在降低了组件之间的耦合性。

###### 2.**处理器映射器：HandlerMapping**

​    HandlerMapping 负责根据用户请求找到 Handler 即处理器，SpringMVC 提供了不同的映射器实现不同的

映射方式，例如：配置文件方式，实现接口方式，注解方式等。

###### 3.**处理器适配器：HandlerAdapter**

​    通过 HandlerAdapter 对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理

器进行执行。

###### 4.**处理器：Handler**

​    它就是我们开发中要编写的具体业务控制器。由 DispatcherServlet 把用户请求转发到 Handler。由

Handler 对具体的用户请求进行处理。

###### 5.**视图解析器：View Resolver**

​    View Resolver 负责将处理结果生成 View 视图，View Resolver 首先根据逻辑视图名解析成物理视图名，即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户。

###### 6.**视图**：View

​    SpringMVC 框架提供了很多的 View 视图类型的支持，包括：jstlView、freemarkerView、pdfView等。最常用的视图就是 jsp。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面。

### 3.基于XML的SpringMVC开发流程

###### 1.pom.xml导入依赖

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
<!-- spring依赖 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.2</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
<!-- SpringMVC依赖 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.3.3</version>
</dependency>

<!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
<!-- servlet-api依赖 -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
</dependency>
```

###### 2.web.xml添加配置

1.配置DispatcherServlet

2.匹配servlet的请求，/标识匹配所有请求

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    
    <!--配置DispatcherServlet-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--关联配置文件，用于初始化时加载-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
    </servlet>
    <!--匹配servlet的请求，/标识匹配所有请求-->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!--/*和/都是拦截所有请求，/会拦截的请求不包含*.jsp,而/*的范围更大，还会拦截*.jsp这些请求-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    
</web-app>
```

###### 3.创建控制器类，需要实现Controller接口

```java
public class FirstController implements Controller {

    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        //创建模型和视图对象
        ModelAndView mv = new ModelAndView();
        //将需要的值传递到model中
        mv.addObject("msg","helloSpringMVC");
        //设置要跳转的视图 index.jsp页面
        mv.setViewName("index");
        return mv;
    }
}
```

###### 4.applicationContext.xml文件配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
		 http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--处理映射器-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"></bean>
    <!--处理器适配器-->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"></bean>

    <!--视图解析器-->
    <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
         <!--前缀后缀和 mv.setViewName("index") 拼接起来使用;-->
        <!--配置前缀-->
        <property name="prefix" value="/"></property>
        <!--配置后缀-->
        <property name="suffix" value=".jsp"></property>
    </bean>

    <bean id="/first" class="cn.yurb.Controller.FirstController"></bean>

</beans>
```

### 4.基于注解的SpringMVC开发流程

###### 1.pom.xml导入依赖

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
<!-- spring依赖 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.2</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
<!-- SpringMVC依赖 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.3.3</version>
</dependency>

<!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
<!-- servlet-api依赖 -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
</dependency>
```

###### 2.web.xml添加配置

1.配置DispatcherServlet

2.匹配servlet的请求，/标识匹配所有请求

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    
    <!--配置DispatcherServlet-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--关联配置文件，用于初始化时加载-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
    </servlet>
    <!--匹配servlet的请求，/标识匹配所有请求-->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!--/*和/都是拦截所有请求，/会拦截的请求不包含*.jsp,而/*的范围更大，还会拦截*.jsp这些请求-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    
</web-app>
```

###### 3.创建控制器类，需要实现Controller接口

```java
//添加@Controller注解
@Controller
public class HelloController{
    /*
    * @RequestMapping就是用来标识此方法用来处理什么请求，其中的/可以取消
    * 取消后默认也是从当前项目的根目录开始查找，一般在编写的时候看个人习惯
    * 同时，@RequestMapping也可以用来加在类上，
    * */
    @RequestMapping("/hello")
    public String hello(Model model){
        model.addAttribute("msg","hello,SpringMVC");
        return "hello";
    }
}
```

###### 4.applicationContext.xml文件配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--自动扫描包，由IOC容器进行控制管理-->
    <context:component-scan base-package="cn.yurb"></context:component-scan>
    <!-- 视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
          id="internalResourceViewResolver">
        <!-- 前缀 -->
        <property name="prefix" value="/WEB-INF/jsp/" />
        <!-- 后缀 -->
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```

### 5.配置细节

###### 1.web.xml之 <Servlet> 配置前端过滤器

```xml
<!--配置DispatcherServlet-->
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--关联配置文件，用于初始化时加载-->
    <!-- 
		细节1：<init-param>元素可选：
			如果<init-param>元素存在并且通过其子元素配置了SpringMVC的路径，则应用程序在启动时会加载配置路径下的配置文件；
			如果没有<init-param>元素，则应用程序会默认到WEB-INF目录下寻找 名称-servlet.xml(格式)的文件。
  	-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </init-param>
    <!--1表示容器在启动时立即加载Servlet-->
    <!-- 
		细节2：<load-on-startup>元素可选：
			如果<load-on-startup>元素的值为1，则在应用程序启动时会立即加载该Servlet;
			如果<load-on-startup>元素不存在，则应用程序会在第一个Servlet请求时加载该Servlet.
 	-->
    <load-on-startup>1</load-on-startup>
</servlet>
```

###### 2.web.xml之 <url-pattern>url-pattern

```xml
<!--匹配servlet的请求，/标识匹配所有请求-->
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <!--/*和/都是拦截所有请求，/会拦截的请求不包含*.jsp,而/*的范围更大，还会拦截*.jsp这些请求-->
    <!-- 
		细节：tomcat下也有一个web.xml文件，所有的项目下web.xml文件都需要继承此web.xml。
     		在服务器的web.xml文件中有一个DefaultServlet用来处理静态资源，url-pattern是/。
     		而我们在自己的配置文件中如果添加了url-pattern=/会覆盖父类中的url-pattern，
				此时在请求的时候DispatcherServlet会去controller中做匹配，找不到则直接报404。

			为什么能处理jsp？
     		在父web.xml(tomcat的web.xml)文件中包含了一个JspServlet的处理，由tomcat进行处理，而不是我们定义的DispatcherServlet。
 	-->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

###### 3.注解方式之RequestMapping

```java
/**
@RequestMapping
作用：
	用于建立请求 URL 和处理请求方法之间的对应关系。

位置：
	类上，请求URL 的第一级访问目录。此处不写的话，就相当于应用的根目录。
	方法上，请求 URL 的第二级访问目录，与类上的使用@ReqquestMapping标注的一级目录一起组成访问虚拟路径。
	注：在整个项目的不同方法上不能包含相同的@RequsetMapping值。

属性：
	value：用于指定请求的URL。它和path属性的作用是一样的。
	method：用于指定请求的方式：POST GET。
	params：表示请求要接受的参数,如果定义了这个属性，那么发送的时候必须要添加参数。
         params有几种匹配规则：
          1、直接写参数的名称，param1,param2
              params = {"username"}
          2、表示请求不能包含的参数，！param1
              params = {"!username"}
          3、表示请求中需要要包含的参数但是可以限制值 param1=values  param1!=value
              params = {"username=123","age"}
              params = {"username!=123","age"}
	headers:填写请求头信息。
          chrome：User-Agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.79 Safari/537.36
          firefox:User-Agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:73.0) Gecko/20100101 Firefox/73.0
	consumers:只接受内容类型是哪种的请求，相当于指定Content-Type。
	produces:返回的内容类型 Content-Type：text/html;charset=utf-8。

通配符支持：
	@Request包含三种模糊匹配的方式，分别是：
    ？：能替代任意一个字符
    *: 能替代任意多个字符和一层路径
    **：能代替多层路径
    @RequestMapping(value = "/**/h*llo?")
    public String hello5(){
        System.out.println("hello5");
        return "hello";
    }
*/
```

### 6.@PathVariable注解

​	如果需要在请求路径中的参数像作为参数应该怎么使用呢？

​	可以使用@PathVariable注解，此注解就是提供了对占位符URL的支持，就是将URL中占位符参数绑定到控制器处理方法的参数中。

```java
@Controller
public class HelloController{

    //当参数和占位符一致时
    @RequestMapping(value = "/pathVariable/{name}")
    public String pathVariable(@PathVariable String name){
        System.out.println(name);
        return "hello";
    }
    
    //当参数和占位符不一致时
    @RequestMapping(value = "/pathVariable/{id}")
    public String pathVariable(@PathVariable("id") String name){
        System.out.println(name);
        return "hello";
    } 
}
```

### 7.REST风格

##### 原因：

```
人话：我们在获取资源的时候就是进行增删改查的操作，如果是原来的架构风格，需要发送四个请求，分别是：

查询：localhost:8080/query?id=1
增加：localhost:8080/insert
删除：localhost:8080/delete?id=1
更新：localhost:8080/update?id=1

按照此方式发送请求的时候比较麻烦，需要定义多种请求，而在HTTP协议中，有不同的发送请求的方式，分别是GET、POST、PUT、DELETE等，我们如果能让不同的请求方式表示不同的请求类型就可以简化我们的查询

GET：获取资源   		/book/1	
POST：新建资源		/book
PUT：更新资源		/book/1
DELETE：删除资源		/book/1

问题：
	我们在发送请求的时候只能发送post或者get，没有办法发送put和delete请求。
```

##### 解决：

###### 1.新建RestController.java类

```java
@Controller
public class RestController {
    @RequestMapping(value = "/user",method = RequestMethod.POST)
    public String add(){
        System.out.println("添加");
        return "success";
    }

    @RequestMapping(value = "/user/{id}",method = RequestMethod.DELETE)
    public String delete(@PathVariable("id") Integer id){
        System.out.println("删除："+id);
        return "success";
    }

    @RequestMapping(value = "/user/{id}",method = RequestMethod.PUT)
    public String update(@PathVariable("id") Integer id){
        System.out.println("更新："+id);
        return "success";
    }

    @RequestMapping(value = "/user/{id}",method = RequestMethod.GET)
    public String query(@PathVariable("id") Integer id){
        System.out.println("查询："+id);
        return "success";
    }
}
```

###### 2.web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--配置DispatcherServlet-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--关联springmvc的配置文件-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
        <!--   表示容器在启动时立即加载Servlet     -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!--匹配servlet的请求，/标识匹配所有请求-->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!--/*和/都是拦截所有请求，/会拦截的请求不包含*.jsp,而/*的范围更大，还会拦截*.jsp这些请求-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>

        <!--  需要新增一个过滤器  -->
    <filter>
        <filter-name>hiddenFilter</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <!--  过滤所有  -->
    <filter-mapping>
        <filter-name>hiddenFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

</web-app>
```

###### 3.applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--自动扫描包，由IOC容器进行控制管理-->
    <context:component-scan base-package="cn.yurb"></context:component-scan>

    <!--视图解析器-->
    <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--配置前缀-->
        <property name="prefix" value="/"></property>
        <!--配置后缀-->
        <property name="suffix" value=".jsp"></property>
    </bean>

</beans>
```

###### 4.新增rest.jsp页面来进行请求

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<form action="/user" method="post">
    <input type="submit" value="增加">
</form>
<form action="/user/1" method="post">
    <!-- name和type的值固定，value为对应类型，不区分大小写 -->
    <input name="_method" value="delete" type="hidden">
    <input type="submit" value="删除">
</form>
<form action="/user/1" method="post">
    <!-- name和type的值固定，value为对应类型，不区分大小写 -->
    <input name="_method" value="put" type="hidden">
    <input type="submit" value="修改">
</form>
<a href="/user/1">查询</a><br/>
</body>
</html>
```

### 8.SpringMVC的请求处理

#### 1.数据响应

##### 1.1 数据响应的方式

###### 页面跳转

```
1.直接返回字符串
2.通过ModelAndView对象返回
```

###### 回写数据

```
1.直接返回字符串
2.返回对象或集合    
```

##### 1.2 页面跳转

###### 1.2.1 返回字符串形式

![SpringMVC数据响应-页面跳转-返回字符串形式](G:\GitResp\java\note\SSM\SpringMVC数据响应-页面跳转-返回字符串形式.jpg)

###### 1.2.2 返回字符串形式--转发forward

```java
@Controller
public class ForWardController {

    /**
     * 当使用转发的时候可以添加前缀forward:index.jsp,此时是不会经过视图解析器的，所以要添加完整的名称
     *
     * forward:也可以由一个请求跳转到另外一个请求
     *
     * @return
     */
    @RequestMapping("/forward01")
    public String forward(){
        System.out.println("1");
        return "forward:/index.jsp";
    }

    @RequestMapping("/forward02")
    public String forward2(){
        System.out.println("2");
        return "forward:/forward01";
    }
}
```

###### 1.2.3 返回字符串形式--重定向 redirect

```java
@Controller
public class RedirectController {


    /**
     * redirect :重定向的路径
     *      相当于 response.sendRedirect("index.jsp")
     *      跟视图解析器无关
     * @return
     */
    @RequestMapping("redirect")
    public String redirect(){
        System.out.println("redirect");
        return "redirect:/index.jsp";
    }

    @RequestMapping("/redirect2")
    public String redirect2(){
        System.out.println("redirect2");
        return "redirect:/redirect";
    }
}
```

###### 1.2.4 ModelAndView对象形式1

在Controller中方法返回ModelAndView对象，并且设置视图名称。

```java
@RequestMapping(value="/hello")
public ModelAndView hello(){
    /*
		Model:模型 作用封装数据
        View：视图 作用展示数据
    */
    ModelAndView modelAndView = new ModelAndView();
    //设置模型数据
    modelAndView.addObject("username","itcast");
    //设置视图名称
    modelAndView.setViewName("success");

    return modelAndView;
}
```

###### 1.2.5 ModelAndView对象形式2

在Controller中方法形参上直接声明ModelAndView，无需在方法中自己创建。

```java
@RequestMapping(value="/hello")
public ModelAndView hello(ModelAndView modelAndView){
    modelAndView.addObject("username","itheima");
    modelAndView.setViewName("success");
    return modelAndView;
}
```

###### 1.2.6 使用Model,Map,ModelMap传输数据到页面

```java
@RequestMapping("output1")
public String output1(Model model){
    model.addAttribute("msg","hello,Springmvc");
    return "output";
}

@RequestMapping("output2")
public String output2(ModelMap model){
    model.addAttribute("msg","hello,Springmvc");
    return "output";
}

@RequestMapping("output3")
public String output1(Map map){
    map.put("msg","hello,Springmvc");
    return "output";
}
```

###### 1.2.7 使用原生的API

```java
@RequestMapping("api")
public String api(HttpSession session, HttpServletRequest request, HttpServletResponse response){
    request.setAttribute("requestParam","request");
    session.setAttribute("sessionParam","session");
    return "success";
}
```

###### 1.2.8 使用session传输数据到页面

@SessionAttribute：此注解可以表示，当向request作用域设置数据的时候同时也要向session中保存一份,此注解有两个参数，一个value（表示将哪些值设置到session中），另外一个type（表示按照类型来设置数据，一般不用，因为有可能会将很多数据都设置到session中，导致session异常）。

```java
@Controller
@SessionAttributes(value = "msg")
public class OutputController {

    @RequestMapping("output1")
    public String output1(Model model){
        model.addAttribute("msg","hello,Springmvc");
        System.out.println(model.getClass());
        return "output";
    }
}
```



##### 1.3 回写数据

###### 1.3.1 直接回写字符串

```java
//将需要回写的字符串直接返回，但此时需要通过@ResponseBody注解告知SpringMVC框架，方法返回的字符串不是跳转是直接在http响应体中返回
@RequestMapping(value="/hello1")
@ResponseBody  //告知SpringMVC框架 不进行视图跳转 直接进行数据响应
public String hello1() throws IOException {
    return "hello itheima";
}

//通过SpringMVC框架注入的response对象，使用response.getWriter().print(“hello world”) 回写数据，此时不需要视图跳转，业务方法返回值为void
@RequestMapping(value="/hello2")
public void hello2(HttpServletResponse response) throws IOException {
    response.getWriter().print("hello itcast");
}
```

###### 1.3.2 回写json格式字符串

```java
@RequestMapping(value="/hello")
@ResponseBody
public String hello() throws IOException {
    User user = new User();
    user.setUsername("lisi");
    user.setAge(30);
    //使用json的转换工具将对象转换成json格式字符串在返回
    //ObjectMapper()需要jackson-databind依赖
    ObjectMapper objectMapper = new ObjectMapper();
    String json = objectMapper.writeValueAsString(user);

    return json;
}
```

###### 1.3.3 SringMVC集成对象或集合json格式的自动转换--xml

在applicationContext.xml文件下添加如下：

```xml
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
        <property name="messageConverters">
            <list>
                <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>
            </list>
        </property>
    </bean>
```

###### 1.3.4 SringMVC集成对象或集合json格式的自动转换--注解

在applicationContext.xml文件下添加如下：

```xml
<mvc:annotation-driven/>
```

不过需要先导入mvc的命名空间：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">
       

</beans>  
```

#### 2.请求参数的处理

##### 2.1 基本参数类型

```java
/*
	Controller中的业务方法的参数名称要与请求参数的name一致，参数值会自动映射匹配。并且能自动做类型转换；
	自动的类型转换是指从String向其他类型的转换。
	http://localhost:8080/hello?username=zhangsan&age=12
*/
@RequestMapping(value="/hello")
@ResponseBody
public void hello(String username,int age) throws IOException {
    System.out.println(username);
    System.out.println(age);
}
```

##### 2.2 POJO类型

```java
//POJO类
public class User {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

```java
/*
	Controller中的业务方法的POJO参数的属性名与请求参数的name一致，参数值会自动映射匹配。
	http://localhost:8080/hello?username=zhangsan&age=12
*/
@RequestMapping(value="/hello")
@ResponseBody
public void hello(User user) throws IOException {
    System.out.println(user);
}
```

##### 2.3 数组类型

```java
/*
	Controller中的业务方法数组名称与请求参数的strs一致，参数值会自动映射匹配。
	http://localhost:8012/arr?strs=12&strs=34
*/
@RequestMapping(value="/hello")
@ResponseBody
public void hello(String[] strs) throws IOException {
    System.out.println(Arrays.asList(strs));
}
```

##### 2.4 集合参数类型

获得集合参数时，要将集合参数包装到一个POJO中才可以。

```jsp
<form action="${pageContext.request.contextPath}/hello" method="post">
        <%--表明是第一个User对象的name age--%>
        <input type="text" name="userList[0].name"><br/>
        <input type="text" name="userList[0].age"><br/>
        <input type="text" name="userList[1].name"><br/>
        <input type="text" name="userList[1].age"><br/>
        <input type="submit" value="提交">
    </form>
```

```java
package com.itheima.domain;

import java.util.List;

public class VO {

    private List<User> userList;

    public List<User> getUserList() {
        return userList;
    }

    public void setUserList(List<User> userList) {
        this.userList = userList;
    }

    @Override
    public String toString() {
        return "VO{" +
                "userList=" + userList +
                '}';
    }
}

```

```java
@RequestMapping(value="/hello")
@ResponseBody
public void hello(VO vo) throws IOException {
    System.out.println(vo);
}
```

##### 2.5 ajax请求发送集合类型数据

当使用ajax提交时，可以指定contentType为json形式，那么在方法参数位置使用@RequestBody可以直接接收集合数据而无需使用POJO进行包装

```jsp
<script src="${pageContext.request.contextPath}/js/jquery-3.3.1.js"></script>
    <script>
        var userList = new Array();
        userList.push({username:"zhangsan",age:18});
        userList.push({username:"lisi",age:28});

        $.ajax({
            type:"POST",
            url:"${pageContext.request.contextPath}/hello",
            data:JSON.stringify(userList),
            contentType:"application/json;charset=utf-8"
        });

    </script>
```

```java
@RequestMapping(value="/hello")
@ResponseBody
public void hello(@RequestBody List<User> userList) throws IOException {
    System.out.println(userList);
}
```

##### 2.6 参数绑定注解@RequestParam

```java
@Controller
public class RequestController {

    /**
     * 如何获取SpringMVC中请求中的信息
     *  默认情况下，可以直接在方法的参数中填写跟请求一样的名称，此时会默认接受参数
     *      如果有值，直接赋值，如果没有，那么直接给空值
     *
     * @RequestParam:获取请求中的参数值,使用此注解之后，参数的名称不需要跟请求的名称一致，但是必须要写
     *      public String request(@RequestParam("user") String username){
     *
     *      此注解还包含三个参数：
     *      value:表示要获取的参数值
     *      required：表示此参数是否必须，默认是true，如果不写参数那么会报错，如果值为false，那么不写参数不会有任何错误
     *      defaultValue:如果在使用的时候没有传递参数，那么定义默认值即可
     *
     *
     * @param username
     * @return
     */
    @RequestMapping("/request")
    public String request(@RequestParam(value = "user",required = false,defaultValue = "hehe") String username){
        System.out.println(username);
        return "success";
    }
}
```

##### 2.7 获得请求头信息@RequestHeader

```java
@Controller
public class RequestController {

    /**
     * 如果需要获取请求头信息该如何处理呢？
     *  可以使用@RequestHeader注解，
     *      public String header(@RequestHeader("User-Agent") String agent){
     *      相当于  request.getHeader("User-Agent")
     *
     *      如果要获取请求头中没有的信息，那么此时会报错，同样，此注解中也包含三个参数,跟@RequestParam一样
     *          value
     *          required
     *          defalutValue
     * @param agent
     * @return
     */
    @RequestMapping("/header")
    public String header(@RequestHeader("User-Agent") String agent){
        System.out.println(agent);
        return "success";
    }
}
```

##### 2.8 获得Cookie值@CookieValue

```java
@Controller
public class RequestController {

    /**
     * 如果需要获取cookie信息该如何处理呢？
     *  可以使用@CookieValue注解，
     *      public String cookie(@CookieValue("JSESSIONID") String id){
     *      相当于
     *      Cookie[] cookies = request.getCookies();
     *      for(Cookie cookie : cookies){
     *          cookie.getValue();
     *      }
     *      如果要获取cookie中没有的信息，那么此时会报错，同样，此注解中也包含三个参数,跟@RequestParam一样
     *          value
     *          required
     *          defalutValue
     * @param id
     * @return
     */
    @RequestMapping("/cookie")
    public String cookie(@CookieValue("JSESSIONID") String id){
        System.out.println(id);
        return "success";
    }
}
```

##### 2.9 获得Servlet相关API

SpringMVC支持使用原始ServletAPI对象作为控制器方法的参数进行注入，常用的对象如下：

HttpServletRequest

HttpServletResponse

HttpSession

```java
@RequestMapping(value="/quick19")
    @ResponseBody
    public void save19(HttpServletRequest request, HttpServletResponse response, HttpSession session) throws IOException {
        System.out.println(request);
        System.out.println(response);
        System.out.println(session);
    }
```

##### 2.10 @RequestBody

​	主要用来接收前端传递给后端的json字符串中的数据的(请求体中的数据的)；GET方式无请求体，所以使用@RequestBody接收数据时，前端不能使用GET方式提交数据，而是用POST方式进行提交。在后端的同一个接收方法里，@RequestBody与@RequestParam()可以同时使用，@RequestBody最多只能有一个，而@RequestParam()可以有多个。

https://blog.csdn.net/justry_deng/article/details/80972817

##### 2.11 @RespsonseEntity定制相应内容

```java
@Controller
public class OtherController {

    @RequestMapping("/testResponseEntity")
    public ResponseEntity<String> testResponseEntity(){

        String body = "<h1>hello</h1>";
        MultiValueMap<String,String> header = new HttpHeaders();
        header.add("Set-Cookie","name=zhangsan");
        return  new ResponseEntity<String>(body,header, HttpStatus.OK);
    }
}
```

#### 3.乱码问题的解决

##### GET请求

```xml
在tomcat的server.xml里把

<Connector connectionTimeout="50000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>

修改为

<Connector connectionTimeout="50000" port="8080" protocol="HTTP/1.1" redirectPort="8443" URIEncoding="UTF-8"/>
```

##### POST请求

在web.xml里新增一个过滤器来进行编码的过滤。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--配置DispatcherServlet-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--关联springmvc的配置文件-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
        <!--   表示容器在启动时立即加载Servlet     -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!--匹配servlet的请求，/标识匹配所有请求-->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!--/*和/都是拦截所有请求，/会拦截的请求不包含*.jsp,而/*的范围更大，还会拦截*.jsp这些请求-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>

        <!--  解决POST乱码问题，该filter要放在其他filter的前面  -->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <!--解决post请求乱码-->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <!--解决响应乱码-->
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    
    
    <!--  REST风格代码需要新增一个过滤器  -->
    <filter>
        <filter-name>hiddenFilter</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <!--  过滤所有  -->
    <filter-mapping>
        <filter-name>hiddenFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>


</web-app>

```

#### 4.静态资源的访问

​	SpringMVC的前端控制器DispatcherServlet的url-pattern配置的是/，会拦截所有的请求，而此时我们没有对应图片的请求处理方法。

通过以下两种方式指定放行静态资源：

•在spring-mvc.xml配置文件中指定放行的资源

​     `<mvc:resources mapping="/js/**"location="/js/"/> `

•使用`<mvc:default-servlet-handler/>`标签

```xml
<!--
此配置表示  我们自己配置的请求由controller来处理，但是不能请求的处理交由tomcat来处理
静态资源可以访问，但是动态请求无法访问
-->
<mvc:default-servlet-handler/>
```

​	如果发现还是不行，需要再添加另外的配置：

```xml
<!--保证静态资源和动态请求都能够访问-->
<mvc:annotation-driven/>
```

### 9.自定义视图解析器

###### 1.编写视图处理器

需要实现ViewReslover接口的resolveViewName方法 和 Ordered接口的 getOrder方法、setOrder方法。

```java
public class MyViewResolver implements ViewResolver, Ordered {
    private int order = 0;
    public View resolveViewName(String viewName, Locale locale) throws Exception {

        //如果前缀是msb:开头的就进行解析
        if (viewName.startsWith("msb:")){
            System.out.println("msb:");
            return new MyView();
        }else{
            //如果不是，则直接返回null,让别的视图解析器处理
            return null;
        }
    }

    //Order属性是指定解析器的优先级，数值越小，优先级越高
    public int getOrder() {
        return this.order;
    }

    public void setOrder(Integer order) {
        this.order = order;
    }
}

```

###### 2.编写视图

需要实现View接口的getContentType方法和render方法。

```java
public class MyView implements View {
    @Override
    public String getContentType() {
        return "text/html";
    }

    @Override
    public void render(Map<String, ?> map, HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        System.out.println("保存的对象是："+map);
        httpServletResponse.setContentType("text/html");
        //在这里渲染数据，取出map传入的值
        PrintWriter out = httpServletResponse.getWriter();
        out.write("<h3>精彩内容即将呈现...Loading</h3>");
        List<Object> lists= (List<Object>) map.get("video");
        out.write("<ul>");
        for(Object object:lists){
            out.write("<li><a href='download'>"+object+"</a></li>");
        }
        out.write("</ul>");
    }
}
```

###### 3.添加配置

在applicationContext.xml文档中配置自定义的视图解析器,交给Spring IOC容器管理。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="cn.yurb"></context:component-scan>
    
    <!-- 默认的视图解析器InternalResourceViewResolver的优先级最低。 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/page/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
    
    <!-- 自定义视图解析器的路径，及设置Order属性值，order的数值越小，优先级越高 -->
    <bean class="cn.yurb.view.MyViewResolver">
        <property name="order" value="1"></property>
    </bean>
</beans>
```

###### 4.编写控制器

```java
@Controller
public class MyViewController {
    @RequestMapping("myview")
    public String myView(Model model){
        List<String> vnames=new ArrayList<String>();
        vnames.add("java疯狂讲义300集");
        vnames.add("java从入门到如入土！！！");
        vnames.add("Spring,SpringMVC从入门到放弃！！！");
        vnames.add("MySql从删库到跑路！！！");
        model.addAttribute("video",vnames);
        return "msb:/index";
    }
}
```

### 10.自定义类型转换器

###### 1.编写自定义类型转换类

需要实现 Converter<S,T>接口。

```java
@Component
public class MyConverter implements Converter<String, User> {

    @Override
    public User convert(String source) {
        User user = new User();;
        System.out.println("-----------------");
        String[] split = source.split("-");
        //要求就是参数的规范是 姓名-年龄
        if (source!=null && split.length==2){
            user.setName(split[0]);
            user.setAge(Integer.parseInt(split[1]));
        }
        return user;
    }
}
```

###### 2.添加配置

在applicationContext.xml文件中添加自定义类型转换器的配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--自动扫描包，由IOC容器进行控制管理-->
    <context:component-scan base-package="cn.yurb"></context:component-scan>

<!--  开启了注解这部分需要屏蔽掉  -->
<!--    &lt;!&ndash;处理映射器&ndash;&gt;-->
<!--    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"></bean>-->
<!--    &lt;!&ndash;处理器适配器&ndash;&gt;-->
<!--    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"></bean>-->

    <!--视图解析器-->
    <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--配置前缀-->
        <property name="prefix" value="/"></property>
        <!--配置后缀-->
        <property name="suffix" value=".jsp"></property>
    </bean>


    <!-- 自定义视图解析器的路径，及设置Order属性值，order的数值越小，优先级越高 -->
    <bean class="cn.yurb.view.MyViewResolver">
        <property name="order" value="1"></property>
    </bean>

    <bean id="/first" class="cn.yurb.Controller.FirstController"></bean>

    <!-- 自定义类型转换器 -->
    <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <ref bean="myConverter"></ref>
            </set>
        </property>
    </bean>
	<!-- 将我们自己的类型转换器添加到 set 集合，使用下面这个标签来使之生效-->
    <mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>

</beans>
```

###### 3.编写控制器

```java
//HTTP访问地址：http://localhost:8012/userconvert?User=lisi-67

@Controller
public class UserConverter {

    @RequestMapping("/userconvert")
    //这里的类型一定要和传入的值类型匹配，否则不生效。
    public String add(@RequestParam("User") User user, Model model){
        System.out.println(user);
        model.addAttribute("user","user");
        return "index";
    }
}
```

### 11.自定义日期格式转换器

​		有时候我们经常需要在页面添加日期等相关信息，此时需要制定日期格式化转换器，此操作非常简单：只需要在单独的属性上添加@DateTimeFormat注解即可，制定对应的格式

###### 1.添加用户类

User.java

```java
package com.mashibing.bean;

import org.springframework.format.annotation.DateTimeFormat;

import java.util.Date;

public class User {

    private Integer id;
    private String name;
    private Integer age;
    private String gender;
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date birth;

    public User() {
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", gender='" + gender + '\'' +
                ", birth=" + birth +
                '}';
    }
}
```

###### 2.添加显示页面

index.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
<form action="dateConvertion" method="post">
  编号：<input type="text" name="id"><br>
  姓名：<input type="text" name="name"><br>
  年龄：<input type="text" name="age"><br>
  性别：<input type="text" name="gender"><br>
  日期：<input type="text" name="birth"><br>
  <input type="submit" value="提交">
</form>
  </body>
</html>

```

###### 3.添加日期格式转换控制器类

DateConvertionController.java

```java
package com.mashibing.controller;

import com.mashibing.bean.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class DateConvertionController {

    @RequestMapping("dateConvertion")
    public String dateConvertion(User user){
        System.out.println(user);
        return "hello";
    }
}
```

###### 4.添加配置

applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"

       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="com.mashibing"></context:component-scan>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/page/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
    <bean class="com.mashibing.view.MyViewResolver">
        <property name="order" value="1"></property>
    </bean>
    <!--此配置表示  我们自己配置的请求由controller来处理，但是不能请求的处理交由tomcat来处理
    静态资源可以访问，但是动态请求无法访问
    -->
    <mvc:default-servlet-handler/>
    <!--保证静态资源和动态请求都能够访问-->
    <mvc:annotation-driven></mvc:annotation-driven>
</beans>
```

​		此时运行发现是没有问题的，但是需要注意的是，如果同时配置了自定义类型转换器之后，那么日期格式转化是有问题的。

applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"

       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="com.mashibing"></context:component-scan>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/page/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
    <bean class="com.mashibing.view.MyViewResolver">
        <property name="order" value="1"></property>
    </bean>
    <!--此配置表示  我们自己配置的请求由controller来处理，但是不能请求的处理交由tomcat来处理
    静态资源可以访问，但是动态请求无法访问
    -->
    <mvc:default-servlet-handler/>
    <!--保证静态资源和动态请求都能够访问-->
    <mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
    <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <ref bean="myConverter"></ref>
            </set>
        </property>
    </bean>
</beans>
```

​		原因就在于ConversionServiceFactoryBean对象中有且仅有一个属性converters，此时可以使用另外一个类来做替换FormattingConversionServiceFactoryBean

applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"

       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="com.mashibing"></context:component-scan>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/page/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
    <bean class="com.mashibing.view.MyViewResolver">
        <property name="order" value="1"></property>
    </bean>
    <!--此配置表示  我们自己配置的请求由controller来处理，但是不能请求的处理交由tomcat来处理
    静态资源可以访问，但是动态请求无法访问
    -->
    <mvc:default-servlet-handler/>
    <!--保证静态资源和动态请求都能够访问-->
<!--    <mvc:annotation-driven></mvc:annotation-driven>-->
    <mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
    <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <ref bean="myConverter"></ref>
            </set>
        </property>
    </bean>
</beans>
```

### 12.数据校验

​		一般情况下我们会在前端页面实现数据的校验，但是大家需要注意的是前端校验会存在数据的不安全问题，因此一般情况下我们都会使用前端校验+后端校验的方式，这样的话既能够满足用户的体验度，同时也能保证数据的安全，下面来说一下在springmvc中如何进行后端数据校验。

​		JSR303是 Java 为 Bean 数据合法性校验提供的标准框架，它已经包含在 JavaEE 6.0 中 。JSR 303 (Java Specification Requests意思是Java 规范提案)通过**在** **Bean** **属性上标注**类似于 @NotNull、@Max 等标准的注解指定校验规则，并通过标准的验证接口对 Bean 进行验证。

JSR303:

![](C:\Users\YuRB\Desktop\java-master\java-master\javaframework\springmvc\04SpringMVC的基本使用3\image\JSR303.png)

Hibernate Validator 扩展注解:

![](C:\Users\YuRB\Desktop\java-master\java-master\javaframework\springmvc\04SpringMVC的基本使用3\image\hibernate.png)

###### 1.导入依赖

​	spring中拥有自己的数据校验框架，同时支持JSR303标准的校验框架，可以在通过添加注解的方式进行数据校验。在spring中本身没有提供JSR303的实现，需要导入依赖的包。

pom.xml

```xml
<!-- https://mvnrepository.com/artifact/org.hibernate/hibernate-validator -->
<!-- 高版本可能不兼容，推荐使用5版本及以下 -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.1.0.Final</version>
</dependency>
```

###### 2.添加请求页面

index.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: root
  Date: 2020/3/12
  Time: 15:23
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
<form action="dataValidate" method="post">
  编号：<input type="text" name="id"><br>
  姓名：<input type="text" name="name"><br>
  年龄：<input type="text" name="age"><br>
  性别：<input type="text" name="gender"><br>
  日期：<input type="text" name="birth"><br>
  邮箱：<input type="text" name="email"><br>
  <input type="submit" value="提交">
</form>
  </body>
</html>

```

###### 3.添加用户类

User2.java

```java
package cn.yurb.domain;


import org.hibernate.validator.constraints.Email;
import org.hibernate.validator.constraints.Length;
import org.springframework.format.annotation.DateTimeFormat;

import javax.validation.constraints.NotNull;
import jakarta.validation.constraints.Past;

import java.util.Date;

public class User2 {

    private Integer id;
    @NotNull
    @Length(min = 5,max = 10)
    private String name;
    private Integer age;
    private String gender;
    @Past
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date birth;
    @Email
    private String email;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", gender='" + gender + '\'' +
                ", birth=" + birth +
                ", email='" + email + '\'' +
                '}';
    }


}
```

###### 4.添加数据校验控制器类

DataValidateController.java

```java
import javax.validation.Valid;

@Controller
public class DataValidateController {
    @RequestMapping("/dataValidate")
    /*
    	在方法上，对属性标上 @Valid 注解，表示我们对这个对象属性需要进行验证。
    	直接添加一个BindingResult，用来存放验证结果。
    */
    public String validate(@Valid User user, BindingResult bindingResult) {
        System.out.println(user);
        if (bindingResult.hasErrors()) {
            System.out.println("验证失败");
            return "redirect:/index.jsp";
        } else {
            System.out.println("验证成功");
            return "hello";
        }
    }
}
```

###### 5.优化

在报错的地方无法出现错误提示，可以换另外一种方式：

index.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
<a href="add">添加用户</a>

  </body>
</html>

```

add.jsp

```jsp
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>$Title$</title>
</head>
<body>
<!-- 设置modelAttribute=”customer”，它的属性值，必须为小写的类型名 -->
<form:form action="dataValidate"  modelAttribute="user2" method="post">
    id:<form:input path="id"></form:input><form:errors path="id"></form:errors> <br/>
    name:<form:input path="name"></form:input><form:errors path="name"></form:errors><br/>
    age:<form:input path="age"></form:input><form:errors path="age"></form:errors><br/>
    gender:<form:input path="gender"></form:input><form:errors path="gender"></form:errors><br/>
    birth:<form:input path="birth"></form:input><form:errors path="birth"></form:errors><br/>
    email:<form:input path="email"></form:input><form:errors path="email"></form:errors><br/>
    <input type="submit" value="submit">
</form:form>
</body>
</html>

```

DataValidateController.java

```java
package com.mashibing.controller;

import com.mashibing.bean.User;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.validation.Valid;


@Controller
public class DataValidateController {
    @RequestMapping("/dataValidate")
    public String validate(@Valid User2 user, BindingResult bindingResult, Model model) {
        System.out.println(user);
        if (bindingResult.hasErrors()) {
            System.out.println("验证失败");
            return "add";
        } else {
            System.out.println("验证成功");
            return "hello";
        }
    }

    @RequestMapping("add")
    public String add(Model model){
        model.addAttribute("user2",new User(1,"zhangsan",12,"女",null,"1234@qq.com"));
        return "add";
    }
}
```

web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
    </context-param>
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    <filter>
        <filter-name>encoding</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encoding</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

###### 6.表单回显错误信息：

DataValidateController.java

```java
package com.mashibing.controller;

import com.mashibing.bean.User;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.validation.Valid;
import java.util.HashMap;
import java.util.List;
import java.util.Map;


@Controller
public class DataValidateController {
    @RequestMapping("/dataValidate")
    public String validate(@Valid User2 user, BindingResult bindingResult, Model model) {
        System.out.println(user);
        Map<String,Object> errorsMap = new HashMap<String, Object>();
        if (bindingResult.hasErrors()) {
            System.out.println("验证失败");
            List<FieldError> fieldErrors = bindingResult.getFieldErrors();
            for (FieldError fieldError : fieldErrors) {
                System.out.println(fieldError.getDefaultMessage());
                System.out.println(fieldError.getField());
                errorsMap.put(fieldError.getField(),fieldError.getDefaultMessage());
            }
            model.addAttribute("errorInfo",errorsMap);
            return "add";
        } else {
            System.out.println("验证成功");
            return "hello";
        }
    }

    @RequestMapping("add")
    public String add(Model model){
        model.addAttribute("user2",new User(1,"zhangsan",12,"女",null,"1234@qq.com"));
        return "add";
    }
}
```

add.jsp

```jsp
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>$Title$</title>
</head>
<body>
<!-- 设置modelAttribute=”customer”，它的属性值，必须为小写的类型名 -->
<form:form action="dataValidate"  modelAttribute="user2" method="post">
    编号:<form:input path="id"></form:input><form:errors path="id"></form:errors>--->${errorInfo.id} <br/>
    姓名:<form:input path="name"></form:input><form:errors path="name"></form:errors>--->${errorInfo.name}<br/>
    年龄:<form:input path="age"></form:input><form:errors path="age"></form:errors>--->${errorInfo.age}<br/>
    性别:<form:input path="gender"></form:input><form:errors path="gender"></form:errors>--->${errorInfo.gender}<br/>
    生日:<form:input path="birth"></form:input><form:errors path="birth"></form:errors>--->${errorInfo.birth}<br/>
    邮箱:<form:input path="email"></form:input><form:errors path="email"></form:errors>--->${errorInfo.email}<br/>
    <input type="submit" value="submit">
</form:form>
</body>
</html>
```

### 13.文件下载

```java
@Controller
public class OtherController {

    @RequestMapping("/download")
    public ResponseEntity<byte[]> download(HttpServletRequest request) throws Exception {
        //获取要下载文件的路径及输入流对象
        ServletContext servletContext = request.getServletContext();
        String realPath = servletContext.getRealPath("/script/jquery-1.9.1.min.js");
        FileInputStream fileInputStream = new FileInputStream(realPath);

        byte[] bytes = new byte[fileInputStream.available()];
        fileInputStream.read(bytes);
        fileInputStream.close();
        //将要下载文件内容返回
        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.set("Content-Disposition","attachment;filename=jquery-1.9.1.min.js");
        return  new ResponseEntity<byte[]>(bytes,httpHeaders,HttpStatus.OK);
    }
}

```

### 