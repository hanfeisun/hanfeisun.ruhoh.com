---
title: Git Stash 历险记
date: '2012-12-31'
description:
categories: Tools
tags: [git]


---
<img src="{{urls.media}}/git-logo.png" >

我第一次用这个命令时被坑过，经过是这样的：我发现有一个类是多余的，想删掉它又担心以后需要查看它的代码，想保存它但又不想增加一个脏的提交。这时我想到了`git stash`，于是那一天我执行了下这个命令就回去睡觉了。 第二天我继续在这个目录里工作，coding了半天才发现前一天的修改都不见了，暂存区里也没有任何昨天修改的纪录。是我忘了保存了吗？

相信git老手们早就看出问题所在了，修改消失了并不是因为我忘了保存，而是`git stash`在保存完当前工作目录和暂存区以后，会用HEAD重置这两者。因为我昨天的修改没有提交，HEAD指向的是前天的版本，所以stash以后工作目录和暂存区就会被前天的的版本所重置。

正确的做法应该是在`git stash`后再执行`git stash apply`，当前的工作目录就恢复回来了。

`git stash apply`相当于利用过去贮藏(stashed)的工作目录快照，恢复当前的工作目录。如果工作目录在贮藏之后发生了变化，恢复时就会产生冲突(conflict)，这种情况下`git stash apply`会对工作目录进行merge操作。

和merge一样，git stash apply之前要保持当前目录是干净的(没有未提交的改变)，否则会保错：

> error: Your local changes to the following files would be overwritten by merge:
> Please, commit your changes or stash them before you can merge.

`git stash apply`只能恢复工作目录，如果想把暂存区也按照贮藏时的暂存区恢复的话，可以加上`--index`，如果暂存区恢复时发生冲突了会怎么办呢？嘿嘿，它会直接报错不允许你这么做：

> Conflicts in index. Try without --index.

个人认为，不能指望利用`git stash`保存暂存区的进度，因为不一定能够成功恢复。


还有一个疑问是`git stash`怎么做到既保存了暂存区，又保存了工作目录的呢？打一下`git log --graph refs/stash -2`这个命令，或者通过`man git-stash`查看它的文档，可以看到其提交的关系是这样的(W是工作目录，I是暂存区，H就是stash时候所在的版本。)：

<pre>
                  .----W
                 /    /
           -----H----I
</pre>


也就是说`git stash`实际上有两次提交，因此能够同时保存暂存区的快照和工作目录的快照。

最后附上一个漫画，我先不说话：
<img src="{{urls.media}}/git-stash-comic.png" >


不知道大家看明白了没有，这个漫画是用来吐槽`git stash`默认只保存跟踪文件的快照，而放任其他文件(untracked files)不管的特点。如果这里在贮藏之后修改了`app/views/users`的内容，因为`git stash`之前没有保存过这个文件，用`git stash pop`或者`git stash apply`都是没办法恢复的，这种情况只能自认倒霉。想把这些untracked files的进度也保存的话，只需添加`-u`参数即可。
