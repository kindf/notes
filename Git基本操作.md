# Git配置

* #### 设置名称：

  * git config --global user.name "名称" 

* #### 设置邮箱：

  * git config --global user.email "邮箱"  

* #### 查看设置：

  * git config --global --list

# Git分支操作

* #### 新建分支：

  * git branch  分支名
  * git checkout -b 分支名

* #### 切换分支：

  * git checkout 分支名

* #### 删除分支：

  * git branch -D 分支名

* #### 合并分支

  * git merge 分支名

# Git基本工作流程

* [![5b8uCt.png](https://z3.ax1x.com/2021/10/27/5b8uCt.png)](https://imgtu.com/i/5b8uCt)

* #### 添加文件到暂存区(staging area)：

  * git add 文件名

* #### 将暂存区文件添加到仓库(local repository)中：

  * git commit -m "提交信息"

* #### 上传远程代码(remote repository)并合并：

  * git push -u origin 分支名

* #### 下载远程代码并合并

  * git pull

# Git暂存区文件操作

* #### 使用暂存区指定文件覆盖工作区对应文件：

  * git checkout 文件名

* #### 删除暂存区文件（工作区不做改变）：

  * git rm --cached 文件名

* #### 回滚暂存区对应文件到当前版本：

  * git reset HEAD 文件名

# Git远端交互

* #### 列出所有remote：

  * git remote -v

* #### 添加远端：

  * git remote add xxx

* #### 删除远端关联：

  * git remote remove xxx
