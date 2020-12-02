# GIT



### 一、新建分支



#### a.新建本地分支

使用`git branch` 新建一个分支

```bash
$ git branch test
$ git branch
* main
  test
```

不会自动切换到新分支中去，并与当前分支指向同一个提交对象。

使用`git checkout -b` 新建并切换到该分支

```bash
$ git checkout -b temp
Switched to a new branch 'temp'
```

相当于下面两条命令

```bash
$ git branch temp
$ git checkout temp
```

使用`git checkout --track`新建一条检出并跟踪远程分支的本地分支

```bash
$ git checkout --track origin/dev
Branch 'dev' set up to track remote branch 'dev' from 'origin'.
Switched to a new branch 'dev'
```

若在本地没有`dev`分支，则该命令可以简化为`git checkout dev`



#### b. 新建远程分支

使用`git push`新建远程分支

```bash
$ git push origin temp
Total 0 (delta 0), reused 0 (delta 0)
remote: 
remote: Create a pull request for 'temp' on GitHub by visiting:
remote:      https://github.com/16302010049/GitBranch-/pull/new/temp
remote: 
To https://github.com/16302010049/GitBranch-.git
 * [new branch]      temp -> temp
```

该命令是`git push origin temp:temp`的简化



### 二、 删除分支



#### a.删除本地分支

使用`git branch -d`删除本地分支

```bash
$ git branch -d temp
Deleted branch temp (was deb9a66).
```



#### b. 删除远程分支

使用`git push --delete`  删除远程分支

```bash
$ git push origin --delete temp
To https://github.com/16302010049/GitBranch-.git
 - [deleted]         temp
```

也可以使用`git push` 推送一个空分支来删除远程分支

```bash
$ git push origin :temp
To https://github.com/16302010049/GitBranch-.git
 - [deleted]         temp
```



### 三、 查看分支

使用`git branch` 查看本地分支

```bash
$ git branch
* dev
  main
  test
```

使用`git branch --all` 查看所有分支(包括远程分支)

```bash
$ git branch --all
* dev
  main
  test
  remotes/origin/HEAD -> origin/main
  remotes/origin/dev
  remotes/origin/hotfix
  remotes/origin/main
```

使用`git branch -vv` 查看本地分支跟踪的远程分支

```bash
$ git branch -vv
* dev  deb9a66 [origin/dev] Initial commit
  main deb9a66 [origin/main] Initial commit
  test deb9a66 Initial commit
```



### 四、修改分支(本地)

使用`git branch --move`修改分支名称

```bash
$ git branch --move test new_test
$ git branch
* dev
  main
  new_test
```

使用`git branch -u` 增加或修改跟踪的远程分支

```bash
$ git branch -u origin/hotfix
Branch 'new_test' set up to track remote branch 'hotfix' from 'origin'.
```



### 五、 分支合并

#### a.   fast-forward模式

切换到`dev`分支并在上面增加一个提交

```bash
$ git checkout dev 
$ touch a.txt
$ git add .
$ git commit -m "C1"
[dev 5876d0e] C1
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 a.txt
```

查看提交历史

```bash
$ git log --graph --oneline
* 5876d0e (HEAD -> dev) C1
* deb9a66 (origin/main, origin/hotfix, origin/dev, origin/HEAD, new_test, main) Initial commit
```

切换回`main`分支并进行合并

```bash
$ git checkout main
Switched to branch 'main'
Your branch is up to date with 'origin/main'.
$ git merge dev
Updating deb9a66..5876d0e
Fast-forward
 a.txt | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 a.txt
```

从`git merge`命令的输出中，可以看到“Fast-forward”,这表明Git此时采用的是fast-forward模式。这是由于被合并的分支(`dev`)指向的提交是当前分支(`main`)指向的提交的后继，因此Git简单地将`main`的指针向前移动到`dev`分支所指向的提交。

#### b. no-ff (no-forward)模式

