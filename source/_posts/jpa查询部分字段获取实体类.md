---
title: jpa查询部分字段获取实体类
date: 2019-07-01 17:07:00
categories: 
tags: [java]
copyright: true #新增,开启
---

代码已经放到github,test测试中的demo2对应的是sql方式,demo3对应的是hql方式,demo1是分页查询,我另一篇文章会讲到{% post_link 分页查询 分页查询 %}
[github地址](https://github.com/qq616023844/springboot-jpa-paging)

#### 前言
我们平时使用jpa查询时,有两种情况,一种是查询全部字段,另一种是查询部分字段,当我们按通常的sql语句写法查询部分字段时,会出现jpa无法自动解析类型的情况,例如这类报错
```
org.springframework.dao.InvalidDataAccessResourceUsageException: could not execute query; SQL [ SELECT sa.name FROM student sa ]; nested exception is org.hibernate.exception.SQLGrammarException: could not execute query
```

#### 解决方案
针对hql和sql分别有两种解决方案

<font size=5>一.</font>

&emsp;hql情况下,我们可以用这种方式来解决,有必要注意的一点是,<font color=red>Student里面一定要有相应的构造类</font>
```java
//TODO 查询部分字段的demo-hql
@Query(value = " SELECT new Student(s.name) FROM Student s")
List<Student> temp03();
```
<font size=5>二.</font>

在sql情况下,我们可以用这种方式解决,首先我们将查出来的数据领jpa解析为map,然后通过我们自己写的map转实体类方法来解决
```java
//TODO 查询部分字段的demo-sql
@Query(value = " SELECT sa.name FROM student sa ",
    nativeQuery = true)
List<Map<String,Object>> temp02();
```
下面是我自己写的一个map转实体类的工具方法
```java
/**将map转换为实体类,在jpa查询部分字段时会用到
* 使用的时候注意,因为int类型会初始化的问题,无法被FASTJSON忽略掉,所以返回的json可能会带有额外的数字0
* 由于是通过属性名来匹配,所以如果数据库字段名和参数名不一致,会导致部分字段映射不到实体,应该这么写
* @Query(value = " select id,bar_code01 barCode01,bar_code02 barCode02,bar_code03 barCode03,name,comment from library_good ",nativeQuery=true)
* 在查询时取别名,将其跟类的属性名一致 */
public static <T>T mapToEntity(Map<String,Object> map,Class<T> targetClass) throws IllegalAccessException, InstantiationException {
    Class superClass;
    Field[] fields;
    
    T target = targetClass.newInstance();
    //接收targetClass的Field
    List<Field> targetfieldList = new LinkedList<>();

    superClass = targetClass;
    while(superClass!=null&&superClass!=Object.class){
        //由于该方法只能获取superClass的参数(private,protect,public等任何声明),但无法获取父类的参数,这里我们迭代一波
        fields = superClass.getDeclaredFields();
        targetfieldList.addAll(Arrays.asList(fields));
        superClass = superClass.getSuperclass();
    }
    //匹配并赋值
    for (Field targetfield : targetfieldList) {
        for (Map.Entry<String, Object> mapEntry : map.entrySet()) {
            if (targetfield.getName().equals(mapEntry.getKey())){
                //暂时保存权限
                boolean targetFlag = targetfield.isAccessible();
                //赋予权限
                targetfield.setAccessible(true);
                //赋值
                targetfield.set(target,mapEntry.getValue());
                //恢复原权限
                targetfield.setAccessible(targetFlag);
                break;
            }
        }
    }
    return target;
}
```
<font color=red>有一点需要注意,由于其底层用了反射,所以无论是通过该种方式取数据还是存数据,均需要setAccessible(true),否则会出现IllegalAccessException异常</font>






