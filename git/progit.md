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
  * 1.在工作目录中修改文件。  
  * 2.暂存文件，将文件的快照放入暂存区域。  
  * 3.提交更新，找到暂存区域的文件，将快照永久性存储到Git仓库目录
* 命令行  
只有在命令模式下你才能执行Git的所有命令，而大多数
* 安装Git  
在linux上安装：可以使用yum: $ sudo yum install git  
在Mac上安装：最简单的方法是安装Xcode Command Line Tools。  
在Windows上安装：在Git官网下载Git for Windows的项目(也叫msysGit)  
从源代码安装：需要安装Git依赖的库：curl、zlib、openssl、expat,还有libiconv.  
### 初次运行Git前的配置(git config)
Git自带一个git config的工具来帮助设置控制Git外观和行为的配置变量。这些变量存储在三个不同的位置：  
  * 1./etc/gitconfig文件：包含系统上每一个用户及他们仓库的通用配置。如果带有--system选项的git config时，它会从此文件读写配置变量  
  * 2.'~/.gitconfig'或'~/.config/git/config'文件：只针对当前用户。可以传递--global选项让Git读写此文件。    
  * 3.当前使用仓库的Git目录中的config文件(就是.git/config)：针对该仓库    
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
## 获取Git仓库(git init/clone)
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
## 记录每次更新到仓库(git status/add/commit/diff)
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
  * 所有空行或者以#开头的行都会被Git忽略
  * 可以使用标准的glob模式匹配
  * 匹配模式可以以(/)开头防止递归
  * 匹配模式可以以(/)结尾指定目录
  * 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号(!)取反
所谓的glob模式是指shell所使用的简化了的正则表达式。*:匹配0个或多个任意字符；[abc]:匹配任何一个列在方括号中的字符；?:只匹配一个任意字符；[0-9]:匹配所有0到9的数字；`**`:匹配任意中间目录，如'a/`**`/z'可以匹配a/z,a/b/z等  
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
## 查看提交历史(git log)
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

