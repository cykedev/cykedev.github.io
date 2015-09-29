---
layout: 	post
title:  	"icloud und caldav"
date:   	2013-10-18
tags:		apple icloud caldav
---
evtl. kann es mal notwendig sein, dirket auf einen icloud kalender per caldav zuzugreifen, z.b. wenn man ihn in thunderbird lightning einbinden möchte/muss.

um an die entsprechenden daten zu kommen braucht man auf jeden fall einen mac. man muss unter `~/Library/Calendars` jetzt den richtigen kalender finden, welcher das ist ist eine gute frage. sieht alles recht kryptisch aus, muss man halt durchprobieren ;-)

in irgendeinem dieser krypischen verzeichnisse ist man jetzt richtig und findet in der `Info.plist` einen text der in etwa so aussieht (die beschreibung ist toll, nicht wahr?)

{% highlight xml %}
<key>CalendarPath</key>
<string>/123456789/calendars/ABCDEF01-2345-6789-ABCD-EF0123456789/</string>
{% endhighlight %}
    
das ist der pfad des entsprechenden kalenders. zusammen mit der basis url `https://p01-caldav.icloud.com:443` kann man sich jetzt die caldav url zusammen bauen, die dann z.b. so aussehen könnte:

    https://p01-caldav.icloud.com:443/123456789/calendars/ABCDEF01-2345-6789-ABCD-EF0123456789/
    
das wars in sehr kurzer form.
