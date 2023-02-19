# git

## 命令 

### 变基与合并

- `git rebase`

  ![WechatIMG2.png](https://img-blog.csdnimg.cn/img_convert/3c4b8271175be2e7475db07b5de80533.png)

  ```shell
  # git rebase出现冲突
  git rebase 
  git add
  git rebase --continue
  ```

  

- `git merge`

  ![image-20210531151838328.png](https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/e587fb1eba2a807440992fb0dd408aac.jpeg)

### 撤回

- 修改但是并未add的文件

  ```shell
  git checkout --目录名 # 删除某个目录下的修改
  git checkout .
  
  git clean -f # 删除当前工作目录下的未跟踪文件，但不删除文件夹运行
  git clean -df # 删除当前工作目录下的未跟踪文件以及文件夹运行
  ```

  

- `git reset`

  ```shell
  # 目标版本号之后的版本会丢
  #  1->2->3->4 （git reset --hard 2）1->2
  git log
  git reset --hard 目标版本号
  git push -f
  ```

- `git revert`

  ```shell
  # 撤销之前的某一版本，同时可以保留该目标版本后面的版本
  # 1->2->3->4 (git revert -n 3,git commit -m ..) 1->2->3->4->5
  git log
  git revert -n 目标版本号
  # 这里可能会出现冲突，那么需要手动修改冲突的文件。而且要git add 文件名
  git commit -m 版本名
  git push
  ```

  

### 剪切

- `git cherry-pick`

  ```shell
  a - b - c - d   master
           \
             e - f - g feature
             
  git checkout master
  git cherry-pick f 
  
  a - b - c - d - f   master
           \
             e - f - g feature
             
  # 转移多个commit git cherry-pick A B
  # 转移多个连续的commit git cherry-pick A..B  不包含A
  #                                    A^..B 包含A
  
  # 遇见冲突
  git add 
  git cherry-pick --continue
  # 或者
  git cherry-pick --abort
  ```

  

## 公共库子模块

### 添加

- `git submodule add 仓库地址  路径`
  - 仓库地址是指子模块仓库地址，路径指将子模块放置在当前工程下的路径    注意：路径不能以 / 结尾（会造成修改不生效）、不能是现有工程已有的目录（不能順利 Clone）
- 命令执行完成，会在当前工程根路径下生成一个名为“.gitmodules”的文件，其中记录了子模块的信息。添加完成以后，再将子模块所在的文件夹添加到工程中即可。

### 删除

- 首先，要在“.gitmodules”文件中删除相应配置信息。然后，执行`git rm –cached`命令将子模块所在的文件从git中删除。

### 克隆

- 当使用git clone下来的工程中带有submodule时，初始的时候，submodule的内容并不会自动下载下来的，此时，只需执行如下命令：`git submodule update --init --recursive`

