---
layout: post
title:  "Mybatis使用Example动态生成sql"
date:   2019-04-05 22:35:53 +0400
categories: Pratice
tags: SpringBoot
author: Eric
description: mybatis使用Example动态生成sql
---  


### 一、mapper接口中的函数及方法    

---- 


| 方法                                                                             | 功能说明                                                               |
| ---------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| int countByExample(UserExample example) thorws SQLException                        | 按条件计数                                                            |
| int deleteByPrimaryKey(Integer id) thorws SQLException                             | 按主键删除                                                            |
| int deleteByExample(UserExample example) thorws SQLException                       | 按条件查询                                                            |
| String/Integer insert(User record) thorws SQLException                             | 插入数据（返回值为ID）                                           |
| User selectByPrimaryKey(Integer id) thorws SQLException                            | 按主键查询                                                            |
| ListselectByExample(UserExample example) thorws SQLException                       | 按条件查询                                                            |
| ListselectByExampleWithBLOGs(UserExample example) thorws SQLException              | 按条件查询（包括BLOB字段）。只有当数据表中的字段类型有为二进制的才会产生。 |
| int updateByPrimaryKey(User record) thorws SQLException                            | 按主键更新                                                            |
| int updateByPrimaryKeySelective(User record) thorws SQLException                   | 按主键更新值不为null的字段                                      |
| int updateByExample(User record, UserExample example) thorws SQLException          | 按条件更新                                                            |
| int updateByExampleSelective(User record, UserExample example) thorws SQLException | 按条件更新值不为null的字段                                      |

---- 


### 二、example实例解析   

>mybatis的逆向工程中会生成实例及实例对应的example，example用于添加条件，相当where后面的部分 
xxxExample example = new xxxExample(); 
Criteria criteria = new Example().createCriteria();      

| 方法                                     | 说明                                        |
| ------------------------------------------ | --------------------------------------------- |
| example.setOrderByClause(“字段名 ASC”); | 添加升序排列条件，DESC为降序      |
| example.setDistinct(false)                 | 去除重复，boolean型，true为选择不重复的记录。 |
| criteria.andXxxIsNull                      | 添加字段xxx为null的条件               |
| criteria.andXxxIsNotNull                   | 添加字段xxx不为null的条件            |
| criteria.andXxxEqualTo(value)              | 添加xxx字段等于value条件              |
| criteria.andXxxNotEqualTo(value)           | 添加xxx字段不等于value条件           |
| criteria.andXxxGreaterThan(value)          | 添加xxx字段大于value条件              |
| criteria.andXxxGreaterThanOrEqualTo(value) | 添加xxx字段大于等于value条件        |
| criteria.andXxxLessThan(value)             | 添加xxx字段小于value条件              |
| criteria.andXxxLessThanOrEqualTo(value)    | 添加xxx字段小于等于value条件        |
| criteria.andXxxIn(List<？>)               | 添加xxx字段值在List<？>条件          |
| criteria.andXxxNotIn(List<？>)            | 添加xxx字段值不在List<？>条件       |
| criteria.andXxxLike(“%”+value+”%”) | 添加xxx字段值为value的模糊查询条件 |
| criteria.andXxxNotLike(“%”+value+”%”) | 添加xxx字段值不为value的模糊查询条件 |
| criteria.andXxxBetween(value1,value2)      | 添加xxx字段值在value1和value2之间条件 |
| criteria.andXxxNotBetween(value1,value2)   | 添加xxx字段值不在value1和value2之间条件 |

Example:   

{% highlight ruby %}    
<!-- 排序 -->
Example example = new Example(Country.class);
example.orderBy("id").desc().orderBy("countryname").orderBy("countrycode").asc();
List<Country> countries = mapper.selectByExample(example);

<!-- 去重 -->
CountryExample example = new CountryExample();
//设置 distinct
example.setDistinct(true);
example.createCriteria().andCountrynameLike("A%");
example.or().andIdGreaterThan(100);
List<Country> countries = mapper.selectByExample(example);

<!-- 设置查询列 -->
Example example = new Example(Country.class);
example.selectProperties("id", "countryname");
List<Country> countries = mapper.selectByExample(example);   

<!-- Example.builder 方式 -->
Example example = Example.builder(Country.class)
        .select("countryname")
        .where(Sqls.custom().andGreaterThan("id", 100))
        .orderByAsc("countrycode")
        .forUpdate()
        .build();
List<Country> countries = mapper.selectByExample(example);  

<!-- and or 查询 -->
Example example = new Example(User.getClass());
        // where 条件
        Criteria criteria = example.createCriteria();
        Criteria criteria1 = example.createCriteria();
        
        criteria.andIn("id", ids);
        criteria1.orLike("des", "%" + des + "%");
        criteria1.orLike("name", "%" + name + "%");
        example.and(criteria1);
        example.and().andEqualTo("status", staus)
 等效于：where id in ( #{ids} ) and ( name like concat(‘%’, #{name} ,’%’) or des like concat(‘%’, #{des} ,’%’) ) and status = #{status} 

{% endhighlight %}    

------   

    

###### [https://www.cnblogs.com/kelthuzadx/p/10924117.html][blog-url]
[blog-url]: https://blog.csdn.net/lyf_ldh/article/details/80976552
