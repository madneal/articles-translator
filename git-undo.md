>原文：[How to undo (almost) anything with Git](https://github.blog/2015-06-08-how-to-undo-almost-anything-with-git/)
>
>译者：[madneal](https://github.com/madneal)
>
>welcome to star my [articles-translator](https://github.com/madneal/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

![](https://github.blog/wp-content/uploads/2019/03/community-twitter.png?resize=1201%2C630)

任何版本控制系统最有用的功能之一就是能够“撤消”错误。在 Git 中，“撤消”可能意味着许多略有不同的事情。

当你进行新的 commit 时，Git 会及时存储你的仓库在该特定时刻的快照；之后，你可以使用 Git 返回到项目的早期版本。

在这篇文章中，我将介绍一些你可能想要“撤消”所做更改的常见场景，以及使用 Git 执行此操作的最佳方法。

## 撤销一个“public”修改

**场景：** 你刚刚运行了 `git push`，将你的修改 push 到 GitHub，现在意识到有一个 commit 有问题。你想把这个 commit 撤销。

**撤销：** `git revert <SHA>`

**结果：** `git revert` 将创建一个与给定 SHA 相反的新 commit。如果旧 commit 是“matter”，则新 commit 是“anti-matter”——旧 commit 中删除的任何内容都将添加到新 commit 中，而旧 commit 中添加的任何内容都将在新 commit 中删除。

这是 Git 最安全、最基本的“撤消”场景，因为它不会更改历史记录，因此你现在可以使用 `git push` 来提交新的 commit来撤消错误的 commit。

## 修复上一个 commit message

**场景：** 你刚刚打错了最新一条 commit message，你使用 `git commit -m "Fxies bug #41"`, 但在 `git push` 之前你意识到应该写 `Fixes bug #42`。

**撤销：** `git commit --amend` 或 `git commit --amend -m "修复 bug #42"`

**结果：** `git commit --amend` 将更新最新 commit 并将其替换为新 commit ，将新 commit 与之前提交的 commit 的内容相结合。若当前没有任何 stage 内容，这只是重写了之前的 commit 消息。

## 撤销本地修改

**场景：** 猫走过键盘并以某种方式保存了更改，然后编辑器崩溃。不过，你还没有 commit 这些修改。你想要撤消该文件中的所有内容 - 只需返回到上次 commit 时的样子即可。

**撤消：** `git checkout -- <bad filename>`

**结果：** `git checkout` 将工作目录中的文件更改为 Git 之前保存的状态。你可以提供要返回的分支名称或特定 `SHA`，或者默认情况下，Git 会假设你要切换到 `HEAD`，即当前分支上的最后一次 commit。

请记住：你以这种方式“撤消”的任何更改实际上都会消失。它们从未被 commit ，因此 Git 无法帮助我们稍后恢复它们。确保你知道你在这里扔掉了什么！ （也许使用 `git diff` 来确认。）

## 重置本地修改

**场景：** 你在本地进行了一些 commit （尚未 push），但一切都很糟糕，你想要撤消最后三个 commit  - 就像它们从未发生过一样。

**撤消方式：** `git reset <last good SHA>` 或 `git reset --hard <last good SHA>`

**结果：** `git reset` 将仓库的历史记录一直回溯到指定的 `SHA`。就好像这些 commit 从未发生过一样。默认情况下，`git reset` 保留工作目录。 commit 已消失，但内容仍在磁盘上。这是最安全的选择，但通常，你会希望一次“撤消” commit 和更改 - 这就是 `--hard` 参数的作用。

## 撤销本地修改后恢复

**场景：** 你进行了一些 commit ，执行了 `git reset --hard` 来“撤消”这些更改（见上文），然后意识到：你想要恢复这些更改！

**撤消：** `git reflog` 和 `git reset` 或 `git checkout`

**结果：** `git reflog` 是恢复项目历史记录的绝佳资源。你可以通过 reflog 恢复几乎任何内容（任何你 commit 的内容）。

你可能熟悉 `git log` 命令，它显示 commit 列表。 `git reflog` 类似，但显示 `HEAD` 更改的时间列表。

一些注意事项：

* `HEAD` 只有在你切换分支时，使用 `git commit` 进行 commit 并使用 `git reset` 取消 commit 时，HEAD 会更改，但是当你 `git checkout -- <bad filename>` 时 `HEAD` 不会变化（来自较早的场景 - 如前所述，这些更改从未 commit ，因此 `reflog` 无法帮助我们恢复这些更改）。
* `git reflog` 不会一直有效。 Git 会定期清理“无法访问”的对象。不要指望在 `reflog` 一直发现几个月前的 commit 。
* 你的转发记录是你的，并且只属于你。你不能使用 `git reflog` 来恢复其他开发人员未 push 的 commit 。

![reflog*](https://github.blog/wp-content/uploads/2015/06/f6b9f054-d891-11e4-8c53-838eff9f40ae.png?resize=1429%2C644)

那么……如何使用 reflog 来“恢复”之前“撤消”的一个或多个 commit ？这取决于你到底想要完成什么：

* 如果你想恢复项目当时的历史记录，请使用 `git reset --hard <SHA>`
* 如果你想在工作目录中重新创建一个或多个文件，而不更改历史记录，请使用 `git checkout <SHA> -- <filename>`
* 如果你想将其中一个 commit 重放到存储库中，请使用 `gitcherry-pick <SHA>`

## 再一次，通过分支

**场景：** 你提交了一些 commit ，然后意识到你在 master 分支上。你希望可以在 `feature` 分支上提交 commit 。

**撤消：** `git branch feature`，`git reset --hard origin/master` 和 `git checkout feature`

**结果：** 你可能习惯使用 `git checkout -b <name>` 创建新分支 - 这是创建新分支并立即 checkout 的流行快捷方式 - 但你不想立即切换到刚刚创建的分支。在这里，`git branch feature` 创建了一个名为 `feature` 的新分支，指向你最近的 commit ，但让你依然在 master 分支上。

接下来，在任何新 commit 之前， `git reset --hard` 将 `master` 回退到 `origin/master`。不过不用担心，它们仍然可以使用。

最后，`git checkout` 切换到新 `feature` 分支，你最近的所有工作都完好无损。

## 分支省时大法

**场景：** 你基于 `master` 分支创建了一个新的 `feature` 分支，但是 `master` 远远落后于 `origin/master`。现在 `master` 分支与 `origin/master` 同步，你希望 `feaute` 的 commit 现在就开始，而不是远远落后。

**撤消方式：** `git checkout feature` 和 `git rebase master`

**结果：** 你可以使用 `git reset`（无 `--hard`，有意保留磁盘上的更改）然后 `git checkout -b <newbranch name>` 来完成此操作，然后重新 commit 更改，但这样，你会丢失 commit 历史记录。有一个更好的方法。


`git rebase master` 做了几件事：

* 首先，它找到当前分支和 `master` 分支之间的共同祖先。
* 然后它将当前的分支重置为该祖先，将所有后续 commit 保存在保留区域中。
* 然后它将当前分支前放到 `master` 的末尾，并在 `master` 最后一次 commit 后重放保留区域的 commit 。

## 批量撤消/重做

**场景：** 你从一个方向开始功能开发，但在中途，你意识到另一种解决方案更好。你有十几个 commit ，但你只想要其中的一些，不想要其它的了。

**撤消：** `git rebase -i <earlier SHA>`

**结果：** `-i` 将 `rebase` 置于“交互模式”。它像上面讨论的 rebase 一样开始，但在重放任何 commit 之前，它会暂停并允许你在重放时轻易修改每个 commit 。

`rebase -i` 将在默认文本编辑器中打开，并显示正在应用的 commit 列表，如下所示：

![rebase-interactive1](https://github.blog/wp-content/uploads/2015/06/f6b1ab88-d891-11e4-97c1-e0630ac74e74.png?resize=1459%2C495)

前两列是关键：第一列是为第二列中的 SHA 标识的 commit 选择的命令。默认情况下，`rebase -i` 假设每个 commit 都使用 `pick` 命令。

要删除 commit ，只需在编辑器中删除该行即可。如果你不再希望项目中存在错误 commit ，则可以删除上面的第 1 行和第 3-4 行。

如果要保留 commit 的内容但编辑 commit 消息，可以使用 `reword` 命令。只需将第一列中的单词 `pick` 替换为单词 `reword` （或只是 `r`）。现在可能你觉得要重写 commit 消息，但这行不通—— `rebase -i` 会忽略 `SHA` 列之后的所有内容。之后的文字实际上只是为了帮助我们记住 `0835fe2` 的含义。当你完成 `rebase -i` 后，系统将提示你输入需要写入的任何新 commit 消息。

如果你想将两个 commit 合并在一起，你可以使用 `squash` 或 `fixup` 命令，如下所示：

![rebase-interactive2](https://github.blog/wp-content/uploads/2015/06/f6b605ca-d891-11e4-98cf-d567ca9f4edc.png?resize=1449%2C339)

`squash` 和 `fixup` 向上合并 commit —— 使用这两个命令的 commit 将被合并到紧邻其之前的 commit 中。在这种情况下，`0835fe2` 和 `6943e85` 将合并为一个 commit ，然后 `38f5e4e` 和 `af67f82` 将合并为另一 commit 。

当你选择 `squash` 时，Git 会提示我们给新的组合 commit 一条新的 commit 消息；`fixup` 将为新 commit 提供列表中第一个 commit 的消息。在这里，你知道 `af67f82` 是一个“ooops” commit ，因此你只需按原样使用来自 `38f5e4e` 的 commit 消息，但你将为通过组合 `0835fe2` 和 `6943e85` 获得的新 commit 编写一条新消息。

当你保存并退出编辑器时，Git 将按从上到下的顺序应用你的 commit 。你可以通过在保存之前更改 commit 顺序来更改 commit 应用的顺序。如果你愿意，你可以通过如下安排将 `af67f82` 与 `0835fe2` 组合起来：

![rebase-interactive3](https://github.blog/wp-content/uploads/2015/06/f6b4a9d2-d891-11e4-9ac9-10039c031d06.png?resize=1445%2C326)

## 修复较早的 commit 

**场景：** 你未能在早期 commit 中包含文件，如果早期 commit 能够以某种方式包含你遗漏的内容，那就太好了。你还没有 push ，但这不是最近的 commit ，所以你不能使用 `commit --amend`。

**撤销：** `git commit --squash <SHA of the earlier commit>` 和 `git rebase --autosquash -i <even earlier SHA>`

**结果：** `git commit --squash` 将创建一个新的 commit ，其中包含类似 `squash!  Earlier commit`。（你可以手动创建带有类似消息的 commit ，但 `commit --squash` 可以节省你的打字时间。）

如果你不想提示你为组合 commit 编写新的 commit 消息，你也可以使用 `git commit --fixup`。在这种情况下，你可能会使用 `commit --fixup`，因为你只想在 rebase 期间使用早期 commit 的 commit 消息。

`rebase --autosquash -i` 将启动交互式 `rebase` 编辑器，但该编辑器将用任何 `sqush!` 和 `!fixup!`  commit 已经与 commit 列表中的 commit 目标配对，如下所示：

![rebase-autosquash](https://github.blog/wp-content/uploads/2015/06/f6a7a1d8-d891-11e4-8784-c32262ff54da.png?resize=1446%2C294)

使用 `--squash` 和 `--fixup` 时，你可能不记得要修复的 commit 的 SHA，只记得它是一到五个 commit 前的。你可能会发现使用 Git 的 `^` 和 `~ `运算符特别方便。 `HEAD^` 是 HEAD 之前的一次 commit 。 `HEAD~4` 是 HEAD 之前的四次 commit ，或者总共是五次向后 commit 。
## Stop tracking a tracked file

## 停止跟踪被跟踪的文件

**场景：** 你不小心将 `application.log` 添加到仓库中，现在每次运行应用程序时，Git 都会报告 `application.log` 中存在未暂存的更改。你将 `*.log` 放入 `.gitignore` 文件中，但它仍然存在 - 你如何告诉 git “撤消”跟踪此文件中的更改？

**撤消：** `git rm --cached application.log`

**结果：** 虽然 `.gitignore` 阻止 Git 跟踪文件的更改，甚至阻止它注意到以前从未跟踪过的文件的存在，但一旦添加并 commit 了文件，Git 将继续注意到该文件中的更改。同样，如果你使用 `git add -f` 来“强制”，或覆盖 `.gitignore`，Git 将继续跟踪更改。以后你不必使用 `-f`` 来添加它。

如果你想从 Git 的跟踪中删除那个应该被忽略的文件， `git rm --cached` 将从跟踪中删除它，但在磁盘上保留该文件不变。由于它现在被忽略，你将不会在 `git status` 中看到该文件，也不会意外地再次 commit 该文件的更改。

这就是使用 Git 撤消任何操作的方法。要了解有关此处使用的任何 Git 命令的更多信息，请查看相关文档：

* [checkout](http://git-scm.com/docs/git-checkout)
* [commit](http://git-scm.com/docs/git-commit)
* [rebase](http://git-scm.com/docs/git-rebase)
* [reflog](http://git-scm.com/docs/git-reflog)
* [reset](http://git-scm.com/docs/git-reset)
* [revert](http://git-scm.com/docs/git-revert)
* [rm](http://git-scm.com/docs/git-rm)
