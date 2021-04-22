### 1. 常用命令

#### 1.1. 基本使用

- 检出 checkout

  ```bash
  # 检出代码
  svn co repo_url
  
  # 检出代码，并指定目录名
  svn co repo_url saved_dir_name
  ```

- 加入版本控制 add

  ```bash
  # 添加指定文件
  svn add /path/to/file
  
  # 添加所有 .sh 后缀的文件
  svn add *.sh
  
  # 递归添加当前目录下的所有新文件
  # 如果使用参数 --no-ignore 则新增时会包含被设置为忽略的文件
  svn add . --force
  ```

- 提交 commit

  ```bash
  svn ci -m '提交更改'
  ```

- 更新 update

  ```bash
  svn up [file|path]
  
  # 更新到指定版本 11
  svn up -r 11 [file|path]
  ```

- 清除锁定 `svn cleanup`

- 重定向仓库地址到新地址

  ```bash
  svn switch --relocate old_repo_url new_repo_url
  ```

- 切换当前项目到指定分支

  ```bash
  svn switch svn://branch_url
  ```

- 检查工作副本下的文件状态 `svn st`

- 查看工作副本的版本库概览信息 `svn info`

- 查看版本库上某个目录下的文件列表（不需要本地检出）

  ```bash
  svn list svn://192.168.1.15/blog
  ```

#### 1.2. 分支和标签

- 创建 branch、tag

  ```bash
  svn cp svn://trunk_url svn://branch_url -m '创建分支'
  svn mkdir svn://branch_url -m '创建空分支'
  svn cp svn://trunk_url tag_url -m '创建标签'
  ```

- 删除 branch、tag

  ```bash
  svn rm svn://branch_url -m '删除分支'
  svn rm svn://branch_url -m '删除标签'
  ```

- 查看 branch、tag

  ```bash
  svn ls ^/branches --verbose
  svn ls ^/tags --verbose
  ```

#### 1.3. 历史记录

- 查看提交记录

  ```bash
  # 查看整个项目或指定文件目录的提交记录
  svn log [file|dir]
  
  # 查看两个指定版本之间的提交记录
  svn log -r 11:5 [file|dir]
  
  # 查看最新的 5 条提交记录
  svn log -l 5 [file|dir]
  
  # 查看带目录信息的提交记录
  svn log -l 5 -v [file|dir]
  ```

- 查看指定版本号的文件内容

  ```
  ➜  svn cat file -r 版本号
  ```

- 查看文件的修改详情（行级）

  ```bash
  # 比较工作副本下的文件与缓存在 .svn 的“原始”拷贝
  svn diff
  
  # 比较工作副本下的文件与指定版本号 5 下的文件
  svn diff -r 5 file
  
  # 比较两个版本库下的文件
  svn diff -r 11:2 file
  ```

- 查看文件每一行的最后修改人

  ```
  ➜  svn blame file
  ```

#### 1.4. 忽略文件

- 基本使用

  ```
  # 忽略指定文件或目录
  svn propset svn:ignore 'file|dir' dir
  
  # 忽略当前目录中所有以 .log 结尾的文件
  # . 表示当前目录，当指定具体目录时，该目录需处于版本控制状态
  svn propset svn:ignore -R '*.log' .
  svn ci -m 'ignore *.log file'
  ```

- 要忽略的文件已经被 add 过了

  - 未提交

  ```
  svn revert -R [file|dir]
  ```

  - 已提交

  ```
  svn del [file|dir]
  
  # 只从 svn 中忽略，而不删除本地文件
  svn del --keep-local [file|dir]
  ```

  然后再执行忽略设置并提交。

- 使用配置文件

  新建 .svnignore 文件，写入类似如下内容：

  ```
  runtime
  *.log
  *.apk  
  *.class
  ```

  然后执行设置

  ```
  svn add .svnignore
  svn propset svn:ignore -R -F .svnignore .
  ```

  提交设置

  ```
  svn ci -m 'add .svnignore and set some ignore'
  ```

#### 1.5. 小技巧

- 创建了一个文件夹，并且把它加入版本控制，但忽略文件夹中的所有内容

  ```
  ➜  svn mkdir dir2 
  ➜  svn propset svn:ignore '*' dir2 
  ➜  svn ci -m 'Adding "dir2" and ignoring its contents.'
  ```

- 创建一个不加入版本控制（忽略）的文件夹

  ```
  ➜  mkdir dir1 
  ➜  svn propset svn:ignore 'dir1' . 
  ➜  svn ci -m 'Ignoring a directory called "dir1".'
  ```

- 导入一个目录 import

  ```
  ➜  svn import /tmp/upload svn://192.168.0.1/repo1 -m 'add module upload'
  ```

### 2. 合并 merge

#### 2.1. 分支合并到主干

> ⚠️**注意：分支合并到主干后应当删除该分支，因为在 SVN 中该分支已经不能进行刷新也不能合并到主干。**

```
# 进入主干目录
➜  cd trunk

# 查看创建分支时的版本号
➜  svn log -q -v -l 1 --stop-on-copy svn://branch_url

# 合并分支上的最新代码到主干
# svn1.8+ 可以不加参数 --reintegrate
➜  svn merge --reintegrate svn://branch_url

# 查看分支中哪些改动已经被合并到主干
➜  svn mergeinfo svn://branch_url

# 查看分支哪些改动还未合并
➜  svn mergeinfo svn://branch_url --show-revs eligible

# ⚠️ 提交本次合并（不提交的话，合并不会生效，也会影响后面的更新操作）
➜  svn ci -m 'merge from branch r11:r5 into trunk'
```

#### 2.2. 主干合并到分支

