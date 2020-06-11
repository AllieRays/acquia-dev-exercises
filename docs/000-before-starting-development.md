# Steps to take every time you start development
In order to keep your local development in sync with the entire dev team. \
Please follow the following steps before you start a new Jira ticket.


### Review your JIRA ticket
Look at the JIRA ticket assigned to you and review your tasks before you start. 

### Fetch all of latest code changes from Github from the upstream 
```
$ git fetch --all
```

### For the purposes of the dev assessment exercise 
```
$ git checkout dev-assessment-exercises
$ git pull upstream dev-assessment-exercises
```

### Sync your origin
Sync any changes from the upstream to your origin fork 
```
$ git push origin
````

### Run blt setup
```
$ blt setup
```

### If you want to log into the site 
```
$ cd docroot
$ drush uli
```

--- 

## After the dev-assessment

### Fetch all of latest code changes from Github from the upstream 
```
$ git fetch --all
$ git checkout develop
$ git pull upstream develop
```

### Rebase your current branch with develop 
```
$ git rebase upstream/develop
```

### Sync your origin
Sync any changes from the upstream to your origin fork 
```
git push origin
````

### Create a feature branch from develop 
```
$ git checkout develop 
$ git checkout -b [JIRA-Ticket-number]:-short-description 
````
for example 
```
$ git checkout -b SCC-001:-faq-content-type
```

### Run blt setup
```
$ blt setup
```

### If you want to log into the site 
```
$ cd docroot
$ drush uli
```

### Commit any new changes and push back to orgin 
```
$ git add . 
$ git commit -m"[JIRA-Ticket-number]: decription of commits."
$ git push origin [JIRA-Ticket-number]:-short-description 
```
