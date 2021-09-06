---
title: 利用git完成服务器自动化部署解决方案
date: 2019-06-24 14:51:00
categories: git
tags: [git]
copyright: true #新增,开启
---

#### 前言
21年回过头来看这篇文章，发现当时的自己是有一定局限性的。当初写这篇的目的，是学校里项目测试时，希望服务器可以自动部署最新代码（仅小项目测试环境）。  
但回过头看，又有很多更好的解决方式
- 部署功能可以用jenkins来取代
- 文件上传可以用文件分发来取代

需要的环境:linux服务器,git,maven,java
<!--more-->

#### 2019.6.25更新
昨天忘记了说一个重要的问题,如果你是在window环境下写的shell脚本到linux环境下运行,由于两者系统换行符不一致,需要在linux中执行vim你的脚本名,进入脚本:set ff=unix,注意":"这个符号需要带着,不明白的请去搜vim命令
**一定要赋予脚本可执行权限**,赋权具体命令下文sh代码有提及
刚才看了些博客,有提到用hook触发,而不是自己去循环访问,思路待定

---
#### 原答案

#### &emsp;首先讲成果,上代码,我会在其中伴随大量讲解
```sh
#!/bin/sh
###本代码中的该项目特有名称均会用其他文本代替,例如我的项目名就用demo代替
###该项目的git文件夹我们暂且称呼为demo

function updateAndRestart(){
	#切换到online分支,这个地方需要根据自己的git分支做适配
	git checkout online
	#git rev-parse online命令用于查看本地的online分支最后一次提交id
	LOCALONLINE=$(git rev-parse online)
    #用于这句话就是打印到控制台,没什么意义
	echo "本地ONLINE为${LOCALONLINE}"
	#从远程仓库fetch,这里选择fetch而不是pull也是为了性能考虑,如有偏差,请根据自己想法修改
	git fetch
	echo "从远程仓库拉取结束"
	#获取已经fetch下来的远程仓库的HEAD
	#如果git rev-parse orgin/online虽然是获取远程仓库online分支的最后一次提交,但是他不会真的去连接远程仓库拉取信息,而是读取本地的远程仓库的缓存信息,所以之前需要git fetch也是为了刷新本地缓存的作用。
    #关于如何直接去远程查看远程仓库最后一次提交这个问题,我找了半天没有找到这个方法
	REMOTEONLINE=$(git rev-parse origin/online)
	echo "远程HEAD为${REMOTEONLINE}"
	#检查远程仓库是否与本地ONLINE一致,若不一致,则证明了远程已更新
	if [[ $LOCALONLINE != $REMOTEONLINE ]]; then
		echo "进入重启-------------------------------------------"
		#将远程分支的更新合并到本地,由于git pull命令可以理解为git fetch+git merge,这一步的意义这里不做赘述
		git merge $REMOTEONLINE
		#杀死所有名为下列的进程
        #这里只讲一点,由于awk命令下文介绍过,那么此时可以想象文本状态是kill -9 进程pid,sh命令是把之前的输出当作脚本来执行,那么就成果实现了批量kill进程
		jps | grep demo.jar|awk '{print "kill -9 " $1}'|sh
		#重新打包jar
		mvn clean package
		#给jar授权
        #这里有必要作下说明,此脚本我是运行在root用户下(这点很重要,如果是其他用户,则在权限方面需要注意做适配)
        #chomd是赋权命令,后面参数则是其权限,参数每部分的具体意义请自行查询
        #这条命令是授予demo.jar的root用户可执行权限(原本被mvn打包后默认为读写权限,没有执行权限)
		chmod 744 /你的目录/demo.jar
        #这个文件我不知道是什么,在window环境下尝试情况,删除了也不会有什么影响。有人说.jar是不带依赖的,original是带依赖的,但是观察文件大小,发现original文件只有几十k,明显不是带着依赖一起打包的样子
		chmod 744 /你的目录/demo.jar.original
		#后台运行jar,具体意义可见后半部分文章
		nohup java -jar /你的目录t/demo.jar >  /你的目录/xxx-`date +%Y-%m-%d-%H-%M-%S`.log 2>&1 &
		sleep 15
	fi
}
###上面的是函数,只有调用时才会运行,首先运行的是下面代码

#首先cd到你的demo存放目录,这一步可有可无,根据你的项目路径做好适配就行
cd /xxx/xxx/demo
#下面这行代码是脚本启动时用来检测是否demo程序正在运行
#这里顺便讲解下shell和java的知识
#   jps该命令可以理解为和ps命令类似,只不过是用来显示java进程
#   |这个管道命令我无法解释,自己去搜索引擎
#   grep用来抓取出包含demo.jar字段的行
#   awk一种文本处理命令,将文本按照我们定义的规则处理
#       这里'{print $1}'代表的是输出每行的第一个参数(在我的linux系统中,每行的第一个参数正好是java进程的pid,其他人需要根据情况适配,注意'引号一定要有)
#   wc -l是统计命令,由于之前都是一行一行打印的,所以此命令可以很轻松统计有多少在运行
#   综上所述,该条命令的意义在于:统计名为demo.jar的java进程的数目,awk这段命令后来想了想属于冗余命令了,可视情况去除
SUMMERTRAINPID=$(jps | grep demo.jar |awk '{print $1}' |wc -l)

if [[ SUMMERTRAINPID!=0 ]]; then
	echo "脚本开始,检测到程序未启动,先启动程序------------------------"
    #summertrain-`date +%Y-%m-%d-%H-%M-%S`
    #下面这条命令是在后台启动demo.jar并且将输出重定向到指定的log文件,如果文件不存在会新建
    #其实nohup java -jar /你的路径/demo.jar >  /你log日志的路径/文件名.log &这条命令就可以做到这一点
    #下面这条命令多出来的几个字段表示的是将error日志也重定向到log文件
    #小提示,在自己的脚本,log日志太长不易于观看,所以可以文件名可以在后缀加上`date +%Y-%m-%d-%H-%M-%S`,参数可以在生成文件时自动加上当前实际后缀,例如demolog-`date +%Y-%m-%d-%H-%M-%S`.log
	nohup java -jar /你的路径/demo.jar >  /你log日志的路径/文件名.log 2>&1 &
    #这里我令他休眠15秒等待java程序启动
	sleep 15
fi

#接下来开始循环检测是否git有更新
while true
do
	echo "自动循环中"
	updateAndRestart
	sleep 10
done
```

写该脚本时参考了git命令,shell语法,linux命令
shell脚本中的空格一定要注意,少一个空格往往意义就会不同
如果团队人数更多,那么该方法不再试用,每次程序启动都需一定时间。