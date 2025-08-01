# 常用指令
```bash
git clone remote-url localdir # 把默认的库在本地文件夹的名字，改成localdir
git clone remote-url --bare # 把远程仓库克隆为bare无工作区仓库
git -c user.name="X" -c user.email="x@xx.com" clone # 临时身份克隆，具体啥意思，没用过
git branch -M main # 把当前分支名改成main
git branch --set-upstram-to oragin/br1 local-br # 如果远程的br1和本地的local-br不一样，在push的时候还是不行，还得指定 分支名，真是奇了怪了
git push -u oragin lcoal-br:remote-br
git push -u oragin local-br # 如果不加":"，代表本地分支名

# 创建分支
git brach <branch-name>
git checkout -b <branch-name>
git switch -c <branch-name>
git branch -d <branch-name> # 删除分支
git branch -D <branch-name> # 删除分支，即使未合并（没有理解）
git branch -m <old-branch-name> <new-branch-name> # 换名字
git branch -m <new-branch-name> # 更改当前分支的名字
git branch --forece <branch-name> <another-branch-name>
# 强行把branch-name分支，指向another-branch-name分支
git push origin --delete <branch-name> # 删除远程分支
git push origin :<branch-name> # 同上（老用法）
```

# git 原理
## 第一部分
有一个存储数据库，理解为KV数据库，K=HashID,V=Object，通常这些对象是如何组织起来的呢，链表堆栈，一般来说，任何一个对象都会有一个指针(hashid)指向对象，放在objs文件夹中，这个实际数据保存的地方。

## 第二部分
各种指向，指向栈顶，就是常说的HEAD指针，放在refs文件夹中

## 第三部分
配置信息，比如 .ignore啥的，

# git 原则
解决冲突问题

解决安全问题，尽可能消除那些命令，会破坏数据关系的，让参与者看到的画面不一样的这些命令

所有的管理，基本都是围绕指针进行的

保证所有提交的对象链，是在一起的，形成了一个DAG图，单项无环图

比如你的commit不在这个链上了，由于各种原因导致脱链了，那就出问题了，



# git概念解析

## 什么是bare仓库
没有工作区的仓库，为啥远程仓库，都是bare仓库呢？大家来共享的，都是要通过push和pull来完成同步一致性的， 不受本地工作区的影响，如果远程库的工作修改了，你就会推送失败，你还要去服务器上进行修正，一般来说，服务器是不能随便进入的，所以，不要工作区，大家共同管理，方便。
其实有工作区的仓库，也可以push，只要你们两者和本地的同不好就可以，如果远程的工作区不提交，基本也没啥影响。

## 暂存区
暂存区的数据独立于仓库中的数据。这是一个数据也存在数据库中。增城区也是各种指针。


## Head^ 和 Head~的区别

```plaintext
Head^ 指向父节点 === Head^1
Head~ 指向父节点 === Head~1

Head^2 指向父节点的第一个兄弟节点，也相当于父层的第二个节点 === Head^1
Head~2 指向父节点的父节点 === Head~2

Head^^2 指向父节点的父节点的第一个兄弟节点，也相当于父父层的第二个节点 === Head^^2
Head~~2 指向父节点的父节点的父节点的父节点 === Head~~~~
```