* 撤销操作
提交完了才发现漏掉几个文件没有添加，或者提交信息写错了。此时可以带有--amend选项的提交命令尝试重新提交：
```
  $ git commit --amend
```
这个命令会将暂存区中的文件提交。如果自上次提交以来你还未做任何修改，那么快照会保持不变，而你修改的只是提交信息。如：
你提交后发现忘记了暂存某些需要的修改：
```
  $ git commit -m 'init commit'
  $ git add forgotten_file
  $ git commit --amend
```
最终你只会有一个提交-第二次提交将代替第一次提交的结果
* 取消暂存的文件
你已修改了两个文件并想要将它们作为两次独立的修改提交，但却意外地输入了git add * 暂存了它们两个。如何只取消暂存两个钟的一个呢？
```
  $ git add * 
  $ git status
  On branch master
  Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)
      renamed: readme.md -> readme
      modified: contributing.md
```
提示使用git reset HEAD <file>...来取消暂存：
```
  $ git reset HEAD contributing.md
  $ git status
  On branch master
  Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)
      renamed: readme.md -> readme
  Changes not stagged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git checkout --<file>..." to discard changes in working directory)
      modified: contributing.md
```
虽然加上--hard选项可以令git reset成为一个危险命令(可能导致工作目录中所有当前进度丢失)，但本例中工作目录内的文件并不会修改。
* 撤销对文件的修改
如果不想保留对contributing.md文件的修改，使用git checkout命令：
```
  $ git checkout --contributing.md
  $ git status
  On branch master
  Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)
      renamed: readme.md -> readme
```
## 远程仓库的使用(git remote/fetch/push)
* 查看远程仓库
git remote：会列出你指定的每个远程服务器的简写。
```
  $ git clone https://github.com/ckrgithub/StudyNote
  $ cd StudyNote
  $ git remote
  origin
```
指定选项-v:显示需要读写远程仓库使用的Git保存的简写与其对应的URL
```
  $ git remote -v
  origin https://github.com/ckrgithub/StudyNote (fetch)
  origin https://github.com/ckrgithub/StudyNote (push)
```
* 添加远程仓库
git remote add <shortname> <url>添加一个新的远程Git仓库
```
  $ git remote add gt https://github.com/ckrgithub/GitNote
  $ git remote -v
  origin https://github.com/ckrgithub/StudyNote (fetch)
  origin https://github.com/ckrgithub/StudyNote (push)
  gt https://github.com/ckrgithub/GitNote (fetch)
  gt https://github.com/ckrgithub/GitNote (push)
```
使用gt代替整个url。如：拉取信息
```
  $ git fetch gt
  remote: Counting objects: 43, done.
  remote: Compressing objects: 100% (36/36), done.
  remote: Total 43 (delta 10), reused 31 (delta 5)
  Unpacking objects: 100% (43/43), done.
  From https://github.com/ckrgithub/GitNote
   * [new branch] master -> gt/master
```
* 从远程仓库中抓取与拉取
```
  git fetch [remote-name]
```
该命令访问远程仓库，从中拉取所有你还没有的数据。执行完后，你将会拥有那个远程仓库总所有分支的引用，可以随时合并或查看。
git fetch命令将数据拉取到你的本地仓库-它不会自动合并或修改你当前的工作。git pull命令会自动抓取然后合并远程分支到当前分支。
* 推送到远程仓库
git push [remote-name] [branch-name].当你想要将master分支推送到origin服务器时(再次说明，clone时通常会自动帮你设置好这两个名字)
```
  $ git push origin master
```
只有当你有所clone服务器的写入权限，并之前没有人推送过时，这条命令才能生效。当其他人推送到上游后你再推送到上游，你的推送会被拒绝。你必须先将他们的工作拉取下来并将其合并进你的工作后才能推送。
* 查看远程仓库
git remote show [remote-name]:
```
  $ git remote show origin
  * remote origin
    Fetch URL: https://github.com/ckrgithub/StudyNote
    Push URL: https://github.com/ckrgithub/StudyNote
    HEAD branch: master
    Remote branches:
      master      tracked
      dev-branch  tracked
    Local branch configured for 'git pull'
      master merges with remote master
    Local ref configured for 'git push'
      master pushes to master (up to date)
```
* 远程仓库的移除与重命名
git remote rename去修改一个远程仓库的简写名
```
  $ git remote rename gt paul
  $ git remote
  origin
  paul
```
这同样也会修改你的远程分支名字，过去引用gt/master的现在会引用paul/master。
git remote rm移除一个远程仓库
```
  $ git remote rm paul
  $ git remote
  origin
```
## 打标签(git tag)
* 列出标签
```
  $ git tag
  v0.1
  v1.3
```
指定特定的模式查找标签
```
  $ git tag -l 'v1.8.5*'
  v1.8.5
  v1.8.5-rc0
  v1.8.5.1
```
* 创建标签
轻量标签(lightweight):很像一个不会改变的分支-它只是一个特定提交的引用
附注标签(annotated)：存储在Git数据库中的一个完整对象。
* 附注标签
指定-a选项：
```
  $ git tag -a v1.4 -m 'my version 1.4'
  $ git tag
  v0.1
  v1.3
  v1.4
```
-m选项指定：一条将会存储在标签中的信息.通过git show命令可以看到标签信息与对应的提交信息：
```
  $ git show v1.4
  tag v1.4
  Tagger: ckrgithub
  Date:   Sat Dec 8 15:43:00 2018 -0700
  my version 1.4
  
  commit  ca82a6dff817ec66f44342007202690a93763949
  Tagger: xxx
  Date:   Sat Dec 8 15:46:00 2018 -0700
    changed the version number
```
* 轻量标签
本质上是将提交校验和存储到一个文件中-没有保存任何其他信息
```
  $ git tag v1.4-ckr
  $ git tag
  v0.1
  v1.3
  v1.4-ckr
```
使用git show不会显示额外标签信息：
```
  $ git show v1.4-ckr
  commit  ca82a6dff817ec66f44342007202690a93763949
  Tagger: xxx
  Date:   Sat Dec 8 15:46:00 2018 -0700
    changed the version number
```
* 后期打标签
你可以对过去的提交打标签。如
```
  $ git log --pretty=oneline
  15027957951b64cf874c3557a0f3547bd83b3ff6 Merge branch 'experiment'
  a6b4c97498bd301d84096da251c98a07c7723e65 beginning write support
  0d52aaab4479697da7686c15f77a3d64d9165190 one more thing
  6d52a271eda8725415634dd79daabbc4d9b6008e Merge branch 'experiment'
  0b7434d86859cc7b8c3d5e1dddfed66ff742fcbc added a commit function
  4682c3261057305bdd616e23b64b0857d832627b added a todo file
  166ae0c4d3f420721acbb115cc33848dfcc2121a started write support
  9fceb02d0ae598e95dc970b74767f19372d61af8 updated rakefile
  964f16d36dfccde844893cac5b347e7b3d44abbc commit the todo
  8a5cbc430f1a9c3d00faaeffd07798508422908a updated readme
```
现在假设在v1.2时忘记了给项目打标签，也就是在'updated rakefile'提交。你可以在之后补上标签。
```
  $ git tag -a v1.2 9fceb02
  $ git tag
  v0.1
  v1.2
  v1.3
  v1.4-ckr
```
* 共享标签
默认情况下，git push命令并不会传送标签到远程仓库服务器上。在创建完标签后你必须显示地推送标签到共享服务器上。
```
  $ git push origin v1.5
```
一次性推送很多标签，可以使用--tags选项的git push命令。
```
  $ git push origin --tags
```
* 检出标签
在Git中并不能真的检出一个标签，因为它们并不能像分支一样来回移动。使用git checkout -b [branchname][tagname]在特定的标签创建一个新分支：
```
  $ git checkout -b version2 v2.0.0
  Switched to a new branch 'version2'
```
## Git 别名
Git并不会在你输入部分命令时自动推断出你想要的命令。如果不想每次都输入完整的Git命令，可以通过git config文件轻松地为每个命令设置别名。
```
  $ git config --global alias.co checkout
  $ git config --global alias.br branch
  $ git config --global alias.ci commit
  $ git config --golbal alias.st status
```
当输入git commit时，只需要输入git ci。在创建你认为存在的命令时这个技术会很有用。
```
  $ git config --global alias.unstage 'reset HEAD --' //会等于下面两个命令
  
  $ git unstage fileA
  $ git reset HEAD -- fileA
```
# Git分支
Git保存的不是文件的变化或者差异，而是一系列不同时刻的文件快照。在进行提交操作时，Git会保存一个提交对象。该提交对象包含一个指向暂存内容快照的指针。首次提交产生的提交对象没有父对象，普通提交操作产生的提交对象有一个父对象，而由多个分支合并产生的提交对象有多个父对象。
假设现在有一个工作目录，里面包含了三个将要被暂存和提交的文件。暂存操作会为每个文件计算校验和，然后会把当前版本的文件快照保存到Git仓库中(Git使用blob对象来保存它们)，最终将校验和加入到暂存区域等待提交：
```
  $ git add README test.txt LICENSE
  $ git commit -m 'init'
```
当使用git commit进行提交操作时，Git会先计算每个子目录的校验和，然后在Git仓库中这些校验和保存为树对象。随后，Git便会创建一个提交对象，它除了包含上面提到的那些信息外，还包含指向这个树对象(项目根目录)的指针。如此一来，Git就可以在需要的时候重现此次保存的快照。现在，Git仓库中有五个对象：三个blob对象(保存着文件快照)、一个树对象(记录着目录结构和blob对象索引)以及一个提交对象(包含着指向前述树对象的指针和所有提交信息)。  

