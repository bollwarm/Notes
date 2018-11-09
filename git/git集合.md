# git集合
git详解
.git目录
下面就开始进入.git目录，通过“ls”命令可以看到.git目录中的文件和子目录：
 
 
对于这些文件和目录，下面给出了一些基本的描述。
•	hooks:这个目录存放一些shell脚本，可以设置特定的git命令后触发相应的脚本；在搭建gitweb系统或其他git托管系统会经常用到hook script
•	info:包含仓库的一些信息
•	logs:保存所有更新的引用记录
•	objects:所有的Git对象都会存放在这个目录中，对象的SHA1哈希值的前两位是文件夹名称，后38位作为对象文件名
•	refs:这个目录一般包括三个子文件夹，heads、remotes和tags，heads中的文件标识了项目中的各个分支指向的当前commit
•	COMMIT_EDITMSG:保存最新的commit message，Git系统不会用到这个文件，只是给用户一个参考
•	config:这个是GIt仓库的配置文件
•	description:仓库的描述信息，主要给gitweb等git托管系统使用
•	index:这个文件就是我们前面提到的暂存区（stage），是一个二进制文件
•	HEAD:这个文件包含了一个档期分支（branch）的引用，通过这个文件Git可以得到下一次commit的parent
•	ORIG_HEAD:HEAD指针的前一个状态
Git引用
Git中的引用是一个非常重要的概念，对于理解分支（branch）、HEAD指针以及reflog非常有帮助。
Git系统中的分支名、远程分支名、tag等都是指向某个commit的引用。比如master分支，origin/master远程分支，命名为V1.0.0.0的tag等都是引用，它们通过该保存某个commit的SHA1哈希值指向某个commit
重新认识HEAD
HEAD也是一个引用，一般情况下间接指向你当前所在的分支的最新的commit上。HEAD跟Git中一般的引用不同，它并不包含某个commit的SHA1哈希值，而是包含当前所在的分支，所有HEAD直接执行当前所在的分支，然后间接指向当前所在分支的最新提交。
为了更形象的解释上面的描述，我们首先查看“.git/HEAD”的内容：
 
这就表示HEAD是一个指向master分支的引用，然后我们可以根据引用路径打开“refs/heads/master”文件，内容如下：
  
根据前面一片文章的介绍，我们通过这个哈希值查看对象的类型和雷人，可以看到这个哈希值对应着一个commit，并通过“git log”可以发现这个commit就是master分支上最新的提交。
 
所以可以看到，所有的内容都是环环相扣的，我们通过HEAD找到一个当前分支，然后通过当前分支的引用找到最新的commit，然后通过commit可以找到整个对象关系模型，看下图：
 
引用和分支
直到现在我们都没有开始介绍分支（branch），先大概展示一下引用和分支的关系。
假设我们现在除了master分支，又创建了一个release-1.0.0.1的分支，再次查看“.git/refs/heads/”目录，可以看到除了master文件之外，又多了一个release-1.0.0.1文件，查看该文件的内容也是一个哈希值。
通过“git show-ref --heads”命令就可以查看所有的头，这些都是HEAD的候选值：
 
HEAD文件的内容是commit的分支，当我们把分支切换到release-1.0.0.1的时候，HEAD文件的内容也会相应的变成：
ref: refs/heads/release-1.0.0.1
日志
我们进入“.git/logs”文件夹，可以看到这个文件夹也有一个HEAD文件和refs目录，这些就是记录commit历史记录的地方。我们可以通过commit的哈希值，把repo退到一个指定的状态。
Git索引index
前面文章我们也提到过index/stage，就是更新的暂存区，下面来看看index文件。
index（索引）是一个存放了已排序的路径的二进制文件，并且每个路径都对应一个SHA1哈希值。在Git系统中，可以通过“git ls-files --stage”来显示index文件的内容：
  
从命令的输出可以看到，所有的记录都对应仓库中的文件（包含全路径）。上面显示的哈希值就是abc.txt的blob对象的哈希值。
现在我们更新abc.txt文件，并通过“git add”添加到暂存区，这时发现index中的abc对象的哈希值已经变化了。
 
对象的存储
前面提到所有的Git对象都会存放在“.git/objects”目录中，对象SHA1哈希值的前两位是文件夹名称，后38位作为对象文件名。下面是之前提到的master最新的commit对象的哈希值：
63a6ea8475b47078dee14ea8ff2c8731927ffbbf
 
