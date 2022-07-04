---
layout: post
read_time: true
show_date: true
title: "Road to Docker: Marker 1, monitoring infrastructure with grafana"
date: 2022-06-23
img: posts/default-post.png
tags: [grafana, monitoring, proxmox, infrastructure, Docker]
category: Docker
author: Scott Powell
description: "Monitoring Proxmox infrastructure using Docker and Grafana"
github:  scottiepowell/
mathjax: yes
---
Monitoring of servers and services is always important in a homelab, especially if i'm out of town or just not active in my man cave working on projects and systems.   

## Concept of Operations (CONOP)
1. install influxdb on docker(using portainer)  
2. configure the metric server on proxmox
3. configure a dashboard (using Grafana)

## Step 1

First thing is to install the influxdb on docker, i'm going to use portainer, portainer is a web-based UI for container technology, it works particularly well for docker installs and maintainece.  Installing portainer is a seperate discussion and another blog post for another time.  A quick google search will land many tutorials for portainer if you don't want to wait on me to make a blog post.
 and scroll down 
First thing is creating a container in portainer, as seen below i've created a container named `influxdb` and we are pulling the image from DockerHub using the alpine distro and version 2.1.  If you want to research version for infuxdb, navigate to (https://hub.docker.com/_/influxdb?tab=tags&page=2) and you will find the 2.1-alpine tag.  One of the strengths of containers is if you want to experiment and test other containers with a different OS or versions, it's a quick rebuild and standup a new container.    

### Portainer containers running

![aaa](https://github.com/scottiepowell/gh-pages-blog.github.io/blob/main/assets/img/posts/2022-06-27/port-img-net.PNG)

![Portainer Image/Container Settings](https://github.com/scottiepowell/gh-pages-blog.github.io/blob/main/assets/img/posts/2022-06-27/port-container-running.PNG)

![aaa](https://github.com/scottiepowell/gh-pages-blog.github.io/blob/main/assets/img/posts/2022-06-27/port-net-settings.PNG)

![aaa](https://github.com/scottiepowell/gh-pages-blog.github.io/blob/main/assets/img/posts/2022-06-27/port-vol-create.PNG)

![aaa](https://github.com/scottiepowell/gh-pages-blog.github.io/blob/main/assets/img/posts/2022-06-27/port-vol-settings.PNG)

![aaa](https://github.com/scottiepowell/gh-pages-blog.github.io/blob/main/assets/img/posts/2022-06-27/proxmox-metric-server.PNG)

## Step 2



## Step 3

In step 1, we use portainer to deploy the influxdb docker container, for grafana, we are going to mix things up and deploy using a docker-compose YAML file.

I'll break the code into smaller chunks and provide some narrative on each section, here is a link to the full code on my github and can skip this section if you are familar with docker-compose and how it utilizes YAML values

___

```
version: '3'

volumes:
  grafana-data:
    driver: local
```

The version with a value of 3 in the code above tells the Docker Daemon that the file will be compatible with a version 1.13.0 and above of the Docker Engine.  The volume creates a grafana volume in docker and mounts the volume in the container at '/var/lib/grafana', this is important for keeping persistent data if the container is removed or destroyed.

___
```
services:
  grafana:
    image: grafana/grafana-oss:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    restart: unless-stopped
```    

The image above tells Docker to pull the latest open-source grafana image 'grafana-oss' from Docker Hub.  Instead of using the tag 'latest' there are other options to use operating systems like Ubuntu, the latest tag defaults to the operating system of alpine.  The ports are mapped from docker to the container on 3000 and the volume is mounted as discussed earlier.
___   
   
networks:
  default:
    name: nginxproxymanager_default
    external: true      
___

The networks section of this docker-compose is critical to get right, this section will tell Docker to install Grafana on a specific network under name.  The **network must be the same as the network that influxDB was installed on** or else the to containers will not be able to communicate.

Now run the following command from the diretory of our docker-compose file

    docker-compose up -d

Verify your container is running with `docker ps`

Security, one important thing to point out is this configuration doesn't include any inherent security features like SSL, the connections are insecure.  Right now, i'm using a reverse proxy to secure the containers and the connection to proxmox is on my internal network, but in the future i'll look at doing a updated posted to describ how to secure the connections between Proxmox, Influx and Grafana with SSL for a production environment.  Right now, that is out of scope.

Once Grafana is installed, navigate to `http://<ip address>:3000` and login with the default username/password of `admin/admin`.  Once logged in navigate to configure > Datasource and install InfluxDB

Install the settings as follows, important, **the hostname in the http setting is the docker hostname of your influxdb container** 

Set the influxdb bucket name, organization and API token that was previusly established with the influxdb container.

There is a really great dashboard for Proxmox at this link, you can create your own dashboard but this dashboard had everything that i wanted

https://grafana.com/grafana/dashboards/15356

Now navigate to the dashboard you have created, my bucket defaulted to a different bucket, make sure you select the right bucket and enjoy your metrics!


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

