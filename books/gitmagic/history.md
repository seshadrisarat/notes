# 关于历史 

Git分布式本性使得历史可以轻易编辑。但你若篡改过去，需要小心：只重写你独自拥有的那部分。正如民族间会无休止的争论谁犯下了什么暴行一样，如果在另一个人的克隆里，历史版本与你的不同，当你们的树互操作时，你会遇到一致性方面的问题。

一些开发人员强烈地感觉历史应该永远不变，不好的部分也不变所有都不变。另一些觉得代码树在向外发布之前，应该整得漂漂亮亮的。Git同时支持两者的观点。像克隆，分支和合并一样，重写历史只是Git给你的另一强大功能，至于如何明智地使用它，那是你的事了。

### 我认错 

刚提交，但你期望你输入的是一条不同的信息？那么键入：

	$ git commit --amend

来改变上一条信息。意识到你还忘记了加一个文件？运行git add来加，然后运行上面的命令。

希望在上次提交里包括多一点的改动？那么就做这些改动并运行：

	$ git commit --amend -a

### 更复杂情况 

假设前面的问题还要糟糕十倍。在漫长的时间里我们提交了一堆。但你不太喜欢他们的组织方式，而且一些提交信息需要重写。那么键入：

	$ git rebase -i HEAD~10

并且后10个提交会出现在你喜爱的$EDITOR。一个例子：

    pick 5c6eb73 Added repo.or.cz link
    pick a311a64 Reordered analogies in "Work How You Want"
    pick 100834f Added push target to Makefile

之后：

- 通过删除行来移去提交。
- 通过为行重新排序行来重新排序提交。
- 替换 `pick` 使用：
   * `edit` 标记一个提交需要修订。
   * `reword` 改变日志信息。
   * `squash` 将一个提交与其和前一个合并。
   * `fixup` 将一个提交与其和前一个合并，并丢弃日志信息。

保存退出。如果你把一个提交标记为可编辑，那么运行

	$ git commit --amend

否则，运行：

	$ git rebase --continue

这样尽早提交，经常提交：你之后还可以用rebase来规整。

### 本地变更之后 

你正在一个活跃的项目上工作。随着时间推移，你做了几个本地提交，然后你使用合并与官方版本同步。在你准备好提交到中心分支之前，这个循环会重复几次。

但现在你本地Git克隆掺杂了你的改动和官方改动。你更期望在变更列表里，你所有的变更能够连续。

这就是上面提到的 `git rebase` 所做的工作。在很多情况下你可以使用 `--onto` 标记以避免交互。

另外参见 `git help rebase` 以获取这个让人惊奇的命令更详细的例子。你可以拆分提交。你甚至可以重新组织一棵树的分支。

### 重写历史 

偶尔，你需要做一些代码控制，好比从正式的照片中去除一些人一样，需要从历史记录里面彻底的抹掉他们。例如，假设我们要发布一个项目，但由于一些原因，项目中的某个文件不能公开。或许我把我的信用卡号记录在了一个文本文件里，而我又意外的把它加入到了这个项目中。仅仅删除这个文件是不够的，因为从别的提交记录中还是可以访问到这个文件。因此我们必须从所有的提交记录中彻底删除这个文件。

	$ git filter-branch --tree-filter 'rm top/secret/file' HEAD

参见 `git help filter-branch` ，那里讨论了这个例子并给出一个更快的方法。一般地， `filter-branch` 允许你使用一个单一命令来大范围地更改历史。

此后，+.git/refs/original+目录描述操作之前的状态。检查命令filter-branch的确做了你想要做的，然后删除此目录，如果你想运行多次filter-branch命令。

最后，用你修订过的版本替换你的项目克隆，如果你想之后和它们交互的话。

### 制造历史 {#makinghistory}

想把一个项目迁移到Git吗？如果这个项目是在用比较有名气的系统，那可以使用一些其他人已经写好的脚本，把整个项目历史记录导出来放到Git里。

否则，查一下 `git fast-import` ，这个命令会从一个特定格式的文本读入，从头来创
建Git历史记录。通常可以用这个命令很快写一个脚本运行一次，一次迁移整个项目。

作为一个例子，粘贴以下所列到临时文件，比如/tmp/history：

    commit refs/heads/master
    committer Alice <alice@example.com> Thu, 01 Jan 1970 00:00:00 +0000
    data <<EOT
    Initial commit.
    EOT

    M 100644 inline hello.c
    data <<EOT
    #include <stdio.h>

    int main() {
      printf("Hello, world!\n");
      return 0;
    }
    EOT


    commit refs/heads/master
    committer Bob <bob@example.com> Tue, 14 Mar 2000 01:59:26 -0800
    data <<EOT
    Replace printf() with write().
    EOT

    M 100644 inline hello.c
    data <<EOT
    #include <unistd.h>

    int main() {
      write(1, "Hello, world!\n", 14);
      return 0;
    }
    EOT


