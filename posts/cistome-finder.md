---
title: "My First Bioinformatics Web Application"
date: '2012-12-23'
description:
categories: Bioinformatics
tags: [CistromeFinder]

---

<img src="{{urls.media}}/cistromefinder.png" align="right">
This tool is called [CistromeFinder](http://cistrome.org/finder). It is made to help biologists to check binding sites around a given gene. Following is an introduction from the [blog](http://www.longwoodgenomics.org/) of my adviser, [Xiaole Shirley Liu](http://www.stat.harvard.edu/faculty_page.php?page=xsliu.html).



>We have been processing all the publicly available ChIP-seq and DNase-seq data in human and mouse. Curious about whether factor X has binding sites near gene Y? CistromeFinder can help you evaluate specific ChIP-seq/DNase-seq dataset and give you the answer to this question.


This is my first Web application. As I was only familar with Python at that time, I chose [Pyramid](http://docs.pylonsproject.org/projects/pyramid/) as web framework, [SQLAlchemy](http://www.sqlalchemy.org/) as ORM and [jQuery UI](http://jqueryui.com/) as front-end framework. As time passed, I found the jQuery UI codes really suffering to maintain, so I discarded all jQuery UI codes and rewrite UI with a much more friendly framework, which is called [HTML KickStart](http://www.99lime.com/).

Two things I've learned:

- For UI, use a high-level UI framework. You will be frustrated to make the UI work with jQueryUI, unless you're very familiar with javascript as well as jQuery. Try HTML KickStart or [Bootstrap](http://twitter.github.com/bootstrap/).

- For web framework, I'm tired of Pyramid + SQLAlchemy. This feeling becomes stronger after I tried [Rails](http://rubyonrails.org/). At first, I chose Pyramid and SQLAlchemy for loose coupling, which is claimed as a feature superior to [Django](https://www.djangoproject.com/). However, it's not as good as it might sound. [One of the comments](http://www.quora.com/How-do-Rails-vs-Flask-vs-Pyramids-compare-as-web-frameworks) is forthright: *The loose coupling means that it's going to just take longer to get to a mature point.*  Even worse is that the documents of both Pyramid and SQLAlchemy are abstract and there're not many good tutorials for them. If I could rewrite the back-end part, I might try Rails or Flask.

The submission of the paper is still on the way. Hope it could be finished before January of next year.

Merry Christmas and Happy New Year! :-)


