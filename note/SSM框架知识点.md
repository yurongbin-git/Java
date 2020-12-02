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
<!-- 添加 context 命名空间 -->

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