首先通过`git reset` 命令将之前的合并回滚

```bash
$ git reset --hard deb9a6
HEAD is now at deb9a66 Initial commit
$ git log --all --graph --oneline
* 5876d0e (dev) C1
* deb9a66 (HEAD -> main, origin/main, origin/hotfix, origin/dev, origin/HEAD, new_test) Initial commit
```

使用`git merge --no-ff`执行合并

```bash
$ git merge --no-ff dev
Merge made by the 'recursive' strategy.
 a.txt | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 a.txt
```

查看此时的提交历史

```bash
$ git log --all --graph --oneline
*   a3783ae (HEAD -> main) Merge branch 'dev' into main
|\  
| * 5876d0e (dev) C1
|/  
* deb9a66 (origin/main, origin/hotfix, origin/dev, origin/HEAD, new_test) Initial commit
```

在这种模式下，即使被合并分支(`dev`)可以顺着当前分支(`main`)到达,也会创建一个新的提交对象，以便形成更清晰的合并历史和分支状态。

#### c. three-way merge模式

首先通过`git reset` 命令将之前的合并回滚

```bash
$ git reset --hard deb9a6
HEAD is now at deb9a66 Initial commit
$ git log --all --graph --oneline
* 5876d0e (dev) C1
* deb9a66 (HEAD -> main, origin/main, origin/hotfix, origin/dev, origin/HEAD, new_test) Initial commit
```

在当前分支(main)上新创建一个提交

```bash
$ touch b.txt
$ git add .
$ git commit -m "C2"
[main 6c4fe1c] C2
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 b.txt
```

查看此时的提交历史

```bash
$ git log --all --graph --oneline
* 6c4fe1c (HEAD -> main) C2
| * 5876d0e (dev) C1
|/  
* deb9a66 (origin/main, origin/hotfix, origin/dev, origin/HEAD, new_test) Initial commit
```

由于要合并的`dev`分支指向的提交(C1)并不是`main`分支指向的提交(C2)的后继，也就是提交历史发生了分叉，此时Git会进行三方合并。此时，Git需要寻找C1与C2的公共祖先作为合并的基准。通过`git merge-base`命令查看该基准

```bash
$ git merge-base dev main
deb9a66e69699929ed9efdcfcad783f70718433e
```

可以看出，基准也就是初始提交。

进行合并并查看提交历史

```bash
$ git merge dev
Merge made by the 'recursive' strategy.
 a.txt | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 a.txt
$ git log --all --graph --oneline
*   7594c92 (HEAD -> main) Merge branch 'dev' into main
|\  
| * 5876d0e (dev) C1
* | 6c4fe1c C2
|/  
* deb9a66 (origin/main, origin/hotfix, origin/dev, origin/HEAD, new_test) Initial commit
```

#### d. squash模式

首先通过`git reset` 命令将之前的合并回滚

```bash
$ git reset --hard 6c4fe1c
HEAD is now at 6c4fe1c C2
$ git log --all --graph --oneline
* 6c4fe1c (HEAD -> main) C2
| * 5876d0e (dev) C1
|/  
* deb9a66 (origin/main, origin/hotfix, origin/dev, origin/HEAD, new_test) Initial commit
```

执行`git merge --squash`命令并提交

```bash
$ git merge --squash dev
Squash commit -- not updating HEAD
Automatic merge went well; stopped before committing as requested
$ git commit -m "C3"
[main cd6f64c] C3
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 a.txt
```

查看此时的提交历史

```bash
$ git log --all --graph --oneline
* cd6f64c (HEAD -> main) C3
* 6c4fe1c C2
| * 5876d0e (dev) C1
|/  
* deb9a66 (origin/main, origin/hotfix, origin/dev, origin/HEAD, new_test) Initial commit
```

squash模式会将`dev`分支上的提交压缩到一个提交对象中，从git历史中看不出合并，但引入了`dev`分支中所有的变更，提交历史也显得比较简洁。



