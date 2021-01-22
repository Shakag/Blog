---
title: git 设置
description: git 相关的一些操作方式......
tags: [前端,后端]
---


### git全局设置

##### 配置文件

> git 总共有 **三个** 配置文件 仓库>全局>系统

* 仓库级  local  

  ```
  文件位置为当前git目录下的 .git/gitconfig
  ```

* 全局级  global  （当前用户）

  ```
  文件位置为 ~/.gitconfig 
  ```

  **~/ 指代当前登录用户的用户目录**

* 系统级  system

  ```
  git 安装目录下的 etc 文件夹下的 gitconfig 文件
  ```

> 执行顺序

* 仓库级 (local) > 全局级 (global) > 系统级 (system)

##### 注意事项

**配置全局级时  git config 后需先跟 --global , 再写具体命令**

* 正确 : **git config --global** user.name shakag
* 错误 : **git config**  user.name shakag **--global**
* 上述错误写法不会生成对应的 **.gitconfig** 文件，原因：未知

##### Windows

> git config 的所有目录

```
当前项目目录下的 config 文件
c/user/用户/当前用户名 (即为~/目录) 下的 .gitconfig 文件 （git config --global设置的为此文件）
git 安装目录下的 etc 文件夹下的 gitconfig 文件
```

> 用途

```
通过 c/用户下的 .gitconfig 配置别名 初始化，复制到所有电脑终端
```

##### Linux

> 配置文件位置

```
system: /etc/gitconfig 
包含系统上每一个用户及他们仓库的通用配置。 如果在执行 git config 时带上 --system 选项，那么它就会读写该文件中的配置变量。
（由于它是系统配置文件，因此你需要管理员或超级用户权限来修改它。）

global: ~/.gitconfig 或 ~/.config/git/config 
只针对当前用户。 你可以传递 --global 选项让 Git 读写此文件，这会对你系统上 所有 的仓库生效。

```

&nbsp;

## .git目录分析

### hooks 文件夹

> 存放git 命令执行时的相关钩子

