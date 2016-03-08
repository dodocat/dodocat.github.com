---
layout: post
title: "Git 分支模型实践--解决分支线性的问题"
summary:
comments: true
cover: /image/git-branch.png
---

# 0. Origin

[A successful Git branching model] 一切起源于这篇文章, 后来的各种 git flow 基本都符合这篇文章的思想.
文章给大家带来这张图
![origin_gitflow.png]

本文以 [A successful Git branching model] 为基础, 探讨保持分支线性的问题.

# 实践

在我的实践过程中, 确定了以下关键原则.
* Git 仓库是分布式, 无中心的.
* 但是希望我们的分支模型里, **同一个** 分支是有中心保持线性的.
* develop 分支的提交粒度是该特性.

拥有了利器, 我们开始工作了:

``` bash
git init
git checkout -b develop
git checkout -b feature1
git commit -m '(╯‵□′)╯︵┻━┻'
git commit -m "┬─┬ ノ( ' - 'ノ)"
git checkout develop
git merge feature1 --no-ff
```

此时我们的分支大概长成这样

```
*   89c7380 - (HEAD, develop) Merge branch 'contents' into develop (36 seconds ago)
|\
| * 5637569 - (contents) add git command (55 seconds ago)
| * 642e6dc - add title (2 minutes ago)
|/
* 43bf08a - (master) scanner online (3 minutes ago)
```

再复杂一点

```
#初始化
git init
git commit
git checkout -b develop
# 开始开发 content
git checkout -b content
git commit
# 此时我们需要去开发另外一个特性 extral
git checkout develop
git checkout -b extral
git commit
# extral 开发完成
git checkout develop
git merge --no-ff extral
# 回来继续开发 content
git checkout content
git commit
# 完成
git checkout develop
git merge --no-ff content
```

此时我们的分支长成这样

```
*   316e0ca - (HEAD, develop) Merge branch 'another' into develop (25 seconds ago)
|\
| * d6595ab - add extral command (50 seconds ago)
| * e0f3b56 - another branch (3 minutes ago)
* |   3ae3288 - Merge branch 'extral' into develop (2 minutes ago)
|\ \
| |/
|/|
| * 4203945 - add extral file (2 minutes ago)
|/
*   89c7380 - Merge branch 'contents' into develop (7 minutes ago)
|\
| * 5637569 - add git command (7 minutes ago)
| * 642e6dc - add title (9 minutes ago)
|/
* 43bf08a - (master) scanner online (10 minutes ago)
```

完美符合理想的分支模型. 并且保留了并行开发的记录.

这个时候我们加入了另外一个小伙伴. 分别在各自的按照这个流程工作.
这时候就有可能出现 `*1 ahead* 2 *behind*` 类似的情况, 此时意味着本地的 `develop` 分支与远程的 develop 分支产生了分叉. 必须先 pull. 但是如果直接 pull 则会产生这样的结果.
`Merge branch 'develop' of <REMOTE> into develop`
此时我们的 develop 分支不再是一条直线. 而且产生了一个无意义的合并节点. 破坏了 develop 分支以特性为粒度按照代码提交时间排序的目标原则.
这时候需要`rebase`.  但是直接执行 `git pull --rebase` 会删除刚才特性分支合并入 develop 分支的节点.  像这样

```
A
| \
B  H
|  |
C  G
|  |
D  F
| /
E
```

`git pull --rebase` 后

```
H
|
G
|
F
|
X <- 远程的新节点
|
B
|
C
|
D
|
E
```

想要保留节点就要加入 `preserve` 参数.

```
git fetch origin
git rebase origin/develop --preserve-merges
```

或者

```
git config pull.rebase preserve
git pull
```

# 总结
* 分支模型的确立是从工程实践出发的. 我们需要多人协同合作, 如果合并进入 develop 分支的都由 pull request 进行. 这时的 develop 分支完全就是中心, 自然是完全线性的. 但是实际情况往往更复杂一些, 我们没有选择完全死板的遵守 pull request 流程. 同时我们又希望保证 develop 分支的线性, 就引入了上面问题, 并解决了它.
* 这里其实就是 `merge` 和 `rebase` 的使用选择问题, 推广开就是我们在**同一个**分支的合并上使用 `rebase`, 在不同分支的合并上使用 `merge` 同时使用 --no-ff 保留分支历史, 保证提交粒度. (当然 master 和 develop 分支外的其它地方自由选择即可).
* 如果特性分支只有一个 commit 节点, 那么其粒度与合并节点相等, 可以选择不使用 --no-ff (新版的 [git flow] 中是这么做的)

> 参考资料:
* [A successful Git branching model]
* [Git flow]
* [5.1 分布式 Git - 分布式工作流程]
* [Learn Git Branching]
* [Make git pull --rebase preserve merge commits]

[A successful Git branching model]:http://nvie.com/posts/a-successful-git-branching-model/
[Git flow]:https://github.com/nvie/gitflow
[origin_gitflow.png]:http://upload-images.jianshu.io/upload_images/43382-3dbfae20399b9280.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "起源"

[5.1 分布式 Git - 分布式工作流程]:https://git-scm.com/book/zh/v2/%E5%88%86%E5%B8%83%E5%BC%8F-Git-%E5%88%86%E5%B8%83%E5%BC%8F%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B
[Learn Git Branching]:http://pcottle.github.io/learnGitBranching/

[Make git pull --rebase preserve merge commits]:(http://stackoverflow.com/questions/11863785/make-git-pull-rebase-preserve-merge-commits)]:http://stackoverflow.com/questions/11863785/make-git-pull-rebase-preserve-merge-commits