在Git系统中有两种对象存储的方式，松散对象存储和打包对象存储
松散对象（loose object）
松散对象存储就是前面提到的，每一个对象都被写入一个单独文件中，对象SHA1哈希值的前两位是文件夹名称，后38位作为对象文件名。
打包对象（packed object）
对应松散存储，把每个文件的每个版本都作为一个单独的对象，它的效率比较低，而且浪费空间。所以就有了通过打包文件（packfile）的存储方式。
Git使用打包文件（packfile）去节省空间。在这个格式中，Git只会保存第二个文件中改变的部分，然后用一个指针指向相似的那个文件。
一般Git系统会自动完成打包的工作，在已经发生过打包的Git仓库中，object/pack目录下回成对出现很多“pack-***.idx”和“pack-***.pak”文件。
在git提交环节，存在三大部分：working tree, index file, commit
2013年04月07日 13:22:27 lksodit_yiyi 阅读数：9163 标签： Git 
这三大部分中：
working tree：就是你所工作在的目录，每当你在代码中进行了修改，working tree的状态就改变了。
index file：是索引文件，它是连接working tree和commit的桥梁，每当我们使用git-add命令来登记后，index file的内容就改变了，此时index file就和working tree同步了。
commit：是最后的阶段，只有commit了，我们的代码才真正进入了git仓库。我们使用git-commit就是将index file里的内容提交到commit中。
总结一下：
git diff：是查看working tree与index file的差别的。
git diff --cached：是查看index file与commit的差别的。
git diff HEAD：是查看working tree和commit的差别的。（你一定没有忘记，HEAD代表的是最近的一次commit的信息）


为了更加清晰的阐释这个关系，来给出一个实例。


[yaya@yaya-desktop]$ cat main.c
#include<stdio.h>
int main(int argc,char *argv[])
{
printf(“hello.\n”);
printf(“he was a student.\n”);
return 0;
}
然后git init, git add . , git commit;
之后你将源代码修改为：


[yaya@yaya-desktop]$ cat main.c
#include<stdio.h>
int main(int argc,char *argv[])
{
printf(“hello.\n”);
printf(“he was a student.\n”);
printf(“he was born in finland.\n”);
return 0;
}




此时你git add .，但不用执行git commit命令。然后你再将源代码改为：
[yaya@yaya-desktop]$ cat main.c
#include<stdio.h>
int main(int argc,char *argv[])
{
printf(“hello.\n”);
printf(“he was a student.\n”);
printf(“he was born in finland.\n”);
printf(“he is very clever!\n”);
return 0;
}
复制代码


这个时候，你执行如下三个命令，仔细查看，我相信你会发现它们三个的区别的！
$ git diff
$ git diff –cached
$ git diff HEAD
讲到这里，基本上对git diff命令有了比较深入的了解了，现在你再使用git status看看输出结果，样子大概是这样：


[yaya@yaya-desktop]$ git status
# On branch master
# Changes to be committed:
#   (use “git reset HEAD <file>…” to unstage)
#
#    modified:   main.c
#
# Changed but not updated:
#   (use “git add <file>…” to update what will be committed)
#
#    modified:   main.c
#很明显可以知道：
Changes to be committed表示已经存在于index file里，但尚未提交。
Changed but not updated表示在working tree已经做修改，但还没有使用git add登记到index file里。
Git最常用功能，这一篇就够了！（结合开发场景）

https://blog.csdn.net/h247263402/article/details/74849182 
毫无疑问，Git是当下最流行、最好用的版本控制系统。Git属于分布式版本控制系统，相较于Subversion等集中式版本控制系统有很明显的优势。对于我们开发人员来说，熟练使用Git是最基本的技能之一。那么，今天就来说一下在开发工作中，使用到的Git的最基本、最常用的功能有那些？
克隆版本库
工作中，当接手维护一个项目时，需要从远程代码库将项目源码克隆到本地。或者，在Github上发现了一个非常好的开源项目，想要搞下来研究研究，第一步也是克隆版本库。
$ git clone git@github.com:hxdaxu/GitDemo.git
正克隆到 'GitDemo'...
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
接收对象中: 100% (3/3), done.
检查连接... 完成。
从Github上克隆代码库至本地的当前目录。
通过暂存区控制提交
Git有暂存区的概念，对于一次Commit只会将加入暂存区的变更提交，我们可以通过暂存区控制提交的内容。
$ echo home page ... done. >> homePage.txt
$ echo show page ... developing... >> showPage.txt
$ git status
位于分支 master
您的分支与上游分支 'origin/master' 一致。
未跟踪的文件:
  （使用 "git add <文件>..." 以包含要提交的内容）

    homePage.txt
    showPage.txt

提交为空，但是存在尚未跟踪的文件（使用 "git add" 建立跟踪）
我们假设homePage功能已经开发完成，showPage功能正在开发中，此时最好将已完成的功能先提交。可以看出，git的提醒功能很强大，通过 “git add <文件名>…” 指令将文件加入暂存区。
$ git add homePage.txt
$ git status
位于分支 master
您的分支与上游分支 'origin/master' 一致。
要提交的变更：
  （使用 "git reset HEAD <文件>..." 以取消暂存）

    新文件：   homePage.txt

未跟踪的文件:
  （使用 "git add <文件>..." 以包含要提交的内容）

    showPage.txt
