---
layout: post
title: Cyrillic fonts in LaTeX
description: Enable cyrillic fonts in LaTeX.
---

There are two common ways to install TeX on a Mac: full TeX Live (2.4 Gb) and BasicTeX (95 Mb), the second distribution is 24 times smaller. It is a subset of full TeX Live and supports most standard TeX typesetting. Cyrillic fonts package isn't included, but it's easy to install them.

To view all installed collections type `$ tlmgr info collections` in your terminal:

![LaTeX collections](/images/2014/07/latex-collections.png)

Two collections are recommended for cyrillic documents: `collection-fontsrecommended` (collection of widely used fonts) and `collection-langcyrillic` for cyrillc typesetting.

Now creating a document with following code will result working russian characters in output file.

```latex
\documentclass[12pt,a4paper]{article}
\usepackage[utf8]{inputenc}
\usepackage[T1]{fontenc}
\usepackage[russian]{babel}

\begin{document}
Привет!
\end{document}
```

Compile with XeLaTex and enjoy!
