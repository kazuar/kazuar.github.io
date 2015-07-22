---
layout: post
title: Faster grep
comments: true
---

If you want to run a faster grep, use the "LC_ALL" flag and set it to "C".

For example: 

{% highlight bash %}
LC_ALL=C grep -in <text> <file_name>
{% endhighlight %}

"LC_ALL=C" will tell grep to search using ASCII locale instead of UTF.

You can also do that on compressed files by running it with zgrep.

For example: 

{% highlight bash %}
LC_ALL=C zgrep -in <text> <file_name>
{% endhighlight %}