homePage已经处于暂存区了，提交只是针对暂存区的，我们将homePage功能提交。showPage不会受影响，可以继续showPage的开发。
$ git commit -m "首页开发完成"
[master ec9541d] 首页开发完成
 1 file changed, 1 insertion(+)
 create mode 100644 homePage.txt
$ git status
位于分支 master
您的分支领先 'origin/master' 共 1 个提交。
  （使用 "git push" 来发布您的本地提交）
未跟踪的文件:
  （使用 "git add <文件>..." 以包含要提交的内容）

    showPage.txt

提交为空，但是存在尚未跟踪的文件（使用 "git add" 建立跟踪）
使用 “git commit -m ” message”” 对暂存区内容提交，”-m”参数是对本次提交添加一个描述信息，Git 不建议没有描述信息的提交。
diff差异比较
在本地修改了代码，准备将代码提交到服务器之前，一般我们需要对修改的代码进行下检查。如果修改点比较多，直接看代码检查时，往往容易遗漏，而且不够直观。这个场景下使用Git差异比较工具再合适不过了，差异比较工具会对比修改前后的文件，将差异点显示出来。
$ git diff
diff --git a/showPage.txt b/showPage.txt
index 0d62ae4..dd01952 100644
--- a/showPage.txt
+++ b/showPage.txt
@@ -1,7 +1,7 @@
 show page ... developing...
 我没被修改。

-这一行被删除了。
-这一行将被修改。
+这一行将被修改。追加，被修改。
+我是新添加的行。

 我也没有被修改。
对showPage修改后，使用 ” git diff ” 查看修改内容。看不懂？？那可不行！
diff --git a/showPage.txt b/showPage.txt 
这一行表明diff的输出格式是git类型的，后面的内容可以理解为是a版本的showPage和b版本的showPage进行比较。
index 0d62ae4..dd01952 100644 这一行涉及到Git对象的概念，意思是版本库index区的0d62ae4对象与工作区的dd01952对象进行比较。对象的ID是版本的哈希值，这里没有显示完全，前面显示出来的部分能确保独一无二即可。后面数字 100 表示文件类型为普通文件，644 表示对文件的操作权限，即 -rw-r–r– 。
--- a/showPage.txt用 ” — ” 表示a版本的showPage，+++ b/showPage.txt用 ” +++ ” 表示b版本的showPage。
@@ -1,7 +1,7 @@这里显示了两个版本的信息，前面的 ” -1,7 ” 表示从 a 版本的第 1 行开始，往下 7 行。后面的 ” +1,7 ” 表示从 b 版本的第 1 行开始，往下 7 行。
再下面展示的是文件内容，以 ” - ” 开头的行表示 a 版本存在，而 b 版本不存在。以 ” + ” 开头的行表示 b 版本存在，而 a 版本不存在。这两个符号都没有的行是上下文，两个版本中都存在。
这里还有一个问题，我们将修改后的文件加入暂存区后，再执行 ” git diff ” 指令发现没有差异信息输出。这是因为直接使用 ” git diff ” 查看的是工作区和暂存区的差异。不同参数的指令还有两个。
$ git diff --cached     // 暂存区和 HEAD 的差异
$ git diff HEAD         // 工作区和 HEAD 的差异
•	1
•	2
（HEAD 可以理解为一个头指针，表示当前工作区的基础版本，关于分支和 HEAD 的概念后面会说） 
使用这两条指令都可以得到和上面相同的差异输出。
git忽略
对于一些编译产生文件，或者本地配置文件是不需要提交到版本库的。但是每次使用 git status 查看状态时，总会提示文件处于未跟踪状态，看着心烦。Git 提供了文件忽略功能，可以将不关心的文件添加到忽略清单中，再使用 git status 就不会看到这些文件了。需要说一点，忽略只对未加入版本库的文件有效。下面介绍两种忽略方式。
第一种是共享式。创建一个名为 ” .gitignore ” 的文件，将要忽略的文件和目录写入，然后把该文件提交到版本库中，这样所有克隆的版本库都可以共享这个忽略文件。（也可以不加入版本库，做为独享忽略文件使用，可以把忽略文件自身也加入到忽略清单中）
$ vi .gitignore
•	1
$ cat .gitignore 
*.class
bin/
*.iml
note.txt
文件名可以使用通配符。
第二种是独享式。编辑.git/info/exclude,将要忽略的文件和目录写入即可。
分支是什么？
git中的每次提交都是基于上次提交的（除了初始提交，它没有父提交），所以整个提交历史可以串成一条线，而一次提交就可以理解为线上的一个节点。如图： 
 
