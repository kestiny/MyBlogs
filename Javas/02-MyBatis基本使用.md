# MyBatis基本使用

MyBatis是一个半自动化的ORM（对象关系映射）框架，她的核心就在于Sql语句和对象的关系映射上面。总的来说，MyBatis的对象关系映射主要有两种方式：xml配置文件和注解方式。

## xml配置文件方式

### 1.namespace命名空间
MyBatis的映射文件内容都包含在一个namespace中，这个namespace中的名称**必须和Mapper接口名称一致**,并且还包括完整的包名。
比如：
```xml
package com.kestiny.mybatis.mapper;
public interface PersonMapper {}

<mapper namespace="com.kestiny.mybatis.mapper.PersonMapper"></mapper>>
```

### 2.Sql语句对应标签
MyBatis对象映射文件的Sql语句对应标签和sql语句完全一致
```xml
<select/>
<delete/>
<insert/>
<update/>
```

以select为例，我们看下标签是怎么和Sql语句进行映射的。
假设我们有一个实体对象Person及其Mapper接口
```java
public class Person {
    private Integer id;
    private String name;
    private Integer sex;
    private String pswd;
    private String mobile;
    private String nickname;
    private Timestamp lastLoginTime;
}

public interface PersonMapper {
    public Person selectById(@Param("id") Integer id);
}
```
我们需要编写对应的映射select配置如下
```xml
<select id="selectById" parameterType="int" resultType="Person">
    select * from person where id = #{id}
</select>
```
其中，
- id：对应的是Mapper中方法的名称，一定要保持一致；
- parameterType：对应的是方法中传入的参数类型；
- resultType：对应的方法返回的对象类型；
- @Param("id")：方法名称中的@Param注解，表示的是此参数传到xml后的名称，可以省略，省略的话名称等同于方法中形参的名称；
- @Param: 多个参数时一定要使用此注解，否则只能按照顺利获取参数，非常麻烦；
- 在标签内写入我们正常的Sql语句即可

其他的几种标签使用方式和select完全一致
```xml
<delete id="delete" parameterType="int">
        delete from person where nid = #{id}
</delete>
<insert id="save" parameterType="Person">
    insert into person
    (nid,name,pswd,mobile,nickname,LastLoginTime)
    values
    (#{id},#{name},#{pswd},#{mobile},#{nickname},#{lastLoginTime})
</insert>
<update id="update" parameterType="Person">
    update person
    <set>
        <if test="name != null">name=#{name},</if>
        <if test="pswd != null">pswd=#{pswd},</if>
        <if test="mobile != null">mobile=#{mobile},</if>
        <if test="nickname != null">nickname=#{nickname},</if>
        <if test="lastLoginTime != null">LastLoginTime=#{lastLoginTime},</if>
    </set>
    where nid=#{id}
</update>
```

### 3.数据库字段名称和实体类属性名称不一致解决
#### 1.数据库字段和实体属性使用驼峰方式
这种方式下，比如数据库字段使用的经典下划线方式**last_login_time**,实体属性使用驼峰方式**lastLoginTime**，针对这种情况，需要在MyBatis核心配置文件中配置开启自动驼峰命名转换即可。
```xml
<settings>
    <!--MyBatis默认是关闭自动驼峰命名转换的-->
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

#### 2.命名完全不一致
当数据库字段名和Java属性名称完全不一致时，就需要使用**resultMap**进行数据库字段和Java属性名称的映射，具体实现如下：
```xml
<resultMap id="BaseResultMap" type="Person">
    <id column="nid" jdbcType="INTEGER" property="id" javaType="int"/>
</resultMap>
<select id="selectAll" resultMap="BaseResultMap">
    select * from person
</select>
```
- 标签名称叫resultMap，它一般需要设置两个属性
   + id：resultMap的名称
   + type:resultMap对应的Java实体类，注意这里的名称规则，如果没有使用应该是包的全名，若想省略包名，则需要配置MyBatis的核心xml文件
       ```xml
        <typeAliases>
            <package name="com.kestiny.mybatis.entities"/>
        </typeAliases>
        ``` 
- resultMap包含多个<id>标签，每个id标签都是一个数据库字段和Java实体属性的对应关系，其中
    + column：数据库字段名
    + property：Java实体类属性名称
    + jdbcType：MyBatis的据类型，和javaType有对应关系
    + javaType：Java类型，和jdbcType有对应关系
    
## 注解方式
注解的实现方式，是把sql语句写在对应的Mapper方法上面，需要写Mapper.xml文件。按说大多数Java库的注解方式都比配置方式简单并且功能一致甚至更强加，但是MyBatis是那个一个例外，注解的方式只对简单sql有用，比较复杂的语句注解应对起来简直太难了！

注解的使用十分简单
```java
@Select("select * from person")
public List<Person> select();

@Delete("delete from person where id=#{id}")
public int delete(String id);

@Update("update person set name=#{name},age=#{age} where id=#{id}")
public int update(Person person);

@Insert("insert into person(id, name,age) values (#{id},#{name},#{age})")
public int add(Person person);
```