```
# 进入分支目录
➜  cd branch

# 合并之前，可以使用 svn mergeinfo 命令查看主干上有哪些待合并的版本信息
➜  svn mergeinfo -v svn://trunk_url --show-revs eligible --log

# 合并主干上最新代码到分支
➜  svn merge svn://trunk_url

# ⚠️ 提交本次合并（不提交的话，合并不会生效，也会影响后面的更新操作）
➜  svn ci -m 'merge from trunk r11:r5 into branch'
```

### 3. 回滚 rollback

#### 3.1. 未提交使用 `revert`

```
# 查看文件状态
➜  svn status

# 回滚单个文件
➜  svn revert /path/to/file_name

# 回滚一个目录
➜  svn revert dir_name

# 回滚整个项目
➜  cd trunk
➜  svn revert -R .
```

#### 3.2. 已提交使用 `merge -r`

1. 先更新到最新代码

   ```
   # 这里得到最新版本号 11
   ➜  svn up
   Updating '.':
   At revision 11.
   ```

2. 找到要回滚到的版本号

   ```
   # 查看最新的 50 次提交记录
   # 最后的参数可以指定文件或目录
   ➜  cd trunk
   ➜  svn log -l 50 [file|dir]
   ```

   假如我们想要回退到的版本是 5，可以执行命令 `svn diff -r 11:5 [file|dir]` 简单查看两个版本的区别。

3. 回滚到指定版本号

   ```
   ➜  svn merge -r 11:5 [file|dir]
   ```

   验证回滚是否成功

   ```
   ➜  svn diff -r [file|dir]
   ```

4. 提交本次回滚操作到仓库

   ```
   ➜  svn ci -m "Revert revision from r28 to r25,because of ..."
   ```

5. 其它地方更新本次回滚操作

   ```
   ➜  svn up
   ```

#### 3.3. 回退误操作的 `svn up`

> **场景**：我们有一个目录是稳定版本的环境，因为误操作执行了 `svn up` 后，代码被更新到了仓库中的最新版本，而最新版还没有测试通过，所以我们需要先将更新的代码回滚，如下：

```
➜  cd trunk
➜  svn merge -r 13410:13962 .
```

⚠️注意：**千万不要提交，否则仓库中的代码也会回滚，而我们只希望本地目录回滚**，等到可以执行 `svn up` 的时候，再恢复：

```
➜  svn revert -R .
➜  svn up
```

### 4. 附录1

#### 4.1. 命令简写对照

- `svn checkout` **--->** `svn co`
- `svn update` **--->** `svn up`
- `svn commit` **--->** `svn ci`
- `svn delete` **--->** `svn rm` 或 `svn del`
- `svn copy` **--->** `svn cp`
- `svn list` **--->** `svn ls`
- `svn status` **--->** `svn st`
- `svn switch` **--->** `svn sw`
- `svn diff` **--->** `svn di`

#### 4.2. 处理冲突

```
# 查找合并时的冲突文件，手工解决冲突
➜  svn st | grep ^C      
# 告知svn冲突已解决
➜  svn resolved filename 
# 提示合并后的版本
➜  svn commit -m ""      
```

- (**p**) postpone 暂时推后处理，我可能要和那个和我冲突的家伙商量一番
- (**df**) diff-full 把所有的修改列出来，比比看
- (**e**) edit 直接编辑冲突的文件
- (**mc**) mine-conflict 如果你很有自信可以只用你的修改，把别人的修改干掉
- (**tc**) theirs-conflict 底气不足，还是用别人修改的吧
- (**s**) show all options 显示其他可用的命令

#### 4.3. 文件符号状态

- **U**: 表示从服务器收到文件更新了
- **G**: 表示本地文件以及服务器文件都已更新,而且成功的合并了
- **A**: 表示有文件或者目录添加到工作目录
- **R**: 表示文件或者目录被替换了.
- **C**: 表示文件的本地修改和服务器修改发生冲突

#### 4.4. 如何迁移仓库

1. 备份源仓库

   `svnadmin dump /path/to/repo > repo.dump`

2. 新建仓库

   `svnadmin create new_repo`

   > 注意：如果是迁移到 Windows 下，使用 VisualSVN 创建新仓库时，不要创建默认目录结构（branches、trunk、tags），否则在导入备份时候可能因为目录冲突发生报错而中断。

3. 从备份文件导入新建的仓库

   `svnadmin load /path/to/new_repo < repo.dump`

**建议**：切换代码的 svn 仓库地址时，最好使用新 svn 地址进行重新检出的方式操作；如果直接使用 `svn switch --relocate svn://svn_old svn://svn_new `进行切换，有可能报类似 `uuid` 冲突的错误。

### 5. 附录2：常见问题处理

- `svn: Can't convert string from 'UTF-8' to native encoding`

  **解决**：

  ```shell
  ➜  echo "export LC_ALL=zh_CN.UTF-8" >> /etc/profile
  ➜  source /etc/profile
  ```

- `SSL handshake failed: SSL 错误：Key usage violation in certificate has been detected.`

  **场景**：在 CentOS 服务器上，通过 svn 命令检出 VisualSVN 管理的仓库代码时。

  **解决**：安装高版本的 svn ，如下

  - 添加软件源 `vim /etc/yum.repos.d/wandisco-svn.repo`

    ```bash
    [WandiscoSVN]
    name=Wandisco SVN Repo
    baseurl=http://opensource.wandisco.com/centos/$releasever/svn-1.8/RPMS/$basearch/
    enabled=1
    gpgcheck=0
    ```

  - 删除旧版本 `yum remove subversion*`

  - 安装新版本

    ```bash
    yum clean all
    yum install subversion
    ```