图中A、B、C、D…均是一次提交，master和branch_feature为两个分支。我们实际操作演示下图中的场景。
$ git branch
* master
使用git branch命令查看本地分支情况，有 ” * ” 号标识的为当前所处分支，可以看出目前本地只有一个分支master。
$ git log --oneline -5
a4ecd06 commit E
d27551e commit D
156bc90 commit C
eaa734e commit B
fc0645e commit A
对 master 分支进行 5 次提交，使用$ git log --oneline -5命令查看最近 5 次的提交信息，提交信息简短显示。git中的一次提交是一个commit对象，有自己的ID和描述，commitID使用40位的SHA1哈希值字符串表示（这里没有完全显示，只要不与其他ID冲突即可）。” commit E ” 是本次提交的描述信息。
$ git branch branch_feature d27551e
$ git branch 
  branch_feature
* master
$ git checkout branch_feature 
切换到分支 'branch_feature'
$ git log --oneline -4
d27551e commit D
156bc90 commit C
eaa734e commit B
fc0645e commit A
使用$ git branch branch_feature d27551e命令基于提交 ” d27551e commit D ” 创建新分支 ” branch_feature ” 。使用检出命令 $ git checkout branch_feature切换到 ” branch_feature ” 分支，然后查看提交历史，当前分支最后提交为commit D ，并且具有自 D 以前的所有提交历史。
那么分支到底是什么呢？分支其实就是一个引用，它指向一次提交，该提交是本分支的最后一次提交。分支的存在形式是.git/refs/heads目录下的一个文件。我们研究一下这个文件。
$ ls .git/refs/heads/
branch_feature  master
这个目录下有两个文件，刚好对应本地的两个分支。看一下这个文件的内容：
$ cat .git/refs/heads/branch_feature 
d27551e1d893e3bb52ec255fda33102419a0f8a8
文件内容是一个id，再看下这个id对应的是什么：
$ git cat-file -t d27551e1d89
commit
是一个commit对象，看一下这个commit对象的详细信息：
$ git cat-file -p d27551e1d89
tree c350f8c30083edb9c60b688e5f09bd9f7896d2f3
parent 156bc9054a5e6a3e3f3ec8914644e70829dcd01c
author huangxu <hxdaxu9934@163.com> 1499238679 +0800
committer huangxu <hxdaxu9934@163.com> 1499238679 +0800

commit D
正是 commit D 。
既然分支是一个引用，当该分支上有新的提交发生时，这个引用会发生什么变化呢？以下创建一个新的提交：
$ echo F > F
$ git add F
$ git commit -m "commit F"
[branch_feature aaea208] commit F
 1 file changed, 1 insertion(+)
 create mode 100644 F
$ git log -1
commit aaea2082dfcb5ec21dcc14dda1e207c53325eeff
Author: huangqingchun <huangqingchun@gionee.com>
Date:   Wed Jul 5 17:47:29 2017 +0800

    commit F
看一下分支文件内容：
$ cat .git/refs/heads/branch_feature
aaea2082dfcb5ec21dcc14dda1e207c53325eeff
•	1
•	2
正是 commit F 的 ID ，分支引用已经指向了提交 F 。分支上有新的提交发生时，分支文件内容会被更新，内容始终是该分支的最后一次提交。
分支也可以理解为一条提交历史线，提交或重置操作会使提交历史线增长或缩短。
现在本地版本库存在了两个分支，在这两个分支上可以分别提交代码，彼此不会受影响。
HEAD是什么？
在对提交重置时，会用到git reset --soft HEAD^命令。HEAD 是什么呢？
HEAD 也是一个引用或者指针，不同于分支，HEAD代表的是当前工作区，它的存在形式是文件.git/HEAD。
$ cat .git/HEAD
ref: refs/heads/branch_feature
•	1
•	2
HEAD 指向了 branch_feature 分支！分支指向具体的提交，这样便确定了工作区的状态。当切换分支时， HEAD 的内容会被改变，指向新的分支。
使用检出命令 git checkout < branch_name > 切换分支，如果此命令后面不使用分支名，而用具体的提交 id ，HEAD 的内容也会变为具体的提交id（分离头指针状态），这个状态下的提交不会被任何分支记录，如非特殊需要，一般也不会 checkout commitID 。
其实，上面我们做的提交，并不是直接作用于分支的，而是 HEAD ，是不过当前 HEAD 是指向分支 branch_feature 的。表现就是提交和重置操作改变了 HEAD ，而 HEAD 指向当前分支，当前分支所指向的具体 commitID 被改变。如果 HEAD 指向具体的 commitID （分离头指针状态），提交或重置操作就不会影响分支，也就不会被分支记录下来。
分支可以理解为分出来的不同提交历史线，头指针HEAD 可以在这些提交历史线之间切换，也可以在某条线上面滑动。
cherry-pick 功能
开发工作中不要在主分支直接操作。如果新功能开发周期比较长，应该创建一个新的本地分支，在该分支开发提交，新功能完成后再合入主分支。
cherry-pick 功能是拣选一个提交应用于当前分支，就是将一个提交放在当前HEAD上形成一个新的完全一样的提交。下面在 branch_feature 分支创建一个提交，然后 cherry-pick 到 master 分支。
$ git diff
diff --git a/showPage.txt b/showPage.txt
index 0d62ae4..aea47d9 100644
--- a/showPage.txt
+++ b/showPage.txt
@@ -1,7 +1,6 @@
 show page ... developing...
 我没被修改。

