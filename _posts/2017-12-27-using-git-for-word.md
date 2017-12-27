---
title: "Using Git for Word"
author: "Vivek Kalyan"
date: "27/12/2017"
---

How to use Git to track Microsoft Word documents.

Sometimes Markdown just does not provide the features I need (often with formatting). During those times, I turn to Microsoft Word. But for all its benefits, it uses its own proprietary extension which makes it hard to implement version control. While looking for solutions, I came across [this article](http://blog.martinfenner.org/2014/08/25/using-microsoft-word-with-git/). Adapting that, I will share what I believe to be a more reproducible way of keeping track of changes in Word using git.

## Install Pandoc

Pandoc is an utility that allows conversion between different markup formats. I use it to covert `.docx` files to markdown, which is then used by git for version control. On Mac, we can install it simply using brew.

`brew install pandoc`

[Check the documentation for other OSs](http://pandoc.org/installing.html). But the following steps assume you are using some sort of UNIX system.

## Configure Git

We configure git to use pandoc whenever it sees a file with `.docx`. Add the following lines to these files (create them if they don't exist)

```
# ~/.gitconfig
[core]
  attributesfile = ~/.gitattributes

[diff "pandoc"]
  textconv=pandoc --to=markdown
  prompt = false
```

```
# ~/.gitattributes
*.docx diff=pandoc
```

The article tells us to commit the `.gitattributes` file for each repository we want to track, but I believe this is a better approach. If `.gitattributes` is committed to the repository and a collaborator comes along and pulls it without first installing pandoc, he will run into errors. And since it needs to be installed anyway, might as install it globally to save the hassle for future projects.

And that's it! 

Note: The article also mentions about comparing revisions by word. I left that out because I do it differently but I don't think either method is better or worse. Refer to my [dotfiles](https://github.com/bowsplinter/dotfiles/blob/master/git/.gitconfig) for how I set mine up. 
