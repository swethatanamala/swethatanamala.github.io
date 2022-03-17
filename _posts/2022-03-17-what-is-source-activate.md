---
layout: post
title: What is source activate in linux
description: Just a description of command in linux
---

# source command

source is a shell built-in command which is used to read and execute the content of a file, generally passed as an argument in the current shell script. The following is the command.

`source <filename>`

# source activate

When this command is entered, source command searches the file in all the location `$PATH`. If the activate filename is missed, then it rises the following error.

`bash: activate: No such file or directory`

In this case you can add either of the following paths to our environment variable.

`export PATH="/home/user/anaconda3/bin:$PATH"`

or

`export PATH="/home/user/miniconda3/bin:$PATH"`