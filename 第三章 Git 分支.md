# 第三章 **Git** **分支**

Git 鼓励在工作流 程中频繁地使用分支与合并，哪怕一天之内进行许多次。 

## **分支简介**

 Git 保存的不是文件的变化或者差异，而是一系列不同时刻的 **快照** 。

进行提交操作时，Git 会保存一个提交对象（commit object）。该提交对象会包含一个指向暂存内容快照的指针。 但不仅仅是这样，该提交对象还包含了作者的姓 名和邮箱、提交时输入的信息以及指向它的父对象的指针。 首次提交产生的提交对象没有父对象，普通提交操作产生的提交对象有一个父对象， 而由多个分支合并产生的提交对象有多个父对象。



 暂存操作会为每一个文件计算校验和（使用我们在 起步 中提到的 SHA-1 哈希算法），然后会把当前版本的文件快照保存到Git 仓库中 （Git 使用 blob 对象来保存它们），最终将校验和加入到暂存区域等待提交。当使用 git commit 进行提交操作时，Git 会先计算每一个子目录（本例中只有项目根目录）的校验和， 然后在 Git 仓库中这些校验和保存为树对象。随后，Git 便会创建一个提交对象， 它除了包含上面提到的那些信息外，还包含指向这个树对象（项目根目录）的指针。 如此一来，Git 就可以在需要的时候重现此次保存的快照。

现在，Git 仓库中有五个对象：三个 blob 对象（保存着文件快照）、一个 **树** 对象 （记录着目录结构和 blob 对象索引）以及一个 **提交** 对象（包含着指向前述树对象的指针和所有提交信息）。

<img src="D:\github_exercise\learn_progit\images\图片15.png" style="zoom:50%"/>

做些修改后再次提交，那么这次产生的提交对象会包含一个指向上次提交对象（父对象）的指针。

<img src="D:\github_exercise\learn_progit\images\图片16.png" style="zoom:40%"/>

Git 的分支，其实本质上仅仅是指向提交对象的可变指针。 Git 的默认分支名字是 master。 在多次提交操作之后，你其实已经有一个指向最后那个提交对象的 master 分支。 master 分支会在每次提交时自动向前移动。

>Git 的 master 分支并不是一个特殊分支。 它就跟其它分支完全没有区别。 之所以几乎每一个仓库都有 master 分支，是因为 git init 命令默认创建它，并且大多数人都懒得去改动它。

<img src="D:\github_exercise\learn_progit\images\图片17.png" style="zoom:40%"/>

### **分支创建**

创建了一个可以移动的新的指针。 比如，创建一个 testing 分支， 你需要使用 git branch 命令：

`$ git branch testing`

这会在当前所在的提交对象上创建一个指针。

<img src="D:\github_exercise\learn_progit\images\图片18.png" style="zoom:40%"/>

图 12. 两个指向相同提交历史的分支

那么，Git 又是怎么知道当前在哪一个分支上呢？ 也很简单，它有一个名为 HEAD 的特殊指针,指向当前所在的本地分支（译注：将 HEAD 想象为当前分支的别名）。在本例中，你仍然在master 分支上。 因为 git branch 命令仅仅 **创建** 一个新分支，并不会自动切换到新分支中去。

<img src="D:\github_exercise\learn_progit\images\图片19.png" style="zoom:40%"/>

图 13. HEAD 指向当前所在的分支

使用 git log 命令查看各个分支当前所指的对象。 提供这一功能的参数是 --decorate。

<img src="images/图片20.png" style="zoom:60%"/>

```
$ git log --oneline --decorate
c1c5569 (HEAD -> master, origin/master, testing) 220713:21-21
6109401 (tag: v1.4) commit33
3be773e aaa:wq
238f5f4 20220713-10:00
702469f commit3
3c6ad48 c2
1984d6f 2022-07-12-14:23
3246ba2 (tag: v1.2, tag: show) commit1
```

正如你所见，当前 master 和 testing 分支均指向校验和以c1c5569 开头的提交对象。

### **分支切换**

要切换到一个已存在的分支，你需要使用 git checkout 命令。 我们现在切换到新创建的 testing 分支 

去：

`$ git checkout testing`

<img src="images/图片21.png" style="zoom:40%"/>

图14. HEAD 指向当前所在的分支

那么，这样的实现方式会给我们带来什么好处呢？ 现在不妨再提交一次：

<img src="images/图片22.png" style="zoom:40%"/>

图15. HEAD 分支随着提交操作自动向前移动

你的 testing 分支向前移动了，但是 master 分支却没有，它仍然指向运行 git checkout 时所

<<<<<<< HEAD
指的对象。 这就有意思了，现在我们切换回 master 分支看看：

` git checkout master`

<img src="images/图片23.png" style="zoom:40%"/>

