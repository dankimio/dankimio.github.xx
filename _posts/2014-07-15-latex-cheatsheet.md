---
layout: post
title: LaTeX cheatsheet
categories: blog
description: Save time with these tips.
---

LaTeX is complicated. There are thousands of packages and there are million ways to do the same thing. I encountered a lot of problems while making my report in LaTeX. I wrote this cheatsheet to make my life a bit easier.

Here's the basic stuff:

* [Enabling cyrillic fonts in LaTeX](/cyrillic-fonts-in-latex/)
* A4, twoside paper: `\documentclass[12pt,a4paper,twoside]{article}`
* Page margins: `\usepackage[top=1in,left=0.85in,right=0.85in]{geometry}`
* Easy way to write LaTeX (preserving case): `\LaTeX`
* `\newpage`. Guess what it does

Sometimes you need to start numbering the pages from the second page. All you need to do is to disable page numbering on your first page and enable it later.

1. On your title page: `\pagenumbering{gobble}`
2. Page where numbering starts: `\pagenumbering{arabic}`

Indentation, margins, alignment:

* Vertical spacing: `\vspace{1cm}`. Will move your content 1 cm below
* Horizonal spacing: `\hspace{-1cm}`. You can use any unit (cm, mm, in, pt, em). Negative values allowed.
* Disable force indentation: `\noindent`
* Wrap code in `\begin{center} \end{center}` to make it centered

Lists and enumeration:

* `itemize` is unordered list, `enumerate` is ordered list
* Each item starts with `\item`

Example:
{% highlight latex %}
\begin{enumerate}
    \item First
    \item Second
\end{enumerate}
{% endhighlight %}

PDFs and graphics

* Use `pdfpages` package to include PDFs. Syntax: `\includepdf[pages=1-6]{path.pdf}`.
* To include graphics use `\includegraphics[width=5.6cm]{image.png}`

Math:

* Wrap all math in dollars: `$ math here $`
* LaTeX also supports multiline math code: `$$ multiline math here $$`. It will render in a separate line, centered
* Subscripts are made using underscore: `$ x_i $`
* If you want a complex expression in your subscript wrap it in curly braces: `$ x_{a,b} $`
* The same applies to superscript (powers): `$ a^{b+c} $`
* You can combine both: `$ a_{b+c}^{d+e} $`
* Greek letters are enabled in math mode: `$ \Alpha \alpha $`. The one that starts with a capital letter will make uppercase alpha

References:

* [Best online LaTeX tutorial I've found](http://en.wikibooks.org/wiki/LaTeX)
* [TeXShop for OS X is good. Didn't find any better](http://pages.uoregon.edu/koch/texshop/)
* [Use smaller MacTeX packages](https://tug.org/mactex/morepackages.html). To install a package type `sudo tlmgr install [package]`
