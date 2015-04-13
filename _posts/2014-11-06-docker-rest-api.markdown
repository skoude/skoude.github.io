---
layout: post
title:  "Docker REST-API testing"
date:   2014-11-06 16:13:25
categories: docker rest api testing
---

Today I was testing the docker REST api, and here’s a quick introduction to it.. 
You can enable docker rest API by editing /etc/default/docker
Edit the file according like this.. 

'''bash
# Use DOCKER_OPTS to modify the daemon startup options.
#DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4"
DOCKER_OPTS="-H tcp://0.0.0.0:8888 -H unix:///var/run/docker.sock"
'''
then run: service docker restart

More information about binding he docker to port and / or sock from here: http://docs.docker.com/articles/basics/#bind-docker

Also I suggest that you add firewall rules to limit the access, use some kind of http proxy (with authentication),   or configure docker to listen only localhost and create an ssh tunnel or vpn connection to server. This way you can get secured access to docker API.
For example firewall:  sudo ufw allow from 192.168.0.4 to any port 888
Then test http GET with you favorite browser the URL http://dockerserverip/images/json

More information can be found from Docker Remote api documentation:  https://docs.docker.com/reference/api/docker_remote_api_v1.14/


