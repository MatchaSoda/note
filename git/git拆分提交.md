# git拆分提交
### 拆分多个文件
这个情况比较好解决，GitBook 上的「拆分提交」部分就写得很清楚了。是通过 rebase 命令来完成的，这个命令还可以修改提交顺序和压缩提交，后者我也经常用。

整个拆分过程的大致步骤就是「告诉 Git 我要修改某条 commit」-「重置 (reset) 到那次 commit 的上一个 commit」->「按需新增 commit」->「修改完毕」。
```bash
# 告诉 git 我要修改某条 commit
$ git rebase -i HEAD~3
# 将某个要拆分的 commit(310154e) 前的 pick 修改为 edit, 如下
pick f7f3f6d changed my name a bit
edit 310154e updated README formatting and added blame
pick a5f4a0d added cat-file

# reset 到那次 commit 的上一个 commit
$ git reset HEAD^

# 按需新增 commit
$ git add README
$ git commit -m 'updated README formatting'
$ git add lib/simplegit.rb
$ git commit -m 'added blame'
$ git rebase --continue

# 修改完毕
$ git rebase --continue
```
### 拆分同个文件
当在一个文件有多个功能改动的时候，这时候拆分 commit 的时候就要对同一个文件修改的内容进行拆分了。拆分过程可以概括为「重置 (reset) 到上一条 commit」->「将未暂存 (unstaged) 的内容拆分成多个部分 (split)」->「将需要的内容添加到暂存 (staged) 区」->「按需新增 commit」，然后循环这个过程。
```bash
# reset 到上一条 commit
$ git reset HEAD^

# 将未暂存(unstaged)的内容拆分成多个部分(split)
## 1.告诉 git 我只要提交文件中的某一部分，它会打印出区别，问你要多这块(hunk)进行怎么处理
$ git add --patch myfile
diff --git a/myfile b/myfile
index 93db4cb..2f113ce 100644
 --- a/myfile
 +++ b/myfile
@@ -1,3 +1,5 @@
+1
 something
 something else
 something again
+2
Stage this hunk [y,n,a,d,/,s,e,?]

# []中的命令分别代表：
# y - stage this hunk
# n - do not stage this hunk
# a - stage this and all the remaining hunks in the file
# d - do not stage this hunk nor any of the remaining hunks in the file
# / - search for a hunk matching the given regex
# s - split the current hunk into smaller hunks
# e - manually edit the current hunk
# ? - print help

## 这里主要是用 s(将内容分块)，y(暂存这一块)，n(先不暂存这一块)
## 比如这里我想再分块，我就输入 s，他就会分成两块
 Split into 2 hunks.
 @@ -1,3 +1,4 @@
+1
 something
 something else
 something again
Stage this hunk [y,n,a,d,/,j,J,g,e,?]? y  # 这次我打算暂存这一块，于是输入 y
@@ -1,3 +2,4 @@
 something
 something else
 something again
+2
Stage this hunk [y,n,a,d,/,K,g,e,?]? n   # 这块下次提交，所以先不存，于是输入 n

# 现在就可以先进行第一次 commit 了。
$ git commit -m "Added first line"
[master cef3d4e] Added first line
 1 files changed, 1 insertions(+), 0 deletions(-)

# 然后就可以提交其他改动，如果还需要分，那就再重复上面的 git add --patch 命令
$ git commit -am "Added last line"
[master 5e284e6] Added last line
1 files changed, 1 insertions(+), 0 deletions(-)
```