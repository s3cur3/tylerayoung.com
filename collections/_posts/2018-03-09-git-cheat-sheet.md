---
title: Git Cheat Sheet
layout: post
authors: ['tyler']
categories: ["Programming"]
---

These are a few of the Git commands I find myself looking up all the time.

*   Show the staged changes:

        $ git diff --cached
    
    Also aliased as:
    
        $ git diff --staged
    
*   Find the most recent common ancestor of two Git branches:
    
        $ git merge-base [branch1] [branch2]
    
*   Find which branch a commit was originally created on:
    
        $ git reflog show --all | grep [commit SHA]
    
*   Find all branches that a commit is on (or that a branch has been merged into):
    
        $ git branch -a --contains [commit SHA or branch name]
    
*   Get the diff between two branches:
    
        $ git diff [branch1]..[branch2]
    
*   Get the diff between two branches, _excluding_ certain subdirectories:
    
        $ git diff [branch1]..[branch2] -- . ':!path/to/exclude' ':!other/path/to-exclude/'
        
    
*   Undo/remove a Git commit that has not been pushed ([scary fine print](https://stackoverflow.com/questions/1611215/remove-a-git-commit-which-has-not-pushed)):
    
        $ git reset --hard HEAD^
    
*   [Extracting a Git subdirectory into its own submodule](https://stackoverflow.com/questions/920165/howto-extract-a-git-subdirectory-and-make-a-submodule-out-of-it)

