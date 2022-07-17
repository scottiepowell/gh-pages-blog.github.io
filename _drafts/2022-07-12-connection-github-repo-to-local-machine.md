---
layout: post
read_time: true
show_date: true
title:  Road to CI/CD: Connecting a gitub repo to a local machine
date:   2022-07-12 15:12:20 -0600
description: How to conneciton a github repo to a local machine to allow local changes
img: posts/default-post.png 
tags: [git, CI/CD]
author: Scott Powell
github:  scottiepowell/
mathjax: yes
---
# CONOP
1. Create the local repo
2. Create a pull request to the gihub repo
3. Commit local repo changes to github repo
4. Push local repo changes to github repo
___

This will be a quick post, for this blog site i use github pages, which runs on jekyll and is easy to maintain and update for me.  I've been updating the site using the web browser but discovered that pull it the repo down locally and working on the posts in VS code is much better.  Here are the commands to do this, easy peasy.

# Step 1

First step is to create the local repo on your local machine, i created a folder called 'blog-github-repo', do a `cd` into the folder that you create and run the following commands.
`git init`
`git pull <github repo name>`

# Step 2

Git init command will initilize a local repository and the workspace to connect to github, the second command will force push the github repo to your local machine repo.  Now when you do a `ls` in the directory that you created earlier there should be all the files from your github repo.  

# Step 3

Now that the repo is present on your local machine, it's time to add a file and commit the file to the local git repo.  the commit portion is very imporant, if you don't commit the changes, the follow on steps will not work.  Below is an example of how to add and commit a file to your local repo.

`git add <directory and file name>`

`git commit -m "file change performed"`

The first command add's the file to the repository and puts the file in version control with git, the second command commits the file to the local repo.

# Step 4

Now, that our file is in commited in git, let's add the github site as the origin site for our local repo.  This is a prepatory command for the git push command.

`git remote add origin <github repo site>`

Verify that the github repo is the 'fetch' and 'push' target for the local repo by running this command `git remote -v` 

Now, push the file to github with following command

`git push origin master`

Now go to your github and check the 'pull request' tab, there should be a request from your local repo.  Accept the pull request and merge the changes into your master branch.  







