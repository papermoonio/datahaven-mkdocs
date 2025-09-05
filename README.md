# mkdocs-template-light-dark

This repository is a bare-bones template for quickly setting up a new documentation site with the mkdocs-material theme.

This template supports dark and light themes. If the project is only using a single theme, please use the [mkdocs-standard-template](https://github.com/papermoonio/mkdocs-template-standard) template.

## Onboarding Checklist

If you're onboarding a new project, make sure to go through this checklist to customize the template:

- [ ] Add the project's logo and favicon to the `material-overrides/assets/images/` directory and name them `XXX-favicon.png`, `XXX-logo.png`, and `XXX-logo-dark.png`. If the images aren't rendering, make sure the file names match the ones defined in the `mkdocs.yml` file
- [ ] Replace all instances of `XXX` with the project's name, including the `XXX.css` file name
- [ ] Update the `XXX.css` file to use the project's branding
    - [ ] Add brand colors
    - [ ] Style code snippets using brand colors
    - [ ] Add brand fonts. If the desired fonts are available through Google Fonts, uncomment the `font` config in the `mkdocs.yml` file and add the font names. For any other type of font, uncomment the `extra_css` config in the `mkdocs.yml` file and add the font to `material-overrides/assets/stylesheets/extra.css`
- [ ] Add the desired social icons and links to the `social` config in the `mkdocs.yml` file
- [ ] Check the copyright information to make sure that it is correct
- [ ] Add the project's Google Analytics property under the `analytics` config in the `mkdocs.yml` file. This should be a unique property specifically for the docs site
- [ ] Check to make sure the GitHub repo URL in the feedback form under the `feedback` config in the `mkdocs.yml` file is correct
- [ ] Create the home page. This template comes with a basic design that can be used, or you can create a custom home page if the project has requested a custom one
- [ ] Create a `docs` repository to hold all of the documentation pages. Make sure that whatever the docs repository is named matches the `docs_dir` config in the `mkdocs.yml` file
- [ ] Update the `.gitignore` file so that the docs repository is ignored
- [ ] Update the `README.md` file to include information on the project, instructions on local development, and any other related information

## About

This repository contains the mkdocs configuration files, theme overrides, and CSS changes.

- [Mkdocs](https://www.mkdocs.org/)
- [Material for Mkdocs](https://squidfunk.github.io/mkdocs-material/)

The actual content is stored in the docs repo and is pulled in during the build process.

- [XXX Docs](https://github.com/XXX/XXX-docs)

## Prerequisites

To get started, you need to have [mkdocs](https://www.mkdocs.org/) installed. All dependencies can be installed with a single command. You can run:

```bash
pip install -r requirements.txt
```

## Getting started

With the dependencies installed, let's proceed to clone the necessary repos. For everything to work correctly, the file structure needs to be the following:

```text
XXX-mkdocs
|--- /material-overrides/ (folder)
|--- /XXX-docs/ (repository)
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
