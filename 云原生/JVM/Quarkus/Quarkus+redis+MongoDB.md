# Quarkus

# 一.什么是quarkus?

​    Quarkus是为GraalVM和HotSpot量身定制的Kubernetes Native Java框架，由最佳的Java库和标准精心打造而成。Quarkus的目标是使Java成为Kubernetes和无服务器环境中的领先平台，同时为开发人员提供统一的反应式和命令式编程模型，以满足更广泛的分布式应用程序架构.

# 二.quarkus的几个特性?

​	1.容器优先:最小的JAVA应用程序,最适合在容器中运行

​	2.云原生:在 Kubernetes 等环境中采用 12 要素原则

​	3.统一命令式与反应式：在一个编程模型下带来非阻塞和命令式开发风格

​	4.微服务优先：快速启动项目编写 Java 应用

# 三.为什么使用quarkus?

​	1.现在大多数使用的框架都是Spring系列,现在元宝正在使用的是Spring家族中的SpringBoot,Spring启动优化是个大难题,特别是启动是的Bean扫描，当应用达到一定规模后，启动非常慢,quarkus打包成Native应用后启动速度对比SpringBoot的JAR包部署方式,启动速度有明显的提升,这是quarkus的一大优势.

​	2.在构建时将进行尽可能多的处理，因此您的应用程序将仅包含运行时实际需要的类。在传统模型中，执行初始应用程序部署所需的所有类都在应用程序的生命周期内徘徊，即使它们仅使用一次。使用Quarkus，它们甚至都不会加载到生产JVM中。由于所有元数据处理已完成，因此这将减少内存使用量，并缩短启动时间.

​	3.Quarkus尽可能避免反射，减少启动时间和内存使用量

