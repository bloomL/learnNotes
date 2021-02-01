Committing Files

1. Git Init

   ```basic
   git init
   ```

2. Git Status

    A **working directory** contains the latest downloaded version from the repository together with any changes that have yet to be committed(尚未提交的任何更改。). As you're working on a project, all changes are made in this working directory.

   ```basic
   git status
   ```

3. Git Add

   要将文件保存或提交到Git存储库，您首先需要将它们添加到暂存区.Git has three areas, a **working directory**, a **staging area** and the **repository itself** Use the command **git add <file|directory>** to add  to the **staging area**.

   **git status**

   ![image-20210126103956430](C:\Users\l'g\AppData\Roaming\Typora\typora-user-images\image-20210126103956430.png)

   **git add hello-world.js**

   **git status**

   ![image-20210126104026851](C:\Users\l'g\AppData\Roaming\Typora\typora-user-images\image-20210126104026851.png)

4. Git commit

   The command **git commit -m "commit message"** moves files from staging to the repository and records the time/date, author and a commit message that can be used to add additional context and reasoning to the changes such as a bug report number.

   ```basic
   git commit -m "Initial Hello World Commit"
   ```

5. Git Ignore

   To ignore these files you create a **.gitignore** file in the root of the repository

   ```basic
   echo '*.tmp' > .gitignore   #输出*.temp到git忽略文件
   git add .gitignore
   git commit -m "gitignore file"
   ```

#### Committing Changes

1. Git Status	allows us to view the changes in the working directory and staging area compared to the repository.(允许我们查看工作目录和暂存区与存储库相比的更改。)

   ```basic
   git status
   ```

2. Git Diff       enables you to compare changes in the working directory against a previously committed version（工作目录中的更改与先前提交的版本进行比较）

   ```basic
   git diff           #默认比较工作目录和HEAD提交
   git diff <commit>  
   ```

3. Git Add     

   ```basic
   git add XXXmodify
   ```

4. Staged Differences   To compare the changes in the staging area against the previous commit(要将暂存区中的更改与上一次提交进行比较)

   ```basic
   git diff --staged
   ```

5. Git Log    view the history of the repository and the commit log.

   ```basic
   git log 
   git log --pretty=format:"%h %an %ar - %s"   
   --pretty	# 内容格式化
   %h			# 提交对象的简短哈希字串
   %an			# 作者（author）的名字
   %ar			# 作者修订日期，按多久以前的方式显示
   %s			# 提交说明
   --oneline	# 精简模式,短哈希子串+提交信息
   ```

6. Git Show  view the changes made in the commit(查看在提交中所做的更改)

   ```basic
   git show 
   git show <commit>
   ```

#### Working Remotely

1. Git Remote

   ```basic
   git remote add <A remote name> <A remote URL> 
   ```

2. Git Push

   The **git push** command is followed by two parameters. The first parameter is the friendly **name of the remote repository** we defined in the first step. The second parameter is the **name of the branch**.

   ```basic
   git push  # 默认将master分支上的提交推送到源远程
   git push origin master
   ```

3. Git Pull

    allows you to sync changes from a remote repository into your local version.

   ```basic
   git pull origin master
   ```

4. Git Log

   ```basic
   git log 	# 仓库历史记录
   git show 	# 查看提交的修改
   git log --grep="#1234"
   ```

5. Git Fetch

   is a great way to review the changes without affecting your current branch.

   The command **git pull** is a combination of two different commands, **git fetch** and **git merge**

   ```basic
   git merge remotes/<remote-name>/<remote-branch-name> master # merge the fetched changes into master
   git fetch
   git checkout	# 访问分支
   git branch -r	# view a list of all the remote branches
   ```

#### Undoing Changes（撤销修改）

1. Git Checkout

    replace everything in the working directory to the last committed version.(将工作目录中的所有内容替换为最后提交的版本)

   ```basic
   git checkout .	# . 当前工作空间所有内容；否则列出目录/文件，并用空格分隔
   ```

2. Git Reset

   move files back from the staging area to the working directory（将文件从暂存区移回工作目录）

   ```basic
   git reset HEAD .	# .当前目录重置所有文件;否则列出用空格分隔的文件
   ```

3. Git Reset Hard

   **git reset --hard** will combine both **git reset** and **git checkout** in a single command.

   files removed from the staging area and the working directory is taken back to the state of the last commit.(从暂存区和工作目录中删除的文件将被带回到上一次提交的状态。)

   ```basic
   git reset --hard <commit-hash>
   
   git reset --hard HEAD
   ```

   Using **HEAD** will clear the state back to the last commit, using **git reset --hard <commit-hash>** allows you to go back to **any** commit state. Remember, **HEAD** is an alias(别名) for the last commit-hash(提交哈希) of the branch.

4. Git Revert

   If you have already committed files but realised you made a mistake then the command **git revert(还原)** allows you to undo the commits(撤销提交). 

   If you haven't pushed your changes then **git reset HEAD~1** has the same affect and will remove the last commit.

   ```basic
   git revert HEAD --no-edit
   
   git revert HEAD...HEAD~2
   
   ```

#### Fixing Merge Conflicts(解决合并冲突)

1. Git Merge

   ```basic
   git merge
   ```

2. Viewing Conflict

   the local changes will appear at the top between *<<<<<<< HEAD* and *=======* 

   remote changes being underneath between *=======* and *>>>>>>> remotes/origin/master*.

3. Resolving Conflict

   The simplest way to fix a conflict is to pick either the local or remote version 

   ```basic
   git checkout --ours staging.txt		# 选择自己版本
   
   git checkout --theirs staging.txt	# 选择远程版本
   ```

#### Experiments Using Branches(分支实验)

1. Git Branch

   ```basic
   git branch <new branch name> <starting branch>
   eg:	git branch new_branch master
   git checkout <new branch name>
   ```

   **git checkout -b <new branch name>** will create and checkout the newly created branch.

2. List Branches

   ```basic
   git brabch
   
   git branch -va	# -a将包括远程分支，-v将包括该分支的HEAD提交消息
   ```

3. Merge To Master

   first need to checkout the **target branch, in this case master**, and then use the **git merge** command to merge in the commits from a branch.

   ```basic
   #1.
   git checkout master
   #2.
   git merge new_branch
   ```

4. Push Branches

   ```basic
   git push <remote_name> <branch_name>
   ```

5. Clean Up Branches

   To delete a branch you need to provide the argument **-d**

   ```basic
   git branch -d <branch_name>
   eg: git branch -d new_branch
   ```

6. Git Blame

   **git blame <file>** shows the revision and author who last modified each line of a file.

   ```basic
   git blame list.html
   
   git blame -L 6,8 list.html		# -L要输出的行
   ```

   