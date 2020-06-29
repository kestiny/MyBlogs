# MyBatis动态SQL

MyBatis的动态SQL比较多，但是常用的也就几个，包括if,where,set,choose.

## if
if语句是条件判断，例如判断type不为空，则增加type语句
```xml
<if test="type != null">type=#{type}</if>
```

## where
在查询语句中，我们经常需要根据给定的不同数据类型，查询不同的语句，动态where此时正当用。
```xml
<select id="select" resultType="person">
    select * from person
    <where>
        <if test="type != null">type=#{type},</if>
        <if test="name != null">and name like #{name},</if>
    </where>
</select>
```
- 如果where标签结果不为空，则自动在sql语句中添加where
- 如果标签内返回以and或者or开头的话，and或者or会自动被剔除
- 标签内的最后一个,号会自动被剔除

## set
```xml
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
set标签的用法同where一致

## choose
choose标签比较像Java语句中是switch选择语句
```xml
<select id="select" resultType="person">
    select * from person
     <where>
         <choose>
             <when test="name != null">name=#{name}</when>
             <when test="type != null">type=#{type}</when>
             <otherwise>id={id}</otherwise>
         </choose>
     </where>
</select>
```
- choose相当于switch语句；
- when相当于case；
- otherwise相当于default；
- 三者需要配合使用，完全一个完整的选择语句

## 总结
动态SQL其实很简单，就是基本的语句拼接，也可以在代码中直接自己写逻辑完成。MyBatis的动态SQL只是帮我们简化了这个流程而已。这种东西，唯手熟尔！