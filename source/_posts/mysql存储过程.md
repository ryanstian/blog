---
title: mysql存储过程
date: 2020-08-04 23:19:00
categories: 
tags: [mysql,存储过程]
copyright: true #新增,开启
---

1.此篇记录简单的mysql存储过程,意在分表,将1个数据库表根据id分为多个表,用于数据迁移。然而实测速度极慢(本地1000W条数据分表,1分钟分了5W条),根据搜集资料显示,瓶颈疑似出现在游标,google回答建议采用临时表代替游标

2.根据搜集资料,现阶段不建议使用存储过程,而是在业务层来处理数据。因为存储过程极其依赖数据库,难以迁移。  
存储过程会加重数据库负担,数据库应该仅承担数据存储职责而非业务职责
<!--more-->

```
注意,如果mysql窗口是打开的mysql这一层级,则需要选择数据库才可继续接下来操作,否则会提示选择数据库

//将结束符暂时更改为//,以免被程序中的语句分隔符;影响
DELIMITER //
//创建存储过程,如果本身已存在同名存储过程,则创建失败
CREATE PROCEDURE wk()
//开始写存储过程
BEGIN
    //语法1:变量声明必须在所有函数之前
    declare i int;
    declare tmp varchar(100);
    declare userId int;
    declare nickname varchar(32);
    declare phone varchar(13);
    declare addr varchar(100);
    declare email varchar(50);
    declare profile_photo_number varchar(50);
    declare tabl varchar(32);
    declare myvar text;
    //语法1:该语句已经属于函数部分,所以变量声明需要在这之前,否则报错
    DECLARE cur CURSOR FOR SELECT * FROM et_user_profile;
    //语法2:set语句给变量赋值
    set i = 0;
    //语法3:while循环语句
    while i < 100 do
        //语法4:此处CONCAT拼接字符串,给分表添加后缀
        SET tmp = CONCAT('et_user_profile_',i);
        //语法5:由于mysql存储过程中的sql语句不支持动态变量,所以只能先拼接出最终字符串,再将字符串转换为执行语句执行
        //语法6:set @语句可以免于提前声明变量,相当于声明+赋值语句
        SET @STMT :=CONCAT('CREATE TABLE IF NOT EXISTS ',tmp,' (
            `id` INT UNSIGNED PRIMARY KEY COMMENT "id follow et_user.id",
            `nickname` VARCHAR(32) NOT NULL COMMENT "nickname",
            `phone` VARCHAR(13) NOT NULL COMMENT "phone",
            `addr` VARCHAR(100) NOT NULL COMMENT "address",
            `email` VARCHAR(50) NOT NULL COMMENT "email",
            `profile_photo_number` VARCHAR(50) NOT NULL COMMENT "profile_photo_number"
        );');
        PREPARE STMT FROM @STMT;
        EXECUTE STMT;
        set i = i +1;
    end while;
    //语法7:打开之前声明的游标
    OPEN cur;
        //语法8:设置循环点
        myloop: LOOP
            //语法9:从游标中提取数据(一次一行,可理解为迭代)
            FETCH cur INTO userId,nickname,phone,addr,email,profile_photo_number;
            //语法10:if语句
            IF done=1 THEN
                //语法11:离开循环点
                LEAVE myloop;
            END IF;
            set tabl = CONCAT('et_user_profile_',floor(userId%100));
            SET @STMT2 := CONCAT('insert into ',tabl,' values(',userId,',','""',',','""',',','""',',','""',',','""',');');
            PREPARE STMT2 FROM @STMT2;
            EXECUTE STMT2;
        END LOOP myloop;
    CLOSE cur;
//语法12:存储过程结束
END //
//语法13:将结束符改回
DELIMITER ;
//语法14:调用刚才创建的存储过程,由于存储过程直接存于mysql中,故可以后重复使用,不必立刻调用
call wk();
```