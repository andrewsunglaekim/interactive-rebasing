# Learning Objectives
- Define a linked list
- Illustrate how git logs or histories are created.
- Use git rebase to replay commits on top of another reference
- Identify how rebase is used to pull in upstream changes
- Use an interactive rebase to reword commit messages
- Use an interactive rebase edit a commit
- Use an interactive rebase to squash a commit(s)
- Use an interactive rebase to drop a commit(s)


# Framing

We generally use rebase to merge in upstream changes to our current branch. It's a process on some teams that have become akin to git pull. We often pull in the latest changes from master and then rebase our feature branch against that master branch.

> PSA: we wont be getting into the merits of merge vs rebase, holy wars are fought over this.

In order to learn how to modify our history rebasing interactively, we need to have a clear understanding of what rebasing is and how it works.

A strong foundation in the knowledge of commit histories and the relationship of one commit to another commit will help us explore what actually happens in a rebase.

Today, we'll try and bridge some of our gaps in our knowledge in order to effectively and purposefully change our git histories.

# [The linked list](https://en.wikipedia.org/wiki/Linked_list)

Straight out of wikipedia:

> A linked list is a linear collection of data elements, whose order is not given by their physical placement in memory. Instead, each element points to the next.

# Commits

A git repo is just a linked list of commits. Every commit besides the initial commit, points to at least 1 commit. To illustrate this lets take a look at a very simple git history.

![simple git history](imgs/simple.png)

In this diagram, `D` points to `C`, `C` points to `B`, and finally `B` points to `A`. Commits only have pointers to their direct ancestors, it is only through these pointers that we can construct git logs and histories.

Let's take a look at a very common problem we come across in git. That problem is how to bring in upstream changes.

![common problem git diagram](imgs/commonProblem.png)

# Merging

One way we can solve this problem is through a merge:

![simple git history](imgs/merge.png)

We might notice that the diagram presenting the common problem had branch names of feature and master, we opt for the letters representing commits here. Depending on *where* merged we merge the branch, we could have two different different diagrams using branches.

Here's one of the ways it could go down:

![master branch merging feature diagram](imgs/masterMergeFeature.png)

And here's another:

![feature branch merging master diagram](imgs/featureMergeMaster.png)

These diagrams are here to illustrate that branches are merely pointers to a commit, a single commit. The reality is that two commits are being merged, not two branches. The above diagrams represent the same thing. They just had different branches pointing to them.

We'll be sticking to labeling commits with letters in this lesson. This will help us conceptualize future diagrams.

# Rebasing

There's one more way to "merge" the code in the following problem:

![common problem with letters diagram](imgs/commonProblemLetters.png)

We can rebase 1 or more commits on top of another commit.

In this case, we would rebase commits `E`, `F`, and `G` on top of commit `D`. If these were branch names the git command to this would look something like this:

```
$ git rebase D G

// or
$ git checkout G
$ git rebase D
```

In order to do this git works the histories back from each of the tips until it finds a common ancestor. In this case, commit `A`. It then grabs the diff between the commit we are rebasing(in our case `G`) and the common ancestor(in our case `A`).

It stores that diff into temporary files:

![stores diff](imgs/storeDiff.png)

It then moves the HEAD(current branch) to where we want to rebase(in our case `D`).

After it moves to the commit were rebasing against, it will then replay the commits that got stored temporarily one by one. It will look something like this:

![diagram replaying commit e](imgs/replayE.png)
<br>
![diagram replaying commit f](imgs/replayF.png)
<br>
<br>
![diagram replaying commit g](imgs/replayG.png)

