---
layout: post
title: "Taking OpenStack for a Spin"
description: ""
category: 
tags: []
by: Kord Campbell
---
{% include JB/setup %}

[OpenStack](http://openstack.org) is an Open Source cloud computing infrastructure platform originally authored by NASA and [RackSpace](http://rackspace.com).  OpenStack provides similar services to Amazon's AWS infrastructure platform, except without selling your first born to pay for Amazon's hellaciously expensive instance time.

OpenStack's components are mostly written in Python and include [Nova](https://github.com/openstack/nova), a fabric controller similar to what EC2 provides, [Swift](https://github.com/openstack/swift), a S3-like storage system, and [Glance](https://github.com/openstack/glance), a service for providing services for virtual disk images.  Other OpenStack projects include [Keystone](https://github.com/openstack/keystone), an identity service, and [Horizon](https://github.com/openstack/horizon), a Django-based UI framework which ties all these services together in a single webpage view.

![](/assets/images/awesomesauce.png)

As of this writing, the most current version of OpenStack is [Essex](https://launchpad.net/nova/essex/essex-rc1).  Essex is currently in release candidate phase, with RC1 available for download for each of the components listed above.  The git repository for the components is [available on Github](https://github.com/openstack).

I floundered around for a few days trying to download and run the various independent components of OpenStack.  The instability of the early versions of Essex and the maze of configuration files you have to tweak make a difficult time of getting it running, at least for me.  Thankfully, a bit of Googling revealed a script written by the fine folks over at Rackspace called [DevStack](http://devstack.org/), which does all the heavy lifting for getting Openstack running.  

_Note: Rackspace doesn't recommend using DevStack for production deployments, but it works great if you just want to give OpenStack a test drive and do some quick development on the instances you start._


## Getting Started

I've put together a 14 minute step-by-step video guide for getting OpenStack installed on an Oneiric instance running on VMWare Fusion under OSX.  The video also guides you through launching an instance and accessing it with a ssh terminal.

<iframe src="http://player.vimeo.com/video/39055026?byline=0&portrait=0" width="420" height="348" frameborder="0">&nbsp;</iframe>

You can [download Oneiric](http://www.ubuntu.com/download/server/download) from Ubuntu's website.  Get it running on a VM or a bare metal box and make sure you install the OpenSSH server so you can ssh into it.


### Using VMWare Fusion

I'm running OpenStack on a dedicated box, but if you run OpenStack on a VM you'll need to ensure you have enough CPU/memory to start instances.  You'll need at least 1.5GB to launch a tiny instance.  Keep in mind that instance will be a bit slow because it's running a [VM on top of another VM](http://en.wikipedia.org/wiki/Simulated_reality#Nested_simulations).  

Here's a screenshot of my VMWare Fusion config set up with a bridged network and about 4GB of RAM with 2 cores assigned:

![](/assets/images/oneiric_settings.png)


### Checking Out Code

Now that you have a fresh install running, you'll need git installed on your new box.  Go ahead and login and do an aptitude update and install it:

<pre class="prettyprint linenums">
sudo apt-get update
sudo apt-get install git
</pre>

Now let's check out the DevStack code from Github:

<pre class="prettyprint linenums">
git clone https://github.com/openstack-dev/devstack.git
</pre>

We need to do a couple of things to configuration files before we fire up DevStack, so let's talk about networking for a second.


### Networking

Nova does the management for your instance's network.  This is similar to the way AWS manages assigning private and public IPs to instances on EC2.  IPs are managed across a set of machines on which you run OpenStack, but you'll still need to configure your routing to make these available to your existing network.

If you have a router that can handle static routes, you could simply map the Nova managed network to the interface on the hypervisor (the base host box) to be able to talk to the instances you launch.  I have an Airport Extreme at my house and it doesn't have a way to do static routes, so I'm going to explain how to work around that limitation.  We'll set it up so our Nova install will get its own network, but we'll use allocated IPs from the existing network so we can provision and map them to the instances we launch with Nova.

Here's are the configs I used in my install.  You'll want to put these in a new file called <code>localrc</code> in the <code>devstack</code> directory you just checked out above.

<pre class="prettyprint linenums">
HOST_IP=10.0.1.20
FLAT_INTERFACE=eth0
FIXED_RANGE=10.0.2.0/24
FIXED_NETWORK_SIZE=256
FLOATING_RANGE=10.0.1.224/27
</pre>

Change the <code>HOST_IP</code> to the address of your hypervisor box (the box on which you are running the devstack script). You can leave the <code>FIXED_RANGE</code> value alone assuming you aren't using <code>10.0.2.0/24</code> on your network.  

Change the <code>FLOATING_RANGE</code> to whatever IPs you run on your local network, but only use the top end of the network by using a <code>/27</code> and starting at the <code>224</code> octet.  Usually you could use an entire <code>/24</code> (253 addresses from 10.0.2.1 thru 10.0.2.254), but I'm purposely using the <code>/27</code> so Nova won't start allocating IPs down low at 10.0.1.1, which is my router.  

Speaking of routers, be sure to block out these upper IPs; my Airport has a max range setting in it that I set to 200, to prevent IP address overlap with the ones Nova will start at 225.

### Add the Oneiric Image

By default devstack only installs a small Cirros distro.  I'm an Ubuntu guy, so I hacked up the <code>stackrc</code> file to include a cloud build of Oneiric which will be downloaded when you run the devstack <code>stack.sh</code> script.  Scroll down to the bottom of the <code>stackrc</code> file and delete/replace the following lines:

<pre class="prettyprint linenums">
case "$LIBVIRT_TYPE" in
    lxc) # the cirros root disk in the uec tarball is empty, so it will not work for lxc
	IMAGE_URLS="http://cloud-images.ubuntu.com/releases/oneiric/release/ubuntu-11.10-server-cloudimg-amd64.tar.gz,http://launchpad.net/cirros/trunk/0.3.0/+download/cirros-0.3.0-x86_64-rootfs.img.gz";;
    *)  # otherwise, use the uec style image (with kernel, ramdisk, disk)
	IMAGE_URLS="http://cloud-images.ubuntu.com/releases/oneiric/release/ubuntu-11.10-server-cloudimg-amd64.tar.gz,http://launchpad.net/cirros/trunk/0.3.0/+download/cirros-0.3.0-x86_64-uec.tar.gz";;
esac
</pre>


### Start the Script

The <code>stack.sh</code> script inside the <code>devstack</code> directory will do the rest of the configuration for you.  It'll download all the components needed to run OpenStack and stuff them in <code>/opt/stack/</code>.  It also downloads the images above and uploads them into Glance so you will have an Ubuntu image you can launch.  Start the install process by running the script:

<pre class='prettyprint linenums'>
cd devstack
./stack.sh
</pre>

The script will prompt you for multiple passwords to use for the various components.  I used the same pass/key for all the prompts, and suggest you do the same.  After the script runs, it'll append the passwords to the <code>localrc</code> file you created earlier:

<pre class='prettyprint linenums'>
HOST_IP=10.0.1.20
FLAT_INTERFACE=eth0
FIXED_RANGE=10.0.2.0/24
FIXED_NETWORK_SIZE=256
FLOATING_RANGE=10.0.1.224/27
MYSQL_PASSWORD=f00bar
RABBIT_PASSWORD=f00bar
SERVICE_TOKEN=f00bar
SERVICE_PASSWORD=f00bar
ADMIN_PASSWORD=f00bar
</pre>


## Managing Using the Horizon UI

Once devstack has OpenStack running, you can connect to Horizon, the management UI.  The <code>stack.sh</code> script will spit out the URL for you to access the UI:

<pre class='prettyprint linenums'>
Horizon is now available at http://10.0.1.x/
</pre>

Plug this into your browser and then use <code>admin</code> and <code>f00bar</code> for your user/pass.  If you made your passwords something else, then obviously use that password here instead of <code>f00bar</code>.


### Create Keypairs/Floating IPs/Security Groups

1. Click on the <code>Project</code> tab on the left side of the screen and then click on <code>Access &amp; Security</code>.  
2. Click on the <code>Allocate IP to Project</code> button at the top and then the <code>Allocate IP</code> button at the bottom of the dialog that pops up.  You should see a message that a new IP has been allocated and it should show up in the list of Floating IPs.
3. Now click on the <code>Create Keypair</code>button under <code>Keypairs</code>.  Create a key named <code>default</code> and accept the download of the private side of the key to save it on your local machine.  You'll need this key later to log into the instance you'll start in a minute.
4. Finally, click on the <code>Edit Rules</code> button on the <code>default</code> security group under <code>Security Groups</code>.  Under <code>Add Rule</code> add rules for pings, ssh and the default web port:

![](/assets/images/security_groups.png)


## Launch an Instance

Click on the <code>Images &amp; Snapshots</code> tab to view the images you have available.  Click on the <code>Launch</code> button next to the flavor of OS you want to launch.  In the dialog that pops up, enter the server name and select the <code>default</code> keypair you created earlier.  You can also paste in a script in the <code>User Data</code> textbox, which will be run after the machine boots.  Here's the one I used to figure out the login user:

<pre class='prettyprint linenums'>
#!/bin/bash
cat /etc/passwd
</pre>

Click on the <code>Launch Instance</code> button at the bottom to launch your instance.  You should be taken to the <code>Instances &amp; Volumes</code> tab and shown the status of the instance:

![](/assets/images/instance_view.png)


### Assign a Floating IP to the Instance

If you are running a network where you can't map static routes in your router, you'll need to assign one of the floating IPs from your network.  Click on the <code>Access &amp; Security</code> tab and then click on the <code>Associate IP</code> next to the IP address to assign to the instance.  

![](/assets/images/assign_ip.png)

Click <code>Associate IP</code> to finish assigning the IP to the new instance.


## Using Your Instance

Remember the default.pem key that got downloaded earlier?  You'll need to go find it and copy it somewhere you won't lose it.  You'll also need to change the permissions on it before you use it to login to your new instance:

<pre class='prettyprint linenums'>
$ mv default.pem /Users/kord/.ssh/
$ cd /Users/kord/.ssh
$ ssh -i default.pem ubuntu@10.0.1.226
Welcome to Ubuntu 11.10 (GNU/Linux 3.0.0-16-virtual x86_64)

  System information as of Thu Mar 22 22:20:13 UTC 2012

  System load:  0.0              Processes:           57
  Usage of /:   9.8% of 9.84GB   Users logged in:     0
  Memory usage: 20%              IP address for eth0: 10.0.2.2
  Swap usage:   0%

ubuntu@ubuntu:~$ uptime
22:20:32 up 3 days,  2:28,  1 user,  load average: 0.00, 0.01, 0.05
</pre>

Once you are logged in you can set up your own account and ssh keys to access the box.  It took me a bit of floundering to figure out the default user for the cloud version of Oreiric was <code>ubuntu</code>.

### More Networking!

If you are running a cluster of servers, you'll need some type of reverse proxy to direct the traffic.  It doesn't appear that OpenStack provides this or load balancer features (yet), but there are a few projects that could have APIs added to them to enable future integration into the OpenStack deployment infrastructure.

In the meantime, check out [Nodejitsu's](http://nodejitsu.com) Node based HTTP proxy on [Github](https://github.com/nodejitsu/node-http-proxy).  It's fairly easy to configure and the performance is decent.  Here's a version I'm running for reference:

<pre class='prettyprint linenums'>
var util = require('util'),
    http = require('http'),
    httpProxy = require('./http-proxy');

//
// Http Proxy Server with Proxy Table
//
httpProxy.createServer({
  router: {
    'spurt.stackgeek.com': '10.0.1.226:80',
    'house.geekceo.com': '10.0.1.19:8080'
  }
}).listen(80);

//
// Target Http Server
//
http.createServer(function (req, res) {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.write('request successfully proxied to: ' + req.url + '\n' + JSON.stringify(req.headers, true, 2));
  res.end();
  console.print("foo");
}).listen(9000);
</pre>


### Maintenance

The OpenStack repos on Github get updated all the time.  If you want to stay up-to-date with these changes, I recommend periodically doing a <code>git pull</code> request in each of the directories that get stuffed into <code>/opt/stack/</code>.  You can kill your instance of OpenStack before you do this by doing a <code>killall screen</code>.

You can also use the <code>rejoin-stack.sh</code> script if you've already run the script and are wanting to fire it back up again after killing it.

That's about .  Be sure to leave comments if you have questions or found errors in the guide! 

Happy stacking!