图 16. 检出时 HEAD 随之移动

这条命令做了两件事。 一是使 HEAD 指回 master 分支，二是将工作目录恢复成 master 分支所指向的快照内容。 也就是说，你现在做修改的话，项目将始于一个较旧的版本。 本质上来讲，这就是忽略 testing 分支所做 的修改，以便于向另一个方向进行开发。

分支切换会改变你工作目录中的文件,在切换分支时，一定要注意你工作目录里的文件会被改变。 如果是切换到一个较旧的分支，你的工作目录会恢复到该分支最后一次提交时的样子。 如果 Git 不能干净利落地完成这个任 务，它将禁止切换分支。

我们不妨再稍微做些修改并提交：

$ vim test.rb

$ git commit -a -m 'made other changes'

现在，这个项目的提交历史已经产生了分叉（参见 项目分叉历史）。 因为刚才你创建了一个新分支，并切换过去进行了一些工作，随后又切换回 master 分支进行了另外一些工作。 上述两次改动针对的是不同分支：你可以在不同分支间不断地来回切换和工作，并在时机成熟时将它们合并起来。 而所有这些工作，你需要的命令只有branch、checkout 和 commit.

<img src="images\图片24.png" style="zoom:43%;" />

图 17. 项目分叉历史

你可以简单地使用 git log 命令查看分叉历史。 运行 git log --oneline --decorate --graph --all ，它会输出你的提交历史、各个分支的指向以及项目的分支分叉情况。

```
$ git log --oneline --decorate --graph --all
* 8e9ab77 (master) bmaster1
| * cb67859 (HEAD -> testing) testing 2
| * d2a9f03 submit testing
|/
* c1c5569 (origin/master) 220713:21-21
* 6109401 (tag: v1.4) commit33
* 3be773e aaa:wq
* 238f5f4 20220713-10:00
* 702469f commit3
* 3c6ad48 c2
* 1984d6f 2022-07-12-14:23
* 3246ba2 (tag: v1.2, tag: show) commit1
```

由于 Git 的分支实质上仅是包含所指对象校验和（长度为 40 的 SHA-1 值字符串）的文件，所以它的创建和销毁都异常高效。 创建一个新分支就相当于往一个文件中写入 41 个字节（40 个字符和 1 个换行符），如此的简单能不快吗？

在 Git 中，任何规模的项目都能在瞬间创建新分支。 同时，由于每次提交都会记录父对 象(上次的提交对象)，所以寻找恰当的合并基础（译注：即共同祖先）也是同样的简单和高效。 这些高效的特性使得 Git 鼓励开发人员频繁地创建和使用分支。

创建新分支的同时切换过去

通常我们会在创建一个新分支后立即切换过去，这可以用 git checkout -b <newbranchname> 一条命令搞定。

## 分支的新建与合并

- git checkout -b issue53

切换分支之前，保持好一个干净的状态。 有一些方法可以绕过这个问题（即，暂存（stashing） 和 修补提交（commit amending））.

git add *

git commit -amend 

git checkout <branch>

---

- git checkout master

 请牢记：当你切换分支的时候，Git 会重置你的工作目录，使其看起来像回到了你在那个分支上最后一次提交的样子。 Git 会自动添加、删除、修改文件以确保此时你的工作目录和这个分支最后一次提交时的样子一模一样。

---

$ git checkout -b hotfix

$ vim index.html

$ git commit -a -m 'fixed the broken email address'

$ git checkout master

---

```
Lenovo@LAPTOP-17JBE9VI MINGW64 /D/github_exercise/learn_progit (hotfix)
$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Lenovo@LAPTOP-17JBE9VI MINGW64 /D/github_exercise/learn_progit (master)
$ git merge hotfix
Updating 8e9ab77..ec094d5
Fast-forward
 index.html | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 index.html

Lenovo@LAPTOP-17JBE9VI MINGW64 /D/github_exercise/learn_progit (master)
```

Fast-forward: 由于你想要合并的分支 hotfix 所指向的提交 C4 是你所在的提交 C2 的直接后继， 因此 Git 会直接将指针向前移动。换句话说，当你试图合并两个分支时， 如果顺着一个分支走下去能够到达另一个分支，那么 Git 在合并两者的时候， 只会简单的将指针向前推进 （指针右移），因为这种情况下的合并操作没有需要解决的分歧——这就叫做 “快进（fast-forward）”。

<img src="D:\github_exercise\learn_progit\images\image-20220714115925844.png" alt="image-20220714115925844" style="zoom: 25%;" />

 git branch -d hotfix





## 分支管理

## 分支开发流

## 远程分支

## 变基
=======
指的对象。 这就有意思了，现在我们切换回 master 分支看看：
>>>>>>> testing
