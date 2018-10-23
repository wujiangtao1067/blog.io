# 仓库配置-用户名和邮件

作用：在提交日志中显示提交者的用户名和邮件。
1. 全局配置（所有git本地仓库如果没有单独的配置，则读取全局配置）
终端直接输入以下命令：

```git config --global user.name ***```

```git config --global user.email ***```

2. 给当前仓库设置单独的用户名和email
进入项目根目录，输入以下命令

```git config user.name ***```
```git config user.email ***```

3. 查看配置信息

```git config --list```

4. 查看配置文件

- 全局配置：```cat ~/.gitconfig```

- 当前项目配置：```cat currentProject/.git/config```



# 与远程仓库的关联与取消

> origin是远程仓库的标识，可随意指定。



1. 关联

```git remote add origin git@....git```

2. 取消

```git remote remove origin```

3. 查看

```git remote -v```



# 分支

> 比如有一个分支dev

#### 查看分支
```git branch```

#### 创建分支

```git branch -b dev```

#### 切换分支

```git checkout dev```

#### 分支合并

例如：将dev分支的内容合并到master

先切换到master分支，然后使用下面的命令

```git merge --no-ff dev```：不使用快进方式合并，保留commit的历史。（使用快进方式合并，如果删除分支，会丢失分支信息。）

#### 删除分支
> ```***```表示分支名



```git branch -d ***```
强行删除```git branch -D ***```

# 添加新功能

添加一个新功能时，你肯定不希望因为一些实验性质的代码，把主分支搞乱了，所以，每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。

```git checkout -b feature-***```

# 修复bug

修复bug时，我们会通过创建新的bug分支```git checkout -b issue-***```进行修复，然后合并，最后删除；
当手头工作没有完成时，先把工作现场```git stash```一下，然后去修复bug，修复后，再```git stash pop```，回到工作现场。