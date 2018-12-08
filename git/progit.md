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
* 查看已暂存和未暂存的修改
git diff将通过文件补丁的格式显示具体哪些行为发生了改变。如：再次修改readme文件后暂存，然后编辑contributing.md文件后先不暂存，运行status命令：
```
  $ git status
  On branch master
  Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)
      modified: readme
  Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git checkout --<file>..." to discard changes in working directory)
      modified: contributing.md
```
要查看尚未暂存的文件更新了哪些内容：
```
  $ git diff
  diff --git a/contributing.md b/contributing.md
  index 8ebb991..643e24f 100644
  --- a/contributing.md
  +++ b/contributing.md
  @@ -65,7 +65,8 @@ branch directly, things can get messy.
    Please include a nice description of your changes when you submit your PR;
    if we have to read the whole diff to figure out why you are contributing in the first place,you are less likely to get feedback and have your change
  -merged in.
  +merged in. Also,split your changes into comprehensive chunks if your patch is longer than a dozen lines.
  If you are starting to work on a particular area,feel free to submit a PR that highlights your work in progress (and not in the PR title that it is)
```
此命令比较的是工作目录中当前文件和暂存区域快照之间的差异，也就是修改之后还没有暂存起来的变化内容。若要查看已暂存的将要添加到下次提交了的内容，可以用git diff --cached 命令。(Git1.6.1以上版本允许使用git diff --staged，效果一样)
```
  $ git diff --staged
  diff --git a/readme b/readme
  new file mode 100644
  index 0000000..03092a1
  --- /dev/null
  +++ b/readme
  @@ -0,0 +1 @@
  +My Project
```
* 提交更新
请一定要确认还有什么修改过的或新建的文件还没有git add过，否则提交的时候不会记录这些还没暂存起来的变化。所以，每次提交前，先用git status查看，然后
在运行git commit,添加-m选项，将提交信息与命令放在同一行。
```
  $ git commit -m "Story 182: Fix a bug"
  [master 463dc4f] Story 182: Fix a bug
    2 files changed,2 insertions(+)
    create mode 100644 readme
```
* 跳过使用暂存区域
git commit -a 命令会自动把所有已跟踪过的文件暂存起来一并提交：
```
  $ git status
  On branch master
  Changes not stagged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git checkout -- <file>..." to discard changes in working directory)
      modified: contributing.md
  no changes add to commit (use "git add" and/or "git commit -a")
  
  $ git commit -a -m 'added new benchmarks'
  [master 83e36c7] added new benchmarks
    1 file changed, 5 insertions(+), 0 deletions(-)
```
* 移除文件
要从Git移除文件，必须从已跟踪文件清单中移除(从暂存区移除),然后提交。可以使用git rm命令完成此项工作，并连带从工作目录中删除指定的文件。
如果手工删除文件，运行git status时就会在"Changes not staged for commit"部分看到：
```
  $ rm readme.md
  $ git status
  On branch master
  Your branch is up-to-date with 'origin/master'.
  Changes not staged for commit:
    (use "git add/rm <file>..." to update what will be committed)
    (use "git checkout -- <file>..." to discard changes in working directory)
      deleted: readme.md
  no changes added to commit (use "git add" and/or "git commit -a")
```
然后再运行git rm记录此次移除文件的操作：
```
  $ git rm readme.md
  rm 'readme.md'
  $ git status
  On branch master
  Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)
      deleted: readme.md
```
如果删除之前修改过并已经放到暂存区域的话，则必须要强制删除选项-f.这是一种安全特性，用于防止误删还没有添加到快照的数据，这样的数据不能被Git恢复.
另一种情况，想把文件从Git仓库中删除，但希望保留在当前工作目录中。当你忘记添加.gitignore文件，不小心把一个很大的日志文件或一推.a这样编译生成文件添加到暂存区时，这一做法尤其有用。使用--cached选项：
```
  $ git rm --cached readme
```
git rm 命令后面可以列出文件或目录的名字，也可以使用glob模式:
```
  $ git rm log/\*.log
```
注意到星号之前的反斜杠,因为Git有他自己的文件模式。此命令删除log/目录下扩展名为.log的所有文件。类似的比如：
```
  $ git rm \*~   //删除以~结尾的所有文件
```
* 移动文件
要在Git中对文件改名：
```
  $ git mv file_from file_to
```
```
  $ git mv readme.md readme
  $ git status
  On branch master
  Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)
      renamed: readme.md -> readme
```
其实，运行git mv相当于运行了三条命令：
```
  $ mv readme.md readme
  $ git rm readme.md
  $ git add readme
```
## 查看提交历史
```
  $ git log
  commit ca82a6dff817ec66f44342007202690a93763949
  Author: ckrgithub
  Date: Sun Dec 28 11:14:00 2018
    changed the version number
    
  commit a11bef06a3f659402fe7563abf99ad00de2209e6
  Author: ckrgithub
  Date: Sun Dec 28 11:13:00 2018
    first commit
```
git log会按提交时间列出所有的更新，最近更新排在最上面。git log 有许多选项：一个常用的选项是-p,用来显示每次提交的内容差异.你也可以加上-1来仅显示最近两次提交：
```
  $ git log -p -1
  commit ca82a6dff817ec66f44342007202690a93763949
  Author: ckrgithub
  Date: Sun Dec 28 11:14:00 2018
    changed the version number
  diff --git a/Rakefile b/Rakefile
  index a874b73..8f94139 100644
  --- a/Rakefile
  +++ b/Rakefile
  @@ -5 7 +5 6 @@ require 'rake/gempackagetask'
     spec = Gem::Specification.new do |s|
    s.platform = Gem::Platform::RUBY
    s.name = "simplegit"
  - s.version = "0.1.0"
  + s.version = "0.1.1"
    s.author = "Scott Chacon"
    s.email = "schacon@gee-mail.com"
    s.summary = "A simple gem for using Git in Ruby code."
```
使用--stat选项：在每次提交的下面列出所有被修改过的文件、有多少文件被修改了以及被修改过的文件的哪些行被移除或是添加了。
```
  $ git log --stat
  commit ca82a6dff817ec66f44342007202690a93763949
  Author: ckrgithub
  Date: Sun Dec 28 11:14:00 2018
    changed the version number
  Rakefile | 2 +-
  1 file changed, 1 insertion(+), 1 deletion(-)
  
```
使用 --pretty :指定使用不同于默认格式的方式展示提交历史。这个选项有一些内径的子选项，如oneline将每个提交放在一行显示
```
  $ git log --pretty=online
  ca82a6dff817ec66f44342007202690a93763949 changed the version number
  a11bef06a3f659402fe7563abf99ad00de2209e6 first commit

```
使用 format：定制要显示的记录格式
```
  $ git log --pretty=format:"%h - %an, %ar : %s"
  ca82a6d - Scott Chacon, 6 years ago : changed the version number
  085bb3b - Scott Chacon, 6 years ago : removed unnecessary test
  a11bef0 - Scott Chacon, 6 years ago : first commit
```
|选项|说明|
|---|---|
|%H|commit的完整hash值|
|%h|commit的简短hash值|
|%T|tree的完整hash值|
|%t|tree的简短hash值|
|%P|parent的完整hash值|
|%p|parent的简短hash值|
|%an|作者的名字|
|%ae|作者的电子邮件|
|%ad|作者修订日期|
|%ar|作者修订日期，按多久以前的方式显示|
|%ch|提交者的名字|
|%ce|提交者的电子邮件|
|%cd|提交日期|
|%cr|提交日期，按多久以前的方式显示|
|%s|提交说明|

