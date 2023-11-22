## Find the nearest parent branch (https://gist.github.com/joechrysler/6073741)
```
git log --decorate \
  | grep 'commit' \
  | grep 'origin/' \
  | head -n 2 \
  | tail -n 1 \
  | awk '{ print $2 }' \
  | tr -d "\n"

#or
git log --pretty=format:'%D' HEAD^ | grep 'origin/' | head -n1 | sed 's@origin/@@' | sed 's@,.*@@'
```

## How to delete all Git commits except the last X:
```
Ok, if you want what I think you want (see my comment), I think this should work:
1. Create branch to save all the commits (and just in case):
   git branch fullhistory
2. While still on master, reset --hard to the commit you want to retain history from:
   git reset --hard HEAD~5
3. Now reset without --hard to the beginning of history, this should leave your workspace untouched, so it remains in HEAD~5 state.
   git reset --soft <first_commit>
4. So now you have empty history on master, and all the changes you need in the workspace. Just commit them.
   git commit -m "all the old changes squashed"
5. Now cherry-pick this 4 commits from fullhistory you want to have here:
   git cherry-pick A..B
Where A is older than B, and remember A is NOT included. So it should be parent of the oldest commit you want to include.
```
(https://stackoverflow.com/questions/11929766/how-to-delete-all-git-commits-except-the-last-five) 

## How to rename a local/remote branch:
```
#To rename the local branch to the new name
git branch -m <old-name> <new-name>

#To delete the old branch on remote
git push origin --delete <old-name>

#Then you should push the new branch to remote
git push origin <new-name>

#To reset the upstream branch for the new-name local branch
git push origin -u <new-name>
```
https://www.w3docs.com/snippets/git/how-to-rename-git-local-and-remote-branches.html

## How to fetch a specific remote branch:

```
git checkout --track -b branch_name origin/branch_name
```

## Detached Head
https://www.cloudbees.com/blog/git-detached-head
```
git checkout origin/test # - results in detached HEAD / unnamed branch
git checkout -b test origin/test # - results in local branch test (with remote-tracking branch origin/test as upstream)

#how to fix:
git fetch origin
git branch -v -a #show the branches
git switch -c test origin/test 
#or prior to Git 2.23
git checkout -b test <name of remote>/test
```
https://stackoverflow.com/questions/1783405/how-do-i-check-out-a-remote-git-branch

## Tell the Local Branch to Track the Remote Branch
```
git branch --track <branch-name> origin/<branch-name>
```

## Source:
```
git remote -v
```

## Compare branches:
```
git diff branch1..branch2
```

## Compare commits between two branches
```
$ git log branch1..branch2
```

```
git log --oneline --graph --decorate --abbrev-commit branch1..branch2
```

## Compare specific files between two branches
```
git diff master..feature -- <file>
```

## Compare diff between a branch and local uncommitted changes:
```
Syntax:

git diff <commit-ish>:../<path> -- <path>
Examples:

git diff origin/master:../README.md -- README.md
git diff HEAD^:../ -- README.md
git diff stash@{0}:../ -- README.md
git diff 1A2B3C4D:../ -- README.md
```

In order to compare two branches using commit abbreviations, use the “git log” command with the following options.
```
$ git log --oneline --graph --decorate --abbrev-commit branch1..branch2
```

## Get a file's history:
```
git log -- [filename]

gitk [filename]
or to follow filename past renames

gitk --follow [filename]
```

## Undo the last commit:
```
git reset --hard HEAD~1
```

## Undo the last commit but leave changes as uncommitted local modifications in your working copy.:
```
git reset --soft HEAD~1
```

## 'Squash' several commits:
```
git reset --soft HEAD~3 &&
git commit
```

## Delete some files from the last commit
```
git reset --soft HEAD^
# or
git reset --soft HEAD~1
#then reset the unwanted files in order to leave them out from the commit (the old way)
git reset HEAD path/to/unwanted_files/*
```

## Undo multiple commits:
```
git reset --hard 0ad5a7a6
```

## Checkout an old commit and make it a new commit:
```
git log #find a hash of a required commit 
git checkout <commit-hash> . #note the dot! It should checkout . (all changes) from the commit to your working-tree
#after that, apply the changes as a new commit
git add ...
git commit 
```

## Change the last commit message:
```
git commit --amend
```

## Change the last commit files:
```
#Edit hello.py and main.py 
git add hello.py git commit 
# Realize you forgot to add the changes from main.py 
git add main.py git commit --amend --no-edit
```

## Delete a file from git but safe in Workspace
```
git rm  --cached <filename>
```

## To clear the untracked files from the local
Git 2.11 and newer versions:
```
git clean  -d  -f .
```
Older versions of Git:
```
git clean  -d  -f ""
```
Where -d can be replaced with the following:
- -x ignored files are also removed as well as files unknown to Git.
- -d remove untracked directories in addition to untracked files.
- -f is required to force it to run.
Here is the [link](http://www.kernel.org/pub/software/scm/git/docs/git-clean.html) that can be helpful as well.

https://git-scm.com/book/en/v2/Git-Internals-Git-References

## Git bash autocompletion
```
source /etc/bash_completion.d/git
# or
source /usr/share/bash-completion/completions/git
```

## How can I selectively merge or pick changes from another branch
```
git checkout source_branch -- path/to/file
# resolve conflicts if any
git commit -am '...'

Or to selectively merge hunks
git checkout -p source_branch -- <paths>...
```

## How to colorize output of git?
```
git config --global color.ui auto
```

## How can I save username and password in Git?
if these commands don't work:
```
git config user.name "your username" 
git config user.password "your password"
```

, we can use the git config to enable credentials storage in Git. It's not safe though, password is in plain text in  ~/.git-credentials 

```
git config --global credential.helper store
```
## 'Personal' gitignore
vim .git/info/exclude