做些修改后再次提交，那么这次生产的提交对象会包含一个指向上次提交对象(父对象)的指针。Git的分支，其实本质上仅仅是指向提交对象的可变指针。Git的默认分支名字是master。在多次提交操作之后，你其实已经有一个指向最后那个提交对象的master分支。它会在每次的提交操作中自动向前移动。  
* 提示：Git的master分支并不是一个特殊分支。它跟其他分支完全没有区别。之所以几乎每个仓库都有master分支，是因为git init命令默认创建它，并且大多数人懒得去改动它。
## 分支创建
```
 $ git branch dev  //会在当前所在的提交对象上创建一个指针
```
在Git中，HEAD是一个指针，指向当前所在的本地分支(注：将HEAD想象为当前分支的别名)  
使用git log命令查看各个分支当前所指的对象
```
 $ git log --oneline --decorate
 f30ab (HEAD, master, dev) add feature #32 - ability to add new 
 34ac2 fixed bug #1328 - stack overflow under certain conditions
 98ca9 initial commit of my project
```
正如你所见，当前master和dev分支均指向校验和以f30ab开头的提交对象。
## 分支切换
```
 $ git checkout dev
```
这样HEAD就指向dev分支了。这样的实现方式会给我们带来什么好处：
```
 $ vim test.txt
 $ git commit -a -m 'made a change'
```
如图所示，你的dev分支向前移动，但master分支没有，它任然指向运行git checkout时所指的对象。我们切换回master分支看看：
```
 $ git checkout master
```
这条命令做了两件事。一是使HEAD指回master分支，二是将工作目录恢复成master分支所指向的快照内容。也就是说，你现在做修改的话，项目将始于一个较旧的版本。本质上来讲，这就是忽略dev分支所做的修改，以便于向另一个方向进行开发。
* 注意：分支切换会改变你工作目录中的文件。在切换分支时，一定要注意你工作目录里的文件会被改变。如果Git不能干净利落地完成这个任务，它将禁止切换分支
```
 $ vim text.txt
 $ git commit -a -m 'made other changes'
```
这个项目的提交历史已经产生了分叉。因为刚才你创建了一个新分支，并切换过去进行了一些工作，随后又切换会master分支进行了另外一些工作。上述两次改动针对的是不同分支：你可以在不同分支间不断地来回切换和工作。可以使用git log查看分叉历史
```
 $ git log --oneline --decorate --graph --all
 * c2b9e (HEAD, master) made other changes
 | * 87ab2 (dev) made a change
 |/
 * f30ab add feature #32 - ability to add new formats to the
 * 34ac2 fixed bug #1328 - stack overflow under certain conditions
 * 98ca9 initial commit of my project
```
Git分支实质上仅是包含所指对象检验和的文件，所以它的创建和销毁都异常高效。创建一个新分支就相当于往一个文件中写入41个字节(40个字符和1个换行符).
## 分支的新建与合并
案例：
    1.开发某个网站
    2.为实现某个新的需求，创建一个分支
    3.在这个分支上开展工作
  正在此时，接到一个电话说有个很严重的问题需要紧急修补：
    1.切换到你的线上分支。
    2.为这个紧急任务新建一个分支，并在其中修复它
    3.在测试通过之后切换回线上分支，然后合并这个修补分支，最后将改动推送到线上分支
    4.切换回你最初工作的分支上，继续工作。
