---
title: 'Security in Code Deployment - part 4: Advantages of using Composer JSON'
description: 'Security in Code Deployment, part 4/5'
tags: 'drupal,drush,composer,deployment'
cover_image: ''
canonical_url: null
published: true
id: 960460
date: '2022-01-19T14:08:33Z'
---

**Part 4/5.** The article was initially published in [Drupal Watchdog 7.01](https://shop.linuxnewmedia.com/us/magazines/drupal-watchdog/eh35014.html), spring 2017.

**Code examples for this article**: [https://github.com/rpsu/dwd-security-in-code-deployment](https://github.com/rpsu/dwd-security-in-code-deployment)

Security in Code Deployment:

* [Part 1: Drush make file](https://dev.to/rpsu/security-in-code-deployment-part-1-drush-make-file-b6i)
* [Part 2: Patch modules with Drush](https://dev.to/rpsu/security-in-code-deployment-part-2-patch-modules-with-drush-4g6a)
* [Part 3: Unknown Drupal codebase](https://dev.to/rpsu/security-in-code-deployment-part-3-unknown-drupal-codebase-2f3)
* Part 4: Advantages of using Composer JSON
* [Part 5: Patching with Composer and Deploy](https://dev.to/rpsu/security-in-code-deployment-part-5-patching-with-composer-and-deploy-3o04)

---
You are most probably familiar with [Composer](https://getcomposer.org/doc/00-intro.md), at least on a conceptual level. It is a dependency manager with all of the functionality of Drush, and more. From the perspective of building the codebase, the difference between Drush and Composer is the recipe file format and the use or Composer, not Drush, as the build tool. Drush and Composer are not mutually exclusive, you can use both; you can even install Drush along with Drupal using Composer.

Composer uses JSON files, which use curly brackets instead of indentation, as in YAML files. However, unlike Drush make files, `composer.json` files are not meant to be edited in quite the same way. Composer keeps the `composer.json` and `composer.lock` files up-to-date according to your instructions. You still need to add some things manually to `composer.json`, such as the patches you want to use in modules.

From a code deployment perspective, using Composer is just a better way to manage all required resources, because it also makes sure all the module requirements are met; you could learn the hard way that something is missing with the use of an (incomplete) Drush make file.

Start by generating the project in a `drupal.local` folder. The following example uses the GitHub-based [Composer template project](https://github.com/drupal-composer/drupal-project). Before you start, make sure you have Composer [installed](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-osx):

```bash
$ composer create-project drupal-composer/drupal-project:8.x-dev drupal.local
--stability dev --no-interaction
$ cd drupal.local
```

Composer can be used with Drupal 7 projects, too. All you have to change is the branch name in the command to `7.x-dev`. Now the `composer.json` and `composer.lock` files have been created. The Drupal root (the `index.php` file) is in the `web` folder, as are other Drupal modules and themes. Other resources (dependencies) are in the `vendor` folder. You might take a look at what is in the folders; just remember that `composer.lock` contains the whole dependency tree, which easily makes it quite long when compared with the `composer.json` file.

Start by adding Drupal.org as one of the package (module) sources; then, start adding your project dependencies. In Listing 13, I’m adding the very latest version of the Token module that is compatible with the defined Drupal core, as well as a specific version of the Field Group module and a specific commit (1fe3649) from Ctools module’s 8.x-3.x branch. Composer will update `composer.json` and `composer.lock` files when you add (or remove) dependencies.

**LISTING 13: Add Modules with Composer**

```bash 
# Configure Drupal.org as a package source
$ composer config repositories.drupal composer https://packages.drupal.org/8
# Add some modules
$ composer require drupal/token
$ composer require drupal/field_group:1.0-rc4
$ composer require drupal/ctools:dev-3.x#1fe3649
```

Once you have committed and pushed changes, your colleagues can get exactly the same codebase every time, with all dependencies included. All they have to do is clone your repository and run:

```bash
$ composer install --prefer-dist --optimize-autoloader
```

Happy coding!