[跳转到详细信息](#git_hooks)

### COMMIT_EDITMSG

> 上一次 git commit 提交的备注信息

```js
git commit -am "这是我的备注"
//COMMIT_EDITMSG 中存储的就是 "这是我的备注" 字符
```

### config

> 仓库的配置文件

可设置一次push多个远程仓库地址

```shell
[remote "origin"]
	url = git@github.com:shakag/xxx.git
	url = git@gitee.com:shakag/xxx.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```



&nbsp;

## git命令

### 1.git add 对应底层命令  

```js
1. git hash-object -w <文件名>  /* 在objects目录中生成git对象（blob) 工作目录->版本库 */ 
2. git update-index ...         /* 版本库->暂存区 */ 
/* 整体流程：工作目录-->版本库-->暂存区  */
```
<img src="/imgs/git_add.svg?center" align=center>

### 2. git commit -m 对应底层命令

>  git commit -am "备注" 
>
> 如果备注用单引号括起来，则会认为单引号也为备注内容，**备注信息**有空格**必须用双引号**

```js
1. git write-tree        /* 在objects目录中生成tree对象 */ 
2. git commit-tree       /* 在objects目录中生成commit对象 */ 
```

### 3. git 远程版本回退

> 初始版本库

<p align="center" ><img src="/imgs/git_1.png" align=center></p>

> 在要回退的位置新建分支	git checkout -b skk 6a74d7f

<p align="center" ><img src="/imgs/git_2.png" align=center></p>



> 将head指向为最新的提交	 git reset --soft d31b5c8
> 
> index 和 repository中的内容不动 ，仍然为回退的版本内容
<p align="center" ><img src="/imgs/git_3.png" align=center></p>

> 提交	git commit -am 'back to v2'
>
> 此时内容为之前内容，但提交对象为最新，所以可以提交，不然会冲突
<p align="center" ><img src="/imgs/git_4.png" align=center></p>

> 切换回 master , 合并分支	git checkout master |  git merge skk
<p align="center" ><img src="/imgs/git_5.png" align=center></p>
&nbsp;

## git_hooks

### git init 初始化脚本

[自定义hooks文件](https://github.com/Shakag/Document/tree/master/Git/hooks)

> Git 钩子是在 Git 仓库中特定事件发生时自动运行的脚本。例如：git commit 时会触发执行 pre-commit **(不带后缀)** 等脚本文件
>
> 初始化带 **sample后缀** 的脚本**并不会执行** ，想要执行需删除后缀名

<p align="center" ><img src="/imgs/git_6.png" align=center></p>

### git commit 相关

> commit 时会先后触发 **pre-commit 、prepare-commit-msg 、commit-msg、post-commit** 脚本

<p align="center" ><img src="/imgs/git_7.png" align=center></p>

##### 1. pre-commit

> `pre-commit` 脚本在每次你运行 `git commit` 命令时，Git 向你询问提交信息或者生产提交对象时被执行。
>
> `pre-commit` 不需要任何参数，以非0状态退出时将放弃整个提交。



##### 2. prepare-commit-msg

> `prepare-commit-msg` 钩子在 `pre-commit` 钩子在文本编辑器中生成提交信息之后被调用。这被用来方便地修改自动生成的 squash 或 merge 提交。

* 它接受1到3个参数 : **$1 (文件路径)、$2 (来源)、 $3**

```
$1 是包含commit msg的文件路径 (一般为.git/COMMIT_EDITMSG)
$2 是commit msg的来源, 可能的值有: 
  `message` (当使用`-m` 或`-F` 选项);
  `template` (当使用`-t` 选项,或`commit.template`配置项已经被设置);
  `merge` (当commit是一个merge或者`.git/MERGE_MSG`存在); 
  `squash`(当`.git/SQUASH_MSG`文件存在);
  `commit`, 且附带该commit的SHA1 (当使用`-c`, `-C` 或 `--amend`).
如果以非0状态退出, 'git commit' 将会被取消.
$3 是相关提交的 SHA1 哈希字串。只有当 -c、-C 或 --amend 选项出现时才需要。
```

* 可以通过 **$(cat $1)** 取得 commit 时的写的 **备注**  
* 也可以通过 **echo "修改备注" > $1** 命令来修改 commit 实际存入的信息

<p align="center" ><img src="/imgs/git_8.png" align=center></p>



##### 3. commit-msg

* 它只有1个参数 : **$1 (文件路径)**

* 同样可以通过 **echo "修改备注" > $1** 命令来 **再次修改** commit 实际存入的信息 
* 如果 **prepare-commit-msg 、commit-msg** 都修改了备注，则以 commit-msg 中的为准，因为执行顺序他的执行顺序在后面

> `commit-msg` 钩子和 `prepare-commit-msg` 钩子很像，但它会在用户输入提交信息之后被调用。这适合用来提醒开发者他们的提交信息不符合你团队的规范。
>
> 传入这个钩子唯一的参数是包含提交信息的文件名。如果它不喜欢用户输入的提交信息，它可以在原地修改这个文件（和 `prepare-commit-msg` 一样），或者它会以非 0 状态退出，放弃这个提交。



##### 4. post-commit

> 这个脚本没有参数，而且退出状态不会影响提交。
>
> `post-commit` 钩子在 `commit-msg` 钩子之后立即被运行 。它无法更改 `git commit` 的结果，所以这主要用于通知用途



##### 5. post-checkout

它接受3个参数 : **$1 (前HEAD )、$2 (新HEAD)、 $3(1或者0)**

- HEAD 前一次提交的引用
- 新的 HEAD 的引用
- 1 或 0，分别代表是分支 checkout 还是文件 checkout。

<p align="center" ><img src="/imgs/git_9.png" align=center></p>

##### [更多hooks参考文档](https://github.com/geeeeeeeeek/git-recipes/wiki/5.4-Git-%E9%92%A9%E5%AD%90%EF%BC%9A%E8%87%AA%E5%AE%9A%E4%B9%89%E4%BD%A0%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%B5%81)

&nbsp;

## 服务器部署

### 步骤

在服务上新建文件夹，并执行 git init

> 也可以 执行 git init --bare  ，但是这样生成的仓库没有工作区，上传不会显示文件

修改 config 中的设置

> 执行 **git config receive.denyCurrentBranch updateInstead命令
>
> 或者 直接在服务器的 **.git / config** 上直接修改

<p align="center" ><img src="/imgs/git_10.png" align=center></p>&nbsp;

克隆到本地

> git clone root@114.114.114.114:/www/git

* root 为 linux 账户
* 114.114.114.114 为服务器地址
* 后面加上**git仓库**目录

&nbsp;

## END


