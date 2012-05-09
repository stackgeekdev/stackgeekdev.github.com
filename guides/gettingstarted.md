Howdy, I’m Kord Campbell, founder of Stackgeek.  This video will walk you through getting started with OpenStack by creating a new project to be used for development.  We’ll launch a couple of instances and install apache in the process.

This video guide assumes you’ve already installed OpenStack on at least a single node.  If you haven’t installed OpenStack yet, hop over to our tutorial pages and read the ‘installing openstack on ubuntu’ guide.  There’s also a video guide for the installation process.  Keep in mind this demo is done using a single node deployment of OpenStack, but it could also be done using a multi-node deployment.

We’ll start our demo by logging into our OpenStack dashboard.  The install process will have prompted you for a password which you’ll use now to login as the admin user.

OpenStack allows you to isolate users, instances and volumes from each other using projects.  You could create a project for an individual developer, or for a development team.  To create a new project, navigate to the admin tab, and click on projects. 

Click on create new project, and enter a name and description for the project.  In this example we’ll use the name “Development”. Click on the ‘create project’ button to create the project.

Now let’s create a new user.  Click on users.  Click on ‘create user’.  We’ll name this user ‘developer’.  Select the project the user will manage and then click create user.  

We’ll go back to projects page briefly to assign another user to the project.  Click on the pulldown next to the project and click on modify users.  use the add to project button to add users to the new project.  pick the role you’d like the user to have for the project and then click add.

For development work, our developers need about 2VPUs,1GB of RAM and 5GB of main disk for their instances.  Let’s create a new flavor for them by clicking on flavors, and then create flavor.  We’ll call this instance m2.medium and give it 2 VPUs, 1GB of RAM and 5GB of main disk, and 1GB of ephemeral disk.  We’ll create a permanent volume for the user once we login.  Click on create flavor to save.

We’ll logout and log back in as the new user.  The new user won’t be able to see other project’s instances or volumes, but they’ll be able to switch between the various projects for which they are a member.

Before we launch an instance, we’ll need to create a developer key which OpenStack can use for authenticating ssh access.  Click on access and security and then click on create keypair.  The private key portion will be downloaded locally.

To launch an instance we’ll click on instances and volumes, and then launch instance.  I’m only using a single image for 12.04 here, but if you add other images with Glance, Openstack’s image manager, you’ll see them here. 

Click on ‘launch’ and then name the instance.  We’ll also tell the instance to install apache while it’s booting.  We do that by writing a small bash script in the user data field.  Pick the flavor you created earlier, and the keypair for the user, and then click on launch instance.

We need a public IP address for this instance and the instance will need to allow ssh and web traffic into it.  Click on access and security again and then click on allocate ip to project.  select the default image pool, in this case mine are IPs local to my home network, and then associate the IP with the instance we just launched.

Click on ‘edit’ rules to add a firewall rule to the instance’s security group.  we’ll add port 22 and 80 for ssh and apache, and then we’ll add ICMP rules to allow pings.

Let’s take a look at our instance now and see where it’s at with booting.  click on instance and volumes again, and then click on the instance name to see the details.  Click on the log tab at the top to view the boot log.  The log view updates every few seconds, so just wait until you see the ssh host key printout before trying to log in to the instance.

Moving over to a terminal, we’ll want to move our key into our .ssh directory, and then change the permissions on it to be private.  We’ll then use the key to log into the new box with it’s public IP address.  Again, my IP here is private, but in a different configuration the IP could be a publicly routable address.

Now we’re logged into the instance, we can view a process list.  Apache is running because we installed it at boot time, so we should be able to go over and view the new site the box is hosting.  

Let’s go ahead and move the www directory over to the new volume we created so any changes we make to it are preserved across instance terminations and starts.  We’ll create a partition using fdisk, and then format it with ext3.

Now let’s move the directory over and create a link to the new directory on the permanent disk.  

Finally we’ll check our content is still being served, and we can begin doing development on our box.  If we want we can terminate this box and launch a new box with different packages, or even snapshot this box and volume so other users can use it for further development.  These snapshots can eventually be used for launching production instances by other projects.

If you have any questions, be sure to use our contact form on the website.  We’re here to help you build your own private cloud!

Have a great day!
