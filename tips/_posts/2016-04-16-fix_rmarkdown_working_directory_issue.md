---
layout: post
title:  "Setting Working Directory Inside R Markdown (.Rmd) Files"
categories: [tips]
tags: [rmarkdown, rmd, setwd, knitr]
description:
---
### Problem

Rmd files use the directory they reside in as the base directory. `setwd()` does not work properly inside chunks and should not be used because of reproducibility issues (the other people may not have the same directory structure as you)[^1]. If your .Rmd file and data or other files are not in the same directory, you're probably having hard time figuring out how to make it work.

### Solution
@yihui has added root.dir option [^2]. You can set this option in one of your chunks and all the paths in the other chunks will be evaluated relative to this directory. This is how to do it:
 
{% highlight r%}
knitr::opts_knit$set(root.dir = 'relative_path_to_root_from_Rmd' )
{% endhighlight %}

### References

[^1]: [Yihui (Package author) comments](https://groups.google.com/forum/#!topic/knitr/knM0VWoexT0)
[^2]: [Knitr project on GitHub](https://github.com/yihui/knitr/issues/277)

