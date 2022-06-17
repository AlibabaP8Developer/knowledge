# 记录每次更新到仓库

1. 检测当前文件状态 : git status
2. 提出更改（把它们添加到暂存区）：git add filename (针对特定文件)、git add *(所有文件)、git add *.txt（支持通配符，所有 .txt 文件）
3. 忽略文件：.gitignore 文件
4. 提交更新: git commit -m "代码提交信息" （每次准备提交前，先用 git status 看下，是不是都已暂存起来了， 然后再运行提交命令 git commit）
5. 跳过使用暂存区域更新的方式 : git commit     -a -m "代码提交信息"。 git commit 加上 -a 选项，Git     就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 git add 步骤。
6. 移除文件 ：git rm filename （从暂存区域移除，然后提交。）
7. 对文件重命名 ：git mv README.md README(这个命令相当于mv README.md README、git rm README.md、git add README 这三条命令的集合)



# 推送改动到远程仓库

- 如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：git remote add origin <server> ,比如我们要让本地的一个仓库和 Github     上创建的一个仓库关联可以这样git remote add origin https://github.com/Snailclimb/test.git
- 将这些改动提交到远端仓库：git push origin master (可以把 *master* 换成你想要推送的任何分支)
            如此你就能够将你的改动推送到所添加的服务器上去了。

 

 

# 远程仓库的移除与重命名

- 将 test 重命名为 test1：git remote rename test test1
- 移除远程仓库 test1:git remote rm test1



# 查看提交历史

在提交了若干更新，又或者克隆了某个项目之后，你也许想回顾下提交历史。 完成这个任务最简单而又有效的工具是 git log 命令。git log 会按提交时间列出所有的更新，最近的更新排在最上面。

可以添加一些参数来查看自己希望看到的内容：

只看某个人的提交记录：

git log --author=bob

 

# 撤销操作

有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。 此时，可以运行带有 --amend 选项的提交命令尝试重新提交：

git commit --amend

取消暂存的文件

git reset filename

撤消对文件的修改:

git checkout -- filename

假如你想丢弃你在本地的所有改动与提交，可以到服务器上获取最新的版本历史，并将你本地主分支指向它：

 git fetch origin
 git reset --hard origin/master



报错：fatal: unable to access 'https://github.com/Homebrew/brew/':

解决方案，执行下面两行命令：

```
git config --global --unset http.proxy 
git config --global --unset https.proxy
```