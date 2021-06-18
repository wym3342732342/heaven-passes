#                         Quarkus+Hibernate

## 1,添加pom文件依赖

```xml-dtd
<!--集成hibernate-->
<dependency>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-hibernate-orm-panache</artifactId>
 </dependency>
<!--集成Mysql-Jdbc-->
 <dependency>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-jdbc-mysql</artifactId>
  </dependency>
<!--YML-->
  <dependency>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-config-yaml</artifactId>
  </dependency>
```

## 2,添加测试依赖方便测试使用

```xml-dtd
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

## 3,添加连接数据库配置文件

```YML
quarkus:
  datasource:
    db-kind: mysql
    jdbc:
      url: jdbc:mysql://8.131.230.37:3306/mallserverTimezone=GMT%2B8&allowMultiQueries=true
      driver: com.mysql.cj.jdbc.Driver
    username: mall
    password: mall
  # 实体属性名称自动映射到数据库蛇形字段名
  hibernate-orm:
    physical-naming-strategy: org.acme.common.CustomPhysicalNamingStrategy
```

## 4,编写测试接口

### (1)测试service层

```java
@ApplicationScoped
public class CmfGoodsAreaService {


    @Inject
    CmfGoodsAreaRepository cmfGoodsAreaRepository;


    /*
     * 功能说明: 查询所有
     */
    public List<CmfGoodsArea> getAll() {
        return cmfGoodsAreaRepository.listAll();
    }


    /*
     * 功能说明: 根据id删除数据
     */
    @Transactional
    public Long deleteById(String id) {
        return cmfGoodsAreaRepository.delete("id", id);
    }


    /*
     * 功能说明:  根据id查询专区信息
     */
    public CmfGoodsArea findArea(CmfGoodsArea cmfGoodsArea) {
        return cmfGoodsAreaRepository.find("id", cmfGoodsArea.getId()).firstResult();
    }


    /*
     * 功能说明: 此方法为新增
     */
    @Transactional
    public Integer saveArea(CmfGoodsArea cmfGoodsArea) {
        cmfGoodsArea.setId(UUID.randomUUID().toString());
        cmfGoodsArea.setCreateTime(new Date());
        cmfGoodsArea.setUpdateTime(new Date());
        try {
            cmfGoodsAreaRepository.persist(cmfGoodsArea);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
        return 1;
    }


    /*
     * 功能说明: 更新专区信息
     */
    @Transactional
    public Integer updateArea(CmfGoodsArea cmfGoodsArea) {
        String hql = "areaTitle='" + cmfGoodsArea.getAreaTitle() + "'," +
                "areaImage='" + cmfGoodsArea.getAreaImage() + "'," +
                "teamMembersMinCount = '" + cmfGoodsArea.getTeamMembersMinCount() + "'," +
                "sort = '" + cmfGoodsArea.getSort() + "' where id =?1";
        return cmfGoodsAreaRepository.update(hql, cmfGoodsArea.getId());
    }
}
```

### (2)测试DAO层

```java
@ApplicationScoped
public class CmfGoodsAreaRepository implements PanacheRepository<CmfGoodsArea> {

}
```

### (3)参考网站

[Quarkus - Creating Your First Application](https://quarkus.io/guides/getting-started)

