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