### 1.新建分支
你已经决定解决你的公司使用的问题#53问题，新建一个分支并同时切换到那个分支上
```
 $ git checkout -b iss53
 Switched to a new branch "iss53"
```
它是下面两条命令的简写：
```
 $ git branch iss53
 $ git checkout iss53
```
继续在#53问题上工作，并做了些提交。
```
 $ vim index.html
 $ git commit -a -m 'added a new footer'
```
现在你接到那个电话，有个紧急问题等你来解决。此时，留意你的工作目录和暂存区里那些还没有被提交的修改，它可能会和你即将检出的分支产生冲突从而阻止Git切换到该分支。
```
 $ git checkout master
 Switched to branch 'master'
 $ git checkout -b hotfix
 Switched to a new branch 'hotfix'
 $ vim index.html
 $ git commit -a -m 'fixed the broken email address'
 [hotfix 1fb7853] fixed the broken email address
  1 file changed, 2 insertions(+)
```
合并回你的master分支来部署到线上。
```
 $ git checkout master
 $ git merge hotfix
 Updating f42c576...3a0874c
 Fast-forward
  index.html | 2 ++
  1 file changed, 2 insertions(+)
```
由于当前master分支所指向的提交是你当前提交(有关hotfix的提交)的直接上游，所以Git只是简单的将指针向前移动。换句话说，当你试图合并两个分支时，如果顺着一个分支走下去能够到达另一个分支，那么Git在合并两者的时候，只会简单的将指针向前推进(指针右移)，因为这种情况下的合并操作没有需要解决的分歧--这就叫做"快进(fast-forward)".  
关于紧急问题解决之后，准备回到被打断之前时的工作中。然而，你应该先删除hotfix分支，因为你已经不再需要它了--master分支已经指向了同一个位置。
```
 $ git branch -d hotfix
 Deleted branch hotfix (3a0874c)
 $ git checkout iss53
 Switched to branch "iss53"
 $ vim index.html
 $ git commit -a -m 'finished the new footer'
 [iss53 ad82d7a] finished the new footer
  1 file changed, 1 insertion(+)
```
### 2.分支的合并
打算将你的工作合并入master分支。为此，你需要合并iss53分支到master分支
```
 $ git checkout master
 Switched to branch 'master'
 $ git merge iss53
 Merge made by the 'recursive' strategy
 index.html |  1+
 1 file changed, 1 insertion(+)
```
这和之前合并hotfix分支的时候看起来不一样。这种情况下，你的开发历史从一个更早的地方开始分叉开来(diverged).因为，master分支所在提交并不是iss53分支所在提交的直接祖先。Git将此次三方合并的结果做了一个新的快照并自动创建一个新的提交指向它。这个被称作一次合并提交，它的特别之处在于他有不止一个父提交。
### 遇到冲突时的分支合并
如果你在两个不同分支中，对同一个文件的同一个部分进行了不同的修改，Git就没法干净的合并它们。如果你对#53问题的修改和有关hotfix的修改都涉及到同一个文件的同一处，在合并它们的时候就会产生合并冲突：
```
 $ git merge iss53
 Auto-merging index.html
 CONFLICT (content): Merge conflict in index.html
 Automatic merge failed; fix conflicts and then commit the result.
```
此时Git做了合并，但没有自动地创建一个新的合并提交。Git会暂停下来，等待你去解决合并产生的冲突。
```
 $ git status
 On branch master
 You have unmerged paths.
   (fix conflixts and run "git commit")
   
 Unmerged paths:
   (use "git add <file>..." to mark resolution)
     both modified:   index.html
 no changes added to commit (use "git add" and/or "git commit -a")
```
任何因包含合并冲突而有待解决的文件，都会以未合并状态标识出来。Git会在有冲突的文件中加入标准的冲突解决标记，这样你可以打开这些包含冲突的人家然后手动解决冲突。
```
  <<<<<< HEAD:index.html
 <div id="footer">contact : email.support@github.com</div>
 ========
 <div id="footer">
   please contact us at support@github.com
 </div>
 >>>>>> iss53:index.html
```
这表示HEAD所指示的版本(也就是你的master分支所在的位置)在这个区段的上半部分(=====的上半部分)，而iss53分支所指示的版本在====的下半部分
为了解决冲突，你必须选择使用有=====分割的两部分中的一个，或者你自行合并这些内容。
```
  <div id="footer">
    please contact us at email.support@github.com
  </div>
```
```
  $ git status
  On branch master
  All conflicts fixed but you are still merging.
    (use "git commit" to conclude merge)
  Changes to be committed:
      modified:  index.html
```
## 分支管理
git branch命令不只是可以创建与删除分支。
```
  $ git branch  //得到当前所有分支列表
    iss53
  * master
    testing
```
注意master分支前的``*``字符：代表当前HEAD指针所指向的分支。如果需要查看每个分支的最后一次提交，可以运行git branch -v 命令：
```
  $ git branch -v
    iss53    93b412c fix javascript issue
  * master   7a98805 Merge branch 'iss53'
    testing  782fd34 add scott to the autor list in the readmes
```
--merged与--no-merged两个有用的选项可以过滤这个列表中已经合并或尚未合并到当前分支的分支、
```
  $ git branch --merged
    iss53
  * master
  
  $ git branch --no-merged
    testing
```
### 分支开发工作流
#### 长期分支
只在master分支上保留完全稳定的代码--有可能仅仅是已经发布或即将发布的代码。还有名为develop或者next的平行分支，被用来做后续开发或者测试稳定性--这些分支不必保持绝对稳定，但一旦达到稳定状态，它们就可以被合并入master分支了。这样，在确保这些已完成的特性分支(短期分支，如：iss53分支)能够通过所有测试，并且不会引入更多bug之后，就可以合并入主干分支中，等待下次发布。
#### 特性分支
特性分支是一种短期分支，它被用来实现单一特性或其相关工作。
#### 远程分支
远程引用是对远程仓库的引用(指针)，包括分支、标签等等。可以通过git ls-remote (remote) 来显示地获得远程引用的完整列表，或者通过git remote show (remote)获得远程分支的更多信息。然而，最常用的是利用远程跟踪分支。  
远程跟踪分支是远程分支状态的引用。它们是不能移动地本地引用当你做任何网络通信操作时，它们会自动移动。它们以(remote)/(branch)形式命名。如：想要看你最后一次与远程仓库origin通信时master分支的状态，你可以查看origin/master分支。你与同事解决一个问题并且他们推送一个iss53分支，你可能有自己的本地iss53分支；但服务器上的分支会指向origin/iss53的提交。假设，你clone [这个项目](https://github.com/ckrgithub/StudyNote)，Git会为你自动将其命名为origin，拉取它的所有数据，创建一个指向它的master分支的指针，并在本地将其命名为origin/master。Git也会给你一个与origin的master分支在指向同一个地方的本地master分支.  
* 注意：origin并无特殊含义。master是当你运行git init时默认的起始分支名字。origin是当你运行git clone时默认的远程仓库名字。如果你运行git clone -o ckr,那么你默认的远程分支名字将会是ckr/master  
你在本地master分支做了些工作，然只要不与origin服务器链接，你的origin/master指针就不会移动.如果要同步你的工作，运行git fetch origin命令，从中抓取本地没有的数据，并更新本地数据库，移动origin/master指针指向新的、更新后的位置。
* 多个远程仓库与远程分支  
假定你有另一个内部Git服务器，仅用于你的sprint小组的开发工作。这个服务器位于git.team1.ourcompany.com。你可以运行git remote add命令添加一个新的远程仓库引用到当前的项目，将这个远程仓库命名为teamone，将其作为整个url的缩写.  
现在，可以运行git fetch teamone 来抓取远程仓库teamone有而本地没有的数据。因为那台服务器上现有的数据是origin服务器上的一个子集，所以Git并不会抓取数据而是会设置远程跟踪分支teamone/master指向teamone的master分支。
#### 推送
当你想要公开分享一个分支时，需要将其推送到有写入权限的远程仓库上。本地分支并不会自动与远程仓库同步-你必须显示地推送想要分享的分支。这样，你就可以把不愿意分享的内容放到私人分支上，而将需要和别人协作的内容推送到公开分支。如果希望和别人一起在名为serverfix的分支上工作，你可以像推送第一个分支那样推送它。运行git push (remote) (branch):
```
  $ git push origin serverfix
  Counting objects: 24, done.
  Delta compression using up to 8 threads.
  Compressing objects: 100%(15/15), done.
  Writing objects: 100% (24/24), 1.91 KiB | 0 bytes/s, done.
  Total 24 (delta 2), reused 0 (delta 0)
  To https://github.com/schacon/simplegit
   * [new branch] serverfix -> serverfix
```
Git自动将serverfix分支名字展开为refs/heads/serverfix:refs/heads/serverfix,意味着，推送本地的serverfix分支来更新远程仓库上的serverfix分支。
#### 跟踪分支
从一个远程跟踪分支检出一个本地分支会自动创建一个叫做"跟踪分支"(有时也叫上游分支)。跟踪分支是与远程分支有直接关系的本地分支。如果在一个跟踪分支上输入git pull，Git能自动地识别去哪个服务器上抓取、合并到哪个分支。
```
 $ git checkout --track origin/serverfix
 Branch serverfix set up to track remote branch serverfix from origin.
 Switched to a new branch 'serverfix'
```
如果想将本地分支与远程分支设置为不同名字，命令如下：
```
 $ git checkout -b sf origin/serverfix
 Branch sf set up to track remote branch serverfix from origin.
 Switched to a new branch 'sf'
```
现在，本地分支sf会自动从origin/serverfix拉取。如果想要修改正在跟踪的上游分支，你可以在任意时间使用-u或--set-upstream-to选项运行git branch来显式地设置。
```
  $ git branch -u origin/serverfix
  Branch serverfix set up to strack remote branch serverfix from origin.
```
* 提示：当设置好跟踪分支后，可以通过@{upstream}或@{u}快捷方式来引用它。如：可以使用git merge @{u}来取代git merge origin/serverfix.  
如果想查看设置的所有跟踪分支，可以使用git branch的-vv选项。这会将所有的本地分支列出来并且包含更多的信息。
```
  $ git branch -vv
    iss53     7e424c3 [origin/iss53: ahead 2] forgot the brackets
    master    1ae2a45 [origin/master] deploying index fix
  * serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] this should do it
    testing   5ea463a trying something new
```
可以看到iss53分支正在跟踪origin/iss53并且ahead是2，意味着本地有两个提交还没有推送到服务器上。master分支正在跟踪origin/master分支并且是最新的。serverfix分支正在跟踪teamone服务器上的server-fix-good分支并且领先3落后1，意味着服务器上有一次提交还没有合并入同时本地有三次提交还没推送。testing分支并没有跟踪任何远程分支。  
需要注意的是这些数字的值来自于你从服务器上最后一次抓取的数据。这个命令并没有连接服务器，它只会告诉你关于本地缓存的服务器数据。如果想捅进最新的领先于落后数字，需要在运行此命令前抓取所有的远程仓库，命名为：$ git fetch --all; git branch -vv
#### 拉取
当git fetch命令从服务器上抓取本地没有的数据时，他并不会修改工作目录中的内容。它只会获取数据然后让你自己合并。然而，git pull在大多数情况下的含义是一个Git fetch紧接着一个git merge命令。
#### 删除远程分支
带有--delete选项的git push命令来删除一个远程分支。
```
  $ git push origin --delete serverfix
  To https://github.com/schacon/simplegit
  - [deleted] serverfix
```
基本上这个命令只是从服务器上移除这个指针。Git服务器通常会保留数据一段时间直到垃圾回收运行。
## 变基
在Git中整合来自不同分支的修改主要有两种方法：merge以及rebase。
### 变基的基本操作
回顾之前，开发任务分叉到两个不同分支，又各自提交了更新。整合分支最容易的方法时merge命令。它会把两个分支的最新快照(C3和C4)以及二者最近的共同祖先(C2)进行三方合并，合并的结果是生成一个新的快照(并提交).  
其实，还有一种方法：你可以提取在C4中引入的补丁和修改，然后在C3的基础上应用一次。在Git中，这种操作叫做变基。可以使用rebase命令将提交到某一分支上的所有修改移至另一个分支上，就像重新播放一样。
```
  $ git checkout develop
  $ git rebase master
  First, rewinding head to replay your work on top of it...
  Applying: added staged command
```
原理是：首先找到这两个分支(即当前develop、变基操作的目标基底分支master)的最近共同祖先C2,然后对比当前分支develop相对于该祖先的历次提交，提取相应的修改并存为临时文件，然后将当前分支指向目标基底C3(master),最后以此将之前另存为临时文件的修改依序引用。  
现在回到master分支，进行一次快进合并。
```
  $ git checkout master
  $ git merge develop
```
此时，C4'指向的快照就和上面使用merge命令合并的C5指向的快照一模一样。这两种整合方法的最终结果没有任何区别，但变基使得提交历史更加整洁。一般使用rebase目的是为了确保在向远程分支推送时能保持提交历史的整洁。注意，无论是通过变基，还是通过三方合并，整合的最终结果所指向的快照始终是一样的，只不过提交历史不同罢了。
### 有趣的变基例子
在对两个分支进行变基时，所生成的"重放"并不一定要在目标分支上应用，也可以指定另一个分支进行应用。你创建了一个特性分支server，为服务端添加一些功能，提交了C3和C4.然后从C3上创建了特性分支client,为客户端添加一些功能，提交了C8和C9.最后，回到server分支，又提交了C10.假设你希望将client中的修改合并到主分支并发布，但暂时并不想合并server中的修改。这时，可以使用git rebase --onto选中在client分支里但不在server分支里的修改(即C8和C9)，将它们在master分支上重放：
```
  $ git rebase --onto master server client
```
意思是：取出client分支，找出处于client分支和server分支的共同祖先之后的修改，然后把它们在master分支上重放一遍。
```
  $ git checkout master
  $ git merge client
```
接下来决定将server分支中的修改也整合进来。使用git rebase [basebranch] [topicbranch]命令可以直接将特性分支(即本例中的server)变基到目标分支(即master)上。这样能省去你先切换到server分支，再对其执行变基命令的多个步骤。
```
  $ git rebase master server
```
然后可以快进合并主分支master了：
```
  $ git checkout master
  $ git merge server
```
### 变基的风险
#### 不要对在你的仓库外有副本的分支执行变基
变基操作的实质是丢弃一些现有的提交，然后相应地新建一些内容一样但实际上不同的提交。如果你已将提交推送至某个仓库，而其他人也已经从该仓库拉取提交并进行了后续工作，此时，如果用git rebase命令重新整理了提交并再次推送，你的同伴因此将不得不再次将他们手头的工作与你的提交进行整合，如果接下来你还要拉取并整合他们修改过的提交，事情会变得一团糟。
### 用变基解决变基
如果团队中的某人强制推送并覆盖了一些你所基于的提交，你需要做的是检查你做了哪些修改，以及他们覆盖了哪些修改。  
实际上，Git除了对整个提交计算SHA-1校验和以外，也对本次提交所引入的修改计算了校验和--即"patch-id"。如果你拉取被覆盖过的更新并将你手头的工作基于此进行变基的话，一般情况下Git都能成功分辨出哪些是你的修改，并把它们应用到新分支上。如：  
有人推送了经过变基的提交，并丢弃了你的本地开发所基于的一些提交这种情况，如果我们不是执行合并，而是执行git rebase teamone/master，Git将会：  
* 检查哪些提交是我们的分支上独有的(C2,C3,C4,C6,C7)
* 检查其中哪些提交不是合并操作的结果(C2,C3,C4)
* 检查哪些提交在对方覆盖更新时并没有被纳入目标分支(只有C2和C3,因为C4其实就是C4')
* 把查到的这些提交应用在teamone/master上面  
从而我们将得到``与你将相同的内容又合并了一次，生产了一个新的提交``中不同的结果。要想上述方案有效，还需要对方在变基时确保C4'和C4是几乎一样的。在本例中，可以使用git pull --rebase直接合并。  
只要你把变基命令当作是在推送前清理提交使之整洁的工具，并只在从未推送至共用仓库的提交上执行变基命令，就不会有事。
### 变基 vs 合并
有一种观点认为，仓库的提交历史即是记录实际发生过什么。它是针对历史的文档，本身就有价值，不能乱改。  
另一种观点认为，提交历史是项目过程中发生的事。软件维护手册也是需要反复修订才能方便使用。
# 服务器上的Git
## 协议
Git可以使用四种主要的协议来传输资料：本地协议(Local),HTTP协议，SSH(Secure Shell)协议及Git协议
### 本地协议
如果你使用共享文件系统，就可以从本地版本库clone、push及pull。像这样去clone一个版本库或者增加一个远程到现有的项目中，使用版本库路径作为URL.
```
 $ git clone /opt/git/project.git
 或者：
 $ git clone file:///opt/git/project.git
```
如果仅是指定路径，Git尝试使用硬链接或直接复制所需要的文件。如果指定(file://)，Git会触发平时用于网络传输资料的进程，那通常是传输效率较低的方法。  
要增加一个本地版本库到现有的Git项目：
```
 $ git remote add local_proj /opt/git/project.git
```
然后，就可以像在网络上一样从远端版本库推送和拉取更新了。
#### 优点
基于文件系统的版本库的优点是简单，并直接使用了现有的文件权限和网络访问权限。
#### 缺点
通常共享文件系统比较难配置，并比起基本的网络连接访问，这不方便从多个位置访问。如果想从家里推送内容，必须先挂载一个远程磁盘，相比网络连接的访问方式，配置不方便，速度也慢。  
这个协议并不保护仓库避免意外的损坏。每个用户都有"远程"目录的完整shell权限，没有方法可以阻止他们修改或删除Git内部文件和损坏仓库。
### HTTP协议
Git通过HTTP通信有两种模式。
#### 智能Http协议
智能Http协议的运行方式和SSH及Git协议类似，只是运行在标准的Http/S端口上并可以使用各种Http验证机制。类似Github的服务，在网页上看到url(如：https://github.com/ckrgithub/StudyNote),和你在clone、push时使用一样。
### 哑Http协议
哑http协议里web服务器仅把裸版本库当作普通文件来对待，提供文件服务。基本上，只需要把一个裸版本库放在http根目录，设置一个叫做post-update的挂钩就可以。
```
 $ cd /var/www/htdocs/
 $ git clone --bare /path/to/git_project gitproject.git
 $ cd gitproject.git
 $ mv hooks/post-update.sample hooks/post-update
 $ chmod a+x hooks/post-update
```
Git自带post-update挂钩会默认执行合适的命令，来确保通过http的获取和克隆操作正常工作。这条命令会在你通过SSH向版本库推送之后被执行；然后别人就可以通过类似下面的命令来clone:
```
 $ git clone https://example.com/gitproject.git
```
### SSH协议
架设Git服务器时常用SSH协议作为传输协议。SSH协议是一个验证授权的网络协议；通过SSH协议clone版本库：
```
 $ git clone ssh://user@server/project.git
```
### Git协议
Git里有一个特殊的守护进程；它监听在一个特定的端口(9418),类似于SSH服务，但访问无需任何授权。要让版本库支持Git协议，需要先创建一个git-daemon-export-ok文件--它是Git协议守护进程为这个版本库提供服务的必要条件
## 在服务器上搭建Git
在开始架设Git服务器前，需要把现有仓库导出为裸仓库--即一个不包含当前工作目录的仓库。为了通过clone你的仓库来创建一个新的裸仓库，你需要在clone命令加上'--bare'选项
```
 $ git clone --bare my_project my_project.git
 Clonig into bare repository 'my_project.git'...
 done.
```
### 把裸仓库放到服务器上
假设一个域名为git.example.com的服务器已经架设好，并可通过SSH连接，你想把所有Git仓库放在/opt/git目录下。假设服务器上存在/opt/git/目录，可以通过命令复制你的裸仓库来创建一个新仓库：
```
 $ scp -r my_project.git user@git.example.com:/opt/git
```
此时，其他通过SSH连接这台服务器并对/opt/git目录拥有可读权限的使用者。
```
 $ git clone user@git.example.com:/opt/git/my_project.git
```
# GitLab
## 安装
Gitlab是一个数据库支持的web应用。为了更快速的启动和运行，你可以下载虚拟机镜像或者在[这里](https://bitnami.com/stack/gitlab)上获取一键安装包，同时调整配置使之符合你特定的环境。可以在Gitlab社区版的[readme](https://gitlab.com/gitlab-org/gitlab-cd/tree/master)找到安装Gitlab的帮助。
## 管理
Gitlab的管理界面使通过网络进入的。将你的浏览器转到已经安装Gitlab的主机名或IP地址，然后以管理员身份登录即可。
### 使用者
Gitlab上的用户指的是对应协作者的账号。每一个用户账号都有一个**命名空间**,即该用户项目的逻辑集合。如果一个叫ckr的用户拥有一个名称是StudyNote的项目，那么这个项目的url会是http://server/ckr/StudyNote.  
移除一个用户有两种方法。"屏蔽(Blocking)"一个用户阻止他登录Gitlab实例，但该用户命名空间下的所有数据任然会被保存。"销毁(Destroying)"一个用户，会彻底的将他从数据库和文件系统中移除。他命名下的所有项目和数据都会被删除，拥有的任何组也会被移除。
### 组
一个Gitlab的组是一些项目的集合，连同关于多少用户可以访问这些项目的数据。每一个组都有一个项目命名空间(与用户一样)，所有如果一个叫training的组拥有一个名称是materials的项目，那么这个项目的url会是http://server/training/materials.
### 项目
一个gitlab的项目相当于git的版本库。每一个项目都有一个可视级别，控制着谁可以看到这个项目页面和仓库。
### 钩子
Gitlab在项目和系统级别上都支持钩子程序。对任意级别，当有相关事件发生时，gitlab的服务器会执行一个包含描述性json数据的http请求。
## 分布式Git
### 集中式工作流
集中式系统中通常使用的是单点协作模型--集中式工作流。这意味着如果两个开发者从中心仓库克隆代码下来，同时作了一些修改，那么只有第一个开发者可用顺利地把数据推送回共享服务器，第二个开发者在推送修改前，必须先将第一个人的工作合并进来。
### 集成管理者工作流
git允许多个远程仓库存在：每个开发者拥有自己仓库的写权限和其他所有人仓库的读权限。这种情形下，通常有个"官方"项目的权威仓库。要为这个项目做贡献，你需要从该项目克隆出一个自己的公开仓库，然后将自己的修改推送上去。接着你可以请求官方仓库的维护者拉取更新合并到主项目。
### 司令官与副官工作流
* 1.普通开发者在自己的特性分支上工作，并根据master分支进行变基。这里是司令官的"master"分支
* 2.副官将普通开发者的特性分支合并到自己的master分支中。
* 3.司令官将所有副官的master分支并入自己的master分支中
* 4.司令官将集成后的master分支推送到参考仓库中，以便所有其他开发者以此为基础进行变基.
### 提交准则
git diff --check:检查空白错误(空白错误：指行尾的空格、tab制表符，和行首空格跟tab制表符)
如果其中一些改动修改了同一个文件，尝试使用git add --patch来部分暂存文件。
# 变基与拣选工作流
为了保持线性的提交历史，有些维护者喜欢在master分支上对贡献过来的工作进行变基和拣选，而不是直接将其合并。当你完成某个特性分支中的工作，并决定要将其整合的时候，你可以在该分支中运行变基命令，在当前master分支(或develop等分支)的基础上重新构造修改。另一种：将引入的工作转移到其他分支的方法是拣选。Git中拣选类似于对特定的某次提交的变基。它会提取该提交的补丁，之后尝试将其重新应用到当前分支上。
```
 $ git cherry-pick commitId //拉取某个commit到当前分支
```
# 为发布打标签
```
 $ git tag -s v1.5 -m 'my signed 1.5 tag'
```
准备一次发布：使用git archive为那些不使用Git的人员创建一个最新的快照归档
```
 $ git archive master --prefix='project/' | gzip > 'git describe master'.tar.gz
 或
 $ git archive master --prefix='project/' --format=zip > 'git describe master'.zip
```
# 制作提交简报
git shortlog:可以快速生成一份包含从上次发布之后项目新增内容的修改日志类文档。它会对你给定范围内的所有提交进行总结：
```
 $ git shortlog --no-merges master --not v1.0.1
``
# GitHub
## 派生(Fork)项目
如果你想要参与某个项目，但没有推送权限，这时可以对这个项目进行'派生'。派生意思是github将在你的空间中创建一个完全属于你的项目副本，且你对其具有推送权限。通过这种方式将修改推送到派生出的项目副本，并通过创建合并请求(Pull Request)来让改动进入源版本库。  
## Markdown
任务列表
```
 - [x] 编写代码
 - [ ] 编写单元测试
 - [ ] 编写文档
```
摘录代码：使用'反引号'
```java
 for(int i=0;i<3;i++){
 
 }
```
引用：使用>符号
```
 > Whether 'tis Nobler in the mind to suffer'
 > The Slings and Arrows of outrageous Fortune
```
表情符号：使用:开头
```
 :<表情名称>:
```
# Git工具
## 选择修订版本
### 简短的SHA-1
Git十分智能，只需要提供SHA-1的前几个字符就可以获得对应的那次提交，当然提供的SHA-1字符数量不少于4个。
**git log**命名查看之前新增一个功能的那次提交
```
 $ git log
 commit 734713b...
 Author: <ckr@gmail.com>
 Date: 2019.01.20
   fixed a bug
 commit d921970...
 Merge: 1c002d...
 Author: <ckr@gmail.com>
 Date: 2019.01.20
   merge commit 'add unit test'
 commit 1c002dd...
 Author: <ckr@gmail.com>
 Date: 2019.01.20
   add readme.md
```
假设这个提交是1c002d...,如果想git show这个提交
```
 $ git show 1c002d
```
使用**git log --abbrev-commit**参数，输出结果里会显示简短且唯一的值
```
 $ git log --abbrev-commit --pretty=oneline
 734713b fixed a bug
 d921970 merge commit 'add unit test'
 1c002dd add readme.md
```
### 分支引用
指明一次提交最直接的方法是有一个指向它的分支引用。
* 引用日志：当你在工作时，git会在后台保存一个引用日志(reflog)，引用日志记录了最近几个月你的HEAD和分支引用所指向的历史。
```
 $ git reflog
 734713b HEAD@{0}: commit: fixed refs handling, added gc auto, updated
 d921970 HEAD@{1}: merge phedders/rdocs: Merge made by recursive.
 1c002dd HEAD@{2}: commit: added some blame and merge stuff
 1c36188 HEAD@{3}: rebase -i (squash): updating HEAD
 95df984 HEAD@{4}: commit: # This is a combination of two commits.
 1c36188 HEAD@{5}: rebase -i (squash): updating HEAD
 7e05da5 HEAD@{6}: rebase -i (pick): updating HE
```
每当你的HEAD所指向的位置发生了变化，Git就会将这个信息存储到引用日志这个历史记录里。如果你想要查看仓库中HEAD在五次前的所指向的提交，你可以使用@{n}来引用reflog中输出的提交记录。
```
 $ git show HEAD@{5}
```
你同样可以使用这个语法来查看某个分支在一定时间前的位置。如：查看你的master分支在昨天时候指向了哪个提交
```
 $ git show master@{yesterday}
```
注意，引用日志只存在于本地仓库，一个记录你在你自己的仓库里做过什么的日志。
* 祖先引用：是另一种指明一个提交的方式。如果在引用的尾部加上一个^，git会将其解析为该引用的上一个提交。
```
 $ git log --pretty=format:'%h %s' --graph
 * 734713b fixed refs handling, added gc auto, updated tests
 * d921970 Merge commit 'phedders/rdocs'
 |\
 | * 35cfb2b Some rdoc changes
 * | 1c002dd added some blame and merge stuff
 |/
 * 1c36188 ignore *.gem
 * 9b29157 add open3_detach to gemspec file list
 
 $ git show HEAD^
 commit d921970aadf03b3cf0e71becdaab3147ba71cdef
 Merge: 1c002dd... 35cfb2b...
 Author: Scott Chacon <schacon@gmail.com>
 Date: Thu Dec 11 15:08:43 2008 -0800
  Merge commit 'phedders/rdocs'
```
另一种指明祖先提交的方法是~。同样是指向第一父提交，因此HEAD~和HEAD^是等价的。而区别在于你后面加数字的时候。HEAD~2代表"第一父提交的第一父提交"
### 提交区间
* 双点：想要查看develop分支中还有哪些提交尚未被合并入master分支。可以使用master..develop来让git显示这些提交。也就是在develop分支中而不是在master分支中的提交。
```
 $ git log master..develop
```
* 多点：查看哪些提交是被包含在某些分支中的一个，但不在你当前的分支上。
```
 $ git log refA..refB
 $ git log ^refA refB
 $ git log refB --not refA
```
* 三点：可以选择出被两个引用中的一个包含但又不被两者同时包含的提交。
```
 $ git log master...develop
```
### 交互式暂存
git add -i:git会进入一个交互式终端模式
### 储藏工作
git status:查看状态
```
 $ git status
 Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
  modified: index.html
 Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working
directory)
  modified: lib/simplegit.rb
```
现在想要切换分支，但还不想要提交之前的工作；所以储藏修改。运行git stash或git stash save
```
 $ git stash
 Saved working directory and index state \
  "WIP on master: 049d078 added the index file"
HEAD is now at 049d078 added the index file
(To restore them type "git stash apply")
```
查看储藏，可以使用git stash list:
```
 $ git stash list
 stash@{0}: WIP on master: 049d078 added the index file
 stash@{1}: WIP on master: c264051 Revert "added file_size"
 stash@{2}: WIP on master: 21d80a5 added number to log
```
想要应用其中一个更旧的储藏，可以通过名字指定它，如：git stash apply stash@{2}。如果不指定一个储藏，git认为指定的是最近的储藏
```
 $ git stash apply
 # On branch master
 # Changed but not updated:
 # (use "git add <file>..." to update what will be committed)
 #
 # modified: index.html
 # modified: lib/simplegit.rb
 #
```
### 创造性的储藏
stash save命令的--keep-index选项:当你做了几个改动并只想提交其中的一部分，过一会再回来处理剩余改动
```
 $ git status -s
 M index.html
  M lib/simplegit.rb
 $ git stash --keep-index
 Saved working directory and index state WIP on master: 1b65b17 added the
 index file
 HEAD is now at 1b65b17 added the index file
 
 $ git status -s
 M index.html
```
另一个经常使用储藏来做的事是储藏未跟踪文件。默认情况下，git stash只会储藏已经在索引中的文件。如果指定--include-untracked或-u标记，git也会储藏任何创建的未跟踪文件
```
 $ git status -s
 M index.html
  M lib/simplegit.rb
 ?? new-file.txt
 $ git stash -u
 Saved working directory and index state WIP on master: 1b65b17 added the
 index file
 HEAD is now at 1b65b17 added the index file
```
### 从储藏创建一个分支
如果储藏了一些工作，将它留在那儿，然后继续在储藏的分支上工作，在重新应用工作时可能会有问题。可以运行git stash branch 创建一个新分支，检出储藏工作时所在的提交，重新在那应用工作。
### 清理工作目录
git clean:去除冗余文件或清理工作目录。  
git clean -f-d:移除工作目录中所有未追踪的文件及空的子目录。  
git clean -d -n:做一次演习
```
 $ git clean -d -n
 Would remove test.o
 Would remove tmp/
```
# 签署工作





























