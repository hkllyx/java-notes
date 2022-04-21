- [分支和合并](#%e5%88%86%e6%94%af%e5%92%8c%e5%90%88%e5%b9%b6)
  - [git branch](#git-branch)
  - [git checkout](#git-checkout)
  - [git log](#git-log)
  - [git merge](#git-merge)
  - [git mergetool](#git-mergetool)
  - [git stash](#git-stash)
  - [git tag](#git-tag)

# 分支和合并

## git branch

列出，创建或删除分支。

```
git branch [-r | -a] [--list]
           [(--merged | --no-merged) [<commit>]]
           [--contains [<commit]] [--no-contains [<commit>]]
           [<pattern>…​]
```

当使用 `--list`（`-l` 也可以，但不支持 pattern） 选项或者不提供任何选项和参数时，列出现有的所有分支，当前分支使用 "* " 开头标志出。可以使用 `-r` 列出远程分支，`-a` 列出所有本地和远程分支。可以提供 pattern 匹配分支名来过滤结果。

`--contains` 选项用于列出包含指定提交的分支（就是分支的顶端提交是指定提交的后代），`--no-contains` 则列出不包含指定提交的分支。

`--merged` 选项可以将匹配 pattern 的分支合并到指定提交（就是该分支的顶端分支可以从指定提交中可达），`--no-merged` 则相反。如果提交不提供，则省缺值为 HEAD。

```
git branch [--set-upstream | --track | --no-track] [-l] [-f] <branchname> [<start-point>]
git branch --unset-upstream [<branchname>]
```

创建一个新的名为 branchname 的分支，它指向 HEAD 的 start-point 处（如果已提供）。注意，此命令不会将工作树切换到新建的分支，如果需要切换，请使用 `git checkout <newbranch>`。

当本地分支是从远程跟踪的分支开始的，Git 将远程分支拉取（git pull）并适当地和该分支合并。这个行为可以通过设置全局配置 branch.autosetupmerge 更改。可以使用 `--track`, `--no-track` 选项覆盖该设置，也可以稍后使用 `--set-upstream` 执行。


```
git branch (-m | -M) [<oldbranch>] <newbranch>
```

重命名分支，重命名后 reflog 条目将指向 newbranch。如果 newbranch 已存在，`-m` 重命名失败，而 `-M` 则将强制重命名。

```
git branch (-d | -D) [-r] <branchname>...
```

删除指定分支。如果分支被删除，分支的 reflog 也会被删除。可以使用 `-r` 删除远程分支。

```
git branch --edit-description [<branchname>]
```

修改分支的描述信息。

## git checkout

切换分支或恢复工作树文件。

```
git checkout [option] [<branch>]
```
用于准备在指定分支上工作，通过更新工作树中的索引和文件，并将 HEAD 指向指定分支来切换到它。对工作树中的文件的本地修改被保留，这样它们就可以提交到指定分支。如果指定分支不存在而恰巧存在同名的远程分支，则命令同 `git checkout -b <branch> --track <remote>/<branch>`。

```
git checkout -b|-B [option] <new_branch> [<start point>]
```

用于创建新的分支并切换到该分支。`-b` 和 `-B` 选项的作用就是使用 `git branch` 创建分支。如果分支已存在，使用 `-b` 则导致失败，而 `-B` 则会强制创建（将原分支重置）。可以使用 `--track` 等选项传递给 `git branch` 命令。

```
git checkout --detach [option] [<branch>]
git checkout <commit>
```

用于准备在指定提交上工作，通过分离 HEAD，并更新工作树中的索引和文件。更新工作树中的索引和文件。对工作树中的文件的本地修改将被保留，这样产生的工作树将是提交中记录的状态加上本地修改。对分支使用 `--detach` 选项可以强制执行次该行为。

```
git checkout [-p|--patch] [<tree-ish>] [--] <pathspec>...
```

当给定 pathspec 或 `--patch` 选项时，不切换分支。它从索引文件或指定的 tree-ish (通常是提交) 中更新工作树中的指定路径。

常见选项：

| 选项 | 说明                                |
| ---- | ----------------------------------- |
| -f   | force，强制执行操作                 |
| -q   | quiet，静默执行操作                 |
| -m   | merge，将当前分支合并到切换目标分支 |

分离 HEAD（DETACHED HEAD）：HEAD 通常指一个指定的分支 (例如 master)。同时，每个分支引用一个特定的提交。

- 一个有三次提交的 reflog，其中一次被标记了，并且 master 分支被签出：

    ```
                HEAD (指向分支 master)
                 |
                 v
    a---b---c  branch 'master' (指向提交 c)
        ^
        |
      tag 'v2.0' (指向提交 b)
    ```

- 在这种状态下创建提交时，将更新分支以引用新的提交。具体来说，`git commit` 创建了一个新的提交 d，它的父节点是提交 c，然后更新 master 分支以引用新的提交 d。

    ```
                    HEAD (指向分支 master)
                     |
                     v
    a---b---c---d  branch 'master' (指向提交 d)
        ^
        |
      tag 'v2.0' (指向提交 b)
    ```

- 有时能够签出不在任何命名分支顶端的提交是很有用的，甚至可以创建一个没有被命名分支引用的新提交。让我们看看当我们签出提交 b 时的情况 (这里我们展示了两种可能的方法):

    ```
    $ git checkout v2.0  # 或者使用 git checkout master^^

       HEAD (指向提交 b)
        |
        v
    a---b---c---d  branch 'master' (指向提交 d)
        ^
        |
      tag 'v2.0' (指向提交 b)
    ```

- 请注意，无论我们使用哪个 checkout 命令，HEAD 现在都直接引用提交 b。这称为处于**分离的 HEAD 状态**。这仅仅意味着 HEAD 引用一个特定的提交，而不是引用一个命名的分支。让我们看看当我们创建一个 commit 时会发生什么：

    ```
    $ edit; git add; git commit

         HEAD (指向提交 e)
          |
          v
          e
         /
    a---b---c---d  branch 'master' (指向提交 d)
        ^
        |
      tag 'v2.0' (指向提交 b)
    ```

- 注意：新提交 e 只被 HEAD 引用，如果将 HEAD 重新指向分支 master，则提交 e 将会被 gc 回收。

## git log

显示提交日志。

```
git log [<options>] [<revision range>] [[--] <path>...]
```

常用选项：

| 选项                                            | 说明                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| -no-decorate, --decorate[=short\|full\|no]      | 输出提交的 ref 名称                                          |
| --pretty                                        | 个性化输出格式                                               |
| -\<num>, -n \<num>, --max-count=\<num>          | 限制输出的提交个数                                           |
| --skip=\<num>                                   | 跳过输出前几个提交                                           |
| --since\|--after\|--until\|--before=\<datetime> | 输出指定日期前 / 后的提交                                    |
| --author\|--committer=\<pattern>                | 输出符合正则表达式的作者的提交                               |
| --grep=\<pattern>                               | 输出符合正则表达式的提交信息的提交。可有多个，结果为并集     |
| -all-match                                      | 搭配 -grep 使用，表示提交信息需要满足所有 --grep，结果为交集 |
| --branches[=\<pattern>]                         | 输出符合正则表达式的分支的提交                               |
| --tags[=\<pattern>]                             | 输出符合正则表达式的标签的提交                               |
| --remotes[=\<pattern>]                          | 输出符合正则表达式的远程的提交                               |
| --graph                                         | 显示树型项图                                                 |
| --abbrev-commit                                 | 缩短提交的 SHA 值                                            |

格式化输出(--pretty)：

```
git log --pretty=oneline
```

每个提交只用一行输出：长 Hash 值 + 提交信息。

```
git log --pretty=format:'...'
```
| 占位符  | 说明                                                  |
| ------- | ----------------------------------------------------- |
| %H      | 提交 hash                                             |
| %h      | 缩短的 commit hash                                    |
| %T      | tree hash                                             |
| %t      | 缩短的 tree hash                                      |
| %P      | parent hashes                                         |
| %p      | 缩短的 parent hashes                                  |
| %an     | 作者名字                                              |
| %ae     | 作者邮箱                                              |
| %ad     | 日期 (--date= 制定的格式)                             |
| %cr     | 提交日期，相对格式 (1 day ago)                        |
| %cn     | 提交者名字                                            |
| %ce     | 提交者 email                                          |
| %d      | ref 名称                                              |
| %e      | 编码                                                  |
| %s      | 提交信息题                                            |
| %f      | 处理后的的提交信息题（如空格使用 - 代替），适合文件名 |
| %m      | 左，右或边界标记（\<>）                               |
| %n      | 换行                                                  |
| %Cred   | 切换到红色                                            |
| %Cgreen | 切换到绿色                                            |
| %Cblue  | 切换到蓝色                                            |
| %Creset | 重设颜色                                              |
| %C(...) | 制定颜色，如 color.branch.* 配置选项中所述            |
| %%      | %                                                     |

示例：

```
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative
```

将上示命令添加到配置的别名中，此后课使用别名快速使用该命令及选项（git log-fmt）：

```
git config --global alias.log-fmt "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative"
```

## git merge

将两个或多个开发历史连接在一起（合并分支）。

```
git merge [-n] [--stat] [--no-commit] [--squash] [--[no-]edit]
          [-s <strategy>] [-X <strategy-option>]
          [--[no-]rerere-autoupdate] [-m <msg>] [<commit>...]
git merge <msg> HEAD <commit>...
git merge --abort
```

## git mergetool

运行合并冲突解决工具来解决合并冲突。

```
git mergetool [--tool=<tool>] [-y | --[no-]prompt] [<file>...]
```

常用选项：

| 选项                       | 说明                                                                                                                                                                                                    |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -t \<tool>, --tool=\<tool> | 使用指定的合并解析程序。有效值包括 emerge，gvimdiff，kdiff3，meld，vimdiff 和 tortoisemerge，如果没有指定合并解析程序，将使用配置变量merge.tool。如果配置变量 merge.tool 没有设置会选择一个合适的默认值 |
| --tool-help                | 输出可能使用的合并工具列表                                                                                                                                                                              |
| -y \| --[no-]prompt        | 是否在每次调用合并解析程序之前提示                                                                                                                                                                      |

## git stash

将更改保存在脏工作目录中。

场景描述：

- 经常有这样的事情发生，当你正在进行项目中某一部分的工作，里面的东西处于一个比较杂乱的状态，而你想转到其他分支上进行一些工作。
- 问题是，你不想提交进行了一半的工作，否则以后你无法回到这个工作点。
- 解决这个问题的办法就是 `git stash` 命令。“储藏”可以获取你工作目录的中间状态 —— 也就是你修改过的被追踪的文件和暂存的变更，并将它保存到一个未完结变更的堆栈中，随时可以重新应用。

```
git stash [save [--patch] [-k|--[no-]keep-index] [-q|--quiet]
          [-u|--include-untracked] [-a|--all] [<message>]]
```

将本地修改储藏到一个新的 stash 中，并且将工作目录恢复到 HEAD 的状态（工作目录修改内容被清空）。

选项及部分：

| 选项及部分              | 说明                                                                                     |
| ----------------------- | ---------------------------------------------------------------------------------------- |
| message                 | 是可选的，但如果需要使用，save 不可省去，防止防止拼错的子命令生成不需要的 stash          |
| -k, --keep-index        | 已经添加到索引中的所有更改都保持不变                                                     |
| -u, --include-untracked | 所有未跟踪的文件也会被隐藏起来，然后用  git clean 进行清理，使工作目录处于非常干净的状态 |
| -a, --all               | 在 -u 的基础上隐藏和清除被忽略的文件                                                     |
| --patch                 | 交互地从  HEAD和工作树之间的差异中选择要隐藏的块。隐式带有 -keep-index 选项              |

```
git stash list [<options>]
```

列出当前所有的 stash

```
git stash show [<stash>]
```

显示指定 stash 和其 origin parent 的区别，没有指定 stash 则默认为最近的一个。

```
git stash (pop | apply) [--index] [-q|--quiet] [<stash>]
```

pop 将指定 stash 从储藏列表中移除并将其状态添加到当前工作目录的顶部，即执行储藏反向操作，工作目录必须与索引匹配。操作可能因为冲突而失败，stash 不会被移除，需要解决冲突并调用 git stash drop 操作手动删除。如果使用 --index 选项，则同时恢复 index（也可能因为冲突而失败）。如果 stash 未指定，默认为 stash@{0}。

apply 类似 pop 操作，但是并不会讲 stash 从储藏列表中移除。

```
git stash branch <branchname> [<stash>]
```

常见病切换到指定名称的分支

           Creates and checks out a new branch named <branchname> starting from the commit at which the <stash> was originally created, applies the changes recorded in <stash> to the new working tree and index. If that succeeds, and <stash> is a reference of the form stash@{<revision>}, it then drops the <stash>. When no <stash> is given, applies the latest one.

           This is useful if the branch on which you ran git stash save has changed
           enough that git stash apply fails due to conflicts. Since the stash is
           applied on top of the commit that was HEAD at the time git stash was run, it
           restores the originally stashed state with no conflicts.


## git tag