​	4.生来就支持GraalVM(很强的一个JVM，支持多语言，native构建应用极小

# 四.quarkus和SpringBoot区别

 (参考:https://blog.csdn.net/z69183787/article/details/108180175?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162381240616780255287885%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162381240616780255287885&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-108180175.first_rank_v2_pc_rank_v29&utm_term=quarkus%E5%92%8Cspring+boot&spm=1018.2226.3001.4187)

## 1.对比应用概况

| 项目              | Spring Boot | Quarkus |
| :---------------- | :---------- | :------ |
| API               | 10个        | 10个    |
| Service           | 10个        | 10个    |
| Service Implement | 10个        | 10个    |
| 打包方式          | Jar         | Native  |

## 2.对比数据

|        | Spring Boot | Quarkus |
| :----- | :---------- | :------ |
| 第一次 | 3.664s      | 0.015s  |
| 第二次 | 3.655s      | 0.007s  |
| 第三次 | 3.338s      | 0.009s  |
| 平均   | 3.552s      | 0.010s  |

## 3.总结

​	Quarkus打包成Native应用后启动速度对比传统Spring Boot的Jar部署方式，启动速度有很明显的提升。个人感觉这个是Quarkus的最大优势，快速部署和启动对于高用户量的应用还是很有帮助的

# 五.quarkus和SpringBoot在使用中的例子

## 1.相关注解

​	1.@Path 相当于springboot 中RequestMapping

​	2.@Produces 返回视图类型，通常我们用MediaType.APPLICATION_JSON

​	3.@Get、@Post、@DELETE、@PUT 对应@GetMapping、@PostMapping、@DeleteMapping

​	4.@PutMapping @ApplicationScoped注解相当于springboot中@component

​	5.@Inject 相当于@Autowired

​	6.@PathParam 相当于@PathVariable

## 2.代码示例

```
@Path("/test/hello")  //作用相当于@RequestMapping
public class TestQuarkusController {

    @Inject
    TestQuarkusService testQuarkusService;


    @GET
    @Produces(MediaType.TEXT_PLAIN)
    @Path("/get")
    public String hello() {
        return "hello";
    }

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    @Path("/find")
    public String helloInject() {

        return testQuarkusService.getName(1);
    }
}
```

# 六.idea整合quarkus 

(参考:https://quarkus.io/ 官方文档)

## 1.添加pom文件 

```java
<dependencies>
    <!-- 实现Rest web服务依赖 -->
    <dependency>
        <groupId>io.quarkus</groupId>
        <artifactId>quarkus-resteasy</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.quarkus</groupId>
            <artifactId>quarkus-bom</artifactId>
            <version>${quarkus.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
    
 <!-- 构建Native Image -->
    <profiles>
        <profile>
            <id>native</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>io.quarkus</groupId>
                        <artifactId>quarkus-maven-plugin</artifactId>
                        <version>${quarkus.version}</version>
                        <executions>
                            <execution>
                                <goals>
                                    <goal>native-image</goal>
                                </goals>
                                <configuration>
                                    <enableHttpUrlHandler>true</enableHttpUrlHandler>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
    
<build>
    <plugins>
        <plugin>
            <groupId>io.quarkus</groupId>
            <artifactId>quarkus-maven-plugin</artifactId>
            <version>${quarkus.version}</version>
            <executions>
                <execution>
                    <goals>
                        <goal>build</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

# 七.启动quarkus

## 1.启动方法一:

​	运行 mvn文件中的quarkus:dev

![image-20210615092707801](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20210618150456.png)

## 2.启动方法二:

​	添加启动类(好处是可以debug运行quarkus,参考:https://my.oschina.net/giegie/blog/4917334)

```java
import io.quarkus.runtime.Quarkus;
import io.quarkus.runtime.QuarkusApplication;
import io.quarkus.runtime.annotations.QuarkusMain;


@QuarkusMain
public class SimpleApplication implements QuarkusApplication {

    public static void main(String[] args) {
        Quarkus.run(SimpleApplication.class,args);
    }

    public int run(String... args) throws Exception {
        Quarkus.waitForExit();
        return 0;
    }
}
```

# 八.配置quarkus配置文件

## 1.创建application.properties

```
quarkus.banner.enabled = true
quarkus.http.port = 8080
```

# 九.quarkus集成redis

## 1.添加pom文件依赖

```java
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-redis-client</artifactId>
</dependency>
```

## 2.添加测试依赖方便测试使用

```java
<!-- Junit -->
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-junit5</artifactId>
    <scope>test</scope>
</dependency>
<!-- Rest接口测试 -->
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <scope>test</scope>
</dependency>
```

## 3.添加redis连接配置文件(本地)

```
#redis
quarkus.redis.hosts = 127.0.0.1:6379
quarkus.redis.database = 0
quarkus.redis.timeout = 6000s
quarkus.redis.password = 123456
```

## 4.编写测试接口 

### (1)controller层:

```java
@Path("/redis")
public class RedisQuarkusController {

    @Inject
    TestRedisService service;

    /**
     * 保存redis
     * @author  wang
     */
    @GET
    @Path("/create/{key}/{value}")
    public BaseResponse create(@PathParam("key") String key , @PathParam("value") Integer value) {
        service.set(key, value);
        TestRedis testRedis = new TestRedis();
        testRedis.setKey(key);
        testRedis.setValue(value);
        return BaseResponse.getSuccessResponse(testRedis);
    }

    /**
     * 保存redis的value
     * @author  wang
     */
    @GET
    @Path("/{key}")
    public TestRedis get(@PathParam("key") String key) {
        TestRedis testRedis = new TestRedis(key, Integer.valueOf(service.get(key)));
        return testRedis;
    }
}
```

### (2)测试service层

```java
@Singleton // 作用相当于Spring中的@Component
public class TestRedisService {

    @Inject // 作用相当于Spring中的@Autowired
    RedisClient redisClient;

    @Inject
    ReactiveRedisClient reactiveRedisClient;

    public String get(String key) {
        return redisClient.get(key).toString();
    }

    public void set(String key, Integer value) {
        redisClient.set(Arrays.asList(key, value.toString()));
    }

    public void increment(String key, Integer incrementBy) {
        redisClient.incrby(key, incrementBy.toString());
    }
}
```

# 十.quarkus集成mongoDB

## 1.添加pom文件依赖

```
<!-- mongoDB驱动依赖 -->
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-mongodb-client</artifactId>
</dependency>
```

## 2.添加mongoDB连接配置文件

```java
#mongo
quarkus.mongodb.users.connection-string = mongodb://127.0.0.1:27017/test
```

## 3.编写测试接口

```java
@Path("/mongo")
public class TestMongoController {


    @Inject
    TestMMongoService fruitService;

    /**
     * 查询mongo数据
     * @author  wang
     */
    @GET
    @Path("/list")
    public List<Fruit> list() {
        List<Fruit> list = fruitService.list();
        return list;
    }

    /**
     * 保存mongo数据
     * @author  wang
     */
    @GET
    @Path("/add/{name}/{description}")
    public List<Fruit> add(@PathParam("name") String name ,@PathParam("description") String description ) {
        Fruit fruit = new Fruit();
        fruit.setDescription(description);
        fruit.setName(name);
        fruitService.add(fruit);
        return list();
    }
}
```

## 4.编写测试service

```java
@ApplicationScoped //作用相当于 @Service
public class TestMMongoService {

    @Inject MongoClient mongoClient;

    public List<Fruit> list(){
        List<Fruit> list = new ArrayList<Fruit>();
        MongoCursor<Document> cursor = getCollection().find().iterator();

        try {
            while (cursor.hasNext()) {
                Document document = cursor.next();
                Fruit fruit = new Fruit();
                fruit.setName(document.getString("name"));
                fruit.setDescription(document.getString("description"));
                list.add(fruit);
            }
        } finally {
            cursor.close();
        }
        return list;
    }

    public void add(Fruit fruit){
        Document document = new Document()
                .append("name", fruit.getName())
                .append("description", fruit.getDescription());
        getCollection().insertOne(document);
    }
 }
```

