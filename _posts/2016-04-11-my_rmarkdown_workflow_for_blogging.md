---
layout: post
title:  "My R Markdown Workflow for Blogging"
categories: [tips]
tags: [Blogging, Jekyll, knitr, Markdown, RMarkdown, Rmd, R, servr]
description: In this post, I explain how to make your R markdown (.Rmd files) work as your blog post using knitr, servr, jekyll, and some tips and tricks.
---

After getting started my blog using Jekyll[^1], I started to wonder how tu setup a process to go from R markdown to blog post seamlessly. I came across the knitr-jekyll[^2] project, which had a similar purpose. The suggested process from knitr-jekyll project is detailed [here](http://yihui.name/knitr-jekyll/2014/09/jekyll-with-knitr.html){:target="_blank"}.

I had slighly different requirements, so I had to make a few modifications. Some of these issues and their solutions are discussed by others[^3] before.

First of all, I prefer to generate the first round of html (or other outputs) using the default format of R markdown and check the results in RStudio. Secondly, I would like the original markdown files reside by their respective projects.

Below, is what I could come up with:

### One-time steps

1. Copy Build.R file from knitr-jekyll project to my github.io project.
2. Create _source directory in the github.io project.
3. The syntax highlighting was not working as I'd like to out-of-the-box. My Jekyll setup uses Rouge as the syntax highlighter and Rouge is compatible with Pygments stylesheets. So, you can pick your favorite style from [richleland's pygment-css](http://richleland.github.io/pygments-css/) project, download it to your css folder and add it to your pages. One more thing, these css files are designed with `codehilite` tag, which should be replaced by your css style for highlight, `highlight` in my case.

### Regular Steps

1. Copy finalized Rmd file from the original project to _source directory.
2. Make sure you are using Jekyll's file naming conventions.
2. Front Matter (The header of the markdown file) modifications:
    * Remove `output` tag
    * Change data format to appropriate format (YYYY-MM-DD, in my case)
    * Add `layout` tag
    * Add `categories`, `tags`, etc. if you like
3. Run `servr::jekyll()` command. I know it's sometimes tricky to get it to work.
4. The last step builds an .md file from your .Rmd and puts it in the _posts directory.
5. Check if the new post works locally. (Stop your RStudio Jekyll process, if needed and run your local Jekyll to test)
6. Commit and push to Github.io if everything looks OK.

## References

[^1]: [Jekyll](https://jekyllrb.com)
[^2]: [knitr-jekyll](https://github.com/yihui/knitr-jekyll), a project by @yihui
[^3]: [Nicole White](http://nicolewhite.github.io/2015/02/07/r-blogging-with-rmarkdown-knitr-jekyll.html) and [Brendan Rocks](http://www.r-bloggers.com/blogging-with-rmarkdown-knitr-and-jekyll/)
