## git教程
### 一.安装
    1. 在linux上安装git:  
    首先，通过 $ git 命令查看系统有没安装git;若没有安装，则通过sudo apt-get install git就可以直接完成git的安装
    2.在Mac OS X上安装git：  
    一是安装homebrew，然后通过homebrew安装git  
    二是在appstore安装Xcode,选择菜单Xcode->Preferences,在弹出窗口中找到downloads选择Command Line Tools点击安装  
    3.在windows上安装git  
    从git官网上下载Git-xxx-bit.exe,点击安装。安装完成后，在Git Bash配置：  
    $ git config --global user.name "your name"  
    $ git config --global user.email "your email"  
    4.创建版本库  
    $ mkdir ckr  //仓库目录  
    $ cd ckr  
    $ pwd   //用于显示当前目录  
    $ git init   
    $ git add readme.md //添加文件到仓库  
    $ git commit -m "add a new file" //-m:输入本次提交的说明  

### 二、时光机穿梭
    1.版本回退  
    $ git log:查看历史提交记录  
    $ git log --pretty=oneline //过滤信息  
    $ git reset --hard HEAD^  //回退到上一个版本  
    $ git reset --hard commit_id   //回退到特定版本  
    $ git reflog //记录每一次命令,可查看commit_id  
    2.工作区和暂存区  
    工作区：ckr目录  
    暂存区：git add添加的文件，实际上时把文件修改添加到暂存区;git commit提交是把暂存区的所有内容提交到当前分支  
    $ git status  //查看状态  
    $ git diff HEAD -- readme.md//查看工作区和版本库里最新版本的区别  
    3.撤销修改  
    $ git checkout -- readme.md  
    一种是readme.md修改后没有被放到暂存区，现在撤销修改回到版本库一模一样的状态  
    一种是readme.md已经添加到暂存区后，又做修改，现在撤销修改回到添加到暂存区后的状态  
    总之，就是让这个文件回到最近一次git commit或git add 状态  
    $ git reset HEAD readme.md //把暂存区的修改回退工作区  
    4.删除文件  
    $ git rm readme.md//删除文件  

### 三、分支管理
    1.分支切换
    $ git checkout -b dev  //切换为dev分支，-b表示创建并切换  
    $ git branch dev //创建分支  
    $ git checkout dev //切换分支  
    $ git branch //查看当前分支，并列出所有分支  
    $ git checkout master  
    $ git merge dev  //合并指定分支dev到当前分支master上,这次合并是"快进模式",但删除分支后，会丢掉分支信息  
    $ git branch -d dev //删除dev分支  
    
    2.解决冲突
    $ git status //查看冲突状态  
    $ git log --graph --pretty=oneline --abbrev-commit //查看合并情况,git log --graph查看分支合并图 
    $ git merge --no-ff -m "merge分支并提交记录"  dev //--no-ff：禁止Fast forward,
    
    3.bug分支
    $ git stash //把当前工作区储藏起来  
    $ git stash list //查看  
    $ git stash apply //恢复，但stash内容并不删除，需要用git stash drop来删除
    $ git stash pop //恢复，同时把stash内容删了
    
    4.rebase
    $ git remote -v //显示更详细的信息，可以显示fetch、push的origin地址
    $ git push origin dev //推送分支
    $ git checkout -b dev origin/dev //创建远程origin的dev分支到本地
    $ git branch --set-upstream-to=origin/dev dev //指定本地dev分支与远程origin/dev分支的链接
    $ git branch --set-upstream branch-name origin/branch-name //建立本地分支和远程分支的关联
    1.1->1.2->2.0->2.1->1.3  
    $ git rebase -i 2.1
    $ git rebase master
    $ git rebase --continue
    $ git rebase--abort

## 感谢
[git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
