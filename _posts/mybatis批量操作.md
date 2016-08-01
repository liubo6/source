---
title: mybatis批量操作
date: 2016-08-01 12:12:11
categories: 数据库
tags: mysql
---
如果一个接口里面有大量的数据新增和更新操作，会因为频繁的连接数据库,操作等缓慢。使用了批量操作之后速度有明显提升

```
jdbc:MySQL://127.0.0.1:3306/test?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true
```
<!--more-->

## 批量插入
```
<insert id="insertbatch" parameterType="java.util.List">
INSERT INTO user (
id, name
)values
<foreach collection="list" item="item" index="index" separator=",">
(
#{item.id},#{item.name}
)
</foreach>
</insert>
```
另一种
```
<insert id="insertBatch" parameterType="java.util.List">
insert into user(id,name)
<foreach collection="list" item="item" index="index" separator="union all">
select #{item.id},
#{item.name}
</foreach>
</insert>
```

## 批量更新
### 根据IDs更新
```
<update id="batchUpdate" parameterType="java.util.List">
update user set state = '0' where id in
<foreach collection="list" item="item" open="(" separator="," close=")">
  #{item}
</foreach>
</update>
```
### 根据条件更新
```
<update id="batchUpdate" parameterType="java.util.List">
<foreach collection="list" item="item" index="index" separator=";">
update user 
<set>
    <if test="item.name != null">
        name=#{item.name}
    <if>
    <if test="item.age!= null">
        age=#{item.age}
    <if>
<set>
where id=#{item.id}
</foreach>
</update>
```

## 批量删除
```
<delete id="batchRemove" parameterType="java.util.List">
DELETE FROM user WHERE id in 
<foreach item="item" index="index" collection="list" open="(" separator="," close=")">
#{item}
</foreach>
</delete>
```
