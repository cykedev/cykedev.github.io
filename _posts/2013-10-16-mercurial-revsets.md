---
layout: 	post
title:  	"mercurial revsets"
meta:		hg
---

kann man mal brauchen: *letzten merge zwischen zwei branches ermitteln*

    max(children(ancestor("branch-a", "branch-b")) and merge())

