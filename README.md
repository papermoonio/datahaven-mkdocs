# DataHaven Documentation Site (MkDocs Framework and Material Theme)

https://docs.datahaven.xyz

## About

This repository contains the mkdocs configuration files, theme overrides, and CSS changes.

- [Mkdocs](https://www.mkdocs.org/)
- [Material for Mkdocs](https://squidfunk.github.io/mkdocs-material/)

The actual content is stored in the docs repo and is pulled in during the build process.

- [DataHaven Docs](https://github.com/datahaven-xyz/datahaven-docs)

## Prerequisites

To get started, you need to have [mkdocs](https://www.mkdocs.org/) installed. All dependencies can be installed with a single command. You can run:

```bash
pip install -r requirements.txt
```

## Getting started

With the dependencies installed, let's proceed to clone the necessary repos. For everything to work correctly, the file structure needs to be the following:

```text
datahaven-mkdocs
|--- /material-overrides/ (folder)
|--- /datahaven-docs/ (repository)
|--- mkdocs.yml
```

To preview changes, build the site by running:

```bash
mkdocs serve
```

After a successful build, the site should be available at `http://127.0.0.1:8000`

## Editing Theme Files

If you're editing any of the files in the `material-overrides` directory, you can run the following command to watch for these changes and render them automatically:

```bash
mkdocs serve --watch-theme
```

Otherwise, you'll need to stop the server (`control + C`) and restart it (`mkdocs serve`) to see the changes.

## Ignoring Excluded Docs Output

Running `mkdocs serve` displays the excluded documents in your terminal. To prevent this effect, you can run:

```bash
mkdocs serve --clean
```

## Disable the Git Dates Plugin

The `git-revision-date-localized` plugin pulls the date of the last git modification of a page. When developing locally, this can slow down your development process, as the plugin checks for the latest dates on every page each time a change is made. To avoid this, you can change your start-up command to turn off the plugin by running:

```bash
export ENABLED_GIT_REVISION_DATE=false
mkdocs serve
```
