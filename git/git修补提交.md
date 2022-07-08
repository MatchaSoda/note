# git修补提交
在修补提交的时候，可以使用git rebase -i，然后e来编辑某个提交，也可以使用
```
$ git checkout -b wip <commitHash>
```
来在某点创建新分支，之后进行修补，期间可以用``git checkout branch -- file``来获取其他提交的文件。  
修改完后commit，``git commit -m"fixup"``。  
再checkout到原来的分支，rebase到wip上边。

rebase成功的话，git rebase -i 就可以看到在提交线中多了一个fixup点，修补完所有提交之后将所有fixup提交前的pick改为f即可。