# Perttu's blog posts warehouse

Blog posts published via https://dev.to/rpsu.

## How to

Manage posts with ´devto-cli´, all instructions [in the repository](https://github.com/sinedied/devto-cli#usage). 

In short (after `npm install -g @sinedied/devto-cli`):

* `dev n posts/2022/01-some-post-file-name.md` creates new post file template
* edit the file, including 
* `dev p [--verbose] [--dry-run] [posts/2022/01-some-post-file-name.md]` publishes (single) posts

Make sure your local `.env` file contains both `DEVTO_TOKEN` to allow publishing and `DEVTO_REPO` for linking assets to correct URL (this repository).

Based on Github marketplace action ["Publish to dev.to"](https://github.com/marketplace/actions/publish-to-dev-to).
