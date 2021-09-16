**删除文件并移出跟踪清单**
$ rm PROJECTS.md

**重命名**
$ git mv file_from file_to

**查看log，显示提交差异，看近两次提交**
$ git log -p -2

**重新提交，覆盖原来的提交信息**
$ git commit --amend

**取消暂存的文件**
$ git reset HEAD CONTRIBUTING.md

**撤销对文件的修改**
$ git checkout -- CONTRIBUTING.md

**查看远程仓库，并显示url**
$ git remote -v

**添加远程仓库，同时指定一个方便使用的简写**
git remote add <shortname> <url> 

**从远程仓库中获得数据**
$ git fetch <remote>
只会将数据下载到你的本地仓库——它并不会自动合并或修改你当前的工作。 当准备好时你必须手动将其合并入你的工作。
git pull 命令自动抓取后合并该远程分支到当前分支。

**查看某个远程仓库**
$ git remote show <remote>

**远程仓库的重命名与移除**
$ git remote rename pb paul
$ git remote remove paul
这同样也会修改你所有远程跟踪的分支名字

**分支切换**
$ git checkout master
分支切换会改变你工作目录中的文件

**查看项目分叉历史**
git log --oneline --decorate --graph --all

**创建新分支的同时切换过去**
git checkout -b

**合并iss52到main，当前为main**
git merge iss52
 一旦暂存这些原本有冲突的文件，Git 就会将它们标记为冲突已解决。

3.3 git分支-分支管理