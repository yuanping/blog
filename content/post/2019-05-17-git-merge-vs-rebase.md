---
title: Git Merge VS Rebase
date: 2019-05-17T16:00:00+08:00
slug: git-merge-vs-rebase
tags: ["Git", "技术分享"]
categories: ["技术分享"]
---

Git rebase命令因为会修改提交历史，所以一般推荐新手不要随意使用，但是如果理解它，谨慎的去使用，会给你在开发过程中带来很多方便。这篇文章，我们对比rebase与merge 两个命令，以及示例在工作中一些常用的使用场景。

# 概念上区别

首先要明白的是rebase其实和merge一样，解决的是同一个问题。两个命令都是把一个分支上的修改合并到另一个分支上，只不过使用的是不同的方式。

我们考虑下面这样一个典型的场景，你打算开发一个新功能，从master分支上check out出一个feature分支，在这个分支上提交这代码。与此同时，其他的同事为master分支贡献了一些代码，做了些提交。

![rebase-01](https://magnet-file.qn.cichang.net/yuanping/blog/git-rebase-01.png)

我们假设你现在需要使用master分支上新提交的代码，为了把master分支上新提交的代码整合到你的feature分支上，现在有两个选择：merge 或是 rebase。

## 使用 Merge

最简单的方式当然是把master分支merge到feature分支上来，使用下面的命令

    git checkout feature
    git merge master

当然你也可以使用一行命令

    git merge feature master

这将会在feature分支上创建一个新的"merge commit" ，在查看提交历史时，可以看到，最终你的分支结构会是下面这样

![rebase-02](https://magnet-file.qn.cichang.net/yuanping/blog/git-rebase-02.png)

Merge命令很好，是因为它是一个不破坏的操作。已经存在的分支，并不会有任何的改变，消除了一些使用rebase命令的潜在陷阱。

当然另一方面，每当你往feature分支上整合其他提交时，也额外多了一个合并提交。如果master分支比较活跃，在你切出feature分支后有很多提交，那么你每merge一次master都会产生一个merge commit。虽然可以通过git log的一些参数过滤掉，但是还是让其他开发人员看你的提交历史时有点费劲。

## 使用Rebase

使用merge的另一个可选就是rebase，你可以使用下面的命令完成

    git checkout feature
    git rebase master

这将会移动整个feature分支的提交到master分支最新提交的末尾。而且与merge不同的是，rebase命令会重写提交的历史，通过基于之前的每个提交创建一个崭新的提交。

![rebase-03](https://magnet-file.qn.cichang.net/yuanping/blog/git-rebase-03.png)

rebase的主要优点是，你将获得一个非常清晰的项目提交历史。首先，它没有像merge命令那样额外的merge commit。其次，从上图可以看出，你可以获得一个非常清晰的线性提交历史。当你使用一些图形工具查看历史时，也比较清晰，没有很多fork。

当然，事物都有两面性，缺点就是在安全性与追溯性上会有隐患。如果你不遵循rebase的黄金规则（下面会描述）重新项目的提交历史，可能是灾难性的。因为修改了提交，有时没办法追溯之前真正的操作，什么时候合并的等信息。

## 交互式rebase

交互式rebase给你了机会可以修改提交，甚至更改提交的顺序。这是个非常强大的工具，因为有可以有更多完全的控制去修改。典型的使用场景是，在把feature合并到master之前用它来清理一些凌乱的提交。

使用交互式rebase，只需要传一个i参数就可以了

    git checkout feature
    git rebase -i master

将会打开一个文本编辑器，展示出所有的提交

    pick 33d5b7a Message for commit #1
    pick 9480b3d Message for commit #2
    pick 5c67e61 Message for commit #3

这个列表将定义执行Rebase命令之后的将是什么样子的。通过改变顺序与修改pick命令，你将可以得到完全你想要的结果。比如，第二个提交实际只是对第一个提交修复了一个小问题，这时你可以使用fixup命令，将第二个与第一个提交合并成一个。

    pick 33d5b7a Message for commit #1
    fixup 9480b3d Message for commit #2
    pick 5c67e61 Message for commit #3

当你保存并关闭文件，Git将会根据这些指令执行rebase，得到的结果如下

![rebase-04](https://magnet-file.qn.cichang.net/yuanping/blog/git-rebase-04.png)

消除一些不是很重要的提交，会让你的feature分支历史看起来非常清晰与容易理解。这是merge命令所不能做到的。

## Rebase的黄金规则

一旦你明白了rebae是什么之后，最重要的是要学会什么时候不适合用。黄金规则其实很简单，就是不要在公开的分支使用它。公开的分支是指，你推到了远端，其他的同事可以访问这个分支，一起协作完成一些工作。

在你每次要执行rebase命令时，一定要先想一下，有其他的同事在看这个分支或一起协作吗？如果答案是”是“，那么千万不要使用rebase。当然如果feature分支还没推到远端，这时你可以放心的使用，修改优化提交历史。

## Force-Pushing

如果你之前已经把feature分支推到了远端，并且其他同事也并没有在这个分支上工作，这时你执行rebase之后，当push到远端时，Git会提示你的分支与远端有冲突，不可以push。但是，你可以通过传 `force` 参数，强制推到远端。

    git push --force

这个命令会把你本地的仓库覆盖远端的仓库，所以需要特别的小心执行这个命令，除非你很清楚你在做什么。

# 使用场景

## 整理本地分支

使用rebase的最典型的场景是，你整理你的本地feature分支。比如你从master分支切出来后，做了很多次的提交，有一些只是修复上次提交的bug，有的是不是很重要的提交，想合并到下一个提交里。为了给Code Review的人一个干净清晰的提交历史，这时最适合使用rebase命令。

在正在进行的开发分支上，周期性的执行交互式的rebase，可以确保你的每次提交都非常有意义。

## 修改提交

rebase不一定需要基于另一个分支，也可以是某个commit。比如，你可以把最近三个提交合并成为一个提交，可以执行下面的命令

    git checkout feature
    git rebase -i HEAD~3

HEAD~3 表示基于当前分支的往前的第三次提交。

![rebase-05](https://magnet-file.qn.cichang.net/yuanping/blog/git-rebase-05.png)

# 总结

如果你喜欢整洁、线性的提交历史，去掉不必要的merge commit，那么当需要从一个分支整合一些提交到你的分支时，你应该使用rebase而不是merge。

如果你喜欢保留你项目的全部的提交历史，避免重写公开分支的提交的风险，你应该使用merge。

最重要的是，你需要在不同的场景下正确的选择合适的命令。