## 数据库脚本准备

```
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) DEFAULT NULL,
  `password` varchar(50) DEFAULT NULL,
  `birthday` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user` VALUES ('1', 'lucy', '123', '2019-12-12');
INSERT INTO `user` VALUES ('2', 'tom', '123', '2019-12-12');
```

### 导入maven坐标

```
<dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.5</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.6</version>
    </dependency>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
    </dependency>

    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.12</version>
    </dependency>
```

### 实体类User

```
public class User {

  private int id;
  private String username;
  private String password;
  private String birthday;
  
  //省略getter/setter方法
}
```

### 主配置文件`SqlMapConfig.xml`

```
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
  <environments default="">
    <environment id="">
      <transactionManager type="JDBC"></transactionManager>
      <dataSource type="POOLED">
        <!-- 数据库连接信息-->
        <property name = "driver" value = "com.mysql.jdbc.Driver"></property>
        <property name = "url" value = "jdbc:mysql://120.27.12.173:3306/zdy_mybatis?characterEncoding=utf8"></property>
        <property name = "username" value = "lxq"></property>
        <property name = "password" value = "Lxq410221@5214"></property>
      </dataSource>
    </environment>
  </environments>

  <!-- 引入sql配置信息 -->
  <mappers>
    <mapper resource = "UserMapper.xml"></mapper>
  </mappers>
</configuration>
```

### UserMapper文件

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.liu.mapper.UserMapper">

  <!--查询用户-->
  <select id="findAll" resultType="com.liu.pojo.User">
    select * from user
  </select>
</mapper>
```



### UserMapper接口

```
public interface UserMapper {
  List<User> findAll(User user);
}
```

### 自定义插件代码编写

```
@Intercepts( //注意看这个大花括号，也就这说这里可以定义多个@Signature对多个地方拦截，都用这 个拦截器
    {
        @Signature(type = StatementHandler.class, //指定拦截的接口,
                    method = "prepare", //拦截的接口内的方法名
                    args = {Connection.class, Integer.class} //拦截方法的参数，按照方法定义的参数顺序写，如果方法重载，可以通过方法名和入参来唯一确定
        )
    }
)
public class MyPlugin implements Interceptor {

  //每次执行操作时，都会进入拦截器的这个方法
  public Object intercept(Invocation invocation) throws Throwable {
    //...增强逻辑
    System.out.println("对方法进行了增强");

    return invocation.proceed();
  }

  /**
   * 将拦截器生成代理对象放入拦截器链中
   * @param o
   * @return
   */
  public Object plugin(Object o) {
    System.out.println("将要包装的目标对象:" + o);
    return Plugin.wrap(o, this);
  }

  public void setProperties(Properties properties) {
    System.out.println("接收到的参数：" + properties);
  }
}
```

### 配置插件

需要在sqlMapConfig.xml中配置自定义插件

```
  <plugins>
    <plugin interceptor="com.liu.plugin.MyPlugin">
      <property name="name" value="sjsdkfd"/>
    </plugin>
  </plugins>
```

需要注意的是该标签需要配置在文件的其他标签前面，否则会有标红提示。

### 测试代码

```
@org.junit.Test
  public void test() throws IOException {
    //加载核心配置文件
    InputStream resourceAsStream = Resources.getResourceAsStream("SqlMapConfig.xml");
    //获得sqlSession工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    //获得sqlSession对象
    SqlSession sqlSession = sqlSessionFactory.openSession();
    //执行sql语句
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    List<User> userList = mapper.findAll(new User());
    for (User user : userList) {
      System.out.println(user);
    }
  }
```

### 输出内容

```
接收到的参数：{name=sjsdkfd}
将要包装的目标对象:org.apache.ibatis.executor.CachingExecutor@306cf3ea
将要包装的目标对象:org.apache.ibatis.scripting.defaults.DefaultParameterHandler@1e6454ec
将要包装的目标对象:org.apache.ibatis.executor.resultset.DefaultResultSetHandler@34f5090e
将要包装的目标对象:org.apache.ibatis.executor.statement.RoutingStatementHandler@31e5415e
对方法进行了增强
User{id=1, username='lucy', password='123', birthday='2019-12-12'}
User{id=2, username='tom', password='123', birthday='2019-12-12'}
User{id=3, username='liujiao', password='dfdfd', birthday='2020-11-13 02:02:01'}
```







