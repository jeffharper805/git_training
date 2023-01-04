- [Learn `git` concepts, not commands](#learn-git-concepts-not-commands)
  - [- Reading history](#--reading-history)
  - [Overview](#overview)
  - [Getting a _Remote Repository_](#getting-a-remote-repository)
  - [Adding new things](#adding-new-things)
  - [Making changes](#making-changes)
  - [Branching](#branching)
  - [Merging](#merging)
  - [Rebasing](#rebasing)
  - [Updating the _Dev Environment_ with remote changes](#updating-the-dev-environment-with-remote-changes)
    - [_Fetching_ Changes](#fetching-changes)
    - [_Pulling_ Changes](#pulling-changes)
    - [Stashing changes](#stashing-changes)
    - [Pulling with Conflicts](#pulling-with-conflicts)
  - [Cherry-picking](#cherry-picking)
  - [Rewriting history](#rewriting-history)
    - [Amending the last Commit](#amending-the-last-commit)
    - [Interactive Rebase](#interactive-rebase)
    - [Fixing a commit that's further in the past](#fixing-a-commit-thats-further-in-the-past)
    - [Public History, why you shouldn't rewrite it, and how to still do it safely](#public-history-why-you-shouldnt-rewrite-it-and-how-to-still-do-it-safely)
  - [Reading history](#reading-history)
As you'll remember from before, revisions in git, aren't only a snapshot of your files but also contain information on where they came from. Each `commit` has one or more parent commits. Our new `merge` commit, has both the last commit from _master_ and the commit we made on the other branch as its parents.
### Fixing a commit that's further in the past

So now we know how to easily `amend` the latest commit and how to rewrite commits in general with an interactive `rebase`.

There is one more helpful feature that makes it easy to fix commits that happened further in the past: `fixup` commits.

Let's recreate our [example from before](#amending-the-last-commit) and from the main branch, switch to a new branch with `git checkout -b fixup_commits`.

Now make some changes to `Alice.txt` and add `Bob.txt`, and then `git add Alice.txt`.

Then `git commit` using a message like "Add important things to Alice and Bob".

And now let's make another change to `Alice.txt`, `git add Alice.txt` and make another `git commit` with a message of you're choice.

If you still remember the `amend` example you'll have noticed that we again forgot to add the changes to `Bob.txt` in our first commit.

Knowing how to do an interactive rebase, we could just add a new commit, rebase, move it before the second commit and squash it into the first.

But that's a lot of manual work and git has a convenient shorthand for fixing commits.

First let's find out the hash of that first commit - I like using `git log --oneline` to quickly find the short hash:

```sh
> git log --oneline
20e59b5 (HEAD -> fixup_commits) Add some more changes to Alice
7acb4bc Add important things to Alice and Bob
e701c3e (origin/master, origin/HEAD, master) made stashing section a little more understandable (#83)
[...]
```

So for me the commit I want to add the `Bob.txt` changes to is `7acb4bc`.

To do that let's `git add Bob.txt`, and then create a new commit that fixes `7acb4bc`.

We do that by simply running `git commit --fixup={the commit hash to fix}`. So for me `git commit --fixup=7acb4bc` will do the trick.

After that my log looks like this:

```sh
> git log --oneline
8b4b1af (HEAD -> fixup_commits) fixup! Add important things to Alice and Bob
20e59b5 Add some more changes to Alice
7acb4bc Add important things to Alice and Bob
e701c3e (origin/master, origin/HEAD, master) made stashing section a little more understandable (#83)
[...]
```

As you can see that new commit is special and it's message is simply the one of the commit it's fixing, prefixed with `fixup!`.

With such `fixup` commits in our history we can simplify the interactive rebase by telling git to `autosquash` them for us.

So if we run `git rebase -i --autosquash origin/master` now the rebase prompt will look like this:

```sh
pick 7acb4bc Add important things to Alice and Bob
fixup 8b4b1af fixup! Add important things to Alice and Bob
pick 20e59b5 Add some more changes to Alice

# Rebase e701c3e..8b4b1af onto e701c3e (3 commands)
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
#
# Note that empty commits are commented out
```

As you can see our `fixup` commit is moved to the correct place below our first commit and will be applied with the `fixup` command - which 'silently' squashes the changes into the commit before it.

After the rebase our history looks like this:

```sh
> git log --oneline
429c09a (HEAD -> fixup_commits) Add some more changes to Alice
55f424e Add important things to Alice and Bob
e701c3e (origin/master, origin/HEAD, master) made stashing section a little more understandable (#83)
[...]
```

And the modified commit (`55f424e` for me) looks like this:

```Diff
> git show 55f424e
commit 55f424ee6f7e2ebdc265628aaca9c3f7222b8c47
Author: UnseenWizzard <nico@riedmann.dev>
Date:   Sun Jul 17 12:30:08 2022 +0200

    Add important things to Alice and Bob

diff --git a/Alice.txt b/Alice.txt
index b4006e7..0c0d2c7 100644
--- a/Alice.txt
+++ b/Alice.txt
@@ -1 +1,2 @@
 Hi! I'm Alice. I'm a file someone added to this repository a while ago.
+Then someone added a line.
diff --git a/Bob.txt b/Bob.txt
new file mode 100644
index 0000000..317a2d6
--- /dev/null
+++ b/Bob.txt
@@ -0,0 +1 @@
+Hi, I'm Bob. I was just added on this branch.
```

And that's all there is to using `fixup` commits - a convenient way to rewrite git history for commits that are further back, where `commit --amend` can't help us.