# 常用指令
```bash
git reset --hard # 从仓库区更新到，暂存区，更新到工作区
git reset --mix  # 从仓库区更新到，暂存区，工作区不变
git reset --soft # 更新仓库区的指针，暂存区不变，工作区也不变

git restore --source=<commit-hash> <file-path>
# 放弃工作区变更的文件的内容，恢复到未变更状态

git commit --date # 可以修改时间 
            --author # 可以指定 用户名和邮箱
git commit --amend --no-edit # 保持commit信息不变  
            --reset-author # 修改作者时间和提交时间
            --date="now" # 修改作者时间和提交时间（同上）

git rebase  # 变基 可以修改每个节点的信息，一旦修改某个节点，之后的节点的hash值，全部要重新计算
# 可以调整提交范围内的顺序也可以合并临近的提交 squash 就是干这个事的

git rebase --onto newbase from to
# 和 git rebase的差异性在于
# rebase 只是两个分支而言
# onto 带有一个范围，前开后闭

git rebase feat/xxx  # 当前branch是main
# 如果 feat/xxx 的base就是main，也就是main之上，只有feat/xxx的提交
# 也即，main分支落后于feat/xxx分支
# 那么，执行上述命令就是fast-forward，
# 就相当于，git merge --fast-forward, merge 默认的是也ff

git rev-parse head~~^
# 查看head~~^的commitid的值

git log --format='%H %<(20,trunc)%s'
## 显示的内容在20个字符后截断，也就是最长显示20个字符内容

git log --reverse --oneline | head -1 
git log --oneline | tail -1 
# 获取commit最初的，也即第一条提交

git log --oneline --follow -- app.txt
# 获取 app.txt 最早出现的提交

git log <branch-name> ...
# 查看另一个分支的log

git log --pretty=full
# 显示所有信息，这里有两个信息，author和committer，这两者可以是不一样的

git restore .
# 放弃本地所有的修改
git restore --stage .
# 放弃暂存区所有的修改

git clean -nfdx
# n 模拟看看那些将被清除
# f 清除本地未跟踪的文件
# f 清除本地未跟踪的目录
# x 清除本地被忽略的文件

git checkout --ours <filenames>
git checkout --theirs <filenames>
# 一旦发生冲突，这两个可以任意检出，数据都还在，直到提交到暂存区中
# theris: stashed or remotes
# ours: current

git cherry-pick <commit-id> # 某个提交
git cherry-pick <commit-id1> <commit-id2> # 多个提交
git cherry-pick <commit-id1>^..<commit-id2> # 范围id1是旧的，id2是新的
# 如果发生冲突，git add . 后，不需要手动commit，执行git cherry-pick --continue 会自动进行commit
# cherry-pick多个提交时，会对应一对一多个提交，不会进行合并

git diff <commit-id1> <commit-id2> --ignore-cr-at-eof # 忽略cr的不同
git diff --ignore-space-at-eof
git diff -b/--ignore-space-change
git diff -w/--ignore-all-space
git diff --diff-filter=M # 只比较改动的文件

git tag -a v1.0.0 -m "version 1.0.0" # 创建标签
git tag -d <tagname> # 删除标签
git tag --list # 列出标签
git push origin v1.0.0 # 推送到远程标签
```
## 标签和分支的不同
| 特性         | 标签（Tag）| 分支（Branch）|
|--------------|---------------------|----------------------------|
| **目的**     | 标记特定时间点的状态（如版本发布） | 支持持续开发的代码线             |
| **可变性**   | 固定不变（指向特定提交）          | 可移动（随新提交向前移动）         |
| **生命周期** | 永久性标记（除非手动删除）        | 临时性（通常合并后删除）           |
| **数量关系** | 一个提交可有多个标签              | 一个提交只能属于一个分支（HEAD位置）|

标签：静态指针  
分支：动态指针  
两者其基础机制，是一样的，只是Tag增加了很多约束

## 高级指令--对历史数据-批量调整

`git filter-branch 批量更新节点信息`

示例1： 修改用户名和邮箱：
```bash
git filter-branch --env-filter '

OLD_EMAIL="old@xxxx.com"
NEW_NAME="you-new-name"
NEW_EMAIL="new@yyyy.com"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$NEW_NAME"
    export GIT_COMMITTER_EMAIL="$NEW_EMAIL"
fi

if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$NEW_NAME"
    export GIT_AUTHOR_EMAIL="$NEW_EMAIL"
fi

' --tag-name-filter cat -- --branches --tags
```

示例2： 文件夹和文件处理
```bash
git filter-branch --index-filter
```

```bash
git mv xxx xxx # 不能用在index-filter上
git rename --cached xxx xxx # 这个可以
```

tree-filter的用法
`git filter-branch --tree-filter`

```bash
git 
```


# 惊奇点
你先对本地的变化进行stash，然后你对本地进行了变基，导致commit都变了，再次stash pop时，是不会merge的，会提示如下：直接abort了
```bash
long@dev001:/mnt/d/dev/git$ git -v
git version 2.43.0
long@dev001:/mnt/d/dev/git$ git stash pop
error: Your local changes to the following files would be overwritten by merge:
        git.md
Please commit your changes or stash them before you merge.
Aborting
```
因为这已经不是同一个代码库了，我想了好久，这个解释应该是正确的


# QA
前提条件：当前本地库和远程库状态一致  
然后，用git commit --amend --author 只修改了用户和邮箱  
然后，再调用 git pull --rebase，
```bash
long@dev001:/mnt/d/dev/git$ git pull --rebase
warning: skipped previously applied commit bf9a508
hint: use --reapply-cherry-picks to include skipped commits
hint: Disable this message with "git config advice.skippedCherryPicks false"
Successfully rebased and updated refs/heads/main.
```
结果，提示 直接 ***跳过原来的修改*** ，直接给 ***复原*** 成了原来的  
这是 ***为啥*** 呢？