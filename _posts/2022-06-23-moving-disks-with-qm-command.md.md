---
layout: post
read_time: true
show_date: true
title: "Road to Homelab:<img src='/assets/img/posts/mile/Mile_marker_1.jpeg'>- moving disks using qm command"
date: 2022-06-23
img: posts/default-post.png
tags: [proxmox, linux, infrastructure, Road to Homelab]
category: Homelab
author: Scott Powell
description: "Moving vm's with the qm command"
github:  scottiepowell/
mathjax: yes
---
I recently bought some new HDD's, a much needed storage upgrade for my proxmox cluster, i've been running into storage issues in my current configuration.  I've got a 1TB HDD that is hosting all my VM's, LXC's and images for my primary node.  I've researched a few different ways to migrate VM's from one physical storage to another, they all have their pro's and con's.  For my particular use case i going to use a three stage method to migrate all my VM's using the proxmox GUI and the QM command.

## Concept of Operations (CONOP)
1. Move contents from current storage to an intermediate USB storage
   > this is a necesary step because i don't have an addtional SATA connector to plug additional storage into 
2. validate the VM is working on the intermediate USB storage
   > remove the old storage from the server 
3. Setup new SATA based storage on server
   > move VM's from USB storage to SATA based storage
4. validate the VM is working on the new SATA based storage   

___

## Step 1

The command to move disks is the QM command

    qm move_disk <vmid> <disk> <storage> [OPTIONS]

First thing with moving the storage, let's navigate to the storage loction on the Proxmox node and take a look at our storage 

    cat /etc/pve/nodes/<node name>/qemu-server/<.conf file of VM>
      
Look at the output of CAT on the line for scsi0, most of the information needed is on that line
   
    scsi0: local-lvm:vm-1070-disk-0,size=60G
      > 1070 is the VM ID
      > scsi0 is the disk you want to move

`barra-usb-4tb-vm` is the directory that the USB storage is configured under, this can be found from command line

type the following command:

    cat /etc/pve/storage.cfg

the output of the command will be all the storage locations, below you can find the directory to move storage into under 'dir:'
   
    dir: barra-usb-4tb-vm
    path /mnt/4tb-barracuda
    content rootdir,images,vztmpl
    nodes proxmox1
    prune-backups keep-all=1
    shared 0
   
Format, for this particular VM i want to change the format from RAW to qcow2, we can do this with the `--format qcow2` option 
   
The full command to move the vm is below:
   
    qm move_disk 1070 scsi0 barra-usb-4tb-vm --format qcow2
   
After the move is complete, go back into the 1070.conf file and verify the move with two lines, the storage will change and unused storage will be the old storage

    scsi0:barra-usb-4tb-vm:1070/vm-1070-disk-0.qcow2,size=60G
   
    barra-usb-4tb-vm:1070/vm-1070-disk-0.qcow2,size=60G
___

## Step 2

Once all the VM's are moved over to the new storage and you are confident that everything is working, go into the Proxmox GUI and remove the volume group, in this case for myself it was 1tb-wd.  If you receive a error or the GUI method will not work, try running `lvremove -f /dev/<name of disk>` and rerun the remove command.

___

## Step 3

Setup your new HDD with the SATA connectors using the following steps:

First, identify the new HDD using the command `lsblk`, in my case this was sdc

Run the following command with parted, if you need to install parted run `sudo apt install parted`.  You can use MBR as a partition table, GPT is a new and better standard so i'm using GPT. 

    parted /dev/sdc mklabel gpt

Now let's make the primary partition using 100% of the disk.

    parted -a opt /dev/sdc mkpart primary ext4 0% 100%

We have a partition now, sdc1, creating a file system is the next task with a volume label for future use since sdc is so generic

    mkfs.ext4 -L wd-4tb /dev/sdc1

Run `lsblk -fs` to verify that the file system was created, the next step is to mount the drive on the server

Make a directory in the mnt directory `mkdir -p /mnt/<folder name>`

Edit the fstab so the file system will automount

open up fstab with a editor `nano /etc/fstab` then add a line `LABEL=wd-4tb /mnt/wd-4tb ext4 defaults 0 2`

    mount -a

___

## Step 4

Now, move the VM's back into your storage attached to the server via SATA connectors.  There are a couple of ways to move the storage.

1. Move using web GUI
2. Move via the command line

The web GUI might be the easier option and if you have a VM that is being stubborn and will not move via GUI, then use the command line, it is the same command `qm move_disk` that was used in STEP 1.

At this point, your HDD's should be moved back to the new storage on your server, i'm going to use the USB for backups.

Cheers
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

