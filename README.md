# StatLab guide to Git branching, merging, and stashing

Last updated: 2023-05-19  
Contributors: [jacob-gg](https://github.com/jacob-gg)

Contents:
- [Branching](#branching)
    - [Creating a branch](#creating-a-branch)
    - [Viewing and moving branches](#viewing-and-moving-branches)
    - [Pushing a branch to the remote](#pushing-a-branch-to-the-remote)
    - [Moving uncommitted work to a new branch](#moving-uncommitted-work-to-a-new-branch)
    - [Deleting branches](#deleting-branches) 
- [Merging](#merging)
    - [Merging branches locally](#merging-branches-locally)
    - [Merging branches at the remote](#merging-branches-at-the-remote)
    - [Resolving merge conflicts](#resolving-merge-conflicts)
- [Stashing](#stashing)
    - [Creating stashes](#creating-stashes)
    - [Viewing stashes](#viewing-stashes)
    - [Applying and deleting stashes](#applying-and-deleting-stashes)

_Throughout this guide, all `code-formatted text` refers to command line commands unless otherwise indicated._

---

## Branching

Branches are copies of a repository. Instead of always working together in the "main" branch of a collaborative repository---where we risk running over each other---we can work in separate branches and then merge them together. Branches are also useful when working solo---they allow for code to be modified and tested in a safe, partitioned environment, and that code, once satisfactory, can be easily merged into the main branch.

### Creating a branch

**Approach 1: Create a new branch and switch to it in one step**

`git checkout -b new_branch` _or_  
`git switch -c new_branch`

**Approach 2: Create a branch and then switch to it:**

`git branch new_branch` _then_  
`git switch new_branch` _or_ `git checkout new_branch`

### Viewing and moving branches

View a repository's existing branches, as well as the branch you're currently in, with:

`git branch`

Move from one branch to another with:

`git checkout branch_name` _or_  
`git switch branch_name`

### Pushing a branch to the remote

Add and commit files to the branch, then:

`git push origin new_branch`

### Moving uncommitted work to a new branch

**Approach 1: Via branch commands**

If you are working in a branch (e.g., the main) and want to move uncommitted changes to a new branch, use:  

`git switch -c new_branch`  
_or_  
`git checkout -b new_branch`

**Approach 2: Via stashing**

Alternatively, you can use the stash (more on the stash below):

First, stash the changes:  
`git stash push -u`  
Then, make a new branch with your stashed changes:  
`git stash branch new_branch stash@{0}`

Note that:
1. `stash@{0}` is a reference to the most-recent stash
    - View all stashes with `git stash list`
2. By default, `git stash push` will stash uncommitted changes to tracked files, but it will _not_ stash new, untracked files and changes thereto
    - To also stash new files, use `git stash push -u` (equivalent to `git stash push --include-untracked`)
    - Assuming that you usually want to stash new, untracked files as well as changes to existing, tracked files, the command with the `-u`/`--include-untracked` option is a reasonable default

### Deleting branches

**Deleting a branch locally**

`git branch -d branch_name`

If you haven't merged the branch into another already, this will produce an error. To force the deletion regardless, use `git branch -D branch_name`.

**Deleting a branch at the remote**

`git push origin :branch_name` _or_  
`git push origin -d branch_name`

---

## Merging

We deal with two kinds of merging: Local and remote (i.e., on GitHub).

### Merging branches locally

 Switch to the target branch (e.g., `main`), then merge in the incoming branch:

`git checkout main`  
`git merge branch_to_be_merged`

Make sure to pull before merging to ensure that the main is up to date (`git pull [origin main]`).

### Merging branches at the remote

Prepare to merge branches at the remote by pushing a branch and then opening a pull request. The workflow will generally look like:

`git checkout -b new_branch`  
_add files, make changes, etc._  
`git add -A`  
`git commit -am 'Some changes'`  
`git push origin new_branch`

Then, open a pull request on GitHub. You or someone else can then review the changes and merge the branch or request edits. Once the branch has been merged, you can safely delete it both locally and at the remote. You can delete the merged branch at the remote using GitHub's interface, or you can use the method described above under "Deleting branches." Also delete the branch locally once it's no longer in use.

### Resolving merge conflicts

Git is usually able to figure out how to merge files and changes together, but not always. If, say, two people modify the same line(s) in a file and then try to push and merge their changes, Git may encounter a merge conflict. Whether the merge conflict arose locally or at the remote influences which interface is most natural to use to resolve it.

**Resolving a merge conflict locally**

Create a merge conflict:

```
git init temp
cd temp
touch file.txt
echo -n 'abcde' > file.txt
git add -A
git commit -am 'create file.txt'
git checkout -b new_branch
echo 'fghij' >> file.txt
git commit -am 'modify file.txt'
git checkout main
echo 'this is overwriting text' > file.txt
git commit -am 'modify file.txt in main'
git merge new_branch
```

This gives:

```
CONFLICT (content): Merge conflict in file.txt
Automatic merge failed; fix conflicts and then commit the result.
```

Upon opening file.txt, you'll see a new addition:

```
<<<<<<< HEAD
this is overwriting text
=======
abcdefghij
>>>>>>> new_branch
```

`<<<<<<< HEAD` and `>>>>>>> new_branch` delineate the start and end of the merge conflict, and `=======` separates the conflicting material that's present in each of the branches. (The material between `HEAD` and `=======` is from the main; the material between `=======` and `>>>>>>> new_branch` is from the incoming branch.)

Modify the file to resolve the merge conflict: Decide which line(s) to keep or otherwise integrate them, then delete the `<<<<<<< HEAD`, `>>>>>>> new_branch`, and `=======` markers.

Save the file, stage it, and then commit to resolve the merge conflict:

`git add file.txt`  
`git commit -am 'resolve file.txt merge conflict between main and new_branch'`

**Resolving a merge conflict at the remote**

If a pull request followed by a merge attempt results in a merge conflict, the issue can be resolved at the remote via a similar interface as in the local merge-conflict example above.

You'll see a "Resolve conflicts" button in the GitHub interface, which will let you address conflicts as above (i.e., by deciding what lines to keep/modify and then deleting the `<<...`, `>>...`, and `==...` conflict-delineation flags).

Screenshots of the process are available [here](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-on-github).

---

## Stashing

Stashing is a way to temporarily save working changes and then revert (undo) those changes in your active branch. This is useful when you need to suddenly work on a different part of a repo but want to save uncommitted changes for future work.

### Creating stashes
Stash the uncommitted changes in your working directory with:

`git stash push` _or_  
`git stash push --include-untracked [-u]` to include _untracked_ (this often means _new_) files, _or_  
`git stash --all [-a]` to include _untracked_ and _ignored_ files

Changes are stashed, and the branch you're working in is returned to the state it was in before your changes. Add the `--message[-m]` flag to include a message describing the stash entry (similar to a commit message):

`git stash push -m 'testing code for doing XYZ'`

### Viewing stashes

View the entries in the stash with:

`git stash list`

Entries take the form `stash@{#}`, with `stash@{0}` being the most-recent.

### Applying and deleting stashes

Stashed changes can be reapplied to the branch you're in with:

`git stash pop` _or_  
`git stash pop stash@{#}` to apply a specific stash (`stash@{0}` is the default)

Git will attempt to apply the stashed changes. If successful, the applied stash will be deleted. If Git encounters conflicts, you'll need to resolve them manually (see "Merge conflicts" above) and then delete the stash entry:

`git stash drop stash@{#}`

To fully clear the stash, use:

`git stash clear`