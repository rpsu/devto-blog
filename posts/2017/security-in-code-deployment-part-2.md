---
title: 'Security in Code Deployment - part 2: Patch modules with Drush'
description: 'Security in Code Deployment, part 2/5'
tags: 'drupal,drush,patching'
cover_image: ''
canonical_url: null
published: true
id: 960452
date: '2022-01-19T14:03:09Z'
---

**Part 2/5.** The article was initially published in [Drupal Watchdog 7.01](https://shop.linuxnewmedia.com/us/magazines/drupal-watchdog/eh35014.html), spring 2017.

**Code examples for this article**: [https://github.com/rpsu/dwd-security-in-code-deployment](https://github.com/rpsu/dwd-security-in-code-deployment)

Security in Code Deployment:

* [Part 1: Drush make file](https://dev.to/rpsu/security-in-code-deployment-part-1-drush-make-file-b6i)
* Part 2: Patch modules with Drush
* [Part 3: Unknown Drupal codebase](https://dev.to/rpsu/security-in-code-deployment-part-3-unknown-drupal-codebase-2f3)
* [Part 4: Advantages of using Composer JSON](https://dev.to/rpsu/security-in-code-deployment-part-4-advantages-of-using-composer-json-ijm)
* [Part 5: Patching with Composer and Deploy](https://dev.to/rpsu/security-in-code-deployment-part-5-patching-with-composer-and-deploy-3o04)

---

Occasionally (or more often) your project could have needs that require you to modify the available module code. Maybe the module has a bug or lacks a required feature, which has in fact already been addressed in the module issue queue on Drupal.org (see “Checking the Issue Queue” box) – after all, Drupal has a large and active community. All you need to do is find out if a working patch is available that fits your needs or fixes the bug in question.

    CHECKING THE ISSUE QUEUE
    Go to the specific module’s page on Drupal.org, find the Issues for section in the sidebar, and search. If a proposed solution to your problem exists, patches can be found after the Issue de- scription. Be sure to read the whole discussion to see if it actually solves your problem.
    
A patch file is a recipe describing what changes should be made and to which files. Explaining and creating patch files is outside the scope of this article, but you should be able to find help on [Drupal.org](https://www.drupal.org/patch).

You could fix the issue manually or apply the patch and leave it fixed on the server, but this solution is against best practices, because the very next codebase update will override the changes you’ve made and reintroduce the issue.

You could also try to remember to apply the patch or changes each time you do updates, but that’s quite an error-prone process. You can and should offload this work to Drush. If you need a custom patch for your module, you have two options: (1) let Drush patch your module using a public patch file or (2) use a local patch file.

Drush is fully capable of patching your code as instructed in the make file. All it needs is the path to the patch file. The path can be a full URL or a path to a local file relative to the make file itself (e.g., Listings 11 and 12). When you build a codebase with patches in the make file, Drush will create a `PATCHES.txt` file for each patched module, which can be found next to the project .info file and Drupal core installation root.

**LISTING 11: INI – Patching a Module**

```ini
;  INI format, patching the module, both public and local patch files
projects[field_group][version] = "1.0-rc4"
; Explain briefly why this patch is needed.
; Get the file path from the drupal.org issue in question and
; use issue number as patch array key.
projects[field_group][patch][2761159] = https://www.drupal.org/files/issues/field_group-empty_group_nonnumeric_index-2761159-2-D8.patch
; Use file in patches-folder (relative to .make-file location)
projects[field_group][patch][other_fix] = patches/field_group-fix_it.patch
```


**LISTING 12: YAML – Patching a Module**

```yaml
# Yaml format, patching the module, both public and local patch files # Note the indentation, we're still inside "projects" array.
field_group:
# Get the latest release within 8.x-1 -version:
# version: 1
# Get the latest development version:
# version: 1.x
# Get the specified version even when it is not the latest release
version: '1.0-rc4'
patch:
  # Explain briefly why this patch is needed.
  # Get the file path from the drupal.org issue in question and
  # use issue number as patch array key.
  2761159: 'https://www.drupal.org/files/issues/field_group-empty_group_nonnumeric_index-2761159-2-D8.patch'
  
  # Use file in patches-folder (relative to .make-file location)
  other_fix: 'patches/field_group-fix_it.patch'
```
In short, out of all possible solutions, do not simply fix the code on your server. Extract the changes regardless of how you’ve made them, patch your code with a Drush make file, and preferably use a public patch file from the Drupal. org issue queue.
