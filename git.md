# Cheatsheet

squash last X commits:
git reset --soft HEAD~X && git commit

undo/redo last commit:
git commit -m "Something terribly misguided"
git reset --soft HEAD~
<< edit files as necessary >>
git add .
git commit -c ORIG_HEAD

undo last commit, leave staged changes:
git reset --soft HEAD~1

difference with last commit:
git diff HEAD~

change most recent commit message:
git commit --amend

nuke last commit forever:
git reset --hard HEAD~1

to reset the time (and author info) on an amended/rebased commit:
git commit --amend --reset-author

to get the latest master and rebase my local changes instead of merging:
git pull --rebase origin master

to delete a remote branch:
git push origin --delete X

to delete local tracking branches of remotely deleted branches:
git fetch --prune origin

to create an archive of a changeset:
git archive -o update.zip HEAD \$(git diff --name-only HASH master)

git config merge.renameLimit 999999
git config --unset merge.renameLimit