### 六、远程操作

使用`git remote` 查看查看远程仓库列表

```bash
$ git remote
origin
```

使用`git remote show <remote>`可以看到某个远程仓库的更多信息，包括仓库的URL及跟踪分支的信息等:

```bash
$ git remote show origin
* remote origin
  Fetch URL: https://github.com/16302010049/GitBranch-.git
  Push  URL: https://github.com/16302010049/GitBranch-.git
  HEAD branch: main
  Remote branches:
    dev    tracked
    hotfix tracked
    main   tracked
  Local branches configured for 'git pull':
    dev      merges with remote dev
    main     merges with remote main
    new_test merges with remote hotfix
  Local refs configured for 'git push':
    dev  pushes to dev  (fast-forwardable)
    main pushes to main (fast-forwardable)
```

使用`git remote add <shortname> <url>`添加远程仓库

```bash
$ git remote add origin https://github.com/16302010049/GitBranch-.git
$ git remote
origin
```

使用`git push <remote> <branch>`推送本地提交

```bash
$ git push origin main
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (5/5), 470 bytes | 470.00 KiB/s, done.
Total 5 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), done.
To https://github.com/16302010049/GitBranch-.git
   deb9a66..cd6f64c  main -> main
```

使用`git fetch <remote>`获取远程仓库的数据

```bash
$ git fetch origin
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 5 (delta 1), reused 5 (delta 1), pack-reused 0
Unpacking objects: 100% (5/5), done.
From https://github.com/16302010049/GitBranch-
   deb9a66..cd6f64c  main       -> origin/main
```

使用`git pull` 获取并合并远程分支的提交

```bash
$ git pull
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (1/1), done.
remote: Total 3 (delta 1), reused 3 (delta 1), pack-reused 0
Unpacking objects: 100% (3/3), done.
From https://github.com/16302010049/GitBranch-
   cd6f64c..96a7ac3  main       -> origin/main
Updating cd6f64c..96a7ac3
Fast-forward
 b.txt | 1 +
 1 file changed, 1 insertion(+)
```



### 七、Git工作流



#### a.  集中式工作流

![集中式工作流。](https://git-scm.com/book/en/v2/images/centralized_workflow.png)

集中式系统中通常使用的是单点协作模型——集中式工作流。 一个中心集线器，或者说 **仓库**，可以接受代码，所有人将自己的工作与之同步。 若干个开发者则作为节点，即中心仓库的消费者与中心仓库同步。

这意味着如果两个开发者从中心仓库克隆代码下来，同时作了一些修改，那么只有第一个开发者可以顺利地把数据推送回共享服务器。 第二个开发者在推送修改之前，必须先将第一个人的工作合并进来，这样才不会覆盖第一个人的修改。



#### b. 集成管理者工作流

![集成管理者工作流。](https://git-scm.com/book/en/v2/images/integration-manager.png)



它的大致流程如下:

1. 项目维护者推送到主仓库。
2. 贡献者克隆此仓库，做出修改。
3. 贡献者将数据推送到自己的公开仓库。
4. 贡献者给维护者发送邮件，请求拉取自己的更新。
5. 维护者在自己本地的仓库中，将贡献者的仓库加为远程仓库并合并修改。
6. 维护者将合并后的修改推送到主仓库。



#### c.主管与副主管工作流

![主管与副主管工作流。](https://git-scm.com/book/en/v2/images/benevolent-dictator.png)



它的大致流程如下:

1. 普通开发者在自己的主题分支上工作，并根据 `master` 分支进行变基。 这里是主管推送的参考仓库的 `master` 分支。
2. 副主管将普通开发者的主题分支合并到自己的 `master` 分支中。
3. 主管将所有副主管的 `master` 分支并入自己的 `master` 分支中。
4. 最后，主管将集成后的 `master` 分支推送到参考仓库中，以便所有其他开发者以此为基础进行变基。

