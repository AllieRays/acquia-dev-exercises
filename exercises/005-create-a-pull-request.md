# Create a Pull Request 
Create a pull request to propose and collaborate on changes to a repository. 
These changes are proposed in a branch, which ensures that the master branch only contains finished and approved work.

As a developer I want to commit my code to the base repository so that I can contribute to the project. 

Scenario 1: Forked Branch \
As a developer \
When I create a branch \
Then I will always create a branch off of my forked repo \
And I will always name my branch with an SCC jira ticket prefix

Scenario 2: Creating a Pull Request \
As a developer \
When I push my latest code to my forked branch \
And I want to create a pull request \
Then I will always make sure my base branch is the correct base branch from the upstream repo a branch

--- 

# Exercise 

#### Step One: Review your JIRA ticket
Look at the JIRA ticket assigned to you and review your tasks before you start. 

#### Step Two: Pull latest code from the upstream 
```
$ git pull upstream dev-assessment-exercises
```

#### Step Three: Check out a new branch 
Check out the base branch you want to base your branch off of. 
```
$ git checkout dev-assessment-exercises
```
Then create a branch from the base branch
```
$ git checkout -b SCC-001-first-pull-request
```

#### Step Four: Create a readme file in this directory 
Navigate to `docs/exercises`
Create a readme with your name. [your-name]-my-notes.md
And add some text 
```
#[your name] read me documentation page 
Hello world. 
```

#### Step Five: Save this file to version control with git 
```
$ git add [your-name]-my-notes.md
$ git commit -m"SCC-001: My first pull request."
$ git push origin SCC-001-first-pull-request
``` 

#### Step Six: Create a PR
Go to your github repo \
and click create a PR. \
Review your code changes \
Review the base branch is dev-assessment-exercises \
Create a PR \
Review [github's documentation about pull requests](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request-from-a-fork) if you need more help with creating a PR.

--- 

## More Documentation 
### Commit Messages Guidelines
Always include the issue number in your commit message. Ex: "SCC-104: Moved install profile to be under /profiles rather than /profiles/custom." \
Optionally include additional information in your commit message if your commit makes multiple changes to the codebase. Example: SCC-104: Moved install profile to be under /profiles rather than /profiles/custom. Added the module security_review Added new install profile configuration option "feature-set"

#### Pull Request Guidelines
Always include a link to the issue  in the PR comments Always review your PR for merge conflicts and fix those. \
Cleaning up your Git repo
```
git fetch -p origin     prune your local "cache" of remote branches firs
git branch -D <branch name>    Deleting a local branch - optional
git push origin --delete <branch name>    push delete changes back up remote  - optional
```

#### Common git commands
```
git command    definition
git remote show origin     shows the origin of the git repo. 
git fetch     updates your remote-tracking branches
git pull    pulls remote changes
git add -p    add each change individually to a single commit
git rebase --interactive HEAD~2    merge your commits together
git clean -fd    remove untracked files and directories
git stash    save and remove local changes but do not commit them
git stash pop    return saved change
sgit fetch
Git add -p lets you add your commits in small batches. By committing one change at a time, you can exclude changes that do not pertain to your commit message. This is helpful when other developers are reviewing your commit messages. Important to note that git add -p will only add changes to files, not add new files. You will still have to use git add for new files. git add -p.
 When you use git add -p you will see the options
 Stage this hunk [y,n,q,a,d,/,e,?]?
 These options in more detail.
y -    stage this hunk
n -    do not stage this hunk
q -    quit; do not stage this hunk or any of the remaining ones
a -    stage this hunk and all later hunks in the file
d -    do not stage this hunk or any of the later hunks in the file
/ -    search for a hunk matching the given regex
j -    leave this hunk undecided, see next undecided hunk
J -    do not stage this hunk or any of the later hunks in the file
k -    leave this hunk undecided, see next undecided hunk
s -    leave this hunk undecided, see next hunk
e -    manually edit the current hunk
? -    print help
```



## More Documentation 
https://www.atlassian.com/git/tutorials/making-a-pull-request
https://www.atlassian.com/git/tutorials/comparing-workflows/forking-workflow
https://www.atlassian.com/git/tutorials/using-branches
