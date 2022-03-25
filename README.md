# Personal Website

## Stack

This website uses Hugo (a static site generator), and the theme "hello-friend-ng" (albeit with some of my modifications).

## Hosting

The hosting is done on GitHub for free.

In the pages section of the repo settings, it has been configured that the site is deployed from the `gh-pages` branch.

On a push to the master branch, the GitHub action runs the workflow `github/workflows/gh-pages.yml`, which builds the website, and pushes the `public` folder to the `gh-pages` branch.
