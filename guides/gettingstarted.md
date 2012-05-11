---
layout: guide
title: "Installing OpenStack on Ubuntu 12.04 LTS in 10 Minutes"
description: ""
category: 
tags: []
by: Kord Campbell
---
<p><small>Last updated on 05/11/12 @ 12:00AM</small></p>
StackGeek has assembled a [few scripts](https://github.com/StackGeek/openstackgeek) and this guide which will give you a working installation of OpenStack Essex in about 10 minutes. Before you start, you'll need the following in place:

1. A box with at least 8GB of RAM, 4 processing cores, (2) hard drives, and (2) ethernet cards.
2. A clean [install of Ubuntu 12.04 LTS](http://www.ubuntu.com/download/server) 64-bit server on your box.
3. One Monster energy drink.

<p><code>Note:</code> Only the primary ethernet card needs to be connected to the network.  If you only have one ethernet card, you can hack the scripts to use the primary interface for your private network.</p>

### Video Guide
The video guide for this tutorial [is on Vimeo](http://vimeo.com/42010112).
<span id="video-41807514"></span>

### Download the Scripts
Login to your box and install <code>git</code> with <code>apt-get</code>.  We'll become root and do an update first.

<pre>
sudo su
apt-get update
apt-get install git
</pre>

Now checkout the StackGeek scripts from Github:

<pre>
git clone git://github.com/StackGeek/openstackgeek.git
cd openstackgeek
</pre>

### Install the Base Scripts
Be sure to take a look at the scripts before you run them.  Keep in mind the scripts will periodically prompt you for input, either for confirming installation of a package, or asking you for information for configuration.  

Start the installation by running the first script:

<pre>
./openstack_base_1.sh
</pre>

When the script finishes you'll see instructions for manually configuring your network.  You can edit the <code>interfaces</code> file by doing a:

<pre>
vim /etc/network/interfaces
</pre>

Copy and paste the network code provided by the script into the file and then edit:

<pre>
auto eth0 
iface eth0 inet static
  address 10.0.1.20
  network 10.0.1.0
  netmask 255.255.255.0
  broadcast 10.0.1.255
  gateway 10.0.1.1
  dns-nameservers 8.8.8.8

auto eth1
</pre>

Change the settings for your network configuration and then restart networking and run the next script:

<pre>
/etc/init.d/networking restart
./openstack_base_2.sh
</pre>

After the second script finishes, you'll need to set up a logical volume for Nova to use for creating snapshots and volumes.  Nova is OpenStack's compute controller process.

Here's the output from the format and volume creation process:

<pre>
root@precise:/home/kord/openstackgeek# fdisk /dev/sdb
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0xb39fe7af.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.

Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-62914559, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-62914559, default 62914559): 
Using default value 62914559

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
root@precise:/home/kord/openstackgeek# pvcreate -ff /dev/sdb1
  Physical volume "/dev/sdb1" successfully created
root@precise:/home/kord/openstackgeek# vgcreate nova-volumes /dev/sdb1
  Volume group "nova-volumes" successfully created
root@precise:/home/kord/openstackgeek#
</pre>

<p><code>Note:</code> Your device names may vary.</p>

### Installing MySql
The OpenStack components use MySQL for storing state information.  Start the install script for MySQL by entering the following:

<pre>
./openstack_mysql.sh
</pre>

You'll be prompted for a password to be used for each of the components to talk to MySQL:

<pre>
Enter a password to be used for the OpenStack services to talk to MySQL (users nova, glance, keystone): f00bar
</pre>

During the installation process you will be prompted for a root password for MySQL.  In our install example we use the same password, 'f00bar'.   At the end of the MySQL install you'll be prompted for your root password again.

<pre>
mysql start/running, process 8796
#######################################################################################
Creating OpenStack databases and users.  Use your database password when prompted.

Run './openstack_keystone.sh' when the script exits.
#######################################################################################
Enter password:
</pre>
	
After MySQL is running, you should be able to login with any of the OpenStack users and/or the root admin account by doing the following:

<pre>
mysql -u root -pf00bar
mysql -u nova -pf00bar nova
mysql -u keystone -pf00bar keystone
mysql -u glance -pf00bar glance
</pre>

### Installing Keystone
Keystone is OpenStack's identity manager.  Start the install of Keystone by doing:

<pre>
./openstack_keystone.sh
</pre>

You'll be prompted for a token, the password you entered for OpenStack's services, and your email address.  The email address is used to populate the user's information in the database.

<pre>
Enter a token for the OpenStack services to auth wth keystone: r4th3rb3t0k3n
Enter the password you used for the MySQL users (nova, glance, keystone): f00bar
Enter the email address for service accounts (nova, glance, keystone): user@foobar.com
</pre>

You should be able to query Keystone at this point.  You'll need to source the <code>stackrc</code> file before you talk to Keystone:

<pre>
. ./stackrc
keystone user-list
</pre>

Keystone should return a list of users:

<pre>
+----------------------------------+---------+------------------------+--------+
|                id                | enabled |         email          |  name  |
+----------------------------------+---------+------------------------+--------+
| b32b9017fb954eeeacb10bebf14aceb3 | True    | kordless@foobar222.com | demo   |
| bfcbaa1425ae4cd2b8ff1ddcf95c907a | True    | kordless@foobar222.com | glance |
| c1ca1604c38443f2856e3818c4ceb4d4 | True    | kordless@foobar222.com | nova   |
| dd183fe2daac436682e0550d3c339dde | True    | kordless@foobar222.com | admin  |
+----------------------------------+---------+------------------------+--------+
</pre>

### Installing Glance
Glance is OpenStack's image manager.  Start the install of Glance by doing:

<pre>
./openstack_glance.sh
</pre>

<p><code>Note:</code> You can safely ignore the <code>SADeprecationWarning</code> warning thrown by Glance when it starts.</p>

The script will download an Ubuntu 12.04 LTS cloud image from StackGeek's S3 bucket.  Go grab some coffee while it's downloading.  Once it's done, you should be able to get a list of images:

<pre>
glance index
</pre>

Here's the expected output:

<pre>
ID                                   Name                           Disk Format          Container Format     Size          
------------------------------------ ------------------------------ -------------------- -------------------- --------------
71b8b5d5-a972-48b3-b940-98a74b85ed6a Ubuntu 12.04 LTS               qcow2                ovf                       226426880
</pre>

We'll cover adding images via Glance in another guide soon.

### Installing Nova
We're almost done installing!  The last component is the most important one as well.  Nova is OpenStack's compute and network manager.  It's responsible for starting instances, creating snapshots and volumes, and managing the network.  Start the Nova install by doing:

<pre>
./openstack_nova.sh
</pre>

#### A Bit on Networking First
You'll immediately be prompted for a few items, including your existing network interface's IP address, the fixed network address, and the floating pool addresses:

<pre>
#############################################################################################################
The IP address for eth0 is probably 10.0.1.35. Keep in mind you need an eth1 for this to work.
#############################################################################################################
Enter the primary ethernet interface IP: 10.0.1.35
Enter the fixed network (eg. 10.0.2.32/27): 10.0.2.32/27
Enter the fixed starting IP (eg. 10.0.2.33): 10.0.2.33
#######################################################################################
The floating range can be a subset of your current network.  Configure your DHCP server
to block out the range before you choose it here.  An example would be 10.0.1.224-255
#######################################################################################
Enter the floating network (eg. 10.0.1.224/27): 10.0.1.224/27
Enter the floating netowrk size (eg. 32): 32
</pre>

<p><code>Note:</code> The script isn't very sophisticated and doesn't use defaults, so be sure you type in things carefully!  You can rerun the script if you mess up.  There's a nice subnet calculator <a href="http://www.subnet-calculator.com/">here if you need help with network masks</a>.  For reference, the <code>/27</code> above is called the 'mask bits' in the calculator.</p>

<p>The fixed network is a set of IP addresses which will be local to the compute nodes.  Think of these addresses as being held and routed internally inside any of the compute node instances.  If you decide to use a larger network, you could use something like <code>10.0.4.0/24</code> and a starting address of <code>10.0.4.1</code>.</p>

The floating network is a pool of addresses which can be assigned to the instances you are running.  For example, you could start a web server and map an external IP to it for serving a site on the Internet.  In the example above we use a private network because we're doing this at the house, but if your routing equipment/network allows it you could assign externally routed IPs to OpenStack instances.

#### Finish Installing Nova
Nova should finish installing after you enter all the network information.  When it's done, you should be able to get a list of images from Glance via Nova:

<pre>
nova image-list
</pre>

And get the expected output we saw earlier from Glance:

<pre>
root@precise:/home/kord/openstackgeek# nova image-list
+--------------------------------------+------------------+--------+--------+
|                  ID                  |       Name       | Status | Server |
+--------------------------------------+------------------+--------+--------+
| 71b8b5d5-a972-48b3-b940-98a74b85ed6a | Ubuntu 12.04 LTS | ACTIVE |        |
+--------------------------------------+------------------+--------+--------+
</pre>

### Installing Horizon
Horizon is the UI and dashboard controller for OpenStack.  Install it by doing:

<pre>
./openstack_horizon.sh
</pre>

When it's done installing, you'll be given a URL to access the dashboard.  You'll be able to login with the user 'admin' and whatever you entered earlier for your password.  If you've forgotten it, simply grep for it in your environment:

<pre>
env |grep OS_PASSWORD
</pre>

### Start Launching Instances
That's it!  You can now watch [the introduction video guide](https://vimeo.com/41807514) which gives an overview of creating a new project, users, and instances.

Be sure to drop us a line if you have any questions or corrections for this guide!
