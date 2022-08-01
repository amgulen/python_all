# git操作（工作中实际开发流程）

## 一，工作中拉取代码到提交代码流程

```bash
1. 在项目主页fork正式仓库
2. git clone 自己fork出来的仓库地址 - git clone git@github.xxx.com:xxx/xxx.git
3. git remote add upstream 正式仓库地址 - git remote add upstream git@github.xxx.com:xxx/xxx.git
4. git remote -v  # 可用此命令查看远端仓库状态
5. git fetch --all # 拉取本地没有的远程内容（其他分支/更新）
6. git checkout -b 分支名 拉出一个新的修复bug分支 - git checkout -b bug_fix
    git branch -a # 查看所有分支
    git branch -v # 查看本地分支
    - git checkout -b hotfix upstream/master # 以后每次切出一个干净的分支开发不同任务
    - git pull upstream master # 更新新切出的分支（拉正式仓库的更新）
    - cd [submodule_name] && git pull # 更新submodule代码 (项目中使用git submodule时)
# 因为git checkout -- file_name 通常是用来撤销修改的，又用它来创建并切换分支，有点容易让人混淆，所以最新版本的Git提供了新的git switch命令来创建or切换分支，推荐使用
- git switch -c dev  # 创建并切换到新的dev分支，等同于git checkout -b dev
- git switch master  # 直接切换到已有的master分支
7. 修改代码，写单元测试，运行pytest，或别的调试 
    docker exec -it container_name bash 进入容器内
    pytest执行命令：pytest -s 目录/文件名.py::类名::函数名
    添加断点 from builtins import breakpoint-->使用Breakpoint()
    n-->下一步
    q-->推出
    p-->打印
8. git add . 
    - git add xxx1.py xxx2.py
        git commit -m "something"
    - git commit -m "bug fix"
        git push -u origin [feature_branch-name] # 更新本地项目到自己仓库 -u选项自动为您设置上游，相当于记录了push到远端分支的默认值，这样当下次我们还想要继续push的这个远端分支的时候推送命令就可以简写成git push即可
    - git push origin bug_fix
    - git push origin bug_fix -f # 强制push,不是non-fastforword时提示push不上去可以强行push, 少用, 保证代码不出现冲突情况下可以用
9. pull request # github页面操作提PR
10. 申请review代码，讲解代码改动
```

## 二，多个commit合并成一个

```bash
- git rebase -i commit_id, 此时的commit_id为你创建分支时的commit_id
- 在打开的文档里, 第一个保留为pick, 其余的改成squash（s）
- 保存退出
```

## 三，若跟master有冲突/master上有更新, 需要rebase master:

```bash
- git fetch --all
- git rebase upstream/master
- 解决完冲突, git add , git commit
- git rebase --continue 继续合并，合并的过程中，还有可能产生冲突，解决方法同上
- push到自己fork的仓库 git push origin feature_branch
- 申请组员review 再次通过
- squash and merge
- git rebace --abort # 撤销rebace
```

## 四，代码不一致处理方法

```bash
- git fetch upstream
- git checkout master
- git merge upstream/master
```

## 五，删除分支

```bash
# 删除本地分支
- git branch -d branch-name # 删除一个本地分支
- git branch -D branch-name # 强制删除一个本地分支

# 删除远程分支
1. 分两部布
- git branch -r -d origin/branch-name # 第一步 删
- git push origin :branch-name # 第二部 push
2. 一步到位
- git push origin -d branch-name
```

## 六，git checkout不推荐切换分支时使用，应该这么用

```bash
- git checkout .  # 放弃工作区中全部的修改 :撤销不了git add
- git checkout -- file_name  # 放弃工作区中某个文件的修改, 先使用 git status 列出文件，再执行, 撤销不了git add
- git checkout -f  # 不可逆操作，强制放弃index和工作区的改动，直接撤销了git add操作和之前改动，回到解放前
```

## 七，git标签管理

```bash
1.标签默认打在最新提交的commit上
- git tag v1.0   
2.我们也可以指定打在某一个commit上，先用git log --pretty=oneline --abbrev-commit查看commit编号
- git tag v0.9 07f52c6
3.还可以创建带有说明的标签，用-a指定标签名，-m指定说明文字：
- git tag -a v0.1 -m "version 0.1 released" 12c0ea3
4.查看标签
- git tag  # 查看都有哪些标签
- git show v1.0  # 查看标签v1.0的详细内容
5.回滚
- git reset --hard v0.9
# 标签总是和某个commit挂钩。如果这个commit既出现在master分支，又出现在dev分支，那么在这两个分支上都可以看到这个标签
# 推送标签
- git push origin <tagname>  # 可以推送一个本地标签
- git push origin --tags  # 可以推送全部未推送过的本地标签
# 删除标签
- git tag -d <tagname>  # 可以删除一个本地标签
- git push origin :refs/tags/<tagname>  # 可以删除一个远程标签
```

