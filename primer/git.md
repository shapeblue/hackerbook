# Git Version Control System

## Configuration

Syntax:

    git config --global <setting> <value>

Frequently used:

    git config --global user.name "[name]"
    git config --global user.email "[email address]"
    git config --global color.ui auto

## Repositories

Create Repo:

    mkdir project && cd project
    git init

Clone repo:

    git clone <url> <optional|folder name>

## Changes

    git rm <file to remove>
    git rm --cached <remove file from git but not locally>
    git mv <original file> <new file>

    git status

    git diff
    git diff --staged

    git add <file|dir>

    git reset <undo file | undo after commit>
    git reset --hard <discard all and reset to commit/branch >

    git commit -s -m "some message"

## Branching and Synchronization

    git branch

    git branch <name of branch to create>

    git branch -d <branch to delete>

    git checkout <branch-name>

    git merge <branch or remote/branch>

    git fetch <origin> <optional|branch>

    git push <remote> <branch name>

    git pull <optinal|--rebase [branch name]>

## Forward Merging

    # Checkout base branch and get the changes
    git checkout <base branch>
    git pull --rebase origin <branch>
    # Checkout the next branch and update it before forward merging changes
    git checkout <next branch>
    git pull --rebase origin <next branch>
    git merge origin/<base branch>
    # Fix conflicts, merge/commit changes and then push the changes
    git push origin <next branch>

## Stashing

    git stash
    git stash list
    git stash <pop|apply>
    git stash drop

## History

    git log

    git log --follow <follow all versions around the file, including renames, moves>

    git diff <branch1>...<branch2>

    git show <commit sha>

## Searching

    git grep <string to search across tracked files>
    git grep -i <insensitive string>

## Squashing Changes

To squash all the changes in your branch against a target branch say `master`
you can do the following:

    git checkout your-branch
    git reset $(git merge-base master your-branch)
    git add -A
    git commit -s # add pretty commit message and description

## Rebasing Branch

This is the most common and challenging task that requires say a PR branch to
be rebased against a new/different branch than say `master`:

0. Easiest way when commits are melded:

    # Note the SHA of the commit on your PR branch
    git log
    # Reset the PR branch against the new branch
    git reset --hard origin/4.11
    # Cherry-pick and fix conflicts
    git cherry-pick <SHA of original commit>
    # Fix conflicts, add/commit it
    git commit --amend -s

1. When new branch is forward branch (say from 4.11 to master):

2. When new branch is a backward (say from master to 4.11):

3. General solution: (say to 4.11)

    git rebase origin/4.11
