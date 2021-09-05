---
title: git使用笔记
date: 2019-06-21 13:02:00
categories: 
tags: [git]
copyright: true #新增,开启
---

#### 详细介绍git使用和配置(不包括安装)

+ #### 什么是git?
    + 首先我用通俗语言解释下,git是一种版本控制工具,你既可以在本地进行版本控制,也可以与搭建好git服务器的远端进行同步
<!--more-->
+ #### 如何使用?
    + windows的可以官方下载安装包,linux可以命令行下载(对于window来说可能需配置环境变量,可有可无)
    </br>
    + 配置全局信息
        + 随便找个地方右键打开git bash
        + ps:这里配置的昵称和邮箱可以随便写,作用体现在,假如你提交了git,那么在git记录中会显示提交者昵称和邮箱,即为下面输入的
        + 输入git config --global user.name "你的昵称"
        + 输入git config --global user.name "你的邮箱"
    </br>
    + 创建git仓库
        + 随便找个地方新建文件夹进去打开git bash(此处建议选一个父文件夹作为git仓库目录)
        + 输入git init
            + 该命令的作用是在当前文件夹下生成git仓库所需文件(注意,这里git仓库通常指的是一个项目,而不是管理多个项目的仓库,而且生成的文件为.git是个隐藏文件夹)
    </br>
    + 使用git
        + 当我们在文件夹下做了操作以后(添加修改删除文件),可以git add .
            + .代表暂存全部文件,当然也可以是其他写法或部分文件
        + 此时我们已经add成功,接下来git commit -m"此次提交的注释"
        + 此时,本地的使用基本就到这里(此外还有分支,冲突等各种概念,不在本篇讲)
    </br>
    + 关联github/码云(也可以是其他的或者自己搭建的git服务器)
        + 首先我们在git bash中生成一对密钥,命令为:ssh-keygen -t rsa -C "你之前填写的邮箱"
            + 其实一般码云或者github都要官方绑定密钥教程,基本都一样
        + 生成密钥后我们把密钥配置到git服务器上
            + 如果是github之类的你从个人setting可以找到配置密钥的地方,如果是个人git服务器则可能需要手动添加
            + 刚才生成的密钥分为公钥和私钥,一般公钥以.pub结尾,这部分涉及到密码学,你只需要知道这是非对称密钥用来代替用户密码做身份验证就好了,具体内容请自行搜索
                + 在window中默认保存在C://user/{你的用户名}/.ssh文件夹中
            + 当我们配置到服务器公钥后,就可以正常的git clone 远程私有仓库等操作
+ #### 可视化界面SourceTree   
    + SourceTree需要注册啥的,可能被墙了,自己解决
    + 这里要提到一点,它仅仅是个可视化界面,仍然需要你安装git
    + 团队里有人出现了SourceTree没有权限的问题,打开密钥设置界面(不同版本打开位置有所不同,大概都是在工具-选项这一块),找到之前我们生成的密钥(上文提到过位置),将私钥添加进去即可

+ #### 解决冲突
    + 这里我不建议大家团队合作时在采用pull来拉代码,这样的话如果有冲突,文件会被标记为冲突,建议用fetch先检测一下
    + 如果是在commit之前拉取,产生了冲突,那么可以针对冲突的文件,抛弃自己的修改(等fetch+merge之后在手动增加回来)
    + 如果是在commit之后拉取产生了冲突,就会出现无法拉取也无法推送的情况,这时候我们可以撤销commit操作,使其返回到上一种情况
    ```
    git reset [--参数] 提交id
    参数有:
        mixed:不删除改动的代码,撤销commit和add
        soft:不删除改动代码,撤销commit
        hard:删除改动代码,撤销commit和add(这种是用指定的提交id强行覆盖掉现有代码)
    示例:
        git reset --soft xxxxxxxxx
    ```
    + 这里我们说的是命令行的操作,如果是sourceTree,那么对应的就是"重置当前分支到此次提交"
    + 如果想要抛弃对方的提交,使自己本次提交强行覆盖掉对方,那么可以见下方代码。如果是sourceTree的话,则需要取选项里面允许强制提交。此外,如果用的github之类的这种托管远程仓库,那么对方可能还会设置了权限,只有拥有者有权限强制推送,项目所属人去github里面设置一下就好(这一点笔者没有实践)
    ```
    git push --force origin
    ```
    
+ #### 一些其他的诸如merge,rebase的用法
    + 这些用法笔者的理解有限,这里附一个知乎的提问,大家可以参考下[在开发过程中使用git rebase还是git merge，优缺点分别是什么？](https://www.zhihu.com/question/36509119)

+ #### 一些其他的bug如何解决
    + 现象:原本中文编码变成了/xxx/xxx之类的八进制码
    解决:修改本地git仓库下的.git隐藏文件夹下config文件,在[core]部分新增quotepath = false字段保存即可

+ #### git忽略规则
&emsp;当我们同步到远程仓库时,配置了.gitignore可以令git按照我们定义的规则,选择性的跟踪本地仓库的文件
&emsp;具体的语法规范这里不做描述,搜索引擎搜教程即可
&emsp;这里我将列出学生team一个springboot项目合作时使用的.gitignore。对官方生成的git做了少许改动

```git
HELP.md
target/
!.mvn/wrapper/maven-wrapper.jar
!**/src/main/**
!**/src/test/**

###团队合作config###
###由于团队合作时每个成员数据库账户密码不一样,所以每个成员都有个个人配置信息,在通用配置里面引入###
person.properties

###springboot###
###此处表示忽略测试文件夹###
/src/test/java

### STS ###
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache

### IntelliJ IDEA ###
.idea
*.iws
*.iml
*.ipr

### NetBeans ###
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/
build/

### VS Code ###
.vscode/


# Log file
*.log

# Compiled class file
*.class

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

hs_err_pid*

```
&emsp;常见问题:配置了.gitignore仍然无法忽略
&emsp;解决方式:git rm --cached filename  
&emsp;删除该filename文件的本地缓存,然后再进行add和commit等操作,等push到远端后,以后再就不会被追踪
&emsp;原因:这是由于在配置.gitignore之前该文件就已经被git追踪造成的,