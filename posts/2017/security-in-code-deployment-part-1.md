---
title: Drush make file
description: Faithfully reproducing a codebase for website recovery
tags: 'drupal drush'
cover_image: ''
canonical_url: null
published: true
---

**Part 1/5.** The article was initially published in [Drupal Watchdog 7.01](https://shop.linuxnewmedia.com/us/magazines/drupal-watchdog/eh35014.html), spring 2017.

**Code examples for this article**: [https://github.com/rpsu/dwd-security-in-code-deployment](https://github.com/rpsu/dwd-security-in-code-deployment)

---

Considering the full life cycle of any website, REST service, or similar utility, development of secure code covers only part of the whole. A properly functioning backup process with tested recovery, along with hardened settings for the web server and database applications running the website, is key to running successful businesses on the web, be they profit or nonprofit. Add some load balancing, architecture with multiple servers, and strict firewall rules to the mix, and you’re good to go, even with bigger sites.

What is far too often ignored is the process of managing the codebase. We’ve all heard stories or seen practices in which nobody knows how exactly to rebuild the production server codebase, requiring it to be cloned from the production site.

You must be able to reproduce the codebase from any given point in time or release version, including each instance you need to have of the website, from local development up to the production site. The codebase must be predictable to the last line of code for each release of your website, and you have a few different ways to get there.

## Full Codebase in Version Control

One way to control your website codebase is to store it entirely in a version control system [such as Git](https://git-scm.com/). Although it is a pretty bulletproof way to manage the codebase, it might require quite a bit of manual work. Drupal core and contributed (contrib) modules and themes and external libraries are already stored elsewhere and are updated regularly, so each of these updates must be imported and committed manually. Additionally, any custom code changes need to be re-implemented during this process. Largely because of these issues, it is often considered unnecessary to store code that is hosted elsewhere in the project repository.

Fortunately, this problem has already been addressed: There is no need to store any publicly available code for Drupal sites in the project repository. With the help of a couple of tools, you can offload the manual processing of each update to your contributed code or libraries.

## Drush Make File to the Rescue

A predictable way to build your Drupal codebase is to use [Drush](http://www.drush.org/) and its ability to take a simple text file and create [a full Drupal codebase with core and contrib modules and themes and external libraries](https://drushcommands.com/). This simple text file is called a "make file", because it usually has the file extension `.make`.
The make file comes in [two flavors](https://github.com/rpsu/dwd-security-in-code-deployment): in the older [plain text (INI) format](https://www.drupal.org/docs/develop/packaging-a-distribution/example-drupalorg-make-file) and in the [newer YAML format](http://www.yaml.org/start.html). Whereas the old-style INI make files have fairly loose format requirements (Listing 1), YAML indenting is important and dictates how your data is interpreted (Listing 2). Both are human readable, but YAML might be easier to scan. If you are working with Drupal 8, the YAML format of Drush make files is recommended; however, INI-style files will also work.

**LISTING 1: INI Make File**

```ini
; Older format is in INI format  
; Line with comment starts with a semicolon  
; Attributes are declared like this:  
; attribute = value  
; Attribute values with spaces and float-like string values must be quoted: ; attribute = "long value with spaces"

; Core must be defined.
core = 8.x
; (Drush) api must be defined, always to '2'.
api = 2

; Here you see the format of an array in a .make-file. Text enclosed
; in brackets are array keys, and each set to the right of the last is
; a layer deeper in the array. Note that the root array element is
; not enclosed in brackets.
; root_element[first_key][next_key] = value

; Define the Drupal core version you need to the last digit projects[drupal][version] = "8.2.3"

; Then define contrib modules etc.
; Define OAuth version to 8.x-2.0, do not include Drupal core version
; This is the same as
; projects[views][version] = "2.0"
projects[oauth] = "2.0"

; Define the where to put the module (should be inside sites/all/modules)
projects[commerce][version] = "2.0-beta3"
projects[commerce][subdir] = "commerce"

; You may also provide default values to all projects
; Now OAuth and other modules with no subdir definition will be placed in
sites/all/modules/contrib defaults[projects][subdir] = "contrib"
```

**LISTING 2: YAML Make File**

```yaml
# YAML format
# Line with comment starts with a hash
#
# Attributes are declared like this:
# attribute: value
# Attribute values with spaces and float-like string values must be quoted:
# attribute: 'long value'

# Core must be defined.
core: 8.x
# (Drush) api must be defined, always to '2'.
# Version numbers that can be interpreted as floats must be quoted.
api: '2'

# Here you see the format of an array in a .make.yml-file. Indented text
# "inside" a parent are array keys, and each set to the right of the last is
# a layer deeper in the array.
# Define the exact Drupal core version you need:
projects:
  drupal:
    # Version numbers that can't be interpreted as floats do not need quotes:
    version: 8.2.3
  # Then define contrib modules etc.
  # Note that we're still inside 'projects' -array/object, hence the indenting:
  oauth:
    version: '2.0'
  commerce:
    subdir: commerce
    version: 2.0-beta3

# You may also provide default values to all projects
# Now OAuth and other modules with no subdir definition will be placed 
# in sites/all/modules/contrib
defaults:
  projects:
    subdir: 'contrib'
```
## Use Resource Versions

To make the codebase building process predictable to the last line of code, it is important to declare all versions, including core and contrib modules and themes and external libraries. Drupal core and module version declarations are explained in Listings 1 and 2, but you can set the contrib versions on the Git commit level, too (Listings 3 and 4).
You need to specify the source URL and some other information to download external libraries in the make file (Listing 5 and 6).

**LISTING 3: INI Git Commit**

```ini
; INI format, get a specific version of ctools module
; Download method defaults to git, so this is not necessary
; projects[ctools][download][type] = "git"
; Optionally provide git branch so Drush can write that to .info file
projects[ctools][download][branch] = "8.x-3.x"
; Git commit hash
projects[ctools][download][revision] = 1fe3649
```
    
**LISTING 4: YAML Git Commit**

```yaml
# YAML format, get a specific version of ctools module
# Note continuous, we're still inside "projects" array.
  ctools:
    download:
      # Download method defaults to git, so this is not necessary
      # type: git
      # Optionally provide git branch so Drush can write that to
      # .info file
      branch: 8.x-3.X
      # Git commit hash
      revision: 1fe3649
```
**LISTING 5: INI – Source URL**

```ini
; INI format, get a specific version of ckeditor library
; ckeditor (library) will be extracted
; Drupal 8: libraries/ckeditor
; Drupal 7: sites/all/libraries/ckeditor
libraries[ckeditor][download][type] = file
libraries[ckeditor][download][request_type] = get
libraries[ckeditor][download][url] = "https://download.cksource.com/CKEditor%20for%20Drupal/edit/ckeditor_4.4.6_edit.zip"
libraries[ckeditor][directory_name] = "ckeditor"
libraries[ckeditor][type] = "library"
```

**LISTING 6: YAML – Source URL**

```yaml
# YAML format, get a specific version of ckeditor module # ckeditor (library) will be extracted
# Drupal 8: libraries/ckeditor
# Drupal 7: sites/all/libraries/ckeditor
libraries: ckeditor:
  download:
    type: file
    request_type: get
    url: 'https://download.cksource.com/CKEditor%20for%20Drupal/edit/ckeditor_4.4.6_edit.zip'
  directory_name: ckeditor
  type: library
```

The library you want to use might not provide releases or files for download. If the code repository isn’t public or you are unable to follow the updates, there is not much to be done besides store that code to your project’s version control – or at least verify downloadable file checksums to prevent unpleasant surprises (Listing 7 and 8). (This project repository is in fact publicly available at GitHub, and you could, instead of relying on a file hash, use a specific commit.)

**LISTING 7: INI – Verify Checksum**

```ini
; INI format, make sure file has not changed verifying file hash
libraries[isotope][download][type] = get
libraries[isotope][download][url] = http://isotope.metafizzy.co/jquery.isotope.min.js
libraries[isotope][download][md5] = ac35f7863494170cddea8ddb144675d5
libraries[isotope][directory_name] = jquery.isotope
libraries[isotope][type] = library
```

**LISTING 8: YAML – Verify Checksum**

```yaml
# Yaml format, make sure file has not changed verifying file hash
libraries:
  isotope:
    download:
      type: get
      url: 'http://isotope.metafizzy.co/jquery.isotope.min.js'
      md5: ac35f7863494170cddea8ddb144675d5
    directory_name: jquery.isotope
    type: library
```

Failing to provide versions or at least to verify the retrieved file checksums, you can’t be sure you have the same codebase when compared with other builds. It is time dependent in the sense that it can’t be predicted when a module will have an update available and will replace some previously downloaded older version of itself. In practice, this might mean one of your colleagues is facing an issue, bug, or even vulnerability that has already been fixed in your codebase – simply because you happened to build your local this week, and your colleague did the same the previous week (Listing 9 and 10).

**LISTING 9: INI – Get Specific Version**

```ini
; INI format
; Get the latest release within 8.x-1 -version:
; projects[role_expose] = 1
; Get the latest development version:
; projects[role_expose] = 1.x
; Get the specified version even when it is not the latest release:
projects[role_expose] = 1.0
```
**LISTING 10: YAML – Get Specific Version**

```yaml
# Yaml format
# Note the indentation, we're still inside "projects" array.
role_expose:
  # Get the latest release within 8.x-1 -version:
  # version: 1
  # Get the latest development version:
  # version: 1.x
  # Get the specified version even when it is not the latest
  # release
  version: '1.0'
```
In short: Always use fixed versions and specific commits, or at least verify checksums if none are available.
