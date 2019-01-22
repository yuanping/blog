---
title: Shell 脚本简介
date: 2019-01-05T16:03:48+08:00
slug: shell-script-introduction
tags: ["Shell", "Bash", "技术分享"]
categories: ["技术分享"]
draft: false
---

Shell脚本仅仅是你写在一个文件里的一组命令，可以一次一起执行。如果你以前了解过Dos的批处，其实差不多一个概念。也就是用各类命令预先放入到一个文件中，方便一次性执行的一个程序文件，主要是方便管理员进行设置或者管理用的。当然bash脚本能比它更强大。

> Shell脚本是一个通过Unix Shell运行的计算机程序，一个命令行的解释器。我们认为Shell脚本是一种脚本语言。Shell脚本的一些典型的操作比如文件操作，程序执行，打印文本等。

Unix有多种Shell，每一个都可以写一本书去讲，在这篇文章里，我们只是讲一些bash脚本的基本元素。

# 基本介绍

换一种说法也就是，shell script是利用shell的功能所写的一个程序，这个程序是使用纯文本文件，将一些shell的语法与指令写在里面，然后用正规表示法，管道命令以及数据流重导向等功能，以达到我们所想要的处理目的。

更明白地来说，shell script就像早期Dos年代的.bat，最简单的功能就是将许多指令汇整写一起，让使用者很容易地就能够一个操作执行多个命令，而shell script更是提供了数组，循环，条件以及逻辑判断等重要功能，让使用者可以直接以shell来写程序，而不必使用类似C程序语言等传统程序编写的语法。

# 需要学习它吗？

你可能会说用Shell脚本实现的功能，我用任何一个编程语言都可以完成，比如Ruby, Python或Go。但是你会发现有一些小的程序使用Shell脚本会更方便与适合。

Shell脚本一般用做一些自动管理任务，封装复杂的配置，把一些命令组装成自己的新命令，让你的工作效率得到更大的提升。比如：

- 自动化你的日常任务
- 创建属于你的命令，可以接受用户的输入
- 在你的Mac电脑与Linux系统上复用一些脚本

# 第一个Shell脚本

打开文本编辑器(可以使用vi/vim命令来创建文件)，新建一个文件my_script.sh，扩展名为sh（sh代表shell），扩展名并不影响脚本执行，见名知意就好，如果你用php写shell 脚本，扩展名就用php好了。

输入一些代码：

    #!/bin/bash
    echo "Hello World !"

`#!` 是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种Shell。在这里是通过`/bin/bash`来运行。

`#!`的英文发音，我们知道`!`一般发音bang，那么`#!`发 hashbang或者shebang

echo命令用于向窗口输出文本

当保存了上面的文件后，我们需要给它一个执行的权限。通过在命令行执行下面命令可以添加执行的权限：

    chmod +x my_script.sh //add execute permission

现在我们用下面任何一行执行刚写的脚本：

    bash my_script.sh
    ./my_script.sh

如果顺利，会看到如下输出：

    Hello World !

# Shell 变量

定义变量时，变量名不加美元符号（$，PHP语言中变量需要），如：

    your_name="yuan ping"

注意，变量名和等号之间不能有空格，这可能和你熟悉的所有编程语言都不一样。同时，变量名的命名须遵循如下规则：

- 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
- 中间不能有空格，可以使用下划线（_）。
- 不能使用标点符号。
- 不能使用bash里的关键字（可用help命令查看保留关键字）。

## **使用变量**

使用一个定义过的变量，只要在变量名前面加美元符号即可，如：

    your_name="yuan ping"
    echo $your_name
    echo ${your_name}

变量名外面的花括号是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界，比如下面这种情况：

    for skill in Ruby Go Java; do
        echo "I am good at ${skill}Language"
    done

如果不给`skill`变量加花括号，写成`echo "I am good at $skillLanuage"`，解释器就会把$skillLanguage当成一个变量（其值为空），代码执行结果就不是我们期望的样子了。

# 循环

增加点难度，我们再写一个复杂些的脚本。实现的需求是从网络上下载一个图片列表文件，文件里面每一行是一个图片的地址。我们需要把这个文件中的图片下载到本地的images目录中。

脚本如下：

    image_url=https://public.qn.cichang.net/images.txt
    filename=$(basename $image_url)
    wget $image_url

    # Download each image from images.txt
    while read p; do
        echo "Download image: $p"
        wget -P images $p
    done <$filename

images.txt文件大概是这样的：

    https://magnet-file.qn.cichang.net/dbdc4754-ff0c-4875-8363-435191b21cf2.jpeg
    https://magnet-file.qn.cichang.net/756676e0-7899-4afb-9d62-45a098d694c9.jpeg
    https://magnet-file.qn.cichang.net/8b18b14a-2395-42f6-a378-fefc27ba30ca.jpeg
    https://magnet-file.qn.cichang.net/069cebfb-6258-494b-9066-be9d6e0b0c39.jpeg

首先，我们把图片列表文件的地址放在变量中。`image_url=https://public.qn.cichang.net/images.txt`

这行命令可以取文件名：`$(basename $image_url)`

然后循环每一行去读取这个文件，把结果放在变量 `p` 中

最后下载每一行的图片地址，-P 是指定文件夹 images `wget -P images $p`

# Shell 数组

bash支持一维数组（不支持多维数组），并且没有限定数组的大小。

类似于 C 语言，数组元素的下标由 0 开始编号。获取数组中的元素要利用下标，下标可以是整数或算术表达式，其值应大于或等于 0。

## **定义数组**

在 Shell 中，用括号来表示数组，数组元素用"空格"符号分割开。定义数组的一般形式为：

    array_name=(value0 value1 value2 value3)

## **读取数组**

读取数组元素值的一般格式是：

    ${数组名[下标]}

例如：

    valuen=${array_name[n]}

使用 **@** 符号可以获取数组中的所有元素，例如：

    echo ${array_name[@]}

# Shell 传递参数

我们可以在执行 Shell 脚本时，向脚本传递参数，脚本内获取参数的格式为：**$n**。**n** 代表一个数字，1 为执行脚本的第一个参数，2 为执行脚本的第二个参数，以此类推……

## **实例**

以下实例我们向脚本传递三个参数，并分别输出，其中 **$0** 为执行的文件名：

    #!/bin/bash

    echo "执行的文件名：$0";
    echo "第一个参数为：$1";
    echo "第二个参数为：$2";

为脚本设置可执行权限，并执行脚本，输出结果如下所示：

    $ chmod +x test.sh
    $ ./test.sh 1 2 3
    Shell 传递参数实例！
    执行的文件名：./test.sh
    第一个参数为：1
    第二个参数为：2

# 条件判断

Sell脚本使用了比较标准的if语句。

    # Syntax of simple if then statement
    if [ 35 -gt 0 ]
    then
        echo "Greater"
    fi

    # output
    Greater

你可能已经注意到了 `fi` 与 `if` 就对应的一对，我们再看看`else`语句

    #Syntax of simple if then statement
    if [ 35 -gt 45 ]
    then
     echo "Greater"
    else
     echo "Lesser"
    fi

    #Output
    Lesser

好了，到这里你应该已经掌握了一些基本的概念，可以写一些简单的Shell脚本了，想了解更多，可以看看官方的指南。还有这个：[https://github.com/alebcay/awesome-shell](https://github.com/alebcay/awesome-shell)