-这一行被删除了。
-这一行将被修改。
+这一行已经被修改。修改人huangxu。

 我也没有被修改。

$ git add showPage.txt 
$ git commit -m "修改showPage"
[branch_feature b6e1a56] 修改showPage
 1 file changed, 1 insertion(+), 2 deletions(-)

$ git status
位于分支 branch_feature
无文件要提交，干净的工作区
然后切换到 master 分支，并查看 branch_feature 分支的提交历史。
$ git checkout master
$ git log branch_feature  --oneline -1
b6e1a56 修改showPage
$ git cherry-pick b6e1a56
[master 25f1dc0] 修改showPage
 1 file changed, 1 insertion(+), 2 deletions(-)
$ git log --oneline -1
25f1dc0 修改showPage
•	1
•	2
•	3
•	4
•	5
•	6
•	7
•	8
使用 git cherry-pick 执行拣选操作，上面的输出看出已经在 master 分支形成了新的提交，下面看一下内容是否发生了变化。
$ cat showPage.txt 
show page ... developing...
我没被修改。

这一行已经被修改。修改人huangxu。

我也没有被修改。
内容也已经发生了变化。
冲突解决
以 cherry-pick 为例，如果双方对同一文件的同一位置分别做了不同的提交，在合入时就会发生冲突，需要将冲突解决才能完成合入。
上个例子中在 branch_feature 修改了showPage 文件，下面在 master 分支修改相同位置并完成提交，这样在 cherry-pick 时就会发生冲突。
$ git diff
diff --git a/showPage.txt b/showPage.txt
index 0d62ae4..cd2ad48 100644
--- a/showPage.txt
+++ b/showPage.txt
@@ -1,7 +1,6 @@
 show page ... developing...
 我没被修改。

-这一行被删除了。
-这一行将被修改。
+这一行被修改。修改人hxdaxu。

 我也没有被修改。
$ git add .
$ git commit -m "hxdaxu修改showPage"
下面执行拣选操作，查看输出信息。
$ git cherry-pick b6e1a56
error: 不能应用 b6e1a56... 修改showPage
提示：冲突解决完毕后，用 'git add <paths>' 或 'git rm <paths>'
提示：对修正后的文件做标记，然后用 'git commit' 提交
提示错误，需要解决冲突，查看下当前状态。
$ git status
位于分支 master

您在执行拣选提交 b6e1a56 的操作。
  （解决冲突并运行 "git cherry-pick --continue"）
  （使用 "git cherry-pick --abort" 以取消拣选操作）

未合并的路径：
  （使用 "git add <file>..." 标记解决方案）

    双方修改：     showPage.txt

修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
•	1
•	2
•	3
•	4
•	5
•	6
•	7
•	8
•	9
•	10
•	11
•	12
•	13
git 给出提示有双方修改的位置，git 是不能帮用户决定以哪次修改为准的。我们使用手动修改的方式解决冲突，先查看一下文件内容：
$ cat showPage.txt 
show page ... developing...
我没被修改。

<<<<<<< HEAD
这一行被修改。修改人hxdaxu。
=======
这一行已经被修改。修改人huangxu。
>>>>>>> b6e1a56... 修改showPage

我也没有被修改。
•	1
•	2
•	3
•	4
•	5
•	6
•	7
•	8
•	9
•	10
•	11
git 对冲突位置做了标记。我们已经知道 HEAD 代表的是当前工作区的一个基础版本，在<<<<<<< HEAD与=======之间的是当前分支的修改，在=======与>>>>>>> b6e1a56... 修改showPage之间是所合入版本的修改，也就是 branch_feature 分支做的修改。下面修改文件内容将冲突解决，标记去掉：
$ cat showPage.txt 
show page ... developing...
我没被修改。

这一行被修改。修改人hxdaxu和huangxu.

我也没有被修改。
•	1
•	2
•	3
•	4
•	5
•	6
•	7
解决冲突后，提交，完成合入操作。
$ git add showPage.txt
$ git commit -m "共同对showPager的修改"
[master bf9069c] 共同对showPager的修改
 1 file changed, 1 insertion(+), 1 deletion(-)
$ git status
位于分支 master
您的分支领先 'origin/master' 共 2 个提交。
  （使用 "git push" 来发布您的本地提交）

