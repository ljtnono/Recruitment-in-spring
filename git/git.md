## git 提交规范

```txt
注意：提交注释请按以下格式
[feat] xxxxxxx 代表提交的文件为新增功能，不需要cherry pick到其他分支
[fix] xxxxxx 代表提交的文件为修复bug，需要cherry pick到其他分支
其他格式类型可以参考(不做强制要求)：
[docs]: 只改动了文档相关的内容
[style]: 不影响代码含义的改动，例如去掉空格、改变缩进、增删分号
[sh]: 只改动了shell脚本相关的内容 
[sql]: 只改动了sql脚本相关的内容
[build]: 构造工具的或者外部依赖的改动，例如webpack，npm
[revert]: 执行git revert打印的message
```

## git平时常用命令

### Git global setup

```bash
git config --global user.name "lingjiatong"
git config --global user.email "935188400@qq.com"
```

### Create a new repository

```bash
git clone git@192.168.172.218:ljtnono/ossbasedocker.git
cd ossbasedocker
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```

### Push an existing folder

```bash
cd existing_folder
git init
git remote add origin git@192.168.172.218:ljtnono/ossbasedocker.git
git add .
git commit -m "Initial commit"
git push -u origin master
```

### Push an existing Git repository

```bash
cd existing_repo
git remote rename origin old-origin
git remote add origin git@192.168.172.218:ljtnono/ossbasedocker.git
git push -u origin --all
git push -u origin --tags
```

## git常用60个命令

### 配置操作

#### 全局配置

```bash
git config --global user.name '你的名字'
git config --global user.email '你的邮箱'
```

#### 当前仓库配置

```bash
git config --local user.name '你的名字'
git config --local user.email '你的邮箱'
```

#### 查看全局配置

```bash
git config --global --list
```

#### 查看当前仓库配置

```bash
git config --local --list
```

#### 删除全局设置

```bash
git config --global --unset 要删除的配置项
```

#### 删除当前仓库配置

```bash
git config --local --unset 要删除的配置项
```

### 本地操作

#### 查看变更情况

```bash
git status
```

#### 将当前目录及其子目录下所有变更都加入到暂存区

```bash
git add .
```

#### 将仓库内所有变更都加入到暂存区

```bash
git add -A
```

#### 将指定文件添加到暂存区

```bash
git add 文件1 文件2 文件3
```

#### 比较工作区和暂存区的所有差异

```bash
git diff
```

#### 比较某文件工作区和暂存区的差异

```bash
git diff 文件
```

#### 比较某文件暂存区和HEAD的差异

```bash
git diff --cached 文件
```

#### 比较某文件工作区和HEAD的差异

```bash
git diff --HEAD 文件
```

#### 创建commit

```bash
git commit
```

#### 将工作区指定文件恢复成和暂存区一致

```bash
git checkout 文件1 文件2 文件3
```

#### 将暂存区指定文件恢复成和HEAD一致

```bash
git reset 文件1 文件2 文件3
```

#### 将暂存区和工作区所有文件恢复成和HEAD一样

```bash
git reset --hard
```

#### 用户difftool比较任意两个commit的差异

```bash
git difftool 提交1 提交2
```

#### 查看哪些文件没被git管控

```bash
git ls-files --others
```

#### 将未处理完的变更先保存到stash中

```bash
git stash
```

#### 临时任务处理完后继续之前的工作

```bash
git stash pop ## 不保留stash
git stash apply ## 保留stash
```

#### 查看所有stash

```bash
git stash list
```

#### 取回某次stash的变更

```bash
git stash pop stash@{数字n}
```

#### 优雅修改最后一次commit

```bash
git add .
git commit --amend
```

### 分支操作

#### 查看当前工作分支及本地分支

```bash
git branch -v
```

#### 查看本地和远端分支

```bash
git branch -av
```

#### 查看远端分支

```bash
git branch -rv
```

#### 切换到指定分支

```bash
git checkout 指定分支
```

#### 基于当前分支创建新分支

```bash
git checkout -b 新分支
## 或者
git branch 新分支
```

#### 基于指定分支创建新分支

```bash
git branch 新分支 指定分支
```

#### 基于某个commit创建分支

```bash
git branch 新分支 某个commit的id
```

#### 创建并切换到该分支

```bash
git checkout -b 新分支
```

#### 安全删除本地某分支

```bash
git branch -d 要删除的分支
```

#### 强行删除本地某个分支

```bash
git branch -D 要删除的分支
```

#### 删除已合并到master分支的所有本地分支

```bash
git branch --merged master | grep -v '^\*\| master' | xargs -n 1 git branch -d 
```

#### 删除远端origin 已不存在的所有本地分支

```bash
git remote prune origin
```

#### 将A分支合并到当前分支且为merge创建commit

```bash
git merge A分支
```

#### 将A分支合并到B分支且为merge创建commit

```bash
git merge A分支 B分支
```

#### 将当前分支基于B分支做rebase，以便将B分支合并到当前分支

```bash
git rebase B分支
```

#### 将A分支基于B分支做rebase，以便将B分支合入到A分支

```shell
git rebase B分支 A分支
```

### 变更历史

#### 当前分支各个commit用一行显示

```shell
git log --oneline
```

#### 显示就近的n个commit

```shell
git log -n
```

#### 用图示显示所有分支的历史

```shell
git log --oneline --graph --all
```

#### 查看涉及到某文件变更的所有commit

```shell
git log 文件
```

#### 某文件各行最后修改对应的commit以及作者

```shell
git blame 文件
```

### 标签操作

#### 查看已有标签

```shell
git tag
```

#### 新建标签

```shell
git tag v1.0
```

#### 新建带备注标签

```shell
git tag -a v1.0 -m "备注文字"
```

#### 给指定的commit打标签

```shell
git tag v1.0 commitid
```

#### 推送一个本地标签

```shell
git push origin v1.0
```

#### 推送全部未推送过的本地标签

```shell
git push origin --tags
```

#### 删除一个本地标签

```shell
git tag -d v1.0
```

#### 删除一个远端标签

```shell
git push origin :refs/tags/v1.0
```

### 远端交互

#### 查看所有远端仓库

```shell
git remote -v
```

#### 添加远端仓库

```shell
git remote add url
```

#### 删除远端仓库

```shell
git remote remove remote名称
```

#### 重命名远端仓库

```shell
git remote rename 旧名称 新名称
```

#### 将远端所有分支和标签的变更都拉到本地

```shell
git fetch remote
```

#### 把远端分支的变更拉到本地，且merge到本地分支

```shell
git pull origin 分支名
```

#### 将本地分支push到远端

```shell
git push origin 分支名
```

#### 删除远端分支

```shell
git push remote --delete 远端分支名
git push remote :远端分支名
```