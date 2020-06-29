# MyBatis环境配置

    MyBatis是一款优秀的持久层框架，她是半自动的化的ORM(Object Relationship Mapping)框架。

## 1.准备工具
- jdk 13.0.2
- maven 3.6.3
- MyBatis 3.5.4
- junit 4.13
- logback 1.2.3
- MySQL-connector-java 8.0.20
- slf4j 1.7.25

## 2.创建一个空的maven项目
- 可以使用IDE创建，比如IDEA或者Eclipse之类的；
- 也可以使用命令行创建
``` 
mvn archetype:generate -DgroupId=com.kestiny -DartifactId=mybatis -Dversion=0.1.0
```

## 3.导入MyBatis依赖包
MyBatis相关依赖包主要是MySQL驱动，都可在maven仓库中查找到；
```xml
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.4</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.20</version>
    <scope>runtime</scope>
</dependency>
```

## 4.编写MyBatis核心配置文件
MyBatis的配置和使用都可参见[MyBatis官方文档](https://mybatis.org/mybatis-3/zh/index.html)
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="application.porperties"/>

    <typeAliases>
        <package name="com.kestiny.mybatis.entities"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="mapper/PersonMapper.xml"/>
        <mapper resource="mapper/SchoolMapper.xml"/>
    </mappers>
</configuration>
```
注意：
1. MyBatis的xml配置文件标签是有顺序的，顺序和官方定位的不符时，是无法正确的使用的，当前版本时的顺序为：properties=>settings=>typeAliases=>typeHandlers=>objectFactory=>objectWrapperFactory=>reflectorFactory=>plugins=>environments=>databaseIdProvider=>mappers
2. 变量的引用需要使用**${}**,变量的配置文件需要在properties resource标签中指定，当然也可以使用代码的方式读取；
3. environment中可以配置多个环境，但是可以生效的只有一个，需要在default中指定；
4. 数据源类型默认为POOLED，目前一共有三种内置数据源类型：UNPOOLED、POOLED和JNDI；

## 5.准备代码和PersonMapper.xml文件
准备代码,参见mybatis模块

编写PersonMapper.xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kestiny.mybatis.mapper.PersonMapper">
    <resultMap id="BaseResultMap" type="Person">
        <id column="nid" jdbcType="INTEGER" property="id"/>
    </resultMap>
    <select id="selectAll" resultMap="BaseResultMap">
        select * from person
    </select>
</mapper>
```

## 6.执行测试
```java
@Test
public void testPerson() {
    try {
        String resource = "mybatis-config.xml";
        InputStream inputStream = inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory build = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = build.openSession();
        PersonMapper mapper = sqlSession.getMapper(PersonMapper.class);
        List<Person> personList = mapper.selectAll();
        logger.info("查找数据：" + personList.size());
        for (Person person : personList) {
            logger.info(person.toString());
        }

        sqlSession.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
注意：
1. 需要从前面的配置文件中，获取MyBatis的连接；
2. 从已有连接中获取一次SqlSession，每次使用完成后，需要把SqlSession释放；
3. 使用前，需要先从session中获取mapper。

## 7. 获取SqlSession的优化
每次都从头创建SqlSessionFactory，在获取SqlSession，十分不便，并且会导致SqlSessionFactory多次创建，一次启动SqlSessionFactory只需要一个就可以了。

编写一个MyBatis工具类
```java
public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            String resource = "mybatis-config.xml";
            InputStream inputStream = inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();
    }
}
```

测试代码简化为
```java
@Test
public void testPerson() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    PersonMapper mapper = sqlSession.getMapper(PersonMapper.class);
    List<Person> personList = mapper.selectAll();
    logger.info("查找数据：" + personList.size());
    for (Person person : personList) {
        logger.info(person.toString());
    }

    sqlSession.close();
}
```