> If at any point during each of the replays of `E`, `F`, and `G` conflict with the commit history of `D` a merge conflict will have to be remedied. Check [here](https://github.com/andrewsunglaekim/advanced_git_workshop#merge-conflicts-5115) on how to resolve merge conflicts during a rebase.

> one more PSA: rebasing **rewrites** history always use with caution.

Once the last commit has replayed the rebase has been completed.

Now that we have laid a foundation for rebasing and how it works. Let's now chat about interactive rebasing.

# Interactive rebasing

So many times in development, we commit things we don't necessarily want to commit. Maybe we misspelled something in our commit message. Maybe we left in a couple of log statements that don't belong. Perhaps we didn't want some of the commits at all.

We often turn to `reset` and say bugger all, let's restart.

Often times, however, we can find our solution in an interactive rebase.

Interactive rebasing works such that we have the ability to interact with the replay of each commit.

When pulling in upstream changes, we generally just do a regular rebase. Since we want the upstream changes as is, we usually don't want to change them through an interactive rebase.

More often then not we interactively rebase a single branch in order to rewrite the history of that branch.

Let's take the finished feature branch above after the sucessful rebase:

![diagram replaying commit f](imgs/linear.png)

In order to change the history of this commit `G` with an interactive rebase, we could do something like this:

```
$ git checkout G
$ git rebase D -i
```

Upon hitting enter, the command line will automatically open up your text editor associated with git.

> If it did not, check [this link](https://help.github.com/en/github/using-git/associating-text-editors-with-git) to associate a default text editor to our git config.

The file it opens in the text editor will look something like this:

```
pick 7bc099f commit message for E
pick 203f7d6 commit message for F
pick 4ea7cd0 commit message for G

# Rebase e8dfdb1..4ea7cd0 onto e8dfdb1 (3 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

> you git sha's will be different and commit message is whatever you make your commit message, they are contrived for the purposes of this lesson. Additionally we'll be using this as the base for all our other commands

This laundry list of things is daunting at first. But the interface is pretty simple. Our job in this first interface is to edit the word `pick` on the left according to how we want to interact with that specific commit.

`pick` is the default action that a normal rebase executes. The comments are there just to help with info, in the first page we are only editing the `pick` word if we want to do something other than the default behavior.

## Rewording a commit message

Let's say we want to reword the commit message for commit `F`. We would edit the file in this way:

```
pick 7bc099f commit message for E
reword 203f7d6 commit message for F
pick 4ea7cd0 commit message for G

# Rebase e8dfdb1..4ea7cd0 onto e8dfdb1 (3 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

> This text looks identical to the first block, except the second line has `reword` instead of `pick`

Once we've made the edit, we then save and close the file. It then automatically open up yet another text editor window to edit your commit message, it will look something like this:

```
commit message for F

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Wed Nov 13 14:06:15 2019 -0500
#
# interactive rebase in progress; onto e8dfdb1
# Last commands done (2 commands done):
#    pick 7bc099f commit 5
#    reword 54dfefd new commit message for F
# No commands remaining.
# You are currently editing a commit while rebasing branch 'master' on 'e8dfdb1'.
#
# Changes to be committed:
#	modified:   test.txt
#
```

> This is a file that we must now edit in order to change the commit message. The comments here are just for direction again.

We may edit it in this way:

```
brand new much more awesome and semantic commit message

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Wed Nov 13 14:06:15 2019 -0500
#
# interactive rebase in progress; onto e8dfdb1
# Last commands done (2 commands done):
#    pick 7bc099f commit 5
#    reword 54dfefd new commit message for F
# No commands remaining.
# You are currently editing a commit while rebasing branch 'master' on 'e8dfdb1'.
#
# Changes to be committed:
#	modified:   test.txt
#
```

All we've done here is make a better commit message, we'll go ahead and save this file and quit that tab. Now we have successfully reworded one of our commit messages through interactive rebasing.

# Edit a commit

We can additionally edit a commit. We might edit the original pane in this way:

```
pick 7bc099f commit message for E
pick 203f7d6 commit message for F
edit 4ea7cd0 commit message for G

# Rebase e8dfdb1..4ea7cd0 onto e8dfdb1 (3 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

> we're going to edit the last commit here for ease as we know it will not cause merge conflicts. Particularly when we are just editing text in 1 file. If we do run into a merge conflict, fix as normal, stage changes and continue the rebase with `git rebase --continue`

After saving the above file and closing it, in the terminal we'll be prompted with something like this:

```
You can amend the commit now, with

  git commit --amend

Once you are satisfied with your changes, run

  git rebase --continue
```

Now we just edit the files however we want. Then we stage them:

```
$ git add .
```

Then we can continue the rebase:

```
$ git rebase --continue
```

Then git will prompt us for a commit message in the text editor:

```
This commit contains awesome new edits

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Wed Nov 13 14:06:15 2019 -0500
#
# interactive rebase in progress; onto e8dfdb1
# Last commands done (2 commands done):
#    pick 7bc099f commit 5
#    edit e2812f2 brand new much more awesome commit message + more!
# No commands remaining.
# You are currently editing a commit while rebasing branch 'master' on 'e8dfdb1'.
#
# Changes to be committed:
#	modified:   test.txt
#
```

Save and quit, and we now have successfully edited a commit rebasing interactively.

# Squashing a commit

We would squash by editing the original interactive rebase prompt in this way:

```
pick 7bc099f commit message for E
pick 203f7d6 commit message for F
squash 4ea7cd0 commit message for G

# Rebase e8dfdb1..4ea7cd0 onto e8dfdb1 (3 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

After saving this and quitting, we should prompted with a second file that should have this type of content:

```
# This is a combination of 2 commits.
# This is the 1st commit message:

commit message for F

# This is the commit message #2:

commit message for G

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Wed Nov 13 15:21:43 2019 -0500
#
# interactive rebase in progress; onto b5edf4c
# Last commands done (4 commands done):
#    pick 9d61789 this is commit 6
#    squash 0f4d856 this is commit 8
# Next commands to do (2 remaining commands):
#    pick 3a2e517 this is commit 9
#    pick dc50b0e this is commit 10
# You are currently rebasing branch 'master' on 'b5edf4c'.
#
# Changes to be committed:
#	modified:   test.txt
#
```

> again don't worry about the comments, it just meta data or hints about what you need to be doing. Yours will be different from what you see on this lesson

The important thing to see here is the two commit messages. By default, if we were to save this file and quit to continue the rebase. The squashed commit would no longer exist, however, the changes of the commit would now exist in the squashed commits old parent with a combined commit message of

```
commit message for F

commit message for G
```

We can also change it to whatever we want in the above text prompt. It might look something like this:

```
SUPER AWESOME SQUASHED COMMIT of F and G

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Wed Nov 13 15:21:43 2019 -0500
#
# interactive rebase in progress; onto b5edf4c
# Last commands done (4 commands done):
#    pick 9d61789 this is commit 6
#    squash 0f4d856 this is commit 8
# Next commands to do (2 remaining commands):
#    pick 3a2e517 this is commit 9
#    pick dc50b0e this is commit 10
# You are currently rebasing branch 'master' on 'b5edf4c'.
#
# Changes to be committed:
#	modified:   test.txt
#
```

If we save and quit the file, we'll now see that the squashed commit was removed, the parent of that commit now has a new message, and a different diff! All through an interactive rebase!

# Dropping commits

We can also drop commits. It might look something like this:

```
pick 7bc099f commit message for E
pick 203f7d6 commit message for F
drop 4ea7cd0 commit message for G

# Rebase e8dfdb1..4ea7cd0 onto e8dfdb1 (3 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

For this one, it's all we have to do. As soon as this file is saved and exited; the interactive rebase will just drop the last commit in this case!

> It should be noted that editing dropping commits can cause merge conflicts, which we would then have to resolve and continue the rebase from there

# PSA - be careful rebasing, always

We commented above, but another reminder here is probably apt. Rebasing **rewrites** history and should always be used with caution. Interactive rebasing is a tool best used on local branches, not branches that are shared.
