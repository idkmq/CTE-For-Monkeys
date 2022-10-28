# Git Cheat sheet


# Remote download branch 

In the case you clone a repository but dont get a the branches, can run the following command to dl any missing branches.

```git
git fetch <remote> <rbranch>:<lbranch>
git checkout <lbranch>
```

...where `<rbranch>` is the remote branch or source ref and `<lbranch>` is the as yet non-existent local branch or destination ref you want to track and which you probably want to name the same as the remote branch or source ref. This is explained under options in the explanation of `<refspec>`.

## Resources
[Stack Overflow thread](https://stackoverflow.com/questions/9537392/git-fetch-remote-branch)

# Merge Branches 

## Scenario:

Lets say you have a `Main` branch and a `Dev` branch you work from. Main has your ready/stable builds, and `Dev` is were you make your changes and has all the most recent updates. 

Now lets say your ready to push and update to `Main` from `Dev`, all we have to do is merge the two and `Main` will get all the recent updates made in `Dev`


1. Move to the branch you wish to merge files to. ( In our Ex. `Main`)
    ```git
    git checkout main
    ```
2. Run the merge command to get all changes from 'Dev`
    ```
    git merge dev
    ```

# Resources 
- [Git merge specific files from other branch](https://jasonrudolph.com/blog/2009/02/25/git-tip-how-to-merge-specific-files-from-another-branch/)
- [Merging Branches](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)