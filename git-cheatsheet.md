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
