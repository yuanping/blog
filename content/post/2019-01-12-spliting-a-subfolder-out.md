---
title: 拆分一个子目录到一个新的Git仓库中
date: 2019-01-12T16:03:48+08:00
slug: splitting-a-subfolder-out-into-a-new-repository
tags: ["Git", "技术分享"]
categories: ["技术分享"]
draft: false
---

我们在项目开发中，可能会有这样的需求，某一个子目录的文件比较通用，可以独立成为一个工具包，以备其他项目复用。这时想把这个子目录拆分出来，并且想保留之前的Git提交历史，把它push到新的代码仓库（repository）中，就会用到Git的 `filter-branch` 命令。

我们下面来讲一下具体怎样做：

1. 打开Terminal，cd到你需要创建新仓库的位置，比如 `~/workspaces`

        cd ~/workspaces

2. 克隆包含子目录的代码库

        git clone https://github.com/USERNAME/REPOSITORY-NAME NEW-REPO

    因为 `filter-branch` 会直接改变当前的仓库，所以最好重新克隆一份，并且指定新的项目名称（NEW-REPO）。

3. 进入到刚才克隆下来的项目目录

        cd NEW-REPO

4. 使用 filter-branch 命令过滤出想拆出来的子目录

        git filter-branch --prune-empty --subdirectory-filter FOLDER-NAME BRANCH-NAME

    - `FOLDER-NAME` : 你希望拆出来的子目录
    - `BRANCH-NAME` : 项目的分支，比如 master

    运行上面的命令之后，当前目录应该只剩下指定的子目录里的文件了
5. 在GitHub 或 GitLab上创建一个新的代码仓库(new repository)。复制仓库的URL。
6. 检查一下当前remote的名称与URL

        git remote -v
        origin  https://github.com/USERNAME/REPOSITORY-NAME.git (fetch)
        origin  https://github.com/USERNAME/REPOSITORY-NAME.git (push)

7. 配置新的仓库地址，粘贴第5步的URL。

        git remote set-url origin https://github.com/USERNAME/NEW-REPOSITORY-NAME.git

8. 确认一下remote的URL是否已改变

        git remote -v
        # Verify new remote URL
        origin  https://github.com/USERNAME/NEW-REPOSITORY-NAME.git (fetch)
        origin  https://github.com/USERNAME/NEW-REPOSITORY-NAME.git (push)

9. 将新的代码提交到新的仓库

        git push -u origin BRANCH-NAME