# Git Commands
git checkout development        ---->   checkout the development branch

git pull                        ---->   pull from remote branch and merge with local branch

git checkout <branch_name>      ---->   checkout an already created branch
git checkout -b <branch_name>   ---->   checkout a new branch with the defined branch name

git status                      ---->   shows what files have been updated


in the future when you rebase development use:
    git rebase --rebase-merges release_candidate

otherwise we'll lose the merge history

-----------------------------------------------
To Rebase after an update
-----------------------------------------------
git checkout development
git pull
git checkout <your_branch>
git rebase --rebase-merges development
git push --force

-----------------------------------------------
* if there are merge conflicts, use the Git UI to resolve them; 'git rebase --continue'; in the
VIM window that pops up type ':wq' to write and quit the changes.  Then continue as normal
