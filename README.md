# Git Tricks

## git checkout -

Have you ever used `cd -`?

```bash
~/code/ $ cd scripts
~/code/scripts $ cd -
~/code/ $
```

You can do the same thing with `git checkout -`!

```bash
~/code/repo/ (master) $ git checkout develop
~/code/repo/ (develop) $ git checkout -
~/code/repo/ (master) $
```

## git status -sb

More compact status with less hints.

I alias this to `gs`.

```bash
~/c/clint (develop) $ git status -sb
## develop...origin/develop [behind 46]
D  manage.py
M  requirements.txt
```

## git cherry-pick

Takes a commit or a range of commits and copies them onto your current branch.

I use this most commonly to get a single commit from another branch or from the reflog.

```
git cherry-pick razzi/3791
```

## git clean -f

Remove untracked files (DANGEROUS but if you still have them open in your editor you can recover them; thanks emacs buffers!)

```
~/c/clint (develop) $ git checkout .
~/c/clint (develop) $ gs
## develop...origin/develop [behind 46]
?? webapp/apps/solar/system/migrations/0010_auto_20170404_1536.py
?? webapp/apps/solar/system/migrations/0011_solarsite_utility_calculation_source.py
~/c/clint (develop) $ git clean -f
Removing webapp/apps/solar/system/migrations/0010_auto_20170404_1536.py
Removing webapp/apps/solar/system/migrations/0011_solarsite_utility_calculation_source.py
~/c/clint (develop) $ django-admin makemigrations
```

## git rebase

Imagine @towens2727 branches off of develop and makes a commit. So do I.

Later, @towens2727's commit is merged into develop.

Now, I have to merge develop into my branch if any conflicts arise.

```
Legend: o = commit
        m = merge commit

       o--to/dev-
      /          \
dev -o------------m---m
      \            \ /
       o-razzi/dev--m
```

Wouldn't the world be simpler if before doing any work myself, I waited for @towens2727 to merge all his code back into develop?


```
       o--to/dev--
      /           \
dev -o-------------m--------------m
                    \            /
                     o-razzi/dev-
```

1 less merge commit!

How can we accomplish this without restricting development to one developer at a time?

Rebasing!

```
~/code/clint (razzi/ur-68) $ git rebase develop
 lib/template/utils.py                                                      |  43 ++++++++-----
 requirements.txt                                                           |   1 -
 2 files changed, 24 insertions(+), 20 deletions(-)
First, rewinding head to replay your work on top of it...
Applying: [UR-68] Add script to compare utility calculation accuracy
Applying: Make percent a percent rather than a ratio
```

First, rebase rewinds your current branch to the point where you and the given branch diverged.

Then, starting from the given branch, it replays your commits.

It's as though you waited for that branch to get to its current state before you started committing!

Commits made this way will have a new commit id, so if you had already pushed commits to that branch on a remote, you'll need to force push.

By default, it doesn't show stats (unlike merge). But this is just a git setting away:

```bash
$ git config --global rebase.stat true
```

# Git settings

Git stores settings in 2 places: ~/.gitconfig and .git/config.

Local .git/config is usually just stuff like remotes.

~/.gitconfig can have:

- aliases

```
[alias]
# Simple abbreviation
	cl = clone

# Staged changes
	new = diff --cached

# Current branch name
	current = rev-parse --abbrev-ref HEAD
```

A useful command to reset the current branch to the origin:

```
$ git reset --hard origin/(git current)
```

- configuration

```
# Make `git pull` a fetch + rebase rather than fetch + merge
[pull]
    rebase = true

# Show stats when rebasing
[rebase]
    stat = true

# Additional file patters to ignore
[core]
    excludesfile = ~/.gitignore_global
```

A couple snippets from my .gitignore_global:

```
*~undo-tree~
.cache
```

This is handy for editor-specific files and things you don't want to bother with in every repository.

