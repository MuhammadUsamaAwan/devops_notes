**Setup**

- `git config --global user.name “[firstname lastname]”` = set a name that is identifiable for credit when review version history
- `git config --global user.email “[valid-email]”` = set an email address that will be associated with each history marker
- You can also add your public SSH Key GitLab>Settings>SSH>Copy>Add Key. To generate key follow [this guide](https://docs.gitlab.com/ee/user/ssh.html#see-if-you-have-an-existing-ssh-key-pair)

**Init**

- `git init` = initialize an existing directory as a Git repository
- `git clone [url]` = retrieve an entire repository from a hosted location via URL.

**Stage & Snapshot**

- `git status` = show modified files in working directory, staged for your next commit
- `git add [file]` = add a file as it looks now to your next commit (stage). Use dot(.) to add all files
- `git reset [file]` = unstage a file while retaining the changes in working directory
- `git commit -m “[descriptive message]”` = commit your staged content as a new commit snapshot

**Banch & Merge**

- `git branch` = list your branches. a \* will appear next to the currently active branch
- `git branch [branch-name]` = create a new branch at the current commit. Naming conventions: temporary branches; feature/feat, bugfix, hotfix, exprimental, WIP. master/main = production branch, dev = development branch, QA/test = testing branch
- `git checkout [branch-name]` = switch to another branch
- `git checkout -b [branch-name]` = create and switch to a new branch

**Inspect & Compare**

- `git log` = show all commits in the current branch’s history

**Tracking Path Changes**

- `git rm -r [file|folder]` = delete the file from project and stage the removal for commit. -r meaning recursive
- `git rm -r --cached [file|folder]` = kept the file from project and stage the removal for commit.

**Share & Update**

- `git remote add origin [url]` = add a git remote repository
- `git fetch` = fetch down all the branches from that git remote
- `git merge [branch]` = merge the specified branch’s history into the current one
- `git push` = transmit local branch commits to the current remote repository branch
- `git push origin [branch]` = transmit local branch commits to the remote repository branch
- `git pull` = fetch and merge any commits from the tracking remote branch
- `git pull --rebase | git pull -r` = fetch and merge any commits from the tracking remote branch without andding a new commit

**Undo Commits / Rewrite History**

- `git commit --amend` = merge the changes into the previous commit
- `git checkout [commit-hash]` = go back to a previous commit
- `git rebase [branch]` = apply any commits of current branch ahead of specified one
- `git reset --hard HEAD~[n]` = undo n number of commits and discard the changes
- `git reset --soft HEAD~[n]` = undo n number of commits and kept the changes
- `git push --force` = push and overwrite the remote branch
- `git revert [commit-hash]` = create a new commit to revert the old commit's changes

**Conflicts**

- `git rebase --continue` = run this command after resolving all conflicts

**Temporary Commits**

- `git stash` = save modified and staged changes
- `git stash list` = list stack-order of stashed file changes
- `git stash pop` = write working from top of stash stack
- `git stash drop` = discard the changes from top of stash stack
