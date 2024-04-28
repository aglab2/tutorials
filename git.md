# git guide

GitHub and git can be used for collaboration for project hosted on GitHub between multiple people. I will attempt to provide basic information about how to make your first Pull Request. Notice that git tends to be complicated, I try to describe only a very simple usecase.

# Terminology

This is very basic representation of git structure

 * commit - a name with diff showing difference between current and previous state of the project
 * branch - a set of commits. A new branch can be created from an existing branch.
 * remote/local - remote and local version of git repository which might be different.
 * fork - repository can be forked for a given collaborator from the "original" repository. Fork might be behind or even independent of the "original" repository. It is _highly_ recommended to make sure forks is not independent.
 * merge - act of putting given commits from a given branch on top of current branch.
 * Pull Request (PR) - a request to "merge" given commits in a given branch

# Example

Let's see an example. Assume there is a repository for Star Display layouts https://github.com/StarDisplayLayouts/layouts. My fork of this repository is https://github.com/aglab2/layouts. When I wanted to make changes in the repository, I made a new "branch" called "jj" where I put my changed https://github.com/aglab2/layouts/commits/jj. I clone my fork locally, make a new commit, push my changes to remote. From my branch "jj" I create a Pull Request on GitHub to "main" repository.

# How to make a simple PR

Start by making a fork of the repository. You will need to create an account on GitHub. I will use "cmder" in this tutorial, WSL should be also sufficient.
Open command line and generate SSH keys
```
ssh-keygen
```

Use default parameters for everything, it will write your SSH key to the paths listed in first 2 lines. From terminal read "id_rsa.pub" file using cat.
```
cat C:\Users\[NAME]\.ssh\id_rsa.pub
```

Go back to GitHub, click on your avavar in top-right corent, press "Settings". Go to SSH and GPG keys, click "New SSH Key" and paste the contents of file "id_rsa.pub". Click "Add SSH Key". This will allow to manipulate "local" repository copy.

Go to "original" repository, for example https://github.com/StarDisplayLayouts/layouts and click "Fork" button. UI will let you create a "fork" of the repository called https://github.com/[YOUR_NAME]/layouts.

To change the content of your fork, the easiest way is to use "git" command line tool in "cmder". Acquire the SSH link, use "Code" > SSH > Copy to clipboard. Clone the "remote" repository to a "local" copy in any folder you wish. You may use these helpful commands to navigate
```
D:
cd D:\git
git clone git@github.com:aglab2/layouts.git ./layouts
```

This will clone repository to "D:\git\layouts".

You may now apply your changes to your local version of the repository.

I highly recommended to make a separate branch to make PR from now. Executing the following command:
```
git checkout -b my-new-feature
```

This will create a new branch called "my-new-feature", you may change the name of the branch. Create a new commit with a name
```
git commit -m "I made a mew feature"
```

Push your changes to remote will fail with message similar to
```
git push --set-upstream origin "my-new-feature"
```

This will create a new "remote" branch letting you make a PR on GitHUb UI as presented by a convenient link that should be written after this command is done:
```
λ git push --set-upstream origin my-new-feature
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
remote:
remote: Create a pull request for 'test' on GitHub by visiting:
remote:      https://github.com/aglab2/tutorials/pull/new/my-new-feature
remote:
To github.com:aglab2/tutorials.git
 * [new branch]      my-new-feature -> my-new-feature
Branch 'my-new-feature' set up to track remote branch 'my-new-feature' from 'origin'.
```

Following this link, change "base" to be "original" repository branch. Create your first PR by pressing button "Create Pull Request". There should be a green checkmark saying "Able to merge" notifying that your changes are done properly.

After your PR is merged, your "fork" branch will behind. When fork is behind, GitHub will provide a convenient UI on your "fork" URL - Sync Fork. You may click "Update Branch" to update the current branch to original ensuring forked branch is up-to-date. 
When remote copy is updated, sync the local copy using command "git pull"
```
git pull
```

To switch back to original branch, use git checkout
```
git checkout main
```

Now you apply more changes again and if another collaborator does changes, perform Fork Update + git pull combination again.

# Other useful commands

Similarly to main, you may switch to any other branch using "checkout".
```
git checkout my-cool-branch
```

To reset all changes to default state use git reset. Notice that your progress will be lost!
```
git reset --hard
```

To reset current branch to remote state for branch "test", you may reset like this
```
git reset --hard origin/test
```

To see current status, use status which shows very nice information of current git state
```
git status
```

[!] For collaboration purposes, it is important to ensure that "fork" is only ever behind or equal to "original". Otherwise PR cannot be merged because of "merge conflict". Please try to make sure your fork is up-to-date when PR is created. [!]