无文件要提交，干净的工作区
•	1
•	2
•	3
•	4
•	5
•	6
•	7
•	8
•	9
•	10
提交后在查看状态，错误信息已经不见了，冲突解决了。
git stash 保存工作进度
当在 branch_feature 开发新功能，还在开发中，突然有个紧急任务，需要切换到 master 分支修复一个 bug 。我们希望切换到 master 分支时有个干净的工作区，这时有两个选择，将未完成的任务提交，或者保存当前工作进度。我们采用保存工作进度的做法：
$ git add settingsPage
$ git status
位于分支 branch_feature
要提交的变更：
  （使用 "git reset HEAD <file>..." 撤出暂存区）

    新文件:       settingsPage
对于未跟踪的文件需要先添加到暂存区，才可以保存进度。
$ git stash 
Saved working directory and index state WIP on branch_feature: b6e1a56 修改showPage
HEAD 现在位于 b6e1a56 修改showPage
$ git status
位于分支 branch_feature
无文件要提交，干净的工作区
使用git stash保存进度后，工作区的变动都被保存了下来，这时可以切换到其他分支处理问题，之后再切换回来恢复进度就可以了：
$ git stash pop
位于分支 branch_feature
要提交的变更：
  （使用 "git reset HEAD <file>..." 撤出暂存区）

    新文件:       settingsPage

丢弃了 refs/stash@{0} (729d13b1a47a81479b75175f7677182acd6c1ce4)
$ git status
位于分支 branch_feature
要提交的变更：
  （使用 "git reset HEAD <file>..." 撤出暂存区）

    新文件:       settingsPage
使用git stash pop恢复进度，工作区的变动又回来了，关于进度保存还有些实用的功能：
$ git stash save "保存settingsPage"
Saved working directory and index state On branch_feature: 保存settingsPage
HEAD 现在位于 b6e1a56 修改showPage
$ git stash list
stash@{0}: On branch_feature: 保存settingsPage
$ git stash pop stash@{0}
位于分支 branch_feature
要提交的变更：
  （使用 "git reset HEAD <file>..." 撤出暂存区）

    新文件:       settingsPage

丢弃了 stash@{0} (04036d18ac1845b900951f94df76c9b378c4ef59)
进度保存时可以使用git stash save "message"添加一个描述。进度保存可以保存多处，使用git stash list查看进度保存列表，并可以选择需要恢复的进度。
文中指令汇总
$ git config --global user.name "Your Name"     // 配置用户名
$ git config --global user.email "Your Email"   // 配置邮箱

$ git add <file name>           // 将修改内容添加至暂存区
$ git add --all                 // 添加所有改变的已跟踪文件和未跟踪文件

$ git diff                      // 工作区和暂存区差异比较
$ git diff --cached             // 暂存区和 HEAD 的差异
$ git diff HEAD                 // 工作区和 HEAD 的差异

$ git status                    // 查看当前状态
$ git commit -m "message"       // 提交说明

$ git reset --soft HEAD^        // 重置上次提交，保留修改的内容
$ git reset --hard HEAD^        // 彻底重置上次提交，不保留修改
$ git reset --hard <commit id>  // 彻底重置至某次提交，不保留修改

$ git log                       // 查看提交历史
$ git log --oneline             // 提交历史简短显示
$ git log -5                    // 查看最近5条提交历史

$ git checkout <branch_name>    // 切换分支
$ git checkout -b <branch_name> // 创建并切换到该分支

$ git branch                    // 查看本地分支列表
$ git branch <branch_name>      // 创建新分支，基于当前HEAD
$ git branch -D <branch_name>   // 删除分支

$ git stash                     // 保存进度
$ git stash pop                 // 恢复最近一次保存的进度
$ git stash save "message"      // 保存进度，并添加一个描述
$ git stash list                // 查看保存的进度列表

$ git cherry-pick <commit ID>   // 拣选一次提交应用于当前HEAD

git架构图解
　　最近又遇到Git了，发现网络上Git的资料确实不咋滴，难懂不全面。没法，自己来整理整理吧。至于Git是什么我就不多说了，相比svn上手确实更难。与svn集中版本库相比较，Git被称作分布式版本库，在分布式的版本库中任何一个库都可以作为中心库看待。如果说svn是颗树，那么Git就像一张网。Svn在每个目录都有一个.svn文件夹存放信息，而git只在根目录才有，这就决定了svn可以单独拉取某个子目录或者某个文件，而git需要全部拉取。传言高版本git可以，反正我没成功。
废话不多说，先上一张图（在此称赞一下亿图图示专家，尝试了n多软件，连一直在用的xmind也无法表达我的思维。这软件还是国产的哦，简洁美观，顶一下）
 
图比较大，如果看不清楚请下载我资源，原创的收1分，望理解~~！
 
