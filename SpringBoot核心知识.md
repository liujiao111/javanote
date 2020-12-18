# SpringBoot核心知识

##  SpringBoot基础知识

#### SpringBoot特点

- 起步依赖
- 自动配置
- 简化监控

springboot解决前端中文乱码问题的两种方式：

- `@RequestMapping(produces = "application/json; charset=utf-8")`
- 配置文件中设置编码：`spring.http.encoding.force-response=true



#### 单元测试

加入单元测试依赖：

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
            <scope>test</scope>
        </dependency>
```

代码：

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class DemoApplicationTests {

  @Autowired
  private HelloController helloController;

  @Test
  public void contextLoads() {
    System.out.println(helloController.test());
  }

}
```

注：

- `@RunWith(SpringRunner.class)`测试启动器，加载`springboot`测试注解
- `@SpringBootTest`标记该类为`springboot`单元测试类，并加载项目上下文环境

#### 热部署

- 引入坐标：

```
  <!-- 引入热部署依赖 --> 
  <dependency>   
  	<groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
  </dependency>
```

- IDEA工具热部署配置

#### 全局配置文件

两种格式的配置文件类型：

- application.properties
- application.yml

@ConfigurationProperties将配置文件中以..为开头

yml文件扩展名可以为.yml或者.yaml

yml中各种数据类型的写法：

```
person:
  family: [fater, mother]
  id: 12
  name: 张三
  map: {k1: v1, k2: v2, k3: v3}
  hobby: [吃饭,睡觉,打豆豆]
  pet: {type: dog, name: 旺财}
```

千万注意：在{}的对象中，key和value值之间一定要隔一个分号，否则无法正确注入



配置文件属性值的两种方式

- 注解@ConfigurationProperties
  配合属性的set方法

- @Value

```
  @Component
  public class Student {
  
    @Value("${person.id}")
    private int id;
    @Value("${person.name}")
    private String name;
    }
    
    //使用
    @Autowired
    private Student student;
```

  需要注意的是：使用@Value注解不支持map、对象等数据类型的注入。



#### 手动加载自定义配置文件

使用`@PropertySource`注解，指定自定义配置文件所在路径：

```
@Component
@PropertySource("classpath:test.properties")
@ConfigurationProperties(prefix = "test")
```



####  手动加bean基于配置类

`@Configuration`注解:

将该方法返回的实例注入ioc容器中：

```
@Configuration //相当于xml中的一个配置
public class MyConfig {
	@Bean(name = "iService") //将返回值对象作为组件添加到spring容器中,标识id默认为方法名    		public MyService myService() {
    	return new MyService();    
    }
}
```

测试：

```
  @Test
  public void testMyConfig() {
    System.out.println(applicationContext.containsBean("iService"));;
  }
```

打印结果为true



#### 随机数设置

```
my.secret=${random.value}         // 配置随机值 
my.number=${random.int}   // 配置随机整数        
my.bignumber=${random.long}    // 配置随机long类型数
my.uuid=${random.uuid}            // 配置随机uuid类型数 my.number.less.than.ten=${random.int(10)}    // 配置小于10的随机整数 
${random.int[1,10]}// 配置1-10的随机整数 (注意：1,10之间不能有空格)
my.number.in.range=${random.int[1024,65536]} // 配置范围在[1024,65536]之间的随机整数
```





#### 参数间引用

```
person:
  id: 0
  name: 年龄可能是：${person.id}
