---
title: "从Mercurial到Git"
date: '2012-12-21'
tags: [git]
categories: [Tools]
---

<img src="{{urls.media}}/mercurial.png" align="right">

一年前，我开始用Mercurial做版本控制，一直觉得它易学易用，该有的功能都有，而且常用的命令就只有pull, push, add, commit, update, revert这几个。其间偶尔用过两次Git，都是在[命令对照表] (http://mercurial.selenic.com/wiki/GitConcepts)的帮助下磕磕绊绊地完成的，每次我都会感觉Git语法的怪异之处，用起来觉得很不舒服。

既然Git这么流行，肯定有它的长处。这周我就尝试学习了一下Git，大致了解了Git的底层机制以及它与Mercurial最大的差异所在。就从我刚转到Git时经常犯的错误开始说起吧。

### hg add 与 git add

Mercurial的世界可以分为两个空间：工作目录(workspace)与版本库(repository)。我们在工作目录中写代码，想保存一下当前的快照就使用提交(commit)命令，然后当前的快照被存放在版本库中。在工作目录中，文件有两个状态：要么是跟踪的(tracked)，要么没有被跟踪(not tracked)。只要跟踪的文件发生了变化，提交时这些变化会被自动添加到版本库中。新建的文件都是未被跟踪的，如果想要开始跟踪这个文件，只需要`hg add 文件名`即可。

Git却不是这样，它在工作目录与版本库之间还有着第三个空间：暂存区(staging area)或者叫索引(index)。一个跟踪中的文件如果发生了变化，需要先把这个变化加入到暂存区中，然后才能通过提交到版本库中。这样一来，工作目录里的**文件的变化**也有了两个状态：暂存(staged)与未暂存(not staged)，请注意这里的用词，暂存用来用来形容的是**文件的变化**而不是**文件本身**。

如果一个文件未被跟踪，那么`git add 文件名`就可以将它的状态改成跟踪，同时将这一变化放入暂存区中：

<pre>
git init;
echo "first line" > new_file;
git add new_file;
git commit -m "add new_file"
</pre>

注意这里的`git add`实际上做了两件事，改变跟踪状态和添加到暂存区。之后`git commit`就可以了，这样看来似乎和hg没什么区别，可是下一次提交里，如果我这样做：

<pre>
echo "add a new line" >> new_file;
git commit -m "Add a new line to new_file"
</pre>

Git会直接报错，说`new_file`的变化没有被添加到暂存区里(Changes not staged for commit)。第一次我不是添加过一次吗？为什么还要再重新添加呢？因为git里将当前文件的变化添加到暂存区的add命令仅仅对那一个时刻有效。如果add后又重新修改了文件，还需要再次把新的变化添加到暂存区。正确的做法应该是这样：

<pre>
echo "add a new line" >> new_file;
git add new_file;
git commit -m "Add a new line to new_file"
</pre>
注意前面这段代码在Mercurial里是不需要的，因为Mercurial不存在暂存区的概念(事实上，大多数版本控制软件也没有)，只要文件被跟踪，在提交的时候，Mercurial就会自动搜集文件相对于版本库头(HEAD)的所有变化，然后将它们添加到版本库里。

看上去Mercurial的操作更简单，也更直观。这一点是Git设计上的缺陷吗？我觉得Git设计成这样是有意为之的，暂存区中存放的变化其实相当于一个临时的提交。比如这样的应用场景：有一段代码需要修改，我用方案A修改以后发现方案B可能会更好，但是在确认方案B更好之前，我不想把方案A提交到代码库中，又希望能保存当前方案A的变化。在Git里可以这样做：


<pre>
echo "Refactor solution A" > code_file;
git add code_file;
# 将方案A保存到暂存区里

echo "Refactor solution B" > code_file;
# 用方案B修改代码，然后做一些测试
</pre>


如果经验证，方案B确实比方案A更好，把方案B添加到暂存区即可： 

<pre>
git add code_file;
</pre>

如果想恢复回方案A，也是可以的：

<pre>
git checkout -- code_file;
</pre>

当然，上面的情景在也可以用储藏/搁置(stash/shelve)或者分支(branch)命令来处理，但是通过这个例子可以看出暂存区的引入增加了Git的灵活性。


### hg revert 与 git revert

>名可名，非常名  --- 《道德经》

上面这句话用来吐槽Mercurial与git之间的术语差异。同样都是名字叫做`revert`的命令，做事的差距咋就这么大呢。

比如某一天，我不小心把自己的工作目录弄乱了或者误删除了一个文件，怎么办呢。在Mercurial里面是很简单的，想恢复整个工作目录是`hg revert -a`，恢复某一个文件是`hg revert file-name`, 也可以用`-v`指定按照哪一个版本去恢复。

Git里按照某一版本恢复文件的命令是`git checkout`，恢复整个工作目录是`git checkout HEAD -- .`, 恢复文件是`git checkout HEAD -- file-name`，可以在--的前面指定要恢复的版本，如果省略，默认的版本是当前的暂存区(见前面的代码)。有趣的是，在Git里也有一个叫revert的命令，但它做得却和`hg revert`根本不是一个事（可以试一试，保证很坑爹），它在Mercurial里的等价命令应该是`hg backout`，不过我几乎从来没在实际场合中用过这个命令，所以不细述其用法。

除了checkout以外，`git reset --hard`也可以按照版本库头恢复工作目录。reset是一个Git很危险的命令，在Mercurial中并没有很类似的命令。


### 远程库与命名空间

Git里的远程库有一个独立的命名空间，比如我从Github上clone了一个项目下来，默认的分支是master，是一个本地库。此外还有一个叫origin的命名空间，是远程库，它的下面有一个origin/master分支。在Mercurial则不存在这样的分法，本地库和远程库采用同一个命名空间，默认的分支叫default。


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


这周主要看的是[Git权威指南](http://book.douban.com/subject/6526452/)这本书，感觉讲的不错，我有点体会到了Git设计成这样的原因，也认识到Hg在Git的基础上做的一些改进。估计以后用Git时能少一些提心吊胆的感觉。 :P
