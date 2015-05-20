---
layout: post
comments: true
title: Using Ansible to deploy Instances and services in openstack
date:   2015-05-19 16:13:25
categories: ansible openstack devops
---

[Ansible](http://www.ansible.com) is a very powerfull platform to deploy and manage any kind of configuration on almost any platform available. It is a superb tool for Devops to automate service configuration and installations. Today I will show you a simple playbook that you can use to deploy instances on Openstack. This will get you started with Ansible and Openstack, and hopefully you will get some idea of how to extend the Ansible playbooks..

### This tutorial will show you how to:

*  Upload image to Glance 
*  Deploy an Openstack instance of that image 
*  Modify the ssh settings so that root login is permitted 
*  Change the root password
*  Restart the ssh -service on instance

You could easily extend this playbook with modules to deploy for example mongodb, apache etc.. 


First thing you have to do is that you have to have an ansible control server (your workstation is fine also), and as a best practice you should keep your ansible playbooks in a version control system like git or something similar. 

To get you started first install Ansible.. Got to [Ansible's web site](www.ansible.com) and go through the installation guide..
On Ubuntu server you can just run
`sudo apt-get install ansible`

and on Mac you can install [Homebrew](http://brew.sh/) and run:
`brew install ansible`


Then on linux or mac, create a  shh-keys (in this example it's called cloud.key) and deploy that ssh-key on your Openstack tenant.. 


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

You may need to edit these parameters on `create_instance.yaml`:

{% highlight yaml %}
      # in here you must know the correct network id. If you are using neuton, you can just use:
      # Kari-MacBook-Pro:skoude_test_project skoude$ neutron net-list
      # +--------------------------------------+----------+-----------------------------------------------------+
      # | id                                   | name     | subnets                                             |
      # +--------------------------------------+----------+-----------------------------------------------------+
      # | 7b4b47a4-93fe-42eb-8d3d-337430c4025f | internal | 6d427371-85fb-4dcb-b136-2394d20b5a27 172.20.0.0/20  |
      # | 82ebf9e9-a103-4913-9c75-50c92906fc26 | external | 67721806-7b55-4fa9-b144-1149f5b2d810 172.20.16.0/20 |
      # +--------------------------------------+----------+-----------------------------------------------------+
      # Kari-MacBook-Pro:skoude_test_project skoude$
      nics:
        - net-id: 7b4b47a4-93fe-42eb-8d3d-337430c4025f
{% endhighlight %}

{% highlight yaml %}
     # The id of the flavor:  nova flavor-list
      flavor_id: 3
        - net-id: 7b4b47a4-93fe-42eb-8d3d-337430c4025f
{% endhighlight %}
When you are done of inspecting the code, you can just execute `./run_create_instance.sh` which asks you the key values and creates on instance in openstack for you. Basically it automatically uploads the Ubuntu 14.04 trusty image to glance and then boots up an instance from that image. Then it changes the ssh settings to allow root login, and changes the root password for what you inserted.  Look for the `create_instance.yaml` for more details. 


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