之后从这个临时文件创建一个Git仓库，键入：

	$ mkdir project; cd project; git init
	$ git fast-import --date-format=rfc2822 < /tmp/history

你可以从这个项目checkout出最新的版本，使用：

	$ git checkout master .

命令`git fast-export` 转换任意仓库到 `git fast-import` 格式，你可以研究其输出来写导出程序， 也以可读格式传送仓库。的确，这些命令可以发送仓库文本文件通过只接受文本的渠道。


### 哪儿错了？ 

你刚刚发现程序里有一个功能出错了，而你十分确定几个月以前它运行的很正常。天啊！这个臭虫是从哪里冒出来的？要是那时候能按照开发的内容进行过测试该多好啊。

现在说这个已经太晚了。然而，即使你过去经常提交变更，Git还是可以精确的找出问题所在：

	$ git bisect start
	$ git bisect bad HEAD
	$ git bisect good 1b6d

Git从历史记录中检出一个中间的状态。在这个状态上测试功能，如果还是有问题：

	$ git bisect bad

如果可以工作了，则把"bad"替换成"good"。Git会再次帮你找到一个以确定的好版本和坏版本之间的状态，通过这种方式缩小范围。经过一系列的迭代，这种二分搜索会帮你找到导致这个错误的那次提交。一旦完成了问题定位的调查，你可以返回到原始状态，键入：

	$ git bisect reset

不需要手工测试每一次改动，执行如下命令可以自动的完成上面的搜索：

	$ git bisect run my_script

Git使用指定命令（通常是一个一次性的脚本）的返回值来决定一次改动是否是正确的：命令退出时的代码0代表改动是正确的，125代表要跳过对这次改动的检查，1到127之间的其他数值代表改动是错误的。返回负数将会中断整个bisect的检查。

你还能做更多的事情: 帮助文档解释了如何展示bisects, 检查或重放bisect的日志,并可以通过排除对已知正确改动的检查，得到更好的搜索速度。

### 谁让事情变糟了？ 

和其他许多版本控制系统一样，Git也有一个"blame"命令：

	$ git blame bug.c

这个命令可以标注出一个指定的文件里每一行内容的最后修改者，和最后修改时间。但不像其他版本控制系统，Git的这个操作是在线下完成的，它只需要从本地磁盘读取信息。

### 个人经验

在一个中心版本控制系统里，历史的更改是一个困难的操作，并且只有管理员才有权这么做。没有网络，克隆，分支和合并都没法做。像一些基本的操作如浏览历史，或提交变更也是如此。在一些系统里，用户使用网络连接仅仅是为了查看他们自己的变更，或打开文件进行编辑。

中心系统排斥离线工作，也需要更昂贵的网络设施，特别是当开发人员增多的时候。最重要的是，所有操作都一定程度变慢，一般在用户避免使用那些能不用则不用的高级命令时。在极端的情况下，即使是最基本的命令也会变慢。当用户必须运行缓慢的命令的时候，由于工作流被打断，生产力降低。

我有这些的一手经验。Git是我使用的第一个版本控制系统。我很快学会适应了它，用了它提供的许多功能。我简单地假设其他系统也是相似的：选择一个版本控制系统应该和选择一个编辑器或浏览器没啥两样。

在我之后被迫使用中心系统的时候，我被震惊了。我那有些脆弱的网络没给Git带来大麻烦，但是当它需要像本地硬盘一样稳定的时候，它使开发困难重重。另外，我发现我自己有选择地避免特定的命令，以避免踏雷，这极大地影响了我，使我不能按照我喜欢的方式工作。

当我不得不运行一个慢的命令的时候，这种等待极大地破坏了我思绪连续性。在等待服务器通讯完成的时候，我选择做其他的事情以度过这段时光，比如查看邮件或写其他的文档。当我返回我原先的工作场景的时候，这个命令早已结束，并且我还需要浪费时间试图记起我之前正在做什么。人类不擅长场景间的切换。

还有一个有意思的大众悲剧效应：预料到网络拥挤，为了减少将来的等待时间，每个人将比以往消费更多的带宽在各种操作上。共同的努力加剧了拥挤，这等于是鼓励个人下次消费更多带宽以避免更长时间的等待。


