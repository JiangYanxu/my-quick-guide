# Git手册

## 概念
> 五种状态:`Committed`已提交,`Staged`已暂存,`Modified`已更改,`Unmodified`未更改,`Untracked`未追踪
> 三个空间:`Working Directory`工作区,`Staging Area`暂存区,`.git directory(Repository)`仓库

## 获得帮助
### manual:`git help <verb>`
### 简短的:`git <verh> -h`

## 未成气候的命令
* `git init`:初始化,创建`.git`文件夹.
* `git clone <url> [DirectoryName]`:克隆url到./DirectoryName文件夹下.
* `git status`:查看当前目录状态.
  * `-s`or`--short`:以简介的方式更改,首列为暂存区,次列为工作区,`??`代表未追踪.
* `git add <FileName>:将未追踪状态或已更改状态的文件添加到暂存区.
* `git diff`:对比工作区与暂存区的内容差别
* `git diff --staged`:对比暂存区与仓库中最后一次提交的差别
  * `--cached`:与`--staged`同义词.
* `git commit [-m "Message!"]`:将暂存区提交到仓库成为一个新版本.
  * `-a`:跳过添加到暂存区步骤,直接从工作区提交到仓库.
* `git rm <file>`:删除工作区file文件,并且将对file的`未追踪状态`添加到暂存区.
  * 相当于: 1. `rm <file>`; 2. `git add <file>`. (2等价于`git rm <file>`)
  * `--cached`:保留工作区文件,将对file的`未追踪状态`添加到暂存区.
    * 即再不变动工作区的情况下,将暂存区的file标记为未追踪状态.
  * `-f`or`--force`:强制将对file的`未追踪状态`添加到暂存区(在暂存区该文件已经被更改后暂存的情况下,不加`-f`报错)
* `git mv <old_name> <new_name>`:改文件名操作.
  * 相当于: 1. `mv <old> <new>`; 2. `git rm <old>`; 3. `git add <new>`.实际上git不追踪改名,不过git能识别这三种操作的组合叫`renamed`.

## 待了解问题
1. `.gitignore`模板
2. glob模式
3. `git difftool`如何调用`emerge`或`vimdiff`软件,以及软件的使用方法. 

## `.gitignore`文件
* 注释以`#`开头
* 可以使用标准的`glob模式`
* 以`/`开头防止递归
* 以`/`结尾指定目录
* 模式前加`!`取反

> [`.gitignore1`模板](https://github.com/github/gitignore)

## git config
### 配置文件
配置文件的位置:
|Configuration File|Option|Instruction|
|--|--|--|
|`/etc/gitconfig`|`--system`|系统级配置文件|
|`~/.gitconfig`|`--global`|用户级配置文件|
|`.git/config`|`--local`(默认)|项目级配置文件|
> Option将决定`git config`的操作目标
> `git config -f <file>`显示给出操作目标

### 实用命令
* `git config --list --show-origin`:包含配置位置的配置列表
* `git config --global user.name "jiangyx"`:为用户级配置用户名
* `git config <key>`:查看一个key的Value

### 使用配置文件
|Key|Value Description|
|--|--|
|user.name|用户名|
|user.email|用户邮箱|
|core.editor|文本编辑器的可执行文件|
|log.date|local,`git log --date=<log.date>`详见[官方文档](https://git-scm.com/docs/git-log#Documentation/git-log.txt---dateltformatgt)|
