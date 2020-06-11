# Configuration management with BLT

## Overview

BLT supports several methods of configuration management (CM) in Drupal 8. All of these rely to varying degrees on Drupal core's configuration entities, which can be "imported" into a database or "exported" to disk as yml files.

BLT _strongly recommends_ a CM workflow based on the [Configuration Split](https://www.drupal.org/project/config_split) module, as described on the [Acquia Knowledge Base](https://support.acquia.com/hc/en-us/articles/360024009393). For most projects, this strikes the best balance of flexibility, reliability, and ease of maintenance and development. This document will also describe a Features-based workflow (analogous to most CM workflows in Drupal 7) that can better accommodate certain multisite architectures, but generally has a much higher development and maintenance overhead.

## General principles

This section describes aspects of BLT's development and deployment process that are common to all CM workflows.

### Basics of configuration management

The primary goal of configuration management is to ensure that all configuration changes can be reviewed, tested, and predictably deployed to production environments. Some simple changes, such as changing a site's name or slogan, might have limited, atomic, and predictable effects, and therefore not require strict change management. Other types of changes, such as modifying field storage schemas, _always_ need to go through a review process. Additionally, different projects might have different degrees of risk tolerance. For instance, some might prefer that configuration be strictly read-only in production, prohibiting even the simple site name change above.

A good CM workflow should be flexible enough to accommodate either of these use cases, and make it easy for developers to make and capture configuration changes, review and test these changes, and reliably deploy these changes to a remote environment.

Generally speaking, a configuration change follows this lifecycle:

1. A developer makes the change in her or his local environment.
2. The developer uses CM commands to export the configuration change to disk.
3. The developer commits the new or updated configuration to VCS and opens a pull request.
4. Automated testing ensures that the configuration can be installed from scratch on a new site as well as imported without conflicts on an existing site.
5. After the change is deployed, deployment hooks automatically import the new or updated configuration.

The way that configuration is captured and deployed between environments in Drupal 8 is typically via YAML files. These YAML files, typically stored in a root `config` directory, or distributed with individual modules in `config/install` directories, represent individual configuration objects that can be synchronized with the active configuration in an environment's database via a variety of methods. See [documentation on core configuration management](https://www.drupal.org/docs/8/configuration-management).

This document addresses the challenge of capturing ("exporting") and deploying ("importing") configuration in a consistent way in order to support the workflow described above.

### How BLT handles configuration updates

BLT-based projects already support this workflow, including automatic imports of configuration updates. BLT defines a generic `drupal:update` task that applies any pending database and configuration updates. This same task can be re-used locally or remotely (via the `drupal:update`, and `artifact:update:drupal` wrappers, respectively) to ensure that configuration changes and database updates behave identically in all environments.

When you run one of these update commands, they perform the following updates (see `drupal:config:import`):

- Database updates: the equivalent of running `drush updb` or hitting `update.php`, this applies any pending database updates.
- Config import: runs the core configuration-import command to import any configuration stored in the root `config` directory. This is either a full or partial import, depending on how BLT is configured.
- Features import (optional): runs features-import-all, which imports any configuration stored in a feature module's `config/install` directory. Note that this only runs if you've configured the `cm.features.bundle` property in `blt/blt.yml`.

There are also pre and post-config import hooks that you can use to run custom commands.

### Config vs content
Drupal’s config system cannot be used to manage entities that Drupal considers to be content, such as nodes, taxonomy terms, and files. This can create conflicts when a configuration entity depends on a content entity, such as:

* You have a block type that includes a file upload field, and you want to place the block in a theme and export the block configuration.
* You have a view that is filtered by a static taxonomy term, and you want to export that view configuration.

In these cases, the exported configuration file for the block or view will define a dependency on a content object (referenced by UUID). If that content doesn’t exist when the configuration is imported, the import will fail.

The solution is to make sure that the referenced content exists before configuration is imported. There are currently two recommended methods for this:

* Use the default_content module to export the referenced content as JSON files, and store these files with a feature or other dedicated module.
* Use Migrate and a custom module to create default content from any number of custom sources, such as JSON files stored with your feature.

### Updating core and contributed modules

Take caution when updating core and contributed modules when using any sort of configuration management process. If those updates make changes to a module’s configuration or schema, you must make sure to also update your exported configurations. Otherwise, the next time configuration is imported (e.g. on deploys), the import of your configuration will clobber changes made by the database updates, and your database and codebase will become out of sync. Failing to take this into account is the most common cause of configuration imports failing BLT's test for overridden configuration.

The best way to prevent such problems is to always follow these steps when updating contributed and core modules. These steps assume you are using Config Split or a similar CM strategy using `drush cex` and `drush cim`.

1. Start from a clean install or database sync, including config import (`blt setup` or `blt drupal:sync`). Verify that active and exported configuration are in sync (running `drush config-export` should report no changes.)
2. Use `composer update` or `composer update drupal/[module_name] --with-dependencies` to download the new module version(s). To update Drupal core, use `composer update webflo/drupal-core-require-dev drupal/core --with-dependencies`.
3. Run `drush updb` to apply any pending updates locally.
4. Export any configuration that changed as part of the database updates or new module versions by running `drush config-export` and checking for any changes on disk using `git status`.
5. Commit any changed configuration, along with the updated `composer.json` and `composer.lock`.

We need to find a better way of preventing this than manually monitoring module updates. Find more information in these issues:
* [Features and contributed module updates](https://www.drupal.org/node/2745685)
* [Testing for schema changes to stored config](https://github.com/acquia/blt/issues/842).

### Ensuring integrity of stored configuration

Configuration stored on disk, whether via the core configuration system or features, is essentially a flat-file database and must be treated as such. For instance, all changes to configuration should be made via the UI or an appopriate API and then exported to disk. You should never make changes to individual config files by hand, just as you would never write a raw SQL query to add a Drupal content type. Even seemingly small changes to one part of the configuration can have sweeping and unanticipated changes. For instance, enabling the Panelizer or Workbench modules will modify the configuration of every content type on the site.

BLT has a built-in test that will help protect against some of these mistakes. After configuration is imported (i.e., during `drupal:update` or `artifact:update:drupal`), it will check if any configuration remains overridden. If so, the build will fail, alerting you to the fact that there are uncaptured configuration changes or possibly a corrupt configuration export. This test acts as a canary and should not be disabled, but if you need to temporarily disable it in an emergency (i.e., if deploys to a cloud environment are failing), you can do so by settings `cm.allow-overrides` to `true`.

Finally, you should enable protected branches in GitHub to ensure that pull requests can only be merged if they are up to date with the target branch. This protects against a scenario where, for instance, one PR adds a new content type, while another PR enables Workbench (which would modify that content type). Individually, each of these PRs is perfectly valid, but once they are both merged they produce a corrupt configuration (where the new content type is lacking Workbench configuration). When used with BLT’s built-in test for configuration overrides, protected branches can quite effectively prevent some forms of configuration corruption.

For an ongoing discussion of how to ensure configuration integrity, see https://www.drupal.org/node/2869910

## Configuration Split Workflow

More information can be found in the article [Managing Configuration with Config Split](https://support.acquia.com/hc/en-us/articles/360024009393) on the Acquia Knowledge Base.

#### Troubleshooting

If for some reason BLT is not working with Config Split, ensure that you are using Drush version 8.1.10 or higher, Config Split version 8.1.0-beta4 or higher, and that `cm.strategy` is set to `config-split` in `blt/blt.yml`.

## Using update hooks to importing individual config files

BLT runs module update hooks before importing configuration changes. For use cases where it is necessary for a configuration change to be imported before the update hook runs, in your hook, you'll need to import the needed configuration from files first. (An example of this would be adding a new taxonomy vocabulary via config, and populating that vocabulary with terms in an update hook.)

Code snippet for importing a taxonomy vocabulary config first before creating terms in that vocabulary:

    use Drupal\taxonomy\Entity\Term;

    // Import taxonomy from config sync directory.
    $vid = 'foo_terms'; // foo_terms is the vocabularly id.
    $vocab_config_id = "taxonomy.vocabulary.$vid";
    $vocab_config_data = foo_read_config_from_sync($vocab_config_id);
    \Drupal::service('config.storage')->write($vocab_config_id, $vocab_config_data);

    Term::create([
      'name' => 'Foo Term 1',
      'vid' => $vid',
    ])->save();

    Term::create([
      'name' => 'Foo Term 2',
      'vid' => $vid',
    ])->save();

This depends on a helper function, which can be added to your custom profile:

    use Drupal\Core\Config\FileStorage;

    /**
     * Reads a stored config file from config sync directory.
     *
     * @param string $id
     *   The config ID.
     *
     * @return array
     *   The config data.
     */
    function foo_read_config_from_sync($id) {
      // Statically cache FileStorage object.
      static $storage;

      if (empty($storage)) {
        global $config_directories;
        $storage = new FileStorage($config_directories[CONFIG_SYNC_DIRECTORY]);
      }
      return $storage->read($id);
    }


### Updating custom fields and schema
There are some configuration changes that Features (and the core config system) doesn’t handle well, including:

* Updating field storage (e.g., changing a single-value field to an unlimited-value field)
* Adding a [new custom block type](https://www.drupal.org/node/2702659) to an existing feature (sadly, you have to create a new feature for every block type)
* Deleting a field (you'll want to remove the field from the feature and then use the code snippet below to actually delete the field)
* Adding a field to some types of content (such as [block content](https://www.drupal.org/node/2661806))
* Adding multiple config entities at once that depend on one another (leading to [cryptic exceptions](https://www.drupal.org/node/2726839) when you run features-import... use the workaround below)

To handle these things, you'll want to use update hooks. For instance, you can use the following snippet of code to create or delete a field:

    use Drupal\field\Entity\FieldStorageConfig;
    use Drupal\field\Entity\FieldConfig;

    // Create a new field.
    module_load_include('profile', ‘foo', 'foo'); // See below; foo is your profile name.
    $storage_values = foo_read_config('field.storage.block_content.field_my_new_field', 'foo_feature');
    FieldStorageConfig::create($storage_values)->save();
    $field_values = foo_read_config('field.field.block_content.foo_my_block.field_my_new_field', 'foo_feature');
    FieldConfig::create($field_values)->save();

    // Delete an existing field.
    $field = FieldStorageConfig::loadByName('block_content', 'field_my_field');
    $field->delete();

This depends on a helper function like this, which I suggest adding to your custom profile (Lightning includes this out of the box):

    use Drupal\Core\Config\FileStorage;
    use Drupal\Core\Config\InstallStorage;

    /**
     * Reads a stored config file from a module's config/install directory.
     *
     * @param string $id
     *   The config ID.
     * @param string $module
     *   (optional) The module to search. Defaults to 'foo' profile (not technically
     *   a module, but profiles are treated like modules by the install system).
     *
     * @return array
     *   The config data.
     */
    function foo_read_config($id, $module = 'foo') {
      // Statically cache all FileStorage objects, keyed by module.
      static $storage = [];

      if (empty($storage[$module])) {
        $dir = \Drupal::service('module_handler')->getModule($module)->getPath();
        $storage[$module] = new FileStorage($dir . '/' . InstallStorage::CONFIG_INSTALL_DIRECTORY);
      }
      return $storage[$module]->read($id);
    }

### Overriding configuration

Drupal normally prevents modules from overriding configuration that already exists in the system, producing an exception like this:

    Configuration objects (foo) provided by bar already exist in active configuration

If you need to override the default configuration provided by another project (or core), the available solutions are:

* Recommended: use Features. Features will prevent a PreExistingConfigException from being thrown when a feature containing pre-existing configuration is installed. Ensure that Features is already enabled before installing any individual features that might contain configuration overrides (simply listing Features as a dependency isn't sufficient).
* Move your config into the a custom profile. Configuration imports for Profiles are treated differently than for modules. Importing pre-existing configuration for a Profile will not throw a PreExistingConfigException.
* Use [config rewrite](https://www.drupal.org/project/config_rewrite), which will allow you to rewrite the configuration of another module prior to installation.
* Use the [config override system](https://www.drupal.org/docs/8/api/configuration-api/configuration-override-system) built into core. This has [some limitations](https://www.drupal.org/node/2614480#comment-10573274) of which you should be wary.

### Other gotchas

Be aware that reverting all features and config on every deploy creates a risk of discarding server-side changes. This risk should be controlled by carefully managing permissions, and must be balanced against the greater risk of allowing for divergent configuration between your DB and VCS.

Configuration Management in Drupal 8 is still being improved early in the Drupal 8 lifecycle, and you should continue to monitor Drupal Core's issue queue and Drupal Planet blog posts for refinements to the CM workflows explained here.

Similarly, Features is a ground-up rewrite in Drupal 8 and is maturing quickly, but may still have some traps. Developers should keep a close eye on exported features, and architects need to carefully review features in PRs for the gotchas and best practices listed above.
