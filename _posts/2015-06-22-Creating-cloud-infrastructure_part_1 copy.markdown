---
layout: post
comments: true
title: Creating cloud infrastructure
date:   2015-06-22 16:13:25
categories: ansible openstack devops
---

Last post I made a little introdcution to [Ansible](http://www.ansible.com) and here I thought I would share some tools, thoughts and methods, what would be the best way to  deploy, run and manage you openstack instances. 

When running openstack, you need some services to take most out of your automated cloud, so that you have easy to deploy and manage infrastructure. The tools you might wan't to use should be easily managed by ansible, puppet, chef or salt and they should be easy to configure and use, so that the overhead of of learning them are as fast and easy as possible. They also need to be highly redundant. 

### This tutorial will show you how to:

*  Give you the idea what tools and services are needed to provision services automatically.
*  Give you the basic understanding of Dnsmasq and how to manage it by using Ansible´
*  Deploy dnsmasq services on openstack
*  Show you some basic shell scripting and Java code to make the Customer UI.


The tools you need are

*  DNS services (internal & External)
*  Version control system to host your code
*  IT Automation tool 
*  Some sort of UI to manage your service automation platform and give users to have an easy way to deploy services
*  Simple and good Backup solution
*  If you wan't to host Google Apps Engine

### So let's go with the DNS services.

#### internal dns services
We need a lightweight dns services for internal networking. Bind and PowerDNS are good products that take much overhead to configure and are way too complex for simple internal dns services.  For these reasons, I chose Dnsmasq, that is really light. 

The first thing you need to do is to provide highly redundant dns services for you openstack hosts.   We can do this with ansible easily, so here's my github project for doing this. 

Then when you have the DNS-services set up, and you need to add new hosts to dns services just call: `dnsmasq-add-host.sh  ip hostname alias`. And alternatively if you wan't to remove dns records you can remove thoste by executing `dnsmasq-remove-host.sh hostname`. 

Now the best part is that when you provision new openstack instances withg ansible you can use these same dnsmasq playbooks in your
own playbooks to add the nodes and remove the nodes. If you need to update node info, just use the add script and then remove the old one. 


#### external dns services

For the external dns services I would definately use rackspace cloud dns. This is because it already has modules for ansible and it's really easy to use through rackspaces console or through API's. 

The procedure for creating and using these are just:
*  Register domain 
*  Point your domain to use rackspace dns servers
*  Now you can just add and remove records from your DNS

I also suggest that you create a wildcard certificate for you servers if you need one :) So when you provision services just use that certificate in you ssl secured services. 


Here is my example rackspace ansible project that you can use on your projects.. Just make shure to modify the parameters in main.yaml. 













 







### Getting started

There are some prerequisites you need to take care of:

* You need to have ansible and git installed
* You need Mac or Linux. Sorry, this is not for Windows users. 
* Openstack must be installed and you must have Tenant(Project) there([DevStack](http://docs.openstack.org/developer/devstack/) is also fine)
* Know the Openstack API Endpoint url for Identity -service. From Dashboard goto `Access & Security` and `API Access` 
* You need to know how to download openstack RC-file (more info [OpenstackRC documentation](http://docs.openstack.org/cli-reference/content/cli_openrc.html) )


First thing you have to do is that you have to have an ansible control server (your workstation is fine also), and as a best practice you should keep your ansible playbooks in a version control system like git or something similar. 

To get you started first install Ansible.. Go to [Ansible's web site](www.ansible.com) and go through the installation guide..


On Ubuntu server you can just run
`sudo apt-get install ansible`

and on Mac you can install [Homebrew](http://brew.sh/) and run:
`brew install ansible`


Then on linux or mac, create a  ssh-keys (in this example it's called cloud.key) and deploy that ssh-key on your Openstack tenant.. 


## 1. Generate ssh-keys

{% highlight bash %}
Kari-MacBook-Pro:~ skoude$ ssh-keygen -t rsa -f cloud.key
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
passphrase too short: have 4 bytes, need > 4
Saving the key failed: cloud.key.
Kari-MacBook-Pro:~ skoude$ ssh-keygen -t rsa -f cloud.key
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in cloud.key.
Your public key has been saved in cloud.key.pub.
The key fingerprint is:
cc:e5:5e:b9:ab:fb:ef:56:65:e0:1f:ae:dd:68:8a:26 skoude@Kari-MacBook-Pro.local
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|              .  |
|          .  . . |
|       o o   ...o|
|        S . o .oo|
|         . . . .o|
|          . . oo.|
|        E .. o+..|
|         o+++*o  |
+-----------------+
Kari-MacBook-Pro:~ skoude$ ls -la cloud*
-rw-------  1 skoude  staff  1675 18 Tou 15:21 cloud.key
-rw-r--r--  1 skoude  staff   411 18 Tou 15:21 cloud.key.pub
Kari-MacBook-Pro:~ skoude$
{% endhighlight %}


## 2. Upload your key to openstack tenant

Now you need to upload your key to opentack tenant.. You can do it by issuing command:
{% highlight bash %}
nova keypair-add --pub-key cloud.pub cloudkey
{% endhighlight %}

## 3. Checkout my example Git -repo
When you have ssh-keys generated you could take a look at my ansible playbooks project on github. 
just do:

{% highlight bash %}
git clone https://github.com/skoude/ansible_and_openstack.git
{% endhighlight %}


## 4. Inspect and execute code
Now,  go to the `playbooks/create_openstack_instance.yaml` and check `create_instance.yaml`. This Yaml -file has comments, and so you should read it through first to understand what it's doing. There is also `create_instance_by_using_env_values.yaml`, which is executed if you wan't to use openstack rc-files environmental variables, but then you need to run `source ./openrc.sh first

When you are done of inspecting the code, you can just execute `./run_create_instance.sh` which asks you the key values and creates on instance in openstack for you. Basically it automatically uploads the Ubuntu 14.04 trusty image to glance and then boots up an instance from that image. Then it changes the ssh settings to allow root login, and changes the root password for what you inserted.  Look for the `create_instance.yaml` for more details. It will also give floating IP for the instance and after running the playbook you should have new instance created on openstack and you can log in with the floating ip assigned. For example: `ssh root@172.20.0.40` 


## 5. Final conclusions

When you are comfortable with this Ansible playbook, you could easily extend it to deploy multiple servers and services at one run. You could for example provision MongoDB, web servers, HAProxy, add the dns and add correct rules to firewall,  by issuing one command.  It would automatically create the whole cluster for you. Or you could use Ansible to deploy and manage the whole openstack infrastucture. It's up to you what you wan't to achieve by using tools like this.  [Ansible Galaxy](https://galaxy.ansible.com/) is a nice place to get started with ready made modules called *Roles*. Also I would suggest you to read *Learning Ansible published by Packt Publishing*  it's really good book. 

At next post I will show you how to use dynamic inventory against openstack. 



{% if page.comments %}
<div id="disqus_thread"></div>
<script type="text/javascript">
    /* * * CONFIGURATION VARIABLES * * */
    var disqus_shortname = 'skoudestechnologyblog';
    
    /* * * DON'T EDIT BELOW THIS LINE * * */
    (function() {
        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
        dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>

{% endif %}


