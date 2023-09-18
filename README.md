# Latihan Git untuk BRIN HPC Workshop 2023

*Materi disadur dari [`APC524-F2022/git-exercise`](https://github.com/APC524-F2022/git-exercise)*

## Introduction

This problem will explore the typical use cases of `git` as a version control for scientific computing. Before proceeding with this problem, make sure you have configured your local `git` configuration and GitHub account properly. Make sure that the GitHub account that you are using has been added to the [class organization](https://github.com/brin-hpc-workshop). If this is not the case, please contact one of the TA.

This assignment assume that you are using a UNIX shell. It is a good practice to log the command that you have used as you might need to purge your solution repositories a couple of times while doing this assignment.

## Part 1 - Initiating and cloning repository

Most of your work will start by initiating a fresh repository or cloning an already established repository of someone else's codebase. As an eager researcher you want to do some experiment with the codebase. However, as a nice person, you do not want to accidentally destroy the original codebase or you might not even have write access in the first place. While you can solve this issue by working locally with a cloned repository then pushing it to the remote at the end of your experimentation, this defeats the use of version control for collaboration and additional source of backup.

GitHub has the concept of **Fork** which creates your own repository based on someone else's repository. Yet, there are several disadvantages of doing this directly especially if you are working with **private** repositories due to the inherent nature that the forked repositories are always linked to the original repository.

In this part, you are going to learn how to initiate and clone an isolated copy of this repository through these steps:

1. Create a **private** repository in the class organization with the name format `git-exercise-{USERID}` such that it is accessible through `https://github.com/brin-hpc-workshop/git-exercise-{USERID}`.
2. Clone this repository to your local machine and make sure you properly get all of the branches. This can be done by using the command `git checkout {branch_name}` after cloning.
3. Rename the `origin` remote pointing to the assignment repository in your local folder to `upstream`.
4. Add your private repository as the new `origin` remote and make sure that all local branches are pointing to the new origin as their upstreams.
5. Run this command:

    ```bash
    git checkout main
    git remote -v > remote_data
    git branch -a > branch_data
    git add .
    git commit -m "Remote exercise"
    git push
    ```

    Make sure that you can see the same branch structure in your private repository and `main` is the main branch.

Make sure that you can see the same branch structure in your private repository and `main` is the main branch.

If you make any mistake at this part, you can also delete and recreate your private repository and start from the beginning.

> After setting this up, you can reset the folder to this point of the assignment whenever you want by using `bash reset.sh`.
>
> **WARNING: This is an irrecoverable operation!**

## Part 2 - Ignoring files

While doing experimentation with your local copy, you might not want to add some files, like the generated plot or the data file, in your commit. The easiest way to do this is by using `.gitignore` file, similar to what is included in this repository. As most of you are going to work with `python` development, the current `.gitignore` has been populated with a list of typical cache file that can be created during your experimentation.

In this part, you will use `.gitignore` file to selectively track files in your repositories through these steps:

1. During the compilation process of a C/C++ program you will typically ended up with the object files, with extensions `.o` and/or `.obj`, which you want to ignore. Add this new restrictions to your `.gitignore` and test that it is working as intended by running:

    ```bash
    echo "This file should not be committed" > $(date +%s).o
    echo "This file should not be committed" > $(date +%s).obj
    echo "This file should be committed" > $(date +%s).object
    git add .
    git commit -m "Gitignore exercise part 1"
    git push
    ```

2. In your `main` branch, the file `old.o` is still currently being tracked, to see that this is the case let's completely change the content of the file before trying to commit using these commands:

    ```bash
    ls -alf > old.o
    git add .
    git commit -m "Gitignore exercise part 2"
    git push
    ```

3. Recall that you can remove a file from being tracked by using `git rm`. Use **an option** of `git rm` to stop tracking `old.o` file without actually removing it from the current directory.

    After doing so run these commands:

    ```bash
    ls -alf > current_files
    git add .
    git commit -m "Gitignore exercise part 3"
    git push
    ```

4. Maybe there are some file ending with `.obj` that you still want to track. Change your `.gitignore` to make sure this file is tracked. After you finish, run these commands:

    ```bash
    echo "This is an important file" > important.obj
    git add .
    git commit -m "Gitignore exercise part 4"
    git push
    ```

## Part 3 - Repository conflict

One of the most common issue that you will encounter is that the remote repository is getting updated while you're working on your local branch.

Let's first try to create diverging commit history in the remote repository through these steps:

1. Separately clone your **private repository** into a different folder in your local machine. Throughout this assignment, this local folder will be referred as `friend_copy`.
2. Update the file `source.py` from the current/old version to the new version in `friend_copy`:
    - Old

        ```python
        # This is the original source code file
        a = 1 + 1
        print(a)
        ```

    - New

        ```python
        # This is the source code file
        a = 1 + 2.5
        print(a)
        ```

3. Commit and push your newly changed `source.py` from `friend_copy` to the remote `main` branch. Use `Friend commit` as the commit message.

Now go to the original folder that you worked on before where both `origin` and `upstream` remotes are defined. Refer this local folder as `local_copy` and follow these steps:

1. **Without pulling**, modify the file `source.py` in `local_copy` to this new version:
    - Old

        ```python
        # This is the original source code file
        a = 1 + 1
        print(a)
        ```

    - New

        ```python
        # This is the experimental source code file
        a = 1 + 1
        b = -10
        print(a)
        print(b)
        ```

2. Commit your newly changed `source.py` to your **local** `main` branch in `local_copy`. Use `Local commit` as the commit message.
3. Attempt to push your commit in `local_copy` to your origin, make sure to log the error of this to `p3_log` file using:

    ```bash
    git push 2> p3_log
    ```

4. The `push` should fail as the remote commit history is different than the commit history in the `local_copy`. The error message suggested that you should try `git pull ...`, so you should and also log it to your log file using:

    ```bash
    echo "----" >> p3_log
    git pull >> p3_log 2>&1
    git merge --abort
    ```

    Your goal right now is to incorporate your friend's change to `source.py` and align the fix history such that the new file has this content:

    ```python
    # This is the source code file
    a = 1 + 2.5
    b = -10
    print(a)
    print(b)
    ```

    There are multiple ways that you can do this with or without overwriting the previous commit. As the conflict is still linear, we can try to put our new commit in `local_copy` after the last commit in the remote. This can be done using `--rebase` option of `git pull`. This option will `rebase` all of the local commits to the end of remote commit. This means that you might have to resolve merge conflict while doing so.

5. Use `git pull --rebase` and solve the merge conflict in `source.py`. After you resolved the merge conflict, execute this command **before** executing `git rebase --continue`:

    ```bash
    echo "----" >> p3_log
    cat source.py >> p3_log
    echo "----" >> p3_log
    git status >> p3_log
    ```

6. Finish your rebasing process using `git rebase --continue`, this will put our new commit after the commit from `friend_copy`.

7. Print out your current commit history using:

    ```bash
    git log --oneline --graph --decorate --all > p3_tree
    ```

8. Commit `p3_log` and `p3_tree` to a new commit of `main` branch after the rebased commit and push the two commits to the remote. Use `Rebase exercise` as the commit message.
