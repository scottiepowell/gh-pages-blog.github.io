---
layout: post
read_time: true
show_date: true
title: "Road to Homelab: marker 1 - moving disks using qm command"
date: 2022-06-23
img: posts/20210402/post7-header.webp
tags: [proxmox, linux, infrastructure, Road to Homelab]
category: Homelab
author: Scott Powell
description: "Moving vm's with the qm command"
---
#
I recently bought some new HDD's, a much needed storage upgrade for my proxmox cluster, i've been running into storage issues in my current configuration.  I've got a 1TB HDD that is hosting all my VM's, LXC's and images for my primary node.  I've researched a few different ways to migrate VM's from one physical storage to another, they all have their pro's and con's.  For my particular use case i going to use a three stage method to migrate all my VM's using the proxmox GUI and the QM command.

# Concept of Operations (CONOP)
1. Move contents from current storage to an intermediate USB storage
   - this is a necesary step because i don't have an addtional SATA connector to plug additional storage into 
2. validate all VM's are FMC on the intermediate USB storage
   - remove the old storage from the server 
4. Move content from intermediate USB storage to new SATA based storage
5. validate all VM's are FMC on the new SATA based storage   


{% comment %} 
//the command is qm move_disk <vmid> <disk> <storage> [OPTIONS]

   //markdown common syntax

   //# Heading level is #'s

   // To create a line break or new line (<br>), end a line with two or more spaces, and then type return.

   // Bold is to ** on each side **bold**

   // italicize text, add one asterisk or underscore before and after a word or phrase.

   // To create a blockquote, add a > in front of a paragraph.

   // Blockquotes can be nested. Add a >> in front of the paragraph you want to nest.

[//]: # (If you need to start an unordered list item with a number followed by a period, you can use a backslash (\) to escape the period.)

[//]: # (Code blocks are normally indented four spaces or one tab. When they’re in a list, indent them eight spaces or two tabs.)

[//]: # (example how to upload a hyperlink to an image ![Tux, the Linux mascot](/assets/images/tux.png))  

[//]: # (If the word or phrase you want to denote as code includes backticks, you can escape it by enclosing the word or phrase in double backticks (``).)

[//]: # (create a horizontal rule, use three or more asterisks (***), dashes (---), or underscores (___) on a line by)    

[//]: # (This is a method of using MD to make a comment)

[Comment test]::  
{% endcomment %}

