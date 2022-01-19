---
title: 'Security in Code Deployment - part 5: Patching with Composer and Deploy'
description: 'Security in Code Deployment, part 5/5'
tags: 'drupal,drush,composer,deployment'
cover_image: ''
canonical_url: null
published: true
id: 960462
date: '2022-01-19T14:12:27Z'
---

**Part 5/5.** The article was initially published in [Drupal Watchdog 7.01](https://shop.linuxnewmedia.com/us/magazines/drupal-watchdog/eh35014.html), spring 2017.

**Code examples for this article**: [https://github.com/rpsu/dwd-security-in-code-deployment](https://github.com/rpsu/dwd-security-in-code-deployment)

Security in Code Deployment:

* [Part 1: Drush make file](https://dev.to/rpsu/security-in-code-deployment-part-1-drush-make-file-b6i)
* [Part 2: Patch modules with Drush](https://dev.to/rpsu/security-in-code-deployment-part-2-patch-modules-with-drush-4g6a)
* [Part 3: Unknown Drupal codebase](https://dev.to/rpsu/security-in-code-deployment-part-3-unknown-drupal-codebase-2f3)
* [Part 4: Advantages of using Composer JSON](https://dev.to/rpsu/security-in-code-deployment-part-4-advantages-of-using-composer-json-ijm)
* Part 5: Patching with Composer and Deploy



---

Composer is able to apply patches if you are using `drupal-composer/drupal-project` or some other patching plugin for Composer (Listing 14).

**LISTING 14: Composer Patching**

```json
"extra": {
  "patches": {
    "drupal/field_group": {
      "A patch with URL": "https://www.drupal.org/files/issues/ field_group-empty_group_nonnumeric_index-2761159-2-D8.patch", 
      "A local patch file, path relative to the composer.json file": "patches/field_group-fix_it.patch"
    }
  }
}
```
After you have added the patch files to the composer. json file, apply the changes and update the `composer.lock` file:

```bash
$ composer install
$ composer update --lock
```

Currently with Drupal 8, all contrib modules and other resources must be added manually one by one. If you are creating a `composer.json` file from a Drupal 7 site, you might benefit by using the [Composer Generate module](https://www.drupal.org/project/composer_generate).

If you are interested in building your Drupal 8 projects with Composer, two good resources are [Wolfgang Ziegler’s presentation](http://www.slideshare.net/nuppla/efficient-development-workflows-with-composer) from Drupal Iron Camp 2016 and the [Composer documentation at Drupal.org](https://www.drupal.org/docs/develop/using-composer).

---

## Deployment Process

For codebase deployment, it is best to rely on a builder tool. Ideally, you commit only your custom code (e.g., modules and themes and a Drush make file or the `composer.json` and `composer.lock` files) to the project repository, with specific versions and, optionally, patching information. With proper versioning (think Git tags), you can test your next release’s code in development and staging environments and reduce the probability of reintroducing old or introducing new issues.

Web server configuration is very much related to deployment security, too. However, because it is too broad a topic to be covered here, just remember to make sure your server does not allow end users access to any files other than those they absolutely need. Patch files, Drupal core, and a module’s text files – especially `CHANGELOG.txt` and build time `PATCHES.txt` – are good examples of files that have no business being visible to the world. Even when most of the attacks against websites are automated, there is no reason to leave information about website internals in the open.

--- 

*Special thanks to [Rami Järvinen](https://www.exove.com/author/rami-jarvinen/) for contributions to this blog post and to [Anastasios Daskalopoulos](https://www.exove.com/author/anastasios-daskalopoulos/) and [James Narraway](https://www.exove.com/author/james-narraway/) for their help correcting my words to much better English.
