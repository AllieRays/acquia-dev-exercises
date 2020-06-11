# Add a module to the site 
In this exercise we will add the [Admin Toolbar](https://www.drupal.org/project/admin_toolbar) module to our local Drupal site. 

## Require a module with composer 
Composer typically works better outside of the VM. 
```
exit vagrant
composer require drupal/admin_toolbar
```

## Install the module with composer 
```
composer update drupal/admin_toolbar --with-dependencies
```

## Use Drush to enable the module 
```
vagrant ssh 
drush en admin_toolbar
```

## Export Active configuration to file system
```
drush cex sync
-y
```

## Review 
Review files that should have changed. \
core.extension.yml \
composer.json \
composer.lock 
