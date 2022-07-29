## 愿景

#git/use

git 在提交的时候不经意会提交很多无用的 commit。为了让 commit 更整洁。所以要对无用的 commit 合并。

## 操作

使用 `git rebase` 命令可以完成这个功能。

```shell
# 从HEAD版本开始往过去数3个版本 
$ git rebase -i HEAD~3 
# 合并指定版本号（不包含此版本） 
$ git rebase -i [commitid]
```

说明：

- `-i（--interactive）`：弹出交互式的界面进行编辑合并
- `[commitid]`：要合并多个版本之前的版本号，注意：`[commitid]` 本身不参与合并

执行命令后会使用默认编辑器，打开操作界面。

```txt
pick 02b7fd9 9
f f016838 10

# Rebase 893949e..f016838 onto 893949e (2 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
```

操作后直接保存退出即可。如果要合并 commit 的话，把需要合并的操作都改成 f 即可。当然有一些操作会保留 message，需要编辑一下 message。
