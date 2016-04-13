---
layout: post
title:  "My R Markdown Workflow for Blogging"
categories: [tips]
tags: [knitr, servr, jekyll, blogging]
---

After getting started my blog using Jekyll[^1], I started to wonder how tu setup a process to go from R markdown to blog post seamlessly. I came across the knitr-jekyll[^2] project, which had a similar purpose. The suggested process from knitr-jekyll project is detailed [here]({% post_url 2014-09-28-jekyll-with-knitr%}).

I had slighly different requirements, so I had to make a few modifications. Some of these issues and their solutions are discussed by others[^3] before.

First of all, I prefer to generate the first round of html (or other outputs) using the default format of R markdown and check the results in RStudio. Secondly, I would like the original markdown files reside by their respective projects.

Below, is what I could come up with:

### One-time steps

1. Copy Build.R file from knitr-jekyll project to my github.io project.
2. Create _source directory in the github.io project.

### Regular Steps

1. Copy finalized Rmd file from the original project to _source directory.
2. Front Matter (The header of the markdown file) modifications:
    * Remove output tag
    * Change data format to appropriate format (YYYY-MM-DD, in my case)
    * Add layout tag
    * Add categories, tags, if you like
3. run `servr::jekyll()` command. I know it's somtimes tricky to get it work.
4. Step 3 builds an .md file from your .Rmd and puts it in the _posts directory.
5. Check if the new post locally. (Stop your RStudio Jekyll process, if needed and run your local Jekyll to test)
6. Commit and push to Github.io if eveything looks OK.

## References

[^1]: [Jekyll](https://jekyllrb.com)
[^2]: [knitr-jekyll](https://github.com/yihui/knitr-jekyll), a project by @yihui
[^3]: [Nicole White](http://nicolewhite.github.io/2015/02/07/r-blogging-with-rmarkdown-knitr-jekyll.html) and [Brendan Rocks](http://www.r-bloggers.com/blogging-with-rmarkdown-knitr-and-jekyll/)