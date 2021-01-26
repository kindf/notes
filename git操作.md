---
title: "git基本操作"
date: 2019-12-10T22:39:34+08:00
draft: true
---

#### git配置相关  

> 配置全局账户<br>  
>> git config --global user.name "名称"  
>> git config --global user.email "邮箱"  

> 配置局部账户  
>> git config --local user.name "名称"  
>> gir config --local user.email "邮箱"  

> 查看配置  
>> git config --global --list  
>> git config --local -list  

#### git本地基本操作
> 本地变更添加到暂存区  
>> git add --all  
>> git add 文件名1 文件名2 ...

> 比较差异  
>> git diff 文件名   (某文件工作区与暂存区差异)  
>> git diff --cache 文件名   (某文件暂存区与HEAD差异)  
>> git diff   (工作区与暂存区差异)  
>> git diff --cache   (暂存区与HEAD所有差异)  

> 回滚
>> git checkout 文件名1 文件名2 ...   (将工作区文件回滚到暂存区版本)  
>> git reset 文件名1 文件名2 ...   (将暂存区文件恢复至与HEAD一样)  
>> git reset --hard   (将暂存区所有文件恢复至与HEAD一样)  

> 查看日志
>> git log  (显示该路径下的commit)  
>> git log -n  (显示最近n个commit)  
>> git log --online  (单行显示)  
>> git log –pretty=oneline (单行显示)  
>> git log 文件名  (涉及到该文件的所有commit)  
>> git blame 文件名  (该文件各行修改对应commit及作者)  
>> git log --online --graph --all  (用图示显示所有的分支历史)  

> 分支与标签
>> git branch 新分支 (基于当前分支创建新分支)  

> 与远端交互
>> git remote -v   (列出所有remote)  
>> git remote add url地址  (增加remote)  
>> git remote remove remote的名称  (删除remote)  
>> git pull   (更新本地分支且merge到本地分支)  
>> git push remote名称 分支名  (把本地分支 push 到远端)  
>> git fetch remote  (把远端所有分支和标签的变更都拉到本地)  


git checkout \-- 文件名:把文件在工作区的修改全部撤销，这里有两种情况：

> 一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

> 一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

git reset HEAD <file>可以把暂存区的修改撤销掉（unstage），重新放回工作区

理解工作区与暂存区的区别？

工作区：就是你在电脑上看到的目录，比如目录下testgit里的文件(.git隐藏目录版本库除外)。或者以后需要再新建的目录文件等等都属于工作区范畴。
版本库(Repository)：工作区有一个隐藏目录.git,这个不属于工作区，这是版本库。其中版本库里面存了很多东西，其中最重要的就是stage(暂存区)，还有Git为我们自动创建了第一个分支master,以及指向master的一个指针HEAD。

我们前面说过使用Git提交文件到版本库有两步：

第一步：是使用 git add 把文件添加进去，实际上就是把文件添加到暂存区。

第二步：使用git commit提交更改，实际上就是把暂存区的所有内容提交到当前分支上。


[菜鸟教程git教程(https://www.runoob.com/git/git-basic-operations.html)](https://www.runoob.com/git/git-basic-operations.html)

[廖雪峰老师的git教程(https://www.liaoxuefeng.com/wiki/896043488029600)](https://www.liaoxuefeng.com/wiki/896043488029600)
