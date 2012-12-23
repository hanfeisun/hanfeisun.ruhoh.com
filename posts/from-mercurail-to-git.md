---
title: "从Mercurial到Git"
date: '2012-12-21'
tags: [Tools]
categories: [Tools]
---

我一年前开始用Mercurial做版本控制，觉得挺好用的，该有的功能都有，而且常用的基本就是pull, push, add, commit, update, revert这几个命令。其间偶尔用过两次Git，都是在[命令对照表] (http://mercurial.selenic.com/wiki/GitConcepts)的帮助下完成的，可是每次我都会感觉Git的某些语法很怪异，感觉很不舒服。

这周我尝试学习了一下Git，大致了解了Git的底层机制以及它与Mercurial的差异所在，说说我从Mercurial转到Git时常犯的几个错误把。

### hg add 与 git add

Mercurial的世界可以分为两个空间：工作目录(workspace)与版本库(repository)。我们在工作目录中写代码，而代码的历史纪录被存放在版本库中。在工作目录中，文件有两个状态：跟踪(tracked)与未跟踪(not tracked)。只要跟踪中的文件发生了变化，那么在提交(commit)的时候，这些变化会被自动添加到版本库中。新建的文件都是未被跟踪的，如果想要开始跟踪这个文件，只需要`hg add 文件名`即可。

在Git里则不是这样，因为Git在工作目录与版本库之间还存在着第三个空间：暂存区(staging area)或者叫索引(index)。一个跟踪中的文件如果发生了变化，需要先把这个变化加入到暂存区中，然后才能通过提交到版本库中。这样一来，工作目录里的**文件的变化**也有了两个状态：暂存(staged)与未暂存(not staged)，请注意这里的用词，暂存用来用来形容的是**文件的变化**而不是**文件本身**。

如果一个文件未被跟踪，那么`git add 文件名`就可以将它的状态改成跟踪，同时将这一变化放入暂存区中：

<pre>
git init;
echo "first line" > new_file;
git add new_file;
git commit -m "add new_file"
</pre>
注意这里的`git add`实际上做了两件事，改变跟踪状态和添加到暂存区。之后`git commit`就可以了，这样看来似乎和hg没什么区别，可是下一次提交我这样做：

<pre>
echo "add a new line" >> new_file;
git commit -m "Add a new line to new_file"
</pre>
Git会直接报错，说`new_file`的变化没有被添加到暂存区里(Changes not staged for commit)，这样做才能提交`new_file`的变化：

<pre>
echo "add a new line" >> new_file;
git add new_file;
git commit -m "Add a new line to new_file"
</pre>
注意前面这段代码在Mercurial里是不需要的，因为Mercurial不存在暂存区的概念(事实上，大多数版本控制软件也没有)，只要文件被跟踪，在提交的时候，Mercurial就会自动搜集文件相对于版本库头(HEAD)的所有变化，然后将它们添加到版本库里。显然Mercurial的操作更简单，也更直观。

这一点是Git的败笔吗？我觉得也不尽然，暂存区中存放的变化其实相当于一个临时的提交。比如这样的应用场景：有一段代码需要修改，我用方案A修改以后发现方案B可能会更好，但是在确认方案B更好之前，我不想把方案A提交到代码库中，又希望能保存当前方案A的变化。在Git里可以这样做：


<pre>
echo "Refactor solution A" > code_file;
git add code_file;
# 将方案A保存到暂存区里

echo "Refactor solution B" > code_file;
# 用方案B修改代码，然后做一些测试
</pre>


如果方案B确实比方案A更好，用方案B更新一下暂存区即可： 

<pre>
git add code_file;
</pre>

如果想变回方案A，也是可以的：

<pre>
git checkout -- code_file;
</pre>

当然，上面的情景在也可以用储藏/搁置(stash/shelve)或者分支(branch)命令来处理，但是可以看出暂存区这一概念增加了Git的灵活性。


### hg revert 与 git revert

>名可名，非常名  --- 《道德经》

上面这句话用来吐槽Mercurial与git之间坑爹的术语差异。同样都是叫`revert`的命令，这差别咋就这么大呢。

比如某一天，我不小心把自己的工作目录弄乱了或者误删除了一个文件，怎么办呢。在Mercurial里面是很简单的，想恢复整个工作目录是`hg revert -a`，恢复某一个文件是`hg revert file-name`, 也可以用`-v`指定按照哪一个版本去恢复。


而在Git里，`git revert`虽然也是revert，但做得根本不是一个事（很坑爹）。Mercurial里`git revert`的等价命令是`hg backout`，不过我貌似从来没在实际场合中用过这个命令。 Git里对应于`hg revert`的命令应该是`git checkout`，恢复整个工作目录是`git checkout HEAD -- .`, 恢复文件是`git checkout HEAD -- file-name`，可以在`--`的前面指定要恢复的版本，如果省略，默认的版本是当前的暂存区(见前面的代码)。

除了checkout以外，`git reset --hard`也可以按照版本库头恢复工作目录。reset也是一个Git很奇葩的命令，在Mercurial中并没有等价的命令。


### 默认的远程库与分支

Git里的远程库有一个独立的命名空间，比如我从Github上clone了一个项目下来，进入到工作目录中时，默认的分支是master，而远程库是一个叫origin的命名空间，它的下面有一个origin/master分支。而Mercurial则好像不把远程库和本地库分隔开来，默认的分支叫default。


如果想把已有的代码库上传到托管网站，Git的做法是这样：

<pre>
git remote add origin ssh://repository.git
git push -u origin master
</pre>

而Mercurial只需要这样：

<pre>
hg push ssh://repository.git
</pre>

个人认为，Git分支和远程库的命名方式(origin, master)初看起来会让人有莫名其妙的感觉。而Mercurial在这方面则更友好。
