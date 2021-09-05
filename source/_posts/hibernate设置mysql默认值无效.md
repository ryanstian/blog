---
title: hibernate设置mysql默认值无效
date: 2019-09-14 00:34:00
categories: 
tags: [hibernate]
copyright: true #新增,开启
---

场景:数据库mysql,框架hibernate
<!--more-->
原因:

&emsp;根据hibernate打印出的sql信息可以发现,如果实体类字段为null,则仍会insert这个字段为null,而mysql设置的默认值生效的前提是,当我们insert一条记录时,我们不指定某字段的值,他才会自动生成默认值,而我们用save的时候指定该字段的值为null,此时如果我们mysql设置的为not null,那么同时也会报错

解决:
```
@DynamicInsert(默认为true)
@DynamicUpdate(默认为true)
```
这两个注解是类注解,作用为:当插入/更新一条记录时,只insert/update改变的信息,而不是将所有字段都update为当前的
即,没有注解时update的sql为
```
Hibernate: select student0_.id as id1_0_0_, student0_.default_value as default_2_0_0_, student0_.password as password3_0_0_, student0_.username as username4_0_0_ from user_student student0_ where student0_.id=?
Hibernate: update user_student set default_value=?, password=?, username=? where id=?
```
有注解时的sql为
```
Hibernate: select student0_.id as id1_0_0_, student0_.default_value as default_2_0_0_, student0_.password as password3_0_0_, student0_.username as username4_0_0_ from user_student student0_ where student0_.id=?
Hibernate: update user_student set username=? where id=?
```
&emsp;可以看到,当需要update时(如果没有做更改则无论有无注解均不会执行update命令),无注解的将所有字段全都update为当前值,有注解的则只update修改值。在每次update之前,均会执行select查询出数据库当前该字段的状态进行比较后在执行更新操作

&emsp;网上有人说@DynamicInsert这个注解的作用是,如果值为null,则不set该值。

&emsp;从片面的角度来看,这句话是对的,如果我们在save时指定id,因为insert对于一条记录只会出现一次,在执行insert之前,hibernate回去数据库执行select查询,如果查询不到数据,那么我们当前值的null和他当前缓存种的值null是相等的,就认为该值没有修改,在后面执行insert时mysql就会给该值赋予默认值。如果我们没指定id,那么他就不会select,同理,跟缓存中的null相等,在组建insert语句时就不会附带该值

&emsp;那么有这么一种情况,假若我们save了一个新记录,然后又update该记录,但是在update时,有个字段我们没有赋值,其为null,那么跟从数据库select出的数据发生了变化,就会set null向数据库中,此时如果我们数据库时not null的,就会报错,反之,则会该字段的值被设为了null