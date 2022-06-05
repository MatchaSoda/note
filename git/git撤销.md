# git撤销
### git文件状态
```
       原始内容
  修改    ↓   
       工 作 区
  暂存    ↓    git add
       暂 存 区
  提交    ↓    git commit
       本地仓库
  推送    ↓    git push
       远程仓库
```
### 已修改 未暂存
已经修改了文件，还未进行``git add``。  
即**工作区**的内容不想要了。
#### 恢复方法
使用以下任意命令
```bash
git checkout .
```
```bash
git checkout -- <FILENAME>
```
```bash
git reset --hard
```
### 已暂存 未提交
已经进行了``git add``，还未进行``git commit``  
即**暂存区**的内容不想要了
#### 恢复方法
使用以下任意命令
```bash
git reset #默认参数[--mixed][HEAD]，回退到最后一次提交，工作区文件内容保持不变。
git checkout . #按需使用，删除工作区的修改
```
```bash
git reset HEAD -- <FILENAME> #仅回退某个文件
```
```bash
git reset --hard #撤销工作区中所有未提交的修改内容，将暂存区与工作区都回到之前的版本
```
**注意：**``git reset --hard [HEAD]``用来回退版本，工作区、暂存区、提交内容都会被删除，危险！
### 已提交 未推送
已经进行了``git commit``，还未进行``git push``  
#### 恢复方法
按需使用以下任意命令：
1. 若只修改上次提交的信息，可以直接使用``git commit --amend``。

2. 使用``git rebase -i``来手动编辑提交（推荐）  
    ```bash
    git rebase -i [HEAD]
    ```
    这条命令会列出HEAD到输入参数的所有提交，根据提示，将``pick``改成``e``后可编辑该提交，修改代码后使用``git commit --amend``去修改提交信息，改好之后用``git rebase --continue``完成修改。  
   


3. 使用远程仓库覆盖本地仓库（不推荐）
    ```bash
    git reset --hard origin/master
    ```
### 已推送
已经进行了``git push``
#### 恢复方法
回滚本地仓库，强制推送覆盖远程仓库
```bash
git reset --hard HEAD^
git push -f
```
### 其他情况
丢弃某个节点后的全部提交
即HEAD指针指向该节点
```bash
git reset --hard <COMMITID>
```