　　下面开始对图详解，假设你是图中的开发者1，红线代表你可以操作的，流程中的虚线代表可以有，但是往往不用的。讲解的顺序图中红圈已经标明。
1：git --bare init
　　--bare代表只存储变化不存储实际文件，作为中心服务器一般都是这样初始化的。初始化后会默认创建一个master分支（怎么创建其他分支参考下面的git push）。这里一个有趣的问题来了，不是说git是分布式的么，任何一个库都是中心库，那么为何还有中心服务器？中心服务器有什么不同？弄一个中心服务器的好处就不多说了，简单解释下中心服务器的不同之处。我们init一个git库后，要想其他人能访问，必须得共享出去。共享的方式有好多种，需要依赖其他软件如gitosis等。具体请查阅这篇文章。这样别人就能通过clone、pull、push、fetch命令跟你打交道了。如果开发者2也安装了发布工具，那么同一个项目你可以跟两个远端库交互。这种情况很少遇到，一般都是大家通过中心服务器来间接交互。
2-1：git clone [-b branchName] url
　　首次参与项目需要先clone下来，-b参数可以指定分支名，不指定默认使用master分支。
　　这时候你本地会有一个项目的拷贝。这时候你可以查看一些信息：
1）git remote show 查看所有远程库，你会发现输出origin，origin就代表你刚才clone下来的远程库。如果开发者2安装了发布软件，你可以使用“git remote add dota 开发者2url”来添加一个远程库且你把他命名为dota，这样你这个项目就可以跟两个远程库打交道了。对应的还有git remote rm dota，git remote rename dota dota2，git remote set-url dota url2。
2）git remote show origin 查看远程库origin的详细信息。
3）git branch [-r -a] 查看分支，不带参数是查看本地分支，-r查看远程的，-a查看所有。其中前面带*号的代表你当前工作目录所处的分支，remotes代表远程库，remotes\origin代表远程库中的origin，remotes\origin\master代表远程库中的origin中的master分支。HEAD那个不懂干嘛
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
2-2：git pull origin branch1
　　从远端origin拉取branch1分支的内容然后合并到当前所处的本地分支。直接git pull的话默认远程库是orgin，默认远程库的分支是master。
2-3：git fetch origin branch1或者git fetch origin branch1:tmp
　　从远端origin拉取branch1分支的内容到本地origin/branch1分支里面，如果本地没有这个分支就创建。更清晰点的应该使用git fetch origin branch1:tmp直接指明本地分支用tmp这个名字。直接git fetch是什么效果还没试过。由此可见git fetch和git pull都是从远程拉取分支到本地，不同的是git pull拉取后马上自动合并。
3：git push origin master:branch1
　　把本地的master分支提交到远程origin的branch1分支上，如果远程没有branch1分支，则创建，所以远程分支的创建方法应该如此。对应的远程分支的删除则是git push origin :branch1，命令中本地分支名留空则能删除远程branch1分支了。值得指出的是，远端分支不需要指明全称，例如你git branch -a看到有一个remotes/origin/dir1/branch1那么你只需要写上dir1/branch1即可。下面来最讨厌的默认情况。
git push origin master（本地master到远程master）
git push（哥不干了）
4：git branch -b branchnew branchold
　　这个是本地分支的创建方法之一（另一种方法参考git checkout），以branchold为模板，创建branchnew分支，这个branchold既可以是本地的也可以是远端的，如果是远端分支的记得输入全名，例如remotes/origin/master。如果没用指明branchold，则默认使用HEAD所指向的提交创建分支。
本地分支的删除使用git branch -d branchname。如果该分支有改动且没有合并到其他分支上，会报错。你可以使用-D强行干掉。
5：git checkout -b branchnew branchold
　　该命令的本意是切换本地分支，可以使用最简单语法git checkout branchname，如果分支不存在则报错。而git checkout -b branchnew branchold是branchnew存在则切换，不存在则以branchold为模版创建一个新分支，然后切换到新分支上。这是创建本地分支的第二个方法。
6：
其实6可以当不存在，如图如果你是开发者1，那么工作目录就是branchA，你用pull、fetch以及即将介绍的merge处理分支的时候，如果branchA被更新了，那么就相当于你的工作目录被更新了。分开是为了下面的介绍。
7：git merge branch1 branch2 branch3......
　　把branch1、branch2、branch3等合并到当前分支。当然，实际中当然是一个一个来了，要不处理冲突可能会吓死你。
8和9：git add、git rm、git reset和git commit
　　到了这里，需要解释下图中黄色底的那些文字的概念了。
　　如果你在branchA上工作，其实branchA还可以细分为三块。
　　work tree：你当前在修改的项目，你能看到的文件。
　　index file：又称staged，当你使用git add file1 file2......系统会在索引文件中记录你打算提交的东西（新添的文件和修改的文件都使用git add，别纠结这个了，删除的文件使用git rm）。
　　commit：真正的branchA分支体。使用git commit -m “msg”可以提交索引文件中记录的提交内容。如果你之前没有使用git add或者git rm把文件修改标记到索引文件中，则提交失败或者提交不全。有一种情况例外，-a参数可以直接提交所有被git管理的文件，不需要使用git add添加到索引文件。至于-m则是必须的。
    git add可以指明一个目录，而git rm则需要一个一个文件来指定，想快捷只能用git commit -a来搞。
