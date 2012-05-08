---
layout: post
title: "Increase the Size of Devstack's Volumes"
category: 
tags: []
by: Kord Campbell
---
{% include JB/setup %}
I've been whacking around on OpenStack for about two months now.  Devstack is awesome sauce and all, but it's default limitation for creating any cloud volumes larger than a total of 2GB has been driving me crazy.  I keep needing volumes which are bigger, end up trying to create them, then get the infinite creation spinner in the UI.  The only way I can get it to go away (but still not be able to make any new volumes) is to whack it from the nova database in MySQL.  That's silly.

Turns out there's a slight mention of this on the [devstack page for a multi-node install](http://devstack.org/guides/multinode-lab.html).  Let's borrow some of those instructions and put them on my blog so I can get massive page rank.  Or not.

The fix is fairly trivial.  Assuming you have a spare hard drive laying around, stuff it into your OpenStack node.  If you don't have a spare drive, go buy two SSDs, install one, and mail me the other one.  I could really use it.

## Fix That Shizzle

The rest of this <code>howto</code> assumes you have a spare empty drive mounted at <code>/dev/sdb</code>.  It's not my fault if you bork your box.

1. Start off by running fdisk from the prompt:

<pre>sudo fdisk /dev/sdb</pre>

Again, be sure you use the actual file handle for the drive!

2. Now create a partition new partition by typing <code>n</code>, then a <code>p</code> for a primary partition.  Use the default start and end values.

3. Set the partition type to Linux LVM by typing <code>t</code> then <code>8e</code>.  Hit <code>w</code> to write out the partition to the drive and exit.

4. From the prompt again, run:

<pre>
pvcreate -ff /dev/sdb1
vgcreate nova-volumes /dev/sdb1
</pre>

The <code>-ff</code> is just there to force the creation of the volume.  You can leave it off if you are scared.

That's it.  The <code>stack.sh</code> will take care of the rest!  You should now be able to create very large volumes at will now!