Pro tip: track your global gitconfig in git! [Here's mine](https://github.com/razzius/personal/blob/master/.gitconfig)

## git rebase -i

```
git rebase -i develop
```

takes you to an interactive edit session where you can edit your commit history:

```
pick 6d162c4904 [UR-68] Add script to compare utility calculation accuracy
fixup c259d4e9ff fix typo
drop f66924000b remove all redis model properties
```

Here I'm using fixup to combine 2 commits, one of which is just fixing a typo.

Note: you can also `git commit --amend` if the typo happened on the most recent commit.

I'm also dropping an overly ambitious commit where I've digressed too far.

The editor has helpful hints!

```
pick 6d162c4904 [UR-68] Add script to compare utility calculation accuracy
pick 0bc6414728 Make percent actually a percent rather than a ratio

# Rebase 6d162c4904..0bc6414728 onto 6deae34b4d (2 commands)
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

# git reflog

Now that we've seen we can drop commits (maybe a little too) easily, let's look at `git reflog`, which makes it really hard to lose work.

```
530db754cd HEAD@{154}: commit: Pass both pre and post solar custom rates to the custom rate schedule calculator
3e3266973c HEAD@{155}: checkout: moving from master to razzi/custom_rate_fix
3e3266973c HEAD@{156}: checkout: moving from staging to master
9999d17683 HEAD@{159}: rebase finished: returning to refs/heads/staging
9999d17683 HEAD@{160}: pull: checkout 9999d176830131409b6891b72d1c968bdcc3dfdd
193c5ae0a8 HEAD@{161}: reset: moving to origin/staging
e3719bbfdd HEAD@{162}: checkout: moving from razzi/5344 to staging
```

Even if I drop these commits, they're still reachable in my local .git history.

Even if I delete the branch that the above commit was on, I can still cherry-pick it a year later.

Checkouts, commits, rebases, and resets are tracked in the reflog.

As long as work is ever committed, you can find it in the reflog.

# git add -p

Sort of like how `git rebase -i` opens an editor of git commits, `git add -p` interactively goes through unstaged changes. You can press y to state, n to not stage, and e to edit the patch!

```bash
$ git add -p
diff --git a/scripts/about_app.py b/scripts/about_app.py
index 834ef0de29..93fae4bea6 100644
--- a/scripts/about_app.py
+++ b/scripts/about_app.py
@@ -44,7 +44,7 @@ def _parse_args():
parser.add_argument('--debug', help='Should only be used for debugging. '
'Prints to log info to std out',
action='store_true')
-    parser.add_argument('--pretty', help='Pretty print JSON',
-    +    parser.add_argument('--light', help='Only retrieve version info',
-                             action='store_true')
-                                  return parser.parse_args()
-
-                                  Stage this hunk [])
```

## git bisect

This command helps find the commit that introduced some bug by marking a known good commit (perhaps develop) and a known bad commit (probably HEAD) then letting git bisect the commits between them.

```
~/c/clint (develop) $ git bisect good master
You need to start by "git bisect start"
Do you want me to do it for you [Y/n]? y
~/c/clint (develop|BISECTING) $ git bisect bad
Bisecting: 73 revisions left to test after this (roughly 6 steps)
[7da829fe6bcf4bc5c6a8d09ea028f8596f5e9b2c] Fix usage_profile_id copypasta
~/c/clint ((7da829fe…)|BISECTING) $ # Check the ui for some bug... no bug at this commit.
~/c/clint ((7da829fe…)|BISECTING) $ git bisect good
Bisecting: 36 revisions left to test after this (roughly 5 steps)``
...
```

If you want to end early, `git bisect reset`.

# Other tips

Rather than stashing, I use WIP commits so that changes are associated with the proper branch, rather than being in a global stash.

```bash
$ function wip
  git add .
  git commit -m "wip $argv"
end
```

I also have a shortcut for `@` to `git reset @^` to undo the WIP commit.

# hub

[hub](https://hub.github.com/) is a command line program for interacting with github.

It provides commands like `pull-request`, which creates a pull request from your current branch.

```sh
$ hub pull-request

...

UR-68 utility comparison script

Also moved all utility scripts to one directory.

# Requesting a pull to sighten:develop from sighten:razzi/ur-68
#
# Write a message for this pull request. The first block
# of text is the title and the rest is the description.
#
# Changes:
#
# 011d1aaf4b (Razzi Abuissa, 9 hours ago)
#    Make percent actually a percent

...

https://github.com/sighten/clint/pull/1234
```

It also wraps every existing git command, so they recommend you alias `git` to `hub`.

# GIT_TRACE

Show what commands git is running. Useful for debugging aliases and hooks.

```sh
~/c/clint (razzi/ur-68) $ export GIT_TRACE=1

~/c/clint (razzi/ur-68) $ gs
00:05:36.091823 git.c:371               trace: built-in: git 'status' '-sb'
## razzi/ur-68...origin/razzi/ur-68
```

For example: where is that "hi" message coming from?

```
~/c/clint (razzi/ur-68) $ git checkout develop
Switched to branch 'develop'
Your branch is up-to-date with 'origin/develop'.
hi

~/c/clint (develop) $ set -xg GIT_TRACE 1
~/c/clint (develop) $ git checkout -
00:09:53.905261 git.c:371               trace: built-in: git 'checkout' '-'
Switched to branch 'razzi/ur-68'
Your branch is up-to-date with 'origin/razzi/ur-68'.
00:09:54.026583 run-command.c:369       trace: run_command: '.git/hooks/post-checkout' '6deae34b4d93fcadd14286b023bfcdb8d7eba2c0' '011d1aaf4bdeb262c78747b0783febc5f2f6d8ab' '1'
hi
```
