---
layout: 	post
title:  	"mercurial revsets"
date:   	2013-10-16
meta:		hg
---

kann man mal brauchen: *letzten merge zwischen zwei branches ermitteln*

    max(children(ancestor("branch-a", "branch-b")) and merge())
