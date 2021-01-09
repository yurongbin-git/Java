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

#### 1.3 JdbcTemplate常用方法执行CRUD

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

### 3.Spring 的配置文件

##### 3.1 Bean 对象的创建（实例化）

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

##### 3.2 Bean 对象的获取

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

##### 3.3 Bean 对象的属性赋值方式

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

##### 3.4 Bean对象的作用范围

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

##### 3.5 Bean 对象的初始化和销毁方法

```xml
<!--bean生命周期表示bean的创建到销毁
        如果bean是单例，容器在启动的时候会创建好，关闭的时候会销毁创建的bean
        如果bean是多例，获取的时候创建对象，销毁的时候不会有任何的调用

    init-method：指定类中的初始化方法名称
    destroy-method：指定类中销毁方法名称
-->
    <bean id="address" class="com.mashibing.bean.Address" init-method="init" destroy-method="destory"></bean>
```

##### 3.6  Bean 对象初始化方法 的 前后处理方法

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

##### 3.7 引用外部配置文件

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

##### 3.8 Spring 基于 xml 文件的自动装配

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

##### 3.9 SpEL的使用

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

### 4. Spring 配置数据库连接池

​	参考 3.7.2

### 5. Spring 注解开发

##### 1.原始注解

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

##### 2.使用注解的步骤

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

##### 3.原始注解的使用

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

##### 4.定义扫描包时要包含的类和不要包含的类

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

##### 5.@AutoWired进行自动注入的注意点：

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

##### 6.自动装配注解：@AutoWired 和 @Resoure 的区别：

```
1、@AutoWired:是spring中提供的注解，@Resource:是jdk中定义的注解，依靠的是java的标准。
2、@AutoWired默认是按照类型进行装配，默认情况下要求依赖的对象必须存在，@Resource默认是按照名字进行匹配的，同时可以指定name属性。
3、@AutoWired只适合spring框架，而@Resource扩展性更好。
```

##### 7.Spring的新注解

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

### 6.Spring整合Junit

##### 6.1 原始Junit测试Spring的问题

在测试类中，每个测试方法都有以下两行代码：

```java
 ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
 IAccountService as = ac.getBean("accountService",IAccountService.class);
```

这两行代码的作用是获取容器，如果不写的话，直接会提示空指针异常。所以又不能轻易删掉。

##### 6.2 上述问题解决思路

让SpringJunit负责创建Spring容器，但是需要将配置文件的名称告诉它。

将需要进行测试Bean直接在测试类中进行注入。

##### 6.3 Spring集成Junit步骤

①导入spring集成Junit的坐标

②使用@Runwith注解替换原来的运行期

③使用@ContextConfiguration指定配置文件或配置类

④使用@Autowired注入需要测试的对象

⑤创建测试方法进行测试

##### 6.4 Spring集成Junit代码实现

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

### 7.Spring AOP

##### 1.概念

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

##### 2.AOP的动态代理技术

```
常用的动态代理技术：
	JDK 代理 : 基于接口的动态代理技术。
	cglib 代理：基于父类的动态代理技术。
```

##### 3.JDK代理

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

##### 4.CGLIB代理

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

##### 5.AOP相关术语

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

##### 6.应用场景

```
- 日志管理
- 权限认证
- 安全检查
- 事务控制
```

##### 7.基于Spring的AOP开发

###### 0.注意点

```
在spring容器中：
	如果有接口，会使用jdk自带的动态代理；容器中保存的组件是代理对象com.sun.proxy.$Proxy。
	如果没有接口，会使用cglib的动态代理；代理对象是 $$EnhancerBySpringCGLIB$$ 类型。
```

###### 1.导入 AOP 相关坐标

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

###### 2.创建目标接口和目标类（内部有切点）

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

###### 3.创建切面类（内部有增强方法）

```java
public class MyAspect {
    //前置增强方法
    public void before(){
        System.out.println("前置代码增强.....");
    }
}
```

###### 4.将目标类和切面类的对象创建权交给 spring

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

###### 5.在 applicationContext.xml 中配置织入关系

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

###### 6.测试代码

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

##### 8.通知方法的执行顺序

```
1、正常执行：@Before--->@After--->@AfterReturning
2、异常执行：@Before--->@After--->@AfterThrowing
```

##### 9.获取 待增强方法 的详细信息

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

##### 10.spring对通知方法的要求

```
	spring对于通知方法的要求并不是很高，你可以任意改变方法的返回值和方法的访问修饰符，但是唯一不能修改的就是方法的参数，会出现参数绑定的错误，原因在于通知方法是spring利用反射调用的，每次方法调用得确定这个方法的参数的值。
```

##### 11.表达式的抽取

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

##### 12.环绕通知的使用

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

##### 13.通知的执行顺序

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

