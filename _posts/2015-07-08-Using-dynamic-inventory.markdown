---
layout: post
comments: true
title: Using dynamic inventory with openstack
date:   2015-07-06 21:13:25
categories: ansible openstack devops
---


### This tutorial will show you how to:

*  Use dynamic inventory
*  Use multiple inventories at the same time


## Getting started with dynamic inventory

There are some prerequisites you need to take care of:

*  You need to have ansible installed (check my last post)
*  You need to have python installed and also you neeed to download the
  openstack.py -file from here [Dynamic inventory file for
  openstack](https://github.com/lukaspustina/dynamic-inventory-for-ansible-with-openstack)
*  You need to cofigure the openstack.yml -file with correct parameters
*  Know the Openstack API Endpoint url for Identity -service. From Dashboard goto `Access & Security` and `API Access` 
*  You need to know how to download openstack RC-file (more info [OpenstackRC documentation](http://docs.openstack.org/cli-reference/content/cli_openrc.html) )

First thing you need to do is download the openstack.py -script and then
edit the openstack.yml -file. 
You can just clone the repo from [Dynamic inventory file for
  openstack](https://github.com/lukaspustina/dynamic-inventory-for-ansible-with-openstack)

  Then edit the openstack.yml -file. Mine looks like this:

{% highlight yml %}
clouds:
  devstack:
    auth:
      auth_url: https://site.site.fi:5000/v2.0
      username: myusername
      password: mypasswd
      project_name: Skoude
{% endhighlight %}


Then you can just test it with executing 
{% highlight bash %}
./openstack.py --list
{% endhighlight %}



Now just edit the `openstack.py` to look for the correct `openstack.yml`
-file. 

{% highlight bash %}
vim openstack.py

 class OpenStackInventory(object):
      def __init__(self, private=False, refresh=False):
          self.openstack_config = os_client_config.config.OpenStackConfig(
              os_client_config.config.CONFIG_FILES.append(
                  'openstack.yml'),
              private)

{% endhighlight %}


This command should list all the instances that you have on your
openstack tenant(=project)

For example on my test openstack project:
{% highlight bash %}
./openstack.py --list

# you should see something like this (it's a long list if you have some
instances running...)...
...
  ],
  "instance-361b13a2-981d-4d85-9b1d-a7091707e884": [
    "ubuntu_node_1"
  ],
  "instance-8693348c-c8ca-4178-956f-957e021ce66b": [
    "ubuntu_node_3"
  ],
  "instance-f490c3eb-5bfd-43e1-87dc-bdd908e71d4e": [
    "ubuntu_node_2"
  ],
  "meta-systemid_test": [
    "ubuntu_node_3",
    "ubuntu_node_2",
    "ubuntu_node_1"
  ],
  "nova": [
    "ubuntu_node_3",
    "ubuntu_node_2",
    "ubuntu_node_1"
  ]
}
{% endhighlight %}


Now you can just use the inventory like
nsible 

{% highlight bash %}
ansible -i openstack.py your_host -m service -a 'name=nginx state=restarted'

#or 

ansible -i openstack.py -u ubuntu your_host -m service -a 'name=ssh
state=restarted'


{% endhighlight %}


## Using dynamic inventory and other inventories at the same time

Create a directory called `inventory` and copy the
`openstack.py`, `openstack.yml` and your normal inventory file (for
example host) to this inventory directory.

Now, because all of our inventories are on a sub folder called
`inventory` we can just point the ansible to use that directory as
inventory file.

For example just execute a command

{% highlight bash %}
ansible -i inventory your_host -m service -a 'name=ssh state=restarted'

{% endhighlight %}

You could also add the inventory file location to you ansible.cfg -file
like:

{% highlight bash %}
 [defaults]
 hostfile = inventory
 # more at http://docs.ansible.com/intro_configuration.html#the-ansible-configuration-file
 host_key_checking=False
 #log_path=log.log
 #remote_user=user
{% endhighlight %}




## Final conclusions

Dynamic inventories in Ansible are very powerfull when you have multiple
systems and clouds that you need to connect to. This gives you the
possibility to connect to all of the nodes you have from the same
playbook. Also in openstack i suggest that you use some `meta`
information when you create instances on openstack. This way you can
just provision services by using the meta -key instead of hostname in
ansible.


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