但你为某个项目发布补丁，然后某个核心成员将你补丁并入项目时，你就是作者，而那个核心成员就是提交者。
当oneline或format与另一个log选项--graph结合使用时尤其有用
```
  $ git log --pretty=format:"%h %s" --graph
  * 2d3acf9 ignore errors from SIGCHLD on trap
  * 5e3ee11 Merge branch 'master' of git://github.com/dustin/grit
  |\
  | * 420eac9 Added a method for getting the current branch.
  * | 30e367c timeout code and tests
  * | 5a09431 add timeout protection to grit
  * | e1193f8 support for heads with slashes in them
  |/
  * d6016bc require time for xmlschema
  * 11d191e Merge branch 'defunkt' into local
```
git log命令支持选项  
|选项|说明|
|---|---|
|-p|按补丁格式显示每个更新之间的差异|
|--stat|显示每次更新的文件修改统计信息|
|--shortStat|只显示--stat中最后的行数修改添加移除统计|
|--name-only|仅在提交信息后显示已修改的文件清单|
|--name-status|显示新增、修改、删除的文件清单|
|--abbrev-commit|仅显示SHA-1的前几个字符，而非所有的40个字符|
|--relative-date|使用较短的相对时间显示(如："2 weeks ago")|
|--graph|显示ASCII图形表示的分支合并历史|
|--pretty|使用其他格式显示历史提交信息|

* 限制输出长度
```
  $ git log --since=2.weeks  //列出最近两周的提交
```
用--autor选项显示指定作者的提交，用--grep选项搜索提交说明中的关键字。(要得到同时满足两个选项搜素条件的提交，必须用--all-match选项。否则，满足任意一个条件的提交都会被匹配出来).用-S可以列出那些添加或移除了某些字符串的提交。如，想找出添加或移除某个特定函数的引用的提交：
```
  $ git log -Sfunction_name
```
用path：指定某些文件或目录的历史提交。
|选项|说明|
|----|----|
|-(n)|仅显示最近的n条提交|
|--since,--after|仅显示指定时间之后的提交|
|--until,--before|仅显示指定时间之前的提交|
|--author|仅显示指定作者相关的提交|
|--committer|仅显示指定提交者相关的提交|
|--grep|仅显示含指定关键字的提交|
|-S|仅显示添加或移除某个关键字的提交|
