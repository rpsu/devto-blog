---
title: 'Security in Code Deployment - part 3: Unknown Drupal codebase'
description: 'Security in Code Deployment, part 3/5'
tags: 'drupal,drush'
cover_image: ''
canonical_url: null
published: true
id: 960458
date: '2022-01-19T14:07:31Z'
---

**Part 3/5.** The article was initially published in [Drupal Watchdog 7.01](https://shop.linuxnewmedia.com/us/magazines/drupal-watchdog/eh35014.html), spring 2017.

**Code examples for this article**: [https://github.com/rpsu/dwd-security-in-code-deployment](https://github.com/rpsu/dwd-security-in-code-deployment)

Security in Code Deployment:

* [Part 1: Drush make file](https://dev.to/rpsu/security-in-code-deployment-part-1-drush-make-file-b6i)
* [Part 2: Patch modules with Drush](https://dev.to/rpsu/security-in-code-deployment-part-2-patch-modules-with-drush-4g6a)
* Part 3: Unknown Drupal codebase
* [Part 4: Advantages of using Composer JSON](https://dev.to/rpsu/security-in-code-deployment-part-4-advantages-of-using-composer-json-ijm)
* [Part 5: Patching with Composer and Deploy](https://dev.to/rpsu/security-in-code-deployment-part-5-patching-with-composer-and-deploy-3o04)

---

Some day you will receive a project that you must start maintaining - *the unknown Drupal codebase*. In the best case, you get an up-to-date Drush make file with properly declared resource versions, database dumps, and `sites/example.com` folder content.. In the worst case, you get a huge, uncompressed file containing the whole Drupal site starting from the Drupal root, including core and contrib modules and a database dump.

In the first case, codebase reviewing and building is fairly easy – all information for the codebase is written into the make file: Build the codebase, extract `sites/example.com` folder contents, import the database, and you’re good to go.

In the worst-case scenario, you must review the entire codebase and create the missing make file yourself. Provided you have Drush installed, generating a make file is fairly simple. Set up the site in your local development environment and enter:

```bash
$ cd DRUPAL_ROOT/sites/example.com  
$ drush make-generate path/to/example.com.make
```

Now you have at least a base for your make file, but the file is not ready yet. Open your example.com.make file and add the missing components. Components that require editing are clearly marked. For example, a missing component might be that the required libraries aren’t yet set because Drush has no way of knowing where they came from – Drush only queries information about the resources from Drupal.org. If you are lucky, you can find `PATCHES.txt` files or similar in the extracted codebase for help.

The next thing you must do is verify that the received codebase has not been tampered with. The previous maintainer might have changed the codebase in some manner. As with so many other things, you can use a Drupal module to offload this tedious and error-prone work to the machine.

[The Hacked! module](https://www.drupal.org/project/hacked) compares the codebase – Drupal core, modules, and themes – to the versions on Drupal. org by downloading the supposedly same files and verifying that they match the local versions. When you also use the [Diff module](https://www.drupal.org/project/diff), it is easy to see what has changed. Both  of these modules have at least beta-versions available for both Drupal 7 and Drupal 8.
