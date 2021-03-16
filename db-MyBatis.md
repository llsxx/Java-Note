## 动态SQL

- if
- choose (when, otherwise)
- trim (where, set)
- foreach

### if

```xml
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```

### choose、when、otherwise

```xml
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

### trim

```xml
  <insert id="insertSelective" parameterType="org.javaboy.vhr.model.HrRole" >
    insert into hr_role
    <trim prefix="(" suffix=")" suffixOverrides="," >
      <if test="id != null" >
        id,
      </if>
      <if test="hrid != null" >
        hrid,
      </if>
      <if test="rid != null" >
        rid,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides="," >
      <if test="id != null" >
        #{id,jdbcType=INTEGER},
      </if>
      <if test="hrid != null" >
        #{hrid,jdbcType=INTEGER},
      </if>
      <if test="rid != null" >
        #{rid,jdbcType=INTEGER},
      </if>
    </trim>
  </insert>
```

#### where

> 1、本质上是trim语句定义出来的
>
> ```xml
> <trim prefix="WHERE" prefixOverrides="AND |OR ">
>   ...
> </trim>
> ```
>
> 2、当select后面都是判断条件时，必须使用；当有确定的条件时，可不用

```xml
<select id="findActiveBlogLike"  resultType="Blog">
  SELECT * FROM BLOG
  <where>
    <if test="state != null">
         state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```

#### set

```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```

> 1、本质上是trim语句定义出来的
>
> ```xml
> <trim prefix="SET" suffixOverrides=",">
>   ...
> </trim>
> ```

### foreach

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

```xml
  <insert id="addRole">
    insert into hr_role (hrid,rid) values
    <foreach collection="rids" item="rid" separator=",">
      (#{hrid},#{rid})
    </foreach>
  </insert>

 Integer addRole(@Param("hrid") Integer hrid, @Param("rids") Integer[] rids);
```



> 1.集合参数collectioin可以是：任何可迭代对象（如 List、Set 等）、Map 对象或者数组对象；
>
> 2.当使用可迭代对象或者数组时，index 是当前迭代的序号，item 的值是本次迭代获取到的元素。当使用 Map 对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值。

## 其他标签

### sql

定义：将要查询的字段定义出来

```xml
   <sql id="Base_Column_List">
   		 id, name, phone, telephone, address, enabled, username, password, userface, remark
   </sql>
```

使用

```xml
    <select id="selectByPrimaryKey" resultMap="BaseResultMap" parameterType="java.lang.Integer">
        select
        <include refid="Base_Column_List"/>   
        from hr
        where id = #{id,jdbcType=INTEGER}
    </select>
```

## 常用API操作

一对多查询

一对一查询

批量删除

## SpringBoot中的配置

方式一：在配置文件中配置

```properties
mybatis:
  mapper-locations: classpath:mappers/*.xml     
  config-location：classpath:mybatis-config.xml  #全局配置文件
  configuration
        map-underscore-to-camel-case：true      #开启驼峰转换
        
#打印sql语句日志设置
logging:
	level:
		com.mengxuegu.blog.article.mapper: debug   //mapper所在路径  

```

方式二:在pom文件中配置resources

```xml
    <build>
       <resources>
            <resource>
                <!--编译时，默认情况下不会将    mapper.xml文件编译进去，
                src/main/java 资源文件的路径，
                **/*.xml 需要编译打包的文件类型是xml文件，
                -->
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
    </build>
```

## 配置文件

mapper模板

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="">
</mapper>

```

