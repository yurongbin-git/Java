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