```





## Springboot高级技术

#### SpringBoot整合mybatis

- 数据库脚本

```
   CREATE DATABASE springbootdata; 
   DROP TABLE IF EXISTS `t_article`;
  CREATE TABLE `t_article` (
    `id` int(20) NOT NULL AUTO_INCREMENT COMMENT '文章id',
    `title` varchar(200) DEFAULT NULL COMMENT '文章标题',
    `content` longtext COMMENT '文章内容',
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;
  
  -- ----------------------------
  -- Records of t_article
  -- ----------------------------
  INSERT INTO `t_article` VALUES ('1', 'Spring Boot基础入门', '从入门到精通讲 解...');
  INSERT INTO `t_article` VALUES ('2', 'Spring Cloud基础入门', '从入门到精通讲 解...');
  
  -- ----------------------------
  -- Table structure for t_comment
  -- ----------------------------
  DROP TABLE IF EXISTS `t_comment`;
  CREATE TABLE `t_comment` (
    `id` int(20) NOT NULL AUTO_INCREMENT COMMENT '评论id',
    `content` longtext COMMENT '评论内容',
    `author` varchar(200) DEFAULT NULL COMMENT '评论作者',
    `a_id` int(20) DEFAULT NULL COMMENT '关联的文章id',
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8;
  
  -- ----------------------------
  -- Records of t_comment
  -- ----------------------------
  INSERT INTO `t_comment` VALUES ('1', '很全、很详细', 'luccy', '1');
  INSERT INTO `t_comment` VALUES ('2', '赞一个', 'tom', '1');
  INSERT INTO `t_comment` VALUES ('3', '很详细', 'eric', '1');
  INSERT INTO `t_comment` VALUES ('4', '很好，非常详细', '张三', '1');
  INSERT INTO `t_comment` VALUES ('5', '很不错', '李四', '2');
  
```

- 导入Maven坐标：（mybatis-spring以及mysql驱动包）

```
  <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.3</version>
      </dependency>
  
      <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.25</version>
      </dependency>
```

- 编写实体类`Comment`

```
  public class Comment {
  
    private Integer id;
  
    private String content;
  
    private String author;
  
    private Integer aId;
    //省略getter/setter/toString方法
    }
```

##### 注解方式

- 编写mapper接口`CommentMapper`

```
  @Mapper
  public interface CommentMapper {
    @Select("select * from t_comment where id = #{id}")
    Comment selectById(Integer id);
  
  }
```

  其中@Mapper接口表示该类是一个mybatis接口文件，能够被springboot扫描并加载到容器中，可以在springboot启动类上添加MapperScan扫描某个包下的mapper文件，避免在大量的接口上添加@Mapper注解。

- 在springboot配置文件中配置数据库连接信息：

```
  # MySQL数据库连接配置
  spring.datasource.url=jdbc:mysql://localhost:3306/springbootdata?serverTimezone=UTC
  spring.datasource.username=root
  spring.datasource.password=123456
  spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

  

- 编写测试方法进行测试接口

````
  @RunWith(SpringRunner.class)
  @SpringBootTest
  public class SpringBootMybatisTests {
  
  
    @Autowired
    private CommentMapper commentMapper;
  
    @Test
    public void selectById() {
      Comment comment = commentMapper.selectById(1);
      System.out.println(comment);
    }
  
  }
````

  

- 运行结果：

```
  Comment{id=1, content='很全、很详细', author='luccy', aId=null}
```

  可以看到comment的aId字段没有值，原因是因为我们在数据库中对应的字段名为a_id，无法正确映射到当前字段上，解决办法：可以在Spring Boot全 局配置文件application.properties中添加开启驼峰命名匹配映射配置：

```
  #开启驼峰命名匹配映射
  mybatis.configuration.map-underscore-to-camel-case=true
```

  输出结果如下：

```
  Comment{id=1, content='很全、很详细', author='luccy', aId=1}
```

  此时已经能够着将字段正确映射到实体类对应字段上。

  

##### 注解方式

- 实体类：

```
  public class Article {
  
    private Integer id;
  
    private String title;
  
    private String content;
    //省略getter/setter/toString方法
    }
```

- 创建Mapper接口

```
  @Mapper
  public interface ArticleMapper {
    Article selectById(Integer id);
  }
```

  

- ArticleMapper.xml：(位置：resources/mapper文件夹下面)

```
  <?xml version="1.0" encoding="UTF-8" ?> 
  <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.liu.springbootdemo.mapper.ArticleMapper">
    <select id="selectById" parameterType="int"  resultType="Article">
    select * from t_article where id = #{id}
    </select>
  </mapper>
```

  

- Springboot文件配置：

```
  #配置MyBatis的xml配置文件路径
  mybatis.mapper-locations=classpath:mapper/*.xml
   #配置XML映射文件中指定的实体类别名路径
  mybatis.type-aliases-package=com.liu.springbootdemo.pojo
```

  

- 测试

```
    @Autowired
    private ArticleMapper articleMapper;
  
    @Test
    public void testxmlmybatis() {
      Article article = articleMapper.selectById(1);
      System.out.println(article);
    }
```

  

- 输出：

```
  Article{id=1, title='Spring Boot基础入门', cotent='从入门到精通讲 解...'}
```

  

#### SpringBoot整合JPA

- 添加依赖

```
    <dependencies>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
      </dependency>
  
  
      <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.25</version>
      </dependency>
```

  

- 编写实体类：@Entity、@Table、@Id、@GenetatedValue、@Column注解

```
  @Entity(name = "t_comment") //设置ORM实体并指定映射的表名
  public class Comment {
  
    @Id// 表明映射对应的主键id
    @GeneratedValue(strategy = GenerationType.IDENTITY)// 设置主键自增策略
    private Integer id;
  
    private String content;
  
    private String author;
  
    @Column(name = "a_id") //指定映射的表字段名
    private Integer aId;
    
    //省略getter/setter/toString方法
    }
```

  

- 创建Repository接口继承JpaRepository接口

```
  public interface CommentRepository extends JpaRepository<Comment, Integer> {
  
  }
```

  

- 编写单元测试进行接口方法测试

```
  @RunWith(SpringRunner.class)
  @SpringBootTest
  public class SpringbootJpaApplicationTests {
  
    @Autowired
    private CommentRepository commentRepository;
  
    @Test
    public void contextLoads() {
      System.out.println(commentRepository.findById(1));
    }
  
  }
```

  输出：

```
  Optional[Comment{id=1, content='很全、很详细', author='luccy', aId=1}]
```

  

  当springboot版本低于2.1.4时，启动可能会报错：

```
  ***************************
  APPLICATION FAILED TO START
  ***************************
  
  Description:
  
  Field usuarioDao in es.uc3m.tiw.Controladores.Controlador required a bean named 'entityManagerFactory' that could not be found.
  
  Action:
  
  Consider defining a bean named 'entityManagerFactory' in your configuration.
```

  只需要将springboot版本号修改为2.1.4及以上即可。

  

#### SpringBoot整合Redis

- 添加redis依赖启动器

```
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
      </dependency>
```

  

- 编写实体类。@RedisHash指定实体类对象的Redis中的存储空间@Id、

```
  @RedisHash("person") //指定操作实体类对象在Redis数据库中的存储空间
  public class Person {
  
    @Id //标识实体类主键
    private String id;
  
    @Indexed //标识对应属性在Redis数据库中生成二级索引
    private String firstname;
  
    @Indexed
    private String lastname;
  
    private Address address;
    //省略getter/setter/toString方法
    
    }
```

  针对实体类中的注解进行说明：

  - @RedisHash("persons")：用于指定操作实体类对象在Redis数据库中的存储空间，此处表示针对 Person实体类的数据操作都存储在Redis数据库中名为persons的存储空间下。
  - @Id：用于标识实体类主键。在Redis数据库中会默认生成字符串形式的HashKey表示唯一的实体 对象id，当然也可以在数据存储时手动指定id。 
  - @Indexed：用于标识对应属性在Redis数据库中生成二级索引。使用该注解后会在Redis数据库中 生成属性对应的二级索引，索引名称就是属性名，可以方便的进行数据条件查询。

- 编写Repository接口，继承CrudRepository接口,并编写查询方法

```
  public interface PersonRepository extends CrudRepository<Person, String> {
  
    List<Person> findByAddress_City(String city);
  
  }
```

  

- Redis数据库连接配置

```
  spring.redis.host=127.0.0.1
  spring.redis.port=6379
  spring.redis.password=
```

  

- 编写单元测试进行接口方法测试

```
@Autowired
  private PersonRepository personRepository;

  @Test
  public void savePersonToRedis() {
    Person person = new Person();
    person.setId("1");
    person.setFirstname("liu");
    person.setLastname("jiao");
    Address address = new Address();
    address.setCity("成都东");
    address.setCountry("四川省");
    person.setAddress(address);
    person.setAddress(address);
    personRepository.save(person);
  }

  @Test
  public void selectPersonToRedis() {

    List<Person> citys = personRepository.findByAddress_City("成都东");
    for (Person city : citys) {
      System.out.println(city);
    }
  }
```

执行`savePersonToRedis`方法后，可以用Redis可视化工具看到数据添加到了person存储空间下，调用`selectPersonToRedis`查询结果如下：

```
Person{id='1', firstname='liu', lastname='jiao', address=Address{city='成都东', country='四川省'}}
```

至此，springboot整合Redis实现增删改查功能结束。



### Spring缓存管理

#### 没有缓存的效果

在没有使用缓存的情况下，每次查询都会打印出sql语句：

```
Hibernate: select article0_.id as id1_0_0_from t_article article0_ where article0_.id=?
Hibernate: select article0_.id as id1_0_0_from t_article article0_ where article0_.id=?
Hibernate: select article0_.id as id1_0_0_from t_article article0_ where article0_.id=?
```



#### 默认缓存管理

  Spring框架支持透明地向应用程序添加缓存对缓存进行管理，其管理缓存的核心是将缓存应用于 操作数据的方法，从而减少操作数据的执行次数，同时不会对程序本身造成任何干扰。
Spring Boot继承了Spring框架的缓存管理功能，通过使用@EnableCaching注解开启基于注解的缓存支 持，Spring Boot就可以启动缓存管理的自动化配置。 



通过以下步骤可以开启并验证springboot默认缓存管理：

- 在SpringBoot启动类上使用@EnableCaching注解开启缓存功能

```
  @EnableCaching
  @SpringBootApplication
  public class Springboot04ThymeleafApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(Springboot04ThymeleafApplication.class, args);
      }
  }
```

  

- 在查询的service层方法上加入@Cacheable注解，并指定唯一的cachanames属性，即可使用默认缓存管理功能

```
  @Cacheable(cacheNames = "article")
      @Override
      public Article findById(Integer id) {
          Optional<Article> repositoryById = articleRepository.findById(id);
          if(repositoryById.isPresent()) {
              return repositoryById.get();
          }
          return null;
      }
```

  

@EnableCaching注解：开启基于注解的缓存支持(用于启动类上)

@Cacheable 将该方法查询结果存放在springboot默认缓存中，cacheNames属性作用：起一个缓存命名空间，对应环境唯一标识。

缓存中key：默认只有一个参数的情况下，key为方法参数值，如果没有参数或者多个参数的情况下，会使用SimpleKeyGenerator类来生成key



缓存的底层结构：ConcurrentMap



缓存执行流程：方法运行之前，先去缓存组件进行查询，按照cacheNames指定的名字获取，第一次获取如果没有的话Cache组件会自动创建



去Cache中查找缓存的内容时，key为方法的参数，如果多个参数或者没有参数，则按照某种策略生成，默认使用KeyGenerator生成，使用SimpleKeyGenerator生成key。

其他注解：

- @CachePut注解：数据更新操作
- @CacheEvict注解：数据删除操作





#### SpringBoot支持的缓存组件

- Generic
- JCache
- EhCache2.x
- Hazelcast
- Infinispan
- Couchbase
- Redis
- Caffieie
- Simple（springboot默认支持的缓存组件）



#### 基于Redis的SpringBoot缓存实现

- 添加redis依赖：

```
  <dependency>
  	<groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
```

- redis连接配置

```
  # Redis服务地址
  spring.redis.host=127.0.0.1
    # Redis服务器连接端
  spring.redis.port=6379
  # Redis服务器连接密码（默认为空）
  spring.redis.password=
```

  

- 方法：

```
  @Cacheable(cacheNames = "article" ,unless = "#result==null")
      @Override
      public Article findById(Article article) {
          Optional<Article> repositoryById = articleRepository.findById(article.getId());
          if(repositoryById.isPresent()) {
              return repositoryById.get();
          }
          return null;
      }
  
      @CachePut(cacheNames = "article",key = "#result.id")
      @Override
      public Integer update(String content, Integer id) {
          return articleRepository.update(content, id);
      }
```

  

- 访问方法报错：

```
  java.lang.IllegalArgumentException: DefaultSerializer requires a Serializable payload but received an object of type [com.lagou.pojo.Article]
  
```

  是因为实体类未实现序列化接口导致。

此外：可以在SpringBoot全局配置文件中配置Redis数据的有效期，但这种方式不够灵活：

```
# 对基于注解的Redis缓存数据统一设置有效期为1分钟，单位毫秒 
spring.cache.redis.time-to-live=60000
```



缓存生效后，可以在redis客户端中查看缓存的数据。

```
\xAC\xED\x00\x05sr\x00\x16com.lagou.pojo.Article\x02\x9Au"Ug\xF5G\x02\x00\x08L\x00\x0Dallow_commentt\x00\x13Ljava/lang/Integer;L\x00\x0Acategoriest\x00\x12Ljava/lang/String;L\x00\x07contentq\x00~\x00\x02L\x00\x07createdt\x00\x10Ljava/util/Date;L\x00\x02idq\x00~\x00\x01L\x00\x08modifiedq\x00~\x00\x03L\x00\x09thumbnailq\x00~\x00\x02L\x00\x05titleq\x00~\x00\x02xpsr\x00\x11java.lang.Integer\x12\xE2\xA0\xA4\xF7\x81\x878\x02\x00\x01I\x00\x05valuexr\x00\x10java.lang.Number\x86\xAC\x95\x1D\x0B\x94\xE0\x8B\x02\x00\x00xp\x00\x00\x00\x01t\x00\x0C\xE9\xBB\x98\xE8\xAE\xA4\xE5\x88\x86\xE7\xB1\xBBt\x00\x0C\xE6\xB5\x8B\xE8\xAF\x95\xE6\x9B\xB4\xE6\x96\xB0sr\x00\x12java.sql.Timestamp&\x18\xD5\xC8\x01S\xBFe\x02\x00\x01I\x00\x05nanosxr\x00\x0Ejava.util.Datehj\x81\x01KYt\x19\x03\x00\x00xpw\x08\x00\x00\x01m\xB1?\xA0\x00x\x00\x00\x00\x00q\x00~\x00\x07ppt\x00\x1D2019\xE6\x96\xB0\xE7\x89\x88Java\xE5\xAD\xA6\xE4\xB9\xA0\xE8\xB7\xAF\xE7\xBA\xBF\xE5\x9B\xBE
```

值是通过JDK默认模式序列化后的HEX格式化存储的数据，不方便直接查看，因此我们通常会自定义序列化格式。

#### 基于API的Redis缓存实现

在Spring Boot整合Redis缓存实现中，除了基于注解形式的Redis缓存实现外，还有一种开发中常用 的方式——基于API的Redis缓存实现。这种基于API的Redis缓存实现，需要在某种业务需求下通过 Redis提供的API调用相关方法实现数据缓存管理；同时，这种方法还可以手动管理缓存的有效期



```
@Service
public class ArticleServiceImpl implements ArticleService {


    @Autowired
    private RedisTemplate redisTemplate;
    @Autowired
    private ArticleRepository articleRepository;


    @Override
    public Page<Article> list(int page, int size) {
        Sort sort = Sort.by(Sort.Direction.DESC, "created");
        PageRequest pageRequest = PageRequest.of(page, size, sort);
        Page<Article> pageList = articleRepository.findAll(pageRequest);
        return pageList;
    }

    @Override
    public Article findById(Article article) {
        Object o = redisTemplate.opsForValue().get("article_" + article.getId());
        if(o != null) {
            return (Article) o;
        }
        Optional<Article> repositoryById = articleRepository.findById(article.getId());
        if(repositoryById.isPresent()) {
            redisTemplate.opsForValue().set("article_" + article.getId(), repositoryById.get());
            return repositoryById.get();
        }
        return null;
    }

    @Override
    public Integer update(String content, Integer id) {
        int update = articleRepository.update(content, id);
        Optional<Article> byId = articleRepository.findById(id);
        redisTemplate.opsForValue().set("article_" + id, byId.get());
        return update;
    }
}

```





####  自定义Redis缓存序列化机制



```
@Configuration
public class RedisConfiguration {

  @Bean
  public RedisTemplate<Object, Object> redisTemplate(
      RedisConnectionFactory redisConnectionFactory) {
    RedisTemplate<Object, Object> template = new RedisTemplate();
    template.setConnectionFactory(redisConnectionFactory);
    // 使用JSON格式序列化对象，对缓存数据key和value进行转换
    Jackson2JsonRedisSerializer jacksonSeial = new Jackson2JsonRedisSerializer(Object.class);
    //解决查询缓存转换异常的问题        
    ObjectMapper om = new ObjectMapper();
    om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
    om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
    jacksonSeial.setObjectMapper(om);
    // 设置RedisTemplate模板API的序列化方式为JSON
    template.setDefaultSerializer(jacksonSeial);
    return template;
  }
}
```

核心原理就是自定义RedisTemplate类的DefaultSerializer序列化方式为自定义的json序列化对象。

测试结果：

```
[
  "com.lagou.pojo.Article",
  {
    "id": 1,
    "title": "2019新版Java学习路线图",
    "content": "测试更新",
    "created": [
      "java.sql.Timestamp",
      1570636800000
    ],
    "modified": null,
    "categories": "默认分类",
    "allow_comment": 1,
    "thumbnail": null
  }
]
```



