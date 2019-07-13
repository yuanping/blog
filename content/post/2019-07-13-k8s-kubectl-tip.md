---
title: Kubernetes访问某个pod的总结
date: 2019-07-13T16:00:00+08:00
slug: k8s-kubectl-exec-pod-tip
tags: ["Kubernetes", "Shell", "技术分享"]
categories: ["技术分享"]
---

# 场景

当应用部署在Kuternetes集群时，有时我们需要某个pod中执行一些命令或查看一些内容，比如需要通过Rails的console查看些数据。

操作步骤可分三步：

1. `kubectl get pods -n magnet` # 首先列出某个命名空间里目前运行中的所以pods
2. `kubectl exec -it pod_name -- bash` # 找到想访问的pod，复制pod的名称，进入pod的命令行
3. `rails console` # 在pod中，进入rails的console

如果经常有这个需求，那么每次都要这样做就比较麻烦了，有简单的办法吗？可以执行一条命令就可以吗？我们来一步一步的优化！

# kubernetes 的命令优化

## kubectl get pods 优化

`kubectl get pods -n magnet` 会返回magnet命名空间下的所有pods，如果部署的服务比较多，并且每个实例都有多个的话，往往会是一个很长的列表。

我们可以通过标签，对服务进行分组、分类，这样就可以通过标签来进行过滤筛选，只列出我们需要的内容。关于k8s的标签更多的操作，可以看官方的[文档](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)。

`kubectl get pods -n magnet -l role=web,name=web-api` 这样就筛选出来标签是 `role=web` 并且 `name=web-api` 的pods。
输出的结果类似下面这样：

```
NAME                   READY   STATUS    RESTARTS   AGE
magnet-web-api-c4qjw   1/1     Running   0          22h
magnet-web-api-n8jk8   1/1     Running   0          22h
magnet-web-api-sbbdh   1/1     Running   0          22h
```

如果想把第一步与第二步的命令合成一个，就需要把一个命令的输出结果，放入第二个命令的 `pod_name` 变量里。我们只想要前面的名字，查资料后，可以通过 `-o name` 参数，支取pod的名称。 执行 `kubectl get pods -n magnet -l role=web,name=web-api -o name` ，输出结果类似下面：

```
magnet-web-api-c4qjw
magnet-web-api-n8jk8
magnet-web-api-sbbdh
```

但是我们只是想要刚才的输出结果的第一行的最开始部分，即(`magnet-web-api-c4qjw`)。我们可以通过加格式输出参数得到：
`kubectl get pods -n magnet -l role=web,name=web-api -o jsonpath='{.items[0].metadata.name}' `
输出结果类似下面：

```
magnet-web-api-c4qjw
```

OK，我们现在已经得到了`pod_name`了，现在只需把它传递到第二步的命令里就可以了。

## 执行一条命令进入pod

我们只需把第二步命令`kubectl exec -it pod_name -- bash` 的 `pod_name` 换成 
`kubectl exec -it $(kubectl get pods -n magnet -l role=web,name=web-api -o jsonpath='{.items[0].metadata.name}' ) -- bash`

这样就可以通过一条命令，进入指定服务实例的第一个pod了。

## 进入pod并执行某个操作

因为进入到指定pod的目的是要进入 Rails console，那么我们可以通过追加 `-c "rails c"` 来实现。把第三步的命令合并成一条，这样就可以直接达到目标：

`kubectl exec -it $(kubectl get pods -n magnet -l role=web,name=web-api -o jsonpath='{.items[0].metadata.name}' ) -- bash -c "rails c"`

但是，每次都把输入这么长的命令那太麻烦了，我们定义一个自定义的shell命令吧。

# 自定义Shell命令

## 找个地方存放我们的命令

我们在home目录创建一个隐藏文件，用来存放命令。使用`.`开头的文件名，只有通过终端才可以访问，这样也可避免误删除。
```
cd ~
touch .custom_command.sh
```
使用你最喜欢的编辑器，打开它，录入下面的代码：
```
#!/bin/bash
function staging_console() {
    kubectl exec -it $(kubectl get pods -n magnet -l role=web,name=web-api -o jsonpath='{.items[0].metadata.name}' ) -- bash -c "rails c
}
```

## 修改文件的执行权限

默认创建的新文件只有只读权限，我们加上执行权限：
`chmod +x .custom_command.sh`

## 让我们的命令在Terminal中可用

通过这个命令，可以让我们刚定义的命令立即可用 `source ~/.my_custom_commands.sh`。但是当你下次打开新的Terminal时，又不可用了，还需要再执行一次。
为了让每次打开Terminal都可用，我们需要把刚才的命令添加到Shell的启动文件中。

Bash是UNIX系统默认的Shell，我个人更喜欢ZSH，那么使用编辑器打开 `~/.zshrc` 文件，把这个命令加到最后一行或者你喜欢的地方 `source ~/.my_custom_commands.sh`。

好的，现在在Terminal中输入 `staging_console`，就会发现直接打开了指定pod的Rails console。

## 给命令加个别名

如果觉得每次输入 `staging_console` 还是麻烦，那么可以通过别名的方式简化

`alias sc=staging_console`

现在在Terminal中输入 `sc`，就可以了。

想了解关于自定义命令更详细的说明与步骤，可以看这篇[文章](https://medium.com/devnetwork/how-to-create-your-own-custom-terminal-commands-c5008782a78e)，写的非常的棒。