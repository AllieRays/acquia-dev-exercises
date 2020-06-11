# Create a View 

As a site visitor, I want to see a listing of FAQ items on the website so that I can find answers to questions.

Scenario 1: View exists in Drupal Spec Tool \
As a site admin,When I am looking at the Drupal Spec tool \
then I will see a View and all associated fields 

Scenario 2: FAQ listing exists \
As a site visitor, \
When I go to /faqs \
then I will see a listing of the faq content type with the following fields \
() title \
() body \
and I will see 10 items with ajax pagenation \
and the faq node title will not link back to the original node


## Exercise 

### Before you start 
#### Review your JIRA ticket
Look at the JIRA ticket assigned to you and review your tasks before you start. 

#### Pull latest code from the upstream 
```
$ git fetch --all
$ git checkout dev-assessment-exercises
$ git reset --hard upstream/dev-assessment-exercises
```

## Sync your origin
Sync any changes from the upstream to your origin fork 
```
git push origin
````

## Run blt setup
```
$ vagrant ssh
$ blt setup
```

---

Make sure you complete the [create content type exercise](https://github.com/acquia-pso/Santa-Clara-County/blob/dev-assessment-exercises/docs/exercises/002-create-content-type.md). 

### Step one: Create a view 
Navigate to `admin/structure/views/add` and create a new view called faq listing \
show *content* of the type *faq* sorted by *title* \
Check the boolean Create a page \
Page display settings *unformatted list*  of *fields* \
Items to display *20* \
Check the boolean use a pager \
*save and edit* 


### Step two: Add fields 
Add the Answer field to view listing \
No need to adjust settings click apply \
Click on Content Title \
Uncheck Link to the Content \
Click apply


## Step three: Save 
Confirm page path is /faq \
Save the view


## Step four: Save configuration 
Open your terminal and export the config \
`drush cex sync` \
Review the configuration before you type `-y`
 

## Step five: Save your code updates with version control. 
Review and commit only updates related to your faq updates \
`git checkout -b SCC-002-faq-view` Create a new feature branch \
`git status` \
`git add -p` and review each steps \
`git add ../config/default/views.view.faq_listing.yml` add new view config file \
`git commit -m"SCC-002: Creating an FAQ view."` \
`git push origin SCC-002-faq-view` 


## Step six: Create a pull request from your origin to the upstream
Go to your github repo \
and click create a PR. \
Review your code changes \
Review the base branch is dev-assessment-exercises \
Create a PR

