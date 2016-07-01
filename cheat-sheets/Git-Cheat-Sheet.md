---
title: Git Cheat Sheet
layout: page
---
# Git Cheat Sheet

Git is a distributed version control system.

## References

- [Github Guide](https://git-scm.com/book/en/v1/Getting-Started-About-Version-Control)
- [Cheat Sheet #1](https://www.git-tower.com/blog/git-cheat-sheet/)
- [Cheat Sheet #2](https://training.github.com/kit/downloads/github-git-cheat-sheet.pdf)
- [Interactive Tutorial](http://try.github.io/)

## Concepts

### Merge

- [guide](https://git-scm.com/book/en/v1/Git-Branching-Basic-Branching-and-Merging)
- combining two branches
- three-way merge
    - common ancestor
    - snapshot to merge into
    - snapshot to merge in
- result of a three-way merge is a merge commit
    - has two parents, one from each branch
- if the branch being merged is directly upstream from the current branch, a "fast-forward" action is take - basically taking the branch pointer and moving ahead. No merge commit is necessary. 

### Rebase

- alternative to merging
- instead of creating an additional commit for the merge, all the commits in the current branch are re-committed
    with changes from the master, which makes it look like changes were done from a static version of the master. 
- a big trade off is not being able to see when changes were applied
- "golden rule" - never rebase a shared branch

### Branch

- A commit is comprised of:
    - commit object - pointer to tree object
    - tree object - links to blob objects
    - blobs - project files
- see this helpful [Diagram](https://git-scm.com/book/en/v1/Git-Branching-What-a-Branch-Is) 
- a branch is a movable pointer to a commit
- the default branch name is "master". Every time a commit is made, this moves forward automatically in fact, every commit made moves some branch forward

### Commit

- a commit is made up of
    - commit object - pointer to tree object
    - tree object - links to blob objects
    - blobs - project files
- an object that contains a pointer to the snapshot of the content that was staged,the author and message metadata, and zero or more pointers to the the commit(s) that were the direct parents of this commit.  

### Snapshot

- state of files at a specific point in time
- Instead of a sequential version number uses codes e.g. 8e1a054 - which is an abbreviation for the full snapshot code
- called a revision in other systems

### HEAD

- special pointer to the local branch you are currently working on
		- to look at a prior state of files, you can move the HEAD pointer to a commit without a branch, but if you are intending to make

### Repository

- a location where Git stores the metadata and object database for your project

### Working Directory

- single checkout of one version of the project
- files from the compressed database in the Git directory are copied into this folder

### Staging Area

- file in your git directory about what will go into the next commit.  

### Remote

- version of your project that is on a remote server / folder etc

### File States in Git

- untracked - file is not in the repository
- unmodified - file is in the repository and identical
- modified - file is in the repository and has been changed
- staged - file is modified and marked as ready for commit

## Git Commands

### Initialize a new repository
```
git init
```

### Show current state of repository
```
git status
```

### Add file(s) to staging
```
git add file.txt
git add '*.txt'
```

### Add all file(s) to staging
```
git add -A
git add --all
git add .
```

### Add all updated files to staging
```
git add -u
git add --update
```

### Restore a directory to last commit
```
git clean -f  (removes untracked files)
git clean --force
git checkout -f (reverts modified files to previous state)
```

### Commit a set of staged changes to the repository
```
git commit -m "Commit Message"
```

### Show history of committed changes
```
git log
```

### Link repository to remote github.com repository
```
git remote add origin https://github.com/brianvp/myrepo
```

### Push local branch to remote repo
```
git push -u origin master
```

### Get changes from remote repo
```
git pull origin master
```
### Show differences between last commit and the remote repo changes pulled in
```
git diff HEAD
git diff --staged
```

### Unstage files
```
git reset myfile.txt
```

### Revert files back to last commit
```
git checkout -- myfile.txt
```

### Make a branch
```
git branch clean_up
```

### remove old branch
```
git branch -d clean_up
```

### modify a commit that hasn't been pushed
```
git commit --amend
```

### change a modified file back to unmodified
```
git checkout -- myfile.txt
```


