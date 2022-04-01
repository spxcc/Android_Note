# git命令学习记录



git init 

git add

git commit -m "xxxx"

git log  //可以查看提交历史，以便确定要回退到哪个版本

git log --pretty=oneline //单行显示log

git reflog  //查看命令历史，以便确定要回到未来的哪个版本

git reset --hard commit_id //回退到某个版本

git reset --hard HEAD^ //回退到上一个版本
git reset --hard HEAD~1 ////回退到上一个版本

git status //查看状态

git diff //查看修改了什么内容

git checkout -- readme.txt
{
一般用于工作区有修改但未add到暂存区，可以用此命令撤回工作区修改的内容

注意：这里有两种情况：
一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

另一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
总之，就是让这个文件回到最近一次git commit或git add时的状态。

}





git reset HEAD readme.txt //如果已经add到了暂存区但未提交，可以用此命令把暂存区的修改撤销掉（unstage），重新放回工作区（撤回add）


git rm test.txt  //删除版本库已经存在（已提交）的文件，并且git commit即可


git remote add origin git@github.com:lansword/GitTest.git//本地和远程仓库建立连接


git push -u origin master
由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令: git push origin master


git clone git@github.com:lansword/Testfirst.git //git clone克隆一个本地库


查看分支：git branch

创建分支：git branch <name>

切换分支：git checkout <name>

创建+切换分支：git checkout -b <name>

合并某分支到当前分支：git merge <name>

删除分支：git branch -d <name>



使用git merge 合并分支时遇到冲突需要先解决冲突然后add、commit之后就合并成功了
{
注意：
	1.在合并出现问题之后，master分支的装填是merging,代表正在合并过程中
	2.此时，两个文件已经是一个完整体了，虽然还是有冲突内容，修改完冲突内容之后，提交，此时两个文件已经合并成功了
}


 git merge --no-ff -m "merge with no-ff" dev //准备合并dev分支，请注意--no-ff参数，表示禁用Fast forward


分支策略:
在实际开发中，我们应该按照几个基本原则进行分支管理：
首先，master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；
那在哪干活呢？干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；
你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。
Git分支十分强大，在团队开发中应该充分应用。
合并分支时，加上--no-ff参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并。


git stash //可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作

git stash list//查看stash列表

git stash apply//恢复，但是恢复后，stash内容并不删除，你需要用git stash drop来删除；

git stash pop//恢复,恢复的同时把stash内容也删了


git branch -D feature-vulcan//已经提交的分支需要使用“-D”强制删除分支

hahaha


git checkout -b dev origin/dev  //本地克隆下来代码之后，可以用该命令切换到其他分支（dev）

git branch --set-upstream-to=origin/dev dev   //git pull失败，原因是没有指定本地dev分支与远程origin/dev分支的链接，根据提示，设置dev和origin/dev的链接




git tag <tagname>用于新建一个标签，默认为HEAD，也可以指定一个commit id；

git tag -a <tagname> -m "blablabla..."//创建带有说明的标签，用-a指定标签名，-m指定说明文字

git tag可以查看所有标签。

git show <tagname>查看标签详细信息

注意：标签总是和某个commit挂钩。如果这个commit既出现在master分支，又出现在dev分支，那么在这两个分支上都可以看到这个标签


命令git push origin <tagname>可以推送一个本地标签；

命令git push origin --tags可以推送全部未推送过的本地标签；

命令git tag -d <tagname>可以删除一个本地标签；

命令git push origin :refs/tags/<tagname>可以删除一个远程标签。

git config --global color.ui true //Git显示颜色，会让命令输出看起来更醒目



1. 查看用户名和地址
   - git config user.name
   - git config user.email

2. 修改用户名和地址
   + git config --global user.name "Your name"
   + git config --global user.email "Your email"

3. 如何退出git diff命令
   - 键入 q即可
