---
layout: 	   post
title:  	   "github pages und jekyll"
meta:		   blog, github, jekyll
---

ich hab mal was neues ausprobiert und meinen blog neu aufgesetzt. zuletzt hatte ich ja ghost im einsatz und war soweit eigentlich ganz zufrieden, hatte aber immer mal wieder probleme mit updates.

darum - öfters mal was neues - probiere ich jetzt mal github pages mit jekyll.

das funktioniert inital erst mal recht einfach, jekyll installieren, neue initiale seite erstellen und direkt mal generieren lassen.

{% highlight bash %}
gem install jekyll
jekyll new .
jekyll serve --watch
{% endhighlight %}

so kann ich unter [http://localhost:4000](http://localhost:4000) eine vorschau der seite sehen.

was man alles so mit jekyll machen kann ist recht gut in der [doku](http://jekyllrb.com/docs/home/) beschrieben.

das ganze kann dann als git repository zu github wandern und direkt von dort ausgeliefert werden. details dazu findet man [hier](https://pages.github.com).

github pagen bieten zudem die möglichkeit eine eigene domain zu verwenden. [hier](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/) die doku.

zudem hat mir der folgende artikel sehr geholfen: [get started with github pages](https://24ways.org/2013/get-started-with-github-pages/)
