> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 https://www.jianshu.com/p/202de00f267f

Git 作为一个目前非常流行的版本管理工具，深受开发者的喜爱。那么怎样才能将 Git 的作用发挥的更好呢？我根据实际的项目经验，归纳总结了以下 Git 工作流的最佳实践。这里所谓的最佳，是经过多次项目经验后，根据自身的情况总结出来的我认为最为合理的方案。如果您有不同的见解，欢迎留言讨论，我们共同进步：）

Git 工作流的最佳实践方案包括如下四个步骤：

#### 1\. 根据 task 创建对应的 branch

当我们开始针对一个 task 编码之前，首先第一步应该要创建一个新的 branch，然后 checkout 到这个新的 branch 上开始编码。我们不应该直接在 master 上进行新的 task 的编码工作，尤其是在团队成员较多的情况下。团队的每个成员都应工作于自己新创建的 branch 上，而不会操作 master 分支，这样做的好处在于 master 始终处于一种 “干净” 的状态，不会因为多人的同时操作而造成过多的冲突，同时也降低了 master 被误操作的可能性。具体的操作如下：

> // 切换到 master 分支；
> **git checkout master**
> // 拉取 master 远程分支的代码；
> **git pull origin master**
> // 创建新的分支并切换到新的分支上。
> **git checkout -b <my branch name>**

#### 2\. 在新建的分支上编码，push 代码到远程分支上

分支创建完毕之后，我们就可以开始在 branch 上进行编码了。这是我们完成 task 的最主要的阶段，绝大部分的工作在此阶段完成，同时它应该也是持续时间最长的阶段。它的主要任务就是完成 task 的编码工作，并最终将代码 push 到当前分支对应的远程分支上去。

首先看一下这个阶段 Git 工作的命令流，示例如下：

> // 创建新的 branch 后的第一天工作结束时，首先将改动的代码放入 index 区；
> **git add .**
> // 然后提交代码到本地仓库；
> **git commit  -m "The first commit message"**

> // 第二天开始工作前，切换到 master 分支；
> **git checkout master**
> // 从 master 的远程分支拉取代码；
> **git pull origin master**
> // 切换到 task 所在的本地分支；
> **git checkout <my branch name>**
> // 将 master 上的最新的代码合并到当前分支上，这里的 - i 的作用是将我们 当前分支之前的 commit 压缩成为一个 commit，这样做的好处在于当我们之后创建 pull request 并进行相应的 code review 的时候，代码的改动会集中在一个 commit，使得 code review 更直观方便；
> **git rebase -i master** 
> // 第二天工作结束之后，将第二天的改动放入 index 区；
> **git add .**
> // 提交代码到当前 branch 的本地仓库；
> **git commit  -m "The second commit message"**

> // 第三天开始工作前，
> **git checkout master** // 同上第二天
> **git pull origin master** // 同上第二天
> **git checkout** // 同上第二天
> **git rebase -i master**  // 同上第二天
> // 第三天工作结束之后，
> **git add .** // 同上第二天
> **git commit  -m "The third commit message"** // 同上第二天

_.........._

> // 最后，当 task 的所有编码完成之后，将代码 push 到远程分支。
> **git push --set-upstream origin <my branch name>**

通过以上的工作流可以看出，我们在完成 task 期间所有的代码都始终存放在我们新创建的 branch 的本地仓库上。只有当所有的编码工作完成之后，才会将最终的代码 push 到当前分支的远程仓库。这样做的好处在于，我们 push 到远程的代码，也就是之后会通过 pull request 被 code review 的代码，始终是针对一个单独 task 的完整代码，这将有利于之后 code review 的执行。不过这样做同时可能会存在一个缺点，那就是最终一次 push 的代码可能会非常庞大，这就要求我们对于 task 粒度的把握应该更合适。我们不应针对一个非常大的 task 创建 branch，完成编码，而是应该尽可能的将 task 分解成一个个粒度较小的子 task，进而针对子 task 创建 branch 完成编码的工作。这是一项非常有技巧的工作，需要丰富的实践经验，它也不是本节要讨论的内容，不再赘述。

#### 3\. 创建 pull request, 进行 code review

当所有的代码都已经被 push 到远程分支后，这时我们还不可以将代码合并到 master 上去，我们应该要做的是创建 pull request。pull request 的作用在于它可以使得代码在 merge 到 master 分支之前，能够被团队成员 code review，从而提高代码的质量以及降低出错的概率。实际项目中我们使用 jira 来帮助我们创建相应的 pull request，当然 Github 本身就具备创建 pull request 的功能。创建 pull request 的操作非常简单，无非就是点击创建 pull request 的按钮，填写 comment 信息，并输入可以进行 code review 的成员名称。当 pull request 创建完成之后，所有可以进行 code review 的团队成员都会收到邮件通知，并通过相应的 pull request 的链接查看代码的改动，从而完成 code review 的工作。这个步骤没有实际的 Git 指令的操作。

#### 4\. 合并代码到 master，并删除之前创建的 branch

当所有的 reviewer 都结束了 code review，且都已经将 pull request 标注为 approved 状态的时候，我们就可以将 branch 合并到 master 上去，并最终 push 到远程 master 分支。示例如下：

> **git checkout master** // 切换到 master 分支；
> **git merge <my branch name>**// 合并之前创建的分支的代码到 master 分支上；
> **git push origin master**// 将 master 的代码 push 到 master 的远程分支；
> **git branch -d <my branch name>**// 删除之前创建的分支。

经过以上四个步骤之后，一个 task 的 Git 的工作流就结束了。之后，我们可以愉快的开始下一个 task 了～～