　　如果一个文件被你add或者rm到index file里面了，你想取消怎么办呢？git reset直接重置index file，git reset filename则取消某个文件在index file的登记。
　　如果你修改了一个文件，想还原怎么办呢？git checkout -- filename，记住--和filename之间有空格。
　　如果你想还原所有的文件到某个版本怎么办呢？git reset --hard HEAD，--和hard之间没有空格，HEAD意义我就不说了，你可以用commit的id替代，通过git log可以看到各个commit的id。
10：git tag tagname
　　标签是什么？其实没那么复杂，跟分支不是一个概念的。只是相当于你在当前分支的当前版本用一些字符标记一下，一般标签内容都是项目版本号。开发到一定程度标记一下，代表到此为止版本1.0.1完成了出来了。
　　本地标签的创建 git tag tagname或者git tag -a v1.1.1 -m ‘my v1.1.1’
　　本地标签的删除 git tag -d tagname
　　本地标签的查看 git tag -l ‘1.4.2*’
　　使用git push的时候默认是不提交标签的，所以当你打了标签要使用git push origin -tags来单独提交标签。某个版本打标签要在commit之后。上述创建标签的-a参数一定要加上-m输入描述。或者直接-s替代-a不需要-m，描述自动帮你加上。
　　远程标签删除使用 git push origin :refs/tags/0.1.3
　　标签详细查看使用 git show v1.1.1
11：git diff
　　这玩意太博大精深了，不做详解，列出常用的。
　　master分支的某个版本号某个文件与当前该文件比较：
　　git diff 0c5ee16a6a4c849d0ae0448caa8ff174399c7c3c ./socket_helper.cpp
　　两个tag（项目版本）之间相差哪些文件，做补丁包常用：
　　git diff tag1 tag2 --stat
　　太多了，我研究不过来，改天吧。
待写问题：冲突识别和解决
一张图诠释Git所有命令
2016年12月02日 18:41:37 wiwief 阅读数：560 标签： git 更多 
个人分类： git 
一张图诠释Git所有命令
 
git 各区域转换图解
 
•	注意：此图解主要针对已跟踪状态文件而言，对于新增文件，有些操作会有问题。
1. 修改本地已被跟踪文件，文件进入未暂存区域。
2. 未暂存区域转到暂存区域
•	git add files
3. 暂存区提交到本地仓库
•	git commit -m 
4. 直接从未暂存区提交到本地仓库
•	git commit -am
•	经测试，对已跟踪的文件可以正确执行，而对于未跟踪文件（即新增文件）则会出错
5. 本地库回退到暂存区
•	git reset –soft hash值
•	git reset –soft origin/master
•	一般回退到暂存区的文件作排查用，不要直接修改，不然会同时出现在暂存区和未暂存区（其实即使修改了也木有太大关系）
6. 本地库回退到未暂存区
•	git reset –mixed hash值
•	git reset –mixed origin/master
•	一般回退到未暂存状态就是为了进一步的修改
7. 本地库回退到文件初始状态（即此版本的）
•	git reset –hard hash值
•	注意这里，通常先执行一次fetch，保证本地版本是origin的最新版本，然后再回退。（最厉害的是，这么操作不会有冲突，直接让文件变成和origin保持一致） 
o	git fetch origin
o	git reset –hard origin/master
o	特别注意：这么操作会使你对文件的修改全部消失，还原成最初状态。
•	(针对上一条情况衍生讲解)通常在推送到origin时，先要pull，然后再推送，一般是修改提交了的文件和pull下来的同一个文件产生冲突（所以建议修改代码前，一定先要pull）
o	git pull
o	git push origin master
8. 暂存区回退到未暂存区
•	git reset – files
•	git rest
o	撤销所有暂存区的文件
9. 未暂存区回退到文件初始状态
•	git checkout – files
10. 暂存区回退到文件初始状态
•	git checkout head – files
Git的工作区、暂存区和分支
工作区和暂存区
Git与其他版本不同的地方就在于它有一个暂存区的概念。
工作区 
就是在电脑上能看到的目录，比如我电脑上的learngit就是一个工作区。 
 
版本库 
上图可以看到有一个.git隐藏目录，这不不算工作区，而是Git的版本库。 
Git版本库存了很多东西，其中最重要的就是被称为stage的暂存区，还有Git为我们自动创建的一个分支master，以及指向master的一个指针HEAD。 
 
当把文件往Git版本库添加的时候，是分两步执行的：
1.	第一步是用git add把文件添加到了暂存区；
2.	第二步使用git commit把暂存区中的文件添加到当前分支上。
git diff三种对比
1. git diff 
将工作区中某个文件和暂存区的比较。
2.git diff –cached 
将暂存区中的某个文件和分支上的作比较。
3.git diff HEAD – 
将工作区中的某个文件与分支上的作比较。

