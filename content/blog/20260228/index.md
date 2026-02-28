---
date: '2026-02-28T13:39:00-05:00'
draft: false
title: '28FEB28 - Git remote track fix'
tags:
  - git
  - lessons learned
---
# Intro
While working on my git repo for my research project found here: [https://github.com/adminprivileges/ebpH](https://github.com/adminprivileges/ebpH) I was making changes to a local branch and pushing it up to the remote github repo without really worrying about where it was going because I assumed it was going to a remote branch. I was wrong and the changes were going directly to main. This isnt really a huge issue since the repo is only for me, but I'm typing this out so hopefully I dont make this mistake somewhere it actually matters and so I can have the steps to fix it id it does. I'll include the actual output to give you a practical example as well and you can check out the repo for the exact changes.

P.S. I should probably change my branch from master to main, it is black history month after all. 

# The Fix

## 1. Get your bearings 
Its a good idea to figure out whats going on here before figuring out how to proceed. The following command should give you an idea of your situation:
```bash
git status
git branch -vv
git log --oneline --decorate --graph --all -n 30
```  
Here was my output
```bash
adminprivileges@computer:~/ebpH$ git status
On branch container-scope
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean

adminprivileges@computer:~/ebpH$ git branch -vv
* container-scope aa5d986 [origin/master] Implemented container-aware key derivation and container context helpers
  master          4baee67 [origin/master: behind 3] bootstrap should be working now.

adminprivileges@computer:~/ebpH$ git log --oneline --decorate --graph --all -n 30
* aa5d986 (HEAD -> container-scope, origin/master, origin/HEAD) Implemented container-aware key derivation and container context helpers
* fefaca8 added arguments to userspace tool as well
* 4e75ad2 added arguments so ebph accepts --scope-mode
* 4baee67 (master) bootstrap should be working now.
* b37dd1f updated bootstrap to include new make logic
* 7748b14 Update Automated Install section header
*   5855fda Merge pull request #1 from adminprivileges/ubuntu2204-pyenv38
|\  
| * 1e8af78 (origin/ubuntu2204-pyenv38) fixed install, oneshot was not the correct choice
| * 77ad345 formatting was off for html
| * e1612d7 Revise initial setup section in README
```
As you can see from this, my current local  branch is `container-scope` at commit `aa5d986` 
- This branch is configured to track `origin/master` which is the crux of my issue. I need to make a `origin/container-scope` to send my local changes to
    ```
    On branch container-scope
    Your branch is up to date with 'origin/master'.
    ```
- the git log confirm that `origin/master` is the same commit hash as my local branch confirming the issue 
    ```
    aa5d986 (HEAD -> container-scope, origin/master, origin/HEAD) Implemented container-aware key derivation and container context helpers
    ```
- There are 3 commits that need to be scrubbed from master and moved to container-scope
    - `aa5d986`
    - `fefaca8`
    - `4e75ad2`

## 2. Create a remote branch for the changes
Here you will create the remote branch you throught you had in the first place by pushing your current changes to a new remote branch (it will get created automatically). At this point you should be tracking this remote branch, but its a good idea to just force it to ensure you dont end up right back here.
```bash
# Create Remote Branch
git push -u origin <REMOTE_BRANCH_NAME>
# Set the correct upstream branch
git branch --set-upstream-to=origin/<REMOTE_BRANCH_NAME> <LOCAL_BRANCH_NAME>
```
In my case I did the following:
```bash
adminprivileges@computer:~/ebpH$ git push -u origin container-scope
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
remote: 
remote: Create a pull request for 'container-scope' on GitHub by visiting:
remote:      https://github.com/adminprivileges/ebpH/pull/new/container-scope
remote: 
To github.com:adminprivileges/ebpH.git
 * [new branch]      container-scope -> container-scope
branch 'container-scope' set up to track 'origin/container-scope'.

adminprivileges@computer:~/ebpH$ git branch --set-upstream-to=origin/container-scope container-scope
branch 'container-scope' set up to track 'origin/container-scope'.
```
## 3.Put everything back where it belongs
- Option A (what i did): 
    - Because my local main branch was already where i wanted it, which is behind those 3 commits due to them being done locally in a branch, I can simply switch to the master branch and push the current state, specifying the master branch.
        ```bash
        git switch master
        git push --force-with-lease origin master
        ```
    - Doing it this way (--force-with-lease) allows me to erase the shame of being bad at git and pretend it never happened. `--force-with-lease` also provides a safer alternative to `--force` which will overwrite remote changes, which isnt a big deal to me right now, but could be to someone working on a team.

        ```bash
        adminprivileges@computer:~/ebpH$ git switch master
        Switched to branch 'master'
        Your branch is behind 'origin/master' by 3 commits, and can be fast-forwarded.
        (use "git pull" to update your local branch)
        adminprivileges@computer:~/ebpH$ git push --force-with-lease origin master
        Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
        To github.com:adminprivileges/ebpH.git
        + aa5d986...4baee67 master -> master (forced update)
        ```

- Option B (what i should have done to remember this mistake):
    - This option doesnt remove the history so you get to live with your mistake showing in your git history, which is good, it also shows that you were able to fix this mistake and learn from it. It works by pulling down the changes from the remote repo, reverting them, and then pushing again. This is useful if you cant force push as well. 
        ```bash
        git switch master
        # Brings the local master to the current container-scope HEAD
        git pull --ff-only origin master
        # Reverts changes to where you want to be 
        git revert 4baee67..HEAD
        # Push the changes with the revert commits
        git push origin master
        ```
## 4. Verify
Now you can simply ensure that 
```bash
# Print your current branch
git rev-parse --abbrev-ref HEAD
# Print its upstream
git rev-parse --abbrev-ref --symbolic-full-name @{u}
git branch -vv
git log --oneline --decorate --graph --all -n 15
```

Now i can see master is tracking master and container-scoped is tracking container-scoped:
- ```bash
    # New Branch
    container-scope aa5d986 [origin/container-scope]
    # Master Branch
    * master          4baee67 [origin/master] 
  ```
Here are the outputs of my verification:
```bash
adminprivileges@computer:~/ebpH$ git rev-parse --abbrev-ref HEAD
master

adminprivileges@computer:~/ebpH$ git rev-parse --abbrev-ref --symbolic-full-name @{u} 
origin/master

adminprivileges@computer:~/ebpH$ git branch -vv
  container-scope aa5d986 [origin/container-scope] Implemented container-aware key derivation and container context helpers
* master          4baee67 [origin/master] bootstrap should be working now.

adminprivileges@computer:~/ebpH$ git log --oneline --decorate --graph --all -n 15
* aa5d986 (origin/container-scope, container-scope) Implemented container-aware key derivation and container context helpers
* fefaca8 added arguments to userspace tool as well
* 4e75ad2 added arguments so ebph accepts --scope-mode
* 4baee67 (HEAD -> master, origin/master, origin/HEAD) bootstrap should be working now.
* b37dd1f updated bootstrap to include new make logic
* 7748b14 Update Automated Install section header
*   5855fda Merge pull request #1 from adminprivileges/ubuntu2204-pyenv38
|\  
| * 1e8af78 (origin/ubuntu2204-pyenv38) fixed install, oneshot was not the correct choice
```

## Lessons learned
1. Double check what youre tracking before the first push
    ```bash
    git branch -vv #look for [origin/<LOCAL_BRANCH_NAME>]
    ```

2. On the first push of a new branch, always set the upstream to a remote branch of the same name.
    ```bash
    git push -u origin HEAD
    ```
3. You can also set git to automatically do this if yours isnt already doing so. 
    ```bash
    git config --global push.autoSetupRemote true
    ```
    - You can check this with the following
        ```bash
        git config --get push.default
        git config --get push.autoSetupRemote
        ```