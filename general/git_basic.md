# Show branch name as PS:
This is one of the functions [bash_it](https://github.com/Bash-it/bash-it) provide.

## `~/.gitconfig`
Pay attention to **[include]** and **[alias]** sections.

# Staging Area

Working directory => staging/index area => git repo (local) => git repo (remote)

When commit to git repo local, all contents are still local. 
When push to remote, you make your commits public.

| Command         |      desccription    |
|-----------------|:--------------------:|
| git add .       | Add current workdirectory to the Object store (repo) and create a index for that (git reset)  |
| git reset --    | (the oposite of git add) reset the index entries for all path to their state at tree-ish.

# File Status Lifecycle
Here we talk about file states in the working directory:
* Tracked: Files that were in the last snapshot
* Unmodified: Tracked files which was NOT changed since your last commit
* Modified: Tracked files which was changed since your last commit
* Staged: Files which was added to the staging area
* Untracked: anything else
When you first clone a repository, all of your files will be tracked and unmodfied. 


|                 | Untracked       | Unmodified           |  Modified           | Staged                |
|:----------------|:----------------|:---------------------|:--------------------|:---------------------:|
| **Untracked**   |                 |                      |                     | git add               |
| **Unmodified**  |                 |                      |                     |                       |
| **Modified**    |                 | git checkout         |                     | git add               |
| **Staged**      |                 |                      |                     |                       |

# Submodule

Submodules allow foreign repositories to be embedded (or linked, see gitlink) within a dedicated subdirectory 
of the source tree, always pointed at a particular commit.

It essentially allows you to attach an external repository inside another repository at a specific path.

Submodules are meant for different projects you would like to make part of your source tree, while the
history of the two projects still stays completely independent and you cannot modify the contents of the
submodule from within the main project

## Commands

```
$ git submodule add git@mygithost:project.git lib/p4java # create .gitmodules and add a submodule entry
$ git submodule update --init # populate the submodule recurisively
$ git submodule rm lib/p4java # remove submodule
```

### Do not forget to update submodule before you check in.

```
$ git submodule update --init # this will get the submodule commit by parent project.
$ cd lib/p4java
$ git pull # this will pull the most recent submodule
$ cd ../..
## Develop my code, and testing
$ git status
$ git add . # do this iff you want to change the submodule reference of the parent project
$ git commit -m"Check in my change to project, and also update the reference to the most recent p4java"
```

## Tig

```
$ tig staging/master # see the difference between master and stagging/master
```

## Branch

```bash
# create a branch
$ git branch my_branch
# or
$ git co -b my_branch

# in either case above, you did not set the upstream, unless you push -u later
$ git branch --set-upstream my_branch origin/my_branch # associate to remote branch
$ git push

# or you can 
$ git co -b my_branch
# push to remote
$ git push -u origin my_branch # this need only once

# or
$ git co -b my_branch orign/my_branch
# git push

# delete a local branch
$ git branch -D my_branch

# delete a remove branch
$ git push origin --delete my_branch
```

## [Rebase](http://git-scm.com/book/en/v2/Git-Tools-Rewriting-History)

Rebase will replay local commits between upstream and branch to the newbase. For more example, see [`git reset` document](http://git-scm.com/docs/git-reset)

The following steps will be executed during rebase (assuming you are on **the current branch**):
* switch to branch if specified
  `git checkout <branch>`
* all changes made by commits in the current branch but that are not in the upstream are 
  saved to a temporary area. This is the same sets of commits shown by
  `git log <upstream>..HEAD`
* the current branch is reset to <upstream>, or <newbase> if specified. This is same as:
  `git reset --hard <upstream>`
* the commits that saved in temporary area are reapplied to **the current branch**, one
  by one, in order.

```bash
# full form rebase
$ git rebase [--onto <newbase>] [<upstream>] [<branch>]

# squash last 2 commit into one
$ git rebase -i HEAD~2 
```

Another way of thinking rebase is:
```bash
# full form
$ git rebase --onto <graft-point> <exclude-from> <include-from>
# shorter form
$ git rebase master
```

The shorter forms use defaults for things you don't specify:
* if you don't specify `--onto`, `<graft-point>` defaults to `<exclude-from>`
* if you don't specify an `<include-from>`, `<include-from>` defaults to the current branch
The commits will be apply:
```
git log <exclude-from>..<include-from>
```

A simiple flow
```
# update master
git co master
git pull
# rebase branch
git co max_projects_count_20150128_WIP
git rebase master
# merge back to master
git co master
git merge max_projects_count_20150128_WIP
```

### Cherry pick
For simple merge, you may not need rebase necessary. There is a more convenient way to do that:

```
dd2e86 - 946992 - 9143a9 - a6fd86 - 5a6057 [master]
           \
            76cada - 62ecb3 - b886a0 [feature]

# to commit 62ecb3 only to master
$ git checkout master
$ git cherry-pick 62ecb3

# to commits 76cada through 62ecb3 to master.
$ git checkout -b newbranch 62ecb3
$ git rebase --onto master 76cada^  # 76cada^ means previous commit, i.e., 946992, which is excluded

```
That’s all. 62ecb3 is now applied to the master branch and commited (as a new commit) in master (in first case). cherry-pick behaves just like merge. If git can’t apply the changes (e.g. you get merge conflicts), git leaves you to resolve the conflicts manually and make the commit yourself.

In second case, the result is that commits 76cada through 62ecb3 are applied to master.

## Revert previous commit

### Reset last commit and re-commit with local changes
```
add/edit
git reset --soft HEAD~1  
git commit
# or keep original commit message and time stamp
git commit -c ORIG_HEAD
```

### Temporarily switch to a different commit
If you want to temporarily go back and fool around
```
git co 0d123456c
# or
git co -b 0d123456c
```

### Hard delete unpublished commits
```
# reset current tip to 0d123456c, all local changes will be abandoned
git rest --hard 0d123456c

# alternatively, if you do want to keep local changes
git stash
git reset --hard 0d123456c
git stach pop
```

### Undo published commits with new commits
```
# This will create 3 separate revert commits, each for a change
git revert a867b4af 25eee4ca 0766c053

# or using ranges, this will revert the last two commits
git revert HEAD~2..HEAD

# reverting a merge commit
git revert -m 1 <merge_commit_sha>

# To get just one, you could use `rease-i` to squash them afterwards
# Or, do it manually (be sure to do this at top level of the repo)
# get the index and work tree into the desired state, without changing HEAD:
git co 0d123456c

# then commit
git commit
```

## Push

```bash
$ git --set-upstream <remote-branch> 
$ git push -u origin local-branch
```

## merge from branch and rebase master

```bash
4624  git show b847499
 4625  git jlfa
 4626  git reset --head HEAD^
 4627  git reset --hard HEAD^
 4628  git show dcfc5b5
 4629  git merge dcfc5b5
 4630  git co WIP_delete_frontend
 4631  git rebase master
 4632  git diff master..origin/master
 4633  git diff origin/master..master
 4634  git co master
 4635  git br -d WIP_delete_frontend
 4636  git push origin :WIP_delete_frontend
 4637  git jlga
```

## `git merge --squash dev_branch`
This will merge the changes(assuming you are in master) from `dev_branch` to master branch, but whithout commit. You need manually commit it (and also manually resolve the conflicts if has). This allow merge the dev_branch changes into one chunch.
```bash
git co dev_branch
... working on dev_branch and comit, push to origin/dev_branch
git co master
git merge --squash dev_branch
... resolve confilcts if necessary, and git add conflicts
git commit -m"merge changes from dev_branch"
git lg # this will show you a linear history
```

## search log
```
git log -g --grep="0052"
```

## delete files
```
git rm .project
git add -A
git commit -m "delete .project"
```

# Term
* `upstream/downstream`: There is no absolute upstream/downstream. When you declared **otherRepo** as a remote one, then you are **pulling from upstream - otherRepo**, and you are **downstream for otherRepo**; you are **pushing to upstream - otherRepo**.
