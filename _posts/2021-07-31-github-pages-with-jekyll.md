---
layout: default
title: "GitHub pages with Jekyll"
description: "Creating this site with GitHub pages and Jekyll"
date: 2021-07-30 12:00:00 -0000
categories: projects b-hub.github.io
tags: dev jekyll how-to
excerpt_separator: <!--excerpt-->
---

In this post I will take you through the steps I used for setting up this website using github pages and jekyll. There is plenty of documentation out there already, so I will give more of an overview of the process and how to resolve some issues I came across. 

<!--excerpt-->

For more detailed steps, please see the linked documentation.

## Quick and Dirty

You can setup a GitHub site with Jekyll very quickly through GitHub's GUI:
- Create a new public repository with a README file
    - It can be private if your account supports private GitHub pages
- Publish your GitHub page by going to repository Settings -> Pages
- Additionally, you can set one of the supported themes by using the 'Theme Chooser'.

Completing all the above steps will result in your repository having an updated README file and a new _config.yml. The _config.yml simply contains your chosen theme (for now).

You can view the site by going to Environments -> github-pages -> View Deployment

### Explanation

By creating a GitHub page, GitHub will automatically build your site using Jekyll, behind the scenes. [Jekyll](https://jekyllrb.com/) is a program which can build static pages from source files.

The entry point for the site is index.md or index.html. If there is no index file, then the README file is used. If neither exist then the site will return a 404 page.

Only .md and .html file types are supported.

## Current Setup

You can follow the detailed guide from GitHub [here](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll). 

The source code for the site can be found [here](https://github.com/b-hub/b-hub.github.io).

### Running Locally

If you want to run the site locally, you may come across the same problem as me. Following the [documentation](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll) on GitHub, make sure you install Ruby version 2.7 (not >= 3.0). As when you finally come to build and serve the site locally it will error. The related discussion can be found on talk.jekyllrb.com [here](https://talk.jekyllrb.com/t/load-error-cannot-load-such-file-webrick/5417/2). Note, this issue may be resolved when you come to it, it which case use the latest version of Ruby.


## Feature Overview

```
---
layout: default
title: "Ben Magistris"
description: "Software Developer"
---
```

Jekyll page variables are enclosed in `---` as shown above.

Custom layouts are defined in _layouts/layout_name.html and can use variables defined in the page.

[Posts](https://jekyllrb.com/docs/posts/) are listed in the _posts folder and can be referenced on a page by using `site.posts`

You can write html and JavaScript as normal, even inside .md files. This includes embedding content with iframes.

Jekyll uses [Liquid](https://shopify.github.io/liquid/) as the templating language. GitHub pages (as of writing) only supports 4.0.3, so some of the features in v5 are unavailable.

## Useful links

- [Setting up a GitHub Pages site with Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll)
- [GitHub Pages Dependencies](https://pages.github.com/versions/)
- [Jekyll Docs](https://jekyllrb.com/docs/)
- [Liquid Docs](https://shopify.github.io/liquid/)
- [RubyInstaller for Windows](https://rubyinstaller.org/)
- [GitHub Jekyll Themes](https://github.com/b-hub/b-hub.github.io/settings/pages/themes?source=main&source_dir=%2F)