> git环境下，n个终端协议分支，m个后台业务代码，错综复杂的依赖如何解决？持续集成系统如何自动化的正确打包？git submodule应该怎么用？本文介绍一下我们项目中的解决方案。

<hr />

##一、	git submodule在团队开发中的应用
###1.1 几个场景和问题
1. 后台代码依赖的终端协议经常更新，终端人员定义协议，后台人员只读。
2. n个后台开发人员，m个终端开发人员，k个终端协议分支，后台工程代码如何准确依赖终端协议？
3. 如何让持续集成系统知道打包编译哪些外部依赖git和分支？系统上点选？前置脚本？
4. 把协议和公共库放在一起管理是否可行？

###1.2 问题分析
1. 协议在开发过程中会经常变动，多个开发人员、多个工程、多个分支，互相依赖的情形种类多。所以协议跟着工程走更合理，
2. 为了让CI系统从git上拉到代码可以直接编译，需要把协议分支信息记录在当前工程里，让开发提交代码后，无需在页面上针对依赖库多次点选

###1.3 解决方案
####思路：通过在工程代码里引入submodule关联协议代码
<img src="./img/submodule_1.png">
###方案一、 引用外部团队的协议文件git的一个分支(只读)
```
git submodule add http://{只读账号名}:{只读账号密码}@github.com/{外部协议名}.git {外部协议名}

1） 修改.gitmodules里的分支指向
git config  -f  .gitmodules  submodule.{外部协议名}.branch  {外部协议分支名}  

2） 重新拉取submodule
删除submodule目录
git submodule init
git submodule update --remote (--remote表示拉取目标分支最新提交)
git add {外部协议名}  （为何要add，后面会解释）

3） 检查变更文件
进入{外部协议名}目录，会发现{外部协议分支名}里的变更出现了，不用在意头指针游离问题（游离指针问题后面会解释）
```

###方案二、 引用团队内部的协议文件git，自己需要开分支加协议(读写)

```
git submodule add http://{只读账号名}:{只读账号密码}@github.com/{内部协议名}.git {内部协议名}


1） 在github.com页面上,创建自己的协议分支，如AA

2） 修改业务工程里.gitmodules里的分支指向
git config  -f  .gitmodules  submodule.{内部协议名}.branch  AA

3） 重新拉取submodule
删除submodule目录
git submodule init
git submodule update --remote
git add {内部协议名}

4） 进入{内部协议名}目录
git checkout AA
这一步很重要，确认已经切到指定分支，没有指针游离，建立版本跟踪

5） 修改jce

6） 提交
一般为了自动化流程会把submodule的只读账号信息放入git url中，导致变更无法push，执行下面这句话，替换url里的用户名。开始走CI流程时记得改回只读账号。
git remote set-url origin http://name@github.com/{内部协议名}.git

7） 完成提交
```

##二、git  submodule常见问题整理
下面来讲讲我们在用submodule时遇到的一些难忘的问题和原理
###2.1  官方文档
https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97
   在官方文档里找答案，大部分场景都覆盖了。(看官方、看官方、看官方) 
###2.2  .gitmodules文件
git config  -f  .gitmodules  submodule.{字模块名}.branch  {分支名}   
修改.gitmodules里的分支指向（也可以直接修改.gitmodules文件）
<img src="./img/submodule_4.png">
修改后重新拉取子模块，会出现游离
<img src="./img/submodule_2.png">

原因如下，表示只拉取了代码，但是没有跟踪。
<img src="./img/submodule_3.png">
这种情况下不用担心，你已经拉取到了submodule的分支文件。
如果你只是使用而不修改submodule文件，即只读，那就不用管它。如果你想修改submodule文件并提交，那你就要主动checkout指定分支，建立git跟踪。
###2.3 自动化编译流程&submodule账号读写权限问题
```
（原因：git push命令无法指定用户名和密码）
1 为了走自动化流程，建议把只读权限的git账号放入git url里
进入submodule文件目录
git remote set-url origin https://{user-read-only}:{passwd}@github.com/{project}.git

2 当你想改动子模块时，通过命令修改下git url里的账户。但是走CI流程时记得改回只读账户
进入submodule文件目录
git remote set-url origin https://{user-read-write}@github.com/{project}.git
```

###2.4 submodule管理机制
a）为何git submodule update了，没有出现submodule里新加的协议？
答：要加上remote参数，**git submodule update --remote**，拉取submodule最新提交。

b）为何经常git status发现submodule是红色的？需要git add么？
答：有必要，可以参考下面这个文章。submodule只记录你当时add的子模块commit id，即
[submodule git: commit id]，当submodule有新提交时，你的工程要更新下这个commit id。


