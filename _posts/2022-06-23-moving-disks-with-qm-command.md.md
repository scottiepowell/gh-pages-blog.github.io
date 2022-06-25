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
I recently bought some new HDD's, a much needed storage upgrade for my proxmox cluster, i've been running into storage issues in my current configuration.  I've got a 1TB HDD that is hosting all my VM's, LXC's and images for my primary node.  I've researched a few different ways to migrate VM's from one physical storage to another, they all have their pro's and con's.  For my particular use case i going to use a three stage method to migrate all my VM's using the proxmox GUI and the QM command.

## Concept of Operations (CONOP)
1. Move contents from current storage to an intermediate USB storage
   > this is a necesary step because i don't have an addtional SATA connector to plug additional storage into 
2. validate the VM is FMC on the intermediate USB storage
   > remove the old storage from the server 
4. Move content from intermediate USB storage to new SATA based storage
5. validate the VM is FMC on the new SATA based storage   

___

The command to move disks is the QM command

    qm move_disk <vmid> <disk> <storage> [OPTIONS]

First thing with moving the storage, let's navigate to the storage loction on the Proxmox node and take a look at our storage 

'cat /etc/pve/nodes/<node name>/qemu-server/<.conf file of VM>'
      
Look at the output of CAT on the line for sci0, most of the information needed is on that line
   
- 'scsi0: local-lvm:vm-1070-disk-0,size=60G'  
   > 1070 is the VM ID
   > scsi0 is the disk you want to move
- barra-usb-4tb-vm is the directory that the USB storage is configured under, this can be found from command line
   > type the following command 'cat /etc/pve/storage.cfg'
   > output will be all the storage locations, below is the directory that i've created with additional details
   
'''dir: barra-usb-4tb-vm
   path /mnt/4tb-barracuda
   content rootdir,images,vztmpl
   nodes proxmox1
   prune-backups keep-all=1
   shared 0
'''
   
Format, for this particular VM i want to change the format from RAW to qcow2, we can do this with the --format option 
   
The full command to move the vm is below:
   
'qm move_disk 1070 scsi0 barra-usb-4tb-vm --format qcow2 --target-vmid 1070'
   
After the move is complete, go back into the 1070.conf file and verify the move with two lines, the storage will change and unused storage will be the old storage

'scsi0:barra-usb-4tb-vm:1070/vm-1070-disk-0.qcow2,size=60G'
   
'barra-usb-4tb-vm:1070/vm-1070-disk-0.qcow2,size=60G'
      
{% comment %}

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
  
{% endcomment %}

