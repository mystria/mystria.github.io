---
layout: post
title:  "Resolving a Sass compile error"
date:   2017-06-13 12:00:00 +0900
categories: Sass Compass Ruby ErrorResolving
comments: true
---
Here is an error when compiling sass by compass.  
  
{% highlight text %}
error sass/your.project.style.scss (C:/Ruby187/lib/ruby/1.8/tempfile.rb:52:in
`initialize': cannot generate tempfile`../.sass-cache/fa2e60b2f342a86305835cc4aed
21511e4d6adde/your.project.component.scssc20170412-8012-328y7d-9')
{% endhighlight %}
  
The reason is long folder name of `.sass-cache` (Maybe over 255 characters).  
You should shorten the compiling folder's full path.  
Put below code to `config.rb` file.  
{% highlight ruby %}
cache_path = "c:/temp/sass"
{% endhighlight %}  
Then the error will be resolved because the cache folder was changed.  

This problem had be reported to Git of Compass but it still open.  
[Compass/compass][compass-git]  

[compass-git]: https://github.com/Compass/compass/issues/1791