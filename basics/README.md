## Basics commands

git add  
 secify a files you want to track  
 add new files, staged modified files, and mark merge conflicts resolve  
 e.g.  
 git add file

git commit  
 store a snapshot in local database  
 e.g.  
 git commit -m "commit msg"

git checkout  
 switch between branches  
 create branches  
 e.g  
 git checkout exisitng-branch  
 git checkout commit-hash  
 git checkout -b new-branch

git log  
 show commit logs  
 e.g.  
 git log -3 --oneline  
 git log -S "code"  
 git log --grep="commit msg"

git reset  
 undo commits  
 e.g  
 git reset HEAD^

git push  
 update remote repository  
 e.g.  
 git push branch-name

git diff  
 show changes between commits  
 e.g  
 git diff HEAD^..HEAD

git show  
 show changes in commit  
 e.g  
 git show HEAD

git rebase  
 reapply commits on top of another branch  
 e.g  
 git rebase -i HEAD~3  
git merge  
 join branches  
 e.g  
 git merge branch-name

git cherry-pick  
 apply existing commits  
 e.g  
 git cherry-pick commit-hash
