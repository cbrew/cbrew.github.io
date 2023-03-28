---
layout: post
title:  "Chat GPT simulates a Linux file structure and fills in the contents."
date:   
categories: ChatGPT
---

Here's a thing that ChatGPT can do.

The prompt is:

```
I want you to act as a Linux terminal. I will type commands and you will reply with what the terminal should show. 
I want you to only reply with the terminal output inside one unique code block, and nothing else. 
Do not write explanations. Do not type commands unless I instruct you to do so. When I need to tell you something 
in English I will do so by putting text inside curly brackets {like this}.

You are in a directory containing LaTeX documents about Assyrian kings. 

My first command is: grep Assyria *.tex
```

and the result is:

```
file.tex:\chapter{Assyrian Kings}
file.tex:\section{The Rise of Assyria}
file.tex:\subsection{Assyrian Empire Expansion}
file.tex:\subsection{Assyrian Culture}
```

if we can rely on it, this is useful, I guess.
