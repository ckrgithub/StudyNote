# 引言
* 第一章：介绍版本控制系统(VCSs)和Git的基本概念。什么是Git,为什么它会成为VCSs大家庭中的一员，它与其它VCSs的区别,以及为什么那么多人都在使用Git。
* 第二章：阐述Git的基本使用--包含你在使用Git时可能遇到的80%的情形。通过阅读本章，你应该能够克隆仓库、查看项目历史、修改文件和贡献更改。
* 第三章：关注于Git的分支模型。分支模型通常被认为Git的杀手级特性.
* 第四章：关于服务器端的Git
* 第五章：阐述多种分布式工作流的细节，以及如何使用Git实现它们.
* 第六章：介绍GitHub托管服务以及深层次的工具
* 第七章：关于Git的高级命令。如掌握可怕的“reset”命令，使用二分搜索识别错误，编辑历史，细节版本选择等.
* 第八章：关于Git环境的自定义配置，包括设置用于增强或促进自定义策略的hook脚本以及按照你所需要的方式进行工作的环境配置
* 第九章：对比Git和其它VCSs
* 第十章：深入Git阴暗而漂亮的实现细节.
## 分布式版本控制系统
客户端并不只是提取最新版本的文件快照，而是把代码仓库完整地镜像下来。这样下来，任何一处协同工作用的服务器发生故障，事后都可以用任何一个镜像出来的本地仓库恢复。因为每次clone操作，实际上都是一次对代码仓库的完整备份。
## Git基础
* 直接记录快照，而非差异比较
Git和其他版本控制系统的主要差别在于Git对待数据的方法。概念上区分，其他大部分系统以文件变更列表的方式存储信息，Git主要对当时的全部文件制作一个快照并保存这个快照的索引。为了高效，如果文件没有修改，Git不再重新存储该文件，而是只保留一个链接指向之前存储的文件。
* 近乎所有操作都是本地执行
浏览项目的历史，Git不需外连到服务器去获取历史，然后再显示出来--它只需直接从本地数据库中读取.  如果想看当前版本与一个月前的版本之间引入的修改，Git会查找到一个月前的文件做一次本地的差异计算。
* Git保证完整性
Git中所有数据哎存储前都计算校验和，然后以检验和来引用。Git用以计算校验和的机制叫做SHA-1散列(hash)。这是一个由40个十六进制字符组成字符串，基于Git中文件的内容或目录结构计算出来。
* Git一般只添加数据
你执行的Git操作，几乎只往Git数据库中增加数据.一旦你提交快照到Git中，就难以再丢失数据。
* 三种状态
已提交(committed)、已修改(modified)和已暂存(staged)。已提交：表示数据已经安全的保存在本地数据库中。已修改：表示修改了文件，但还没保存到数据中。已暂存：表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。
由此引入Git项目的三个工作区域的概念：Git仓库、工作目录以及暂存区域.
Git仓库目录：Git用来保存项目的元数据和对象数据库的地方。从其他计算器clone仓库时，拷贝的就是这里的数据.
工作目录：对项目的某个版本独立提取出来的内容。
暂存区域：是一个文件，保存了下次将提交的文件列表信息，一般在Git仓库目录中。
基本Git工作流：
1.在工作目录中修改文件。
2.暂存文件，将文件的快照放入暂存区域。
3.提交更新，找到暂存区域的文件，将快照永久性存储到Git仓库目录
* 命令行
只有在命令模式下你才能执行Git的所有命令，而大多数
* 安装Git
在linux上安装：可以使用yum: $ sudo yum install git
在Mac上安装：最简单的方法是安装Xcode Command Line Tools。
在Windows上安装：在Git官网下载Git for Windows的项目(也叫msysGit)
从源代码安装：需要安装Git依赖的库：curl、zlib、openssl、expat,还有libiconv.
### 初次运行Git前的配置
Git自带一个git config的工具来帮助设置控制Git外观和行为的配置变量。这些变量存储在三个不同的位置：
1./etc/gitconfig文件：包含系统上每一个用户及他们仓库的通用配置。如果带有--system选项的git config时，它会从此文件读写配置变量
2.~/.gitconfig或~/.config/git/config文件：只针对当前用户。可以传递--global选项让Git读写此文件。
3.当前使用仓库的Git目录中的config文件(就是.git/config)：针对该仓库
每个级别覆盖上一级别的配置，所以.git/config的配置变量会覆盖/etc/config中的配置变量
在Window系统，Git会查找$HOME目录下的.gitconfig文件。Git同样也会寻找/etc/gitconfig文件，但只限于MSys的根目录，即安装Git时所选的目标位置。
* 用户信息
当安装完Git应该做的第一件事就是设置你的用户名称与邮件地址。这样做很重要，因为每个Git的提交都会使用这些信息，并且它会写入到你的每次提交中。
```
  $ git config --global user.name "ckr"
  $ git config --global user.email ckr@google.com
```
如果使用了--global选项，那么该命令只需要运行一次，因为之后无论你在该系统上做任何事，Git都会使用那些信息。当你想针对特定项目使用不同的用户名称与邮件地址时，可以在那个项目目录下运行没有--global选项的命令来配置
* 文本编辑器
当Git需要你输入信息时会调用它，如果没配置，使用操作系统默认的文本编辑器，通常是Vim。
```
  $ git config --global core.editor "code --wait"
```
* 检查配置信息
使用git config --list命令列出所有Git当时能找到的配置
* 获取帮助
```
  $ git help <verb>
  $ git <verb> --help
  $ man git-<verb>
  $ git help config//获取config命令的手册
```
# Git基础
## 获取Git仓库
有两种取得Git项目仓库方法。第一种：在现有项目或目录下导入所有文件到Git中；第二种：从一个服务器克隆一个现有的Git仓库
* 在现有目录中初始化仓库
```
  $ git init
```
该命令将创建名为.git的子目录，这个子目录含有初始化的Git仓库中所有的必须文件，这些文件是Git仓库的骨干.
如果你是在一个已经存在文件的文件夹(而不是空文件夹)中初始化Git仓库来进行版本控制的话，你应该开始跟踪这些文件并提交。
```
  $ git add *.c
  $ git add LICENSE
  $ git commit -m 'init project version'
```
* clone现有的仓库
Git clone是该Git仓库服务器上的几乎所有数据，而不是仅仅复制完成你的工作所需要文件。如：
```
  $ git clone https://github.com/ckrgithub/StudyNote
```
这会在当前目录下创建一个名为"StudyNote"的目录，并在这个目录下初始化一个.git文件夹,从远程仓库拉取下所有数据放入.git文件夹，然后从中读取最新版本的文件的拷贝。如果你想在clone远程仓库时，自定义本地仓库名字：
```
  $ git clone https://github.com/ckrgithub/StudyNote MyNote
```
## 记录每次更新到仓库
已跟踪的文件：那些纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后，它们的状态可能处于未修改，已修改或放入暂存区。
为跟踪文件：除了已跟踪文件外的所有其他文件。
* 检查当前文件状态
```
  $ git status
  On branch master
  nothing to commit, working directory clean
```
在项目创建一个新的readme文件。
```
  $ echo 'StudyNote' > README
  $ git status
  On branch master
  Untracked files:
    (use "git add <file>..." to include in what will be committed)
      README
  nothing added to commit but untracked files present (use "git add" to track)
```
可以看到，新建的readme文件出现在Untracked files下面。
* 跟踪新文件
```
  $ git add README
  $ git status
  On branch master
  Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)
    new file: README
```
只要在Changes to be committed这行下面，就说明是已暂存状态。
* 暂存已修改文件
如果你修改了一个名为contributing.md的已被跟踪的文件，然后运行git status命令：
```
  $ git status
  On branch master
  Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)
      new file: README
  Changes not stagged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git checkout --<file>..." to discard changes in working directory)
      modified:  contributing.md
```
出现在Changes not staged for commit下面，说明已跟踪文件的内容发生了变化，但还没放到暂存区。运行git add命令：添加内容到下一次提交中，而不是讲一个文件添加到项目中要更合适。现在运行git add后：
```
  $ git add contributing.md
  $ git status
  On branch master
  Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)
      new file: README
      modified: contributing.md
```
再次在contributing.md里加注释，运行git status：
```
  $ vim contributing.md
  $ git status
  On branch master
  Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)
      new file:  README
      modified:  contributing.md
  Changes not stagged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git checkout --<file>..." to discard changes in working directory)
      modified:  contributing.md
```
contributing.md文件同时出现在暂存区和非暂存区。实际上Git只暂存了你运行git add命令时的版本，如果你现在提交，contributing.md版本是你最后一次运行git add命令时的版本，而不是你运行git commit时，在工作目录中的当前版本。
* 状态简览
git status 命令的输出十分详细，但其用语有些繁琐。使用git status -s:
```
  $ git status -s
    M READNE
   MM Rakefile
   A  lib/git.rb
   M  lib/simplegit.rb
   ?? LICENSE.txt
```
新添加的未跟踪文件前面有??标记，新添加到暂存区中的文件前面有A标记，修改过的文件前面有M标记。
* 忽略文件
创建一个名为.gitignore的文件，列出要忽略的文件模式，如：
```
  $ cat .gitignore
  *.[oa]    //git忽略所有以.o或.a结尾的文件。
  *~        //Git忽略所有以波浪符结尾的文件
```
文件.gitignore的格式规范如下：
  所有空行或者以#开头的行都会被Git忽略
  可以使用标准的glob模式匹配
  匹配模式可以以(/)开头防止递归
  匹配模式可以以(/)结尾指定目录
  要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号(!)取反
所谓的glob模式是指shell所使用的简化了的正则表达式。*:匹配0个或多个任意字符；[abc]:匹配任何一个列在方括号中的字符；?:只匹配一个任意字符；[0-9]:匹配所有0到9的数字；**:匹配任意中间目录，如'a/**/z'可以匹配a/z,a/b/z等
我们在看一个.gitignore文件例子：
```
  # no .a files
  *.a
  # but do track lib.a,even though you are ignoring .a files above
  !lib.a
  # only ignore the TODO file in the current directory, not subdir/TODO
  /TODO
  # ignore all files in the build/ directory
  build/
  # ignore doc/notes.txt,but not doc/server/arch.txt
  doc/*.txt
  # ignore all .pdf files in the doc/ directory
  doc/**/*.pdf
```

