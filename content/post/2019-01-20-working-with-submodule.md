---
title: Git submodule 的使用
date: 2019-01-12T16:03:48+08:00
slug: working-with-submodule
tags: ["Git", "技术分享"]
categories: ["技术分享"]
draft: false
---

我们在工作中，随着项目的发展，可能会遇到一个项目的某些文件可以抽出来作为一个工具包或库，可以被其他项目使用。Git提供了一个工具解决这个问题：`submodule`

Submodule 允许你包含或内嵌一个或多个代码仓库作为一个子目录在你的项目中。不过submodule不一定是最佳的方式，需要根据项目的实际情况来选择，在文章的后面，我会说一些缺点与其他实现方式。

# 添加一个submodule

假设我们现在开发一个汽车的项目，发现公司其他同事之前研发的轮胎项目比较适合，可以直接使用，没有必要再去造一个新的轮子。

![Car](https://magnet-file.qn.cichang.net/yuanping/blog/tesla-car.jpg)

你可以轮胎项目作为汽车项目的一个子模块，在汽车项目中执行下面命令导入：

    git submodule add https://github.com/<user>/tire tire

执行成功后，在你的项目中就会有 `tire` 这个文件夹，如果你的Git版本比较低，打开 tire 文件夹，会发现里面什么也没有，不用担心，再执行下面的命令就可以了。

    git submodule update --init --recursive

当然，如果你是最新版本的Git，Git会自动帮你做这些。

如果顺利，这时你的项目的`tire`文件夹里已经有了轮胎项目的所有代码了。你可以提交代码，应用这些改变。如果你用的是GitHub或GitLab，你会发现 `tire` 文件夹的图标样式与普通的文件夹是有一些区别的，当你点击 `tire` 文件夹，会自动跳转到 `tire` 的代码仓库（repository）里。

在命令行中，你在不同的文件夹中执行git 命令，也有所不同。比如你在汽车项目的普通目录里执行git log与在 tire 目录中执行，会发现显示的是不同的 repository 的提交历史。

    cd ~/projects/car
    git log # log shows commits from Project Car
    cd ~/projects/car/others
    git log # still commits from Project Car
    cd ~/projects/car/tire
    git log # commits from Tire

# 参与一个使用submodule的项目

当有一个新的员工参与Car项目的开发时，使用 git clone 去下载Car项目代码，你会发现项目里 tire 目录中什么也没有。

原因是，Git希望我们显式的去指定是否下载submodule目录中的内容。在这里，你可以使用 `git submodule update --init --recursive` 来解决，但是还有一个更简单的命令，在 `clone` 时，指定一个参数，就可以自动下载 submodule 目录中的所有内容。

    git clone --recursive <project url>

# 抽出submodule作为独立代码仓库

如果你开发一段时间Car项目之后，发现发动机这块代码可能会被其他项目使用，所有想单独创建一个Engine代码仓库，但是想把之前的提交历史也保留，这时就需要用到 `git filter-branch` 来实现。关于 `filter-branch` 的具体使用，可以看这篇[文章](https://yuanping.github.io/2019/01/splitting-a-subfolder-out-into-a-new-repository/)。

第一步，先创建一个 car 项目的拷贝，用来生成 engine 项目。

    cd ..
    cp -r car engine

我们使用 `cp` 带 `-r` 参数，表示拷贝car目录里所有的文件包括文件夹中的文件到 engine 中。

然后进入engine项目，抽出engine目录的内容。

    cd engine
    git filter-branch --prune-empty --subdirectory-filter engine master

这时，engine目录里只有发动机相关的代码了，我们在GitHub上创建好新的代码仓库，然后设置为新的 repository url。

    git remote set-url origin https://github.com/<user>/engine

最后，把代码推到GitHub上就OK了。

现在我们再切换到car项目里，把engine目录换成submodule。

    cd ../car
    git rm -r engine
    git commit -m "Remove engine (preparing for submodule)"

最后，在用之前我们讲的命令，把engine作为submodule添加进来。

    git submodule add https://github.com/<user>/engine engine
    git commit -m "engine submodule"

# 使用submodule的建议

- 在使用submodule把一个repository添加到项目时，需要考虑一下是不是最好的方式，submodule适合一些简单的场景，复杂的场景需要专门的包依赖管理工具。现在每个语言基本都有自己的包管理工具了，可以处理复杂的依赖管理。比如`Ruby`有 `rubygems`, `Node.js` 有 `npm`。
- Git默认不会下载submodule的代码，所以当项目中使用了submodule时，你需要告诉项目伙伴，使用 `git clone --recursive` 命令下载所有的 submodule代码。
- 注意，项目成员不会自动更新submodule里的代码，当你更新了submodule，你需要提醒项目成员使用 git submodule update 来更新，否则会有很多奇怪的现象。
- 需要自己权衡，便利性与代码一致性。简单的情况，比如给 hugo 安装一个新的模板样式，用submodule就非常方便。如果在项目中，希望项目成员使用的submodule代码都一致，需要指定SHA去锁定。