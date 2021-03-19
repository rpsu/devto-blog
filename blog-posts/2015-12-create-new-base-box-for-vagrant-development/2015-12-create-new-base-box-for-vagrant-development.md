---
published: true
title: "Create new base box for Vagrant development"
cover_image: "https://raw.githubusercontent.com/rpsu/devto-blog/main/blog-posts/2015-12-create-new-base-box-for-vagrant-development/assets/slam-logo.jpg"
description: "A walkthrough of base image creation with Scientific Linux 7 (12/2015)"
tags: vagrant, scientific linux, sl7, localdev
series:
canonical_url: https://web.archive.org/web/20160624001745/http://www.exove.com/techblog/create-new-base-box-for-vagrant-development/

---

This blog post has been initially released in [Exove](https://www.exove.com)'s Tech blog, but [can only be found on the Archive Org's magnificient web archive](https://web.archive.org/web/20160624001745/http://www.exove.com/techblog/create-new-base-box-for-vagrant-development/) since the blog has disappeared after the publication day.  
As turned out, the "first post" ended up being the only one, not a serie of blog posts.

--

This is a first post of series "Get your local [Drupal] development environment in decent shape". Although this is targeted to Drupal developers, first three parts are fully CMS agnostic. So you could use this as a basis to get any other - say Wordpress, Ez or any other PHP based - CMS developing platform up and running.

## Distro of our choise is Scientific Linux 7

We at [http://www.exove.com](at Exove) are using [http://scientificlinux.org/](Scientific Linux 7) [SL7] as development environment distribution. It is a close relative to Red Hat Entrerprice Linux (and therefore also Centos et.al.) and there are [http://scientificlinux.org/about/why-make-scientific-linux/)(no significant differences to Red Hat) used so often as production site platform. It is fairly lightweight and as easy to configure as any Red Hat derivative.

Here you can find a list of Scientific Linux source mirrors. **Pick the closest one** to you instad of blindly copy-pasting all URL in this blog post. It is really not that difficult.

* [http://scientificlinux.org/downloads/sl-mirrors/]()

For this blog post's purpose the closest mirror is the one in Finland, so URL's are something like this:
[http://www.nic.funet.fi/pub/mirrors/scientificlinux.org/scientific/7/x86_64/iso/]()

## Tools you should have installed

These instructions assume you are using

* OS X (10.9 or newer should be fine)
* VirtualBox 5.0.0 or newer
* Vagrant 

Note that OS X El Capitano works best with VirtualBox 5 and Vagrant 1.7.3 or newer.

## Download  a Scientific Linux installation media 

Start by downloading SL7 installation media. Netinst-image is the smallest and all we need here. Other images have a lot of stuff we have no need for.

[http://www.nic.funet.fi/pub/mirrors/scientificlinux.org/scientific/7/x86_64/iso/SL-7.1-x86_64-netinst.iso]()

**NOTE** Check SHA1, SHA256 before using any such image.
 
## Set up VirtualBox 

Launch VirtualBox and start by creating new virtual machine instance with name 'SL7-server'. **Do not power it up**, but configure it's settings first (CMD + S sould bring up the configure options).

Turn off some unnecessary hardware options and configure network and "insert" the installation media:

1. General / Advanced / Shared clipboard

	* Set to **none**

1. General / Advanced / Drag'n'drop

	* Set to **none**

1. System / Motherboard / Pointing device:

	* Use **PS/2** (instead of available USB -options)

1. Audio

	* Set **disabled**

1. Network - Adapter 1 

	* Use **NAT**

1. Network - Adapter 1 / Advanced / Port Forwarding

	* Set forwarding SSH **2222 -> 22**

1. Network - Adapter 2,3,4 

	* Set **disabled** 

1. Ports / USB 

	* Enable USB Controller -> **unchecked**

1. SerialPort 

	* Enable Serial port -> **unchecked**

1. Storage 

	* Add installation media to DVD slot

## Install SL7 server in virtual machine
Power up the machine and start regular installation using UI provided by VirtualBox. These setting may vary depending your needs. For example most people might not set timezone and keyboard like this.

1. Set used keyboards (feel free to skip this part)  
	* Finnish Macintosh  
	* English US

2. Set timezone (feel free to skip this part)  
	* Europe/Helsinki

3. Enable networking (disabled by default)

4. Set installation source *to the closest mirror to you* 
	* [http://www.nic.funet.fi/pub/mirrors/scientificlinux.org/scientific/7/x86_64/os/]()  
	Wait for a while to fetch repository data from the mirror.

5. Choose installation type

	* We'll install now a minimal server with only limited amount of packages to keep the box small.

6. Install

	Start installation, and...

6. During the OS installation process (which will take a few minutes)

	* Set root passwd to vagrant
	* Add new user vagrant with password vagrant

7. REBOOT

	The installation media will be "ejected" automatically.

# Install/remove packages
Now you can install the tools you wish to have already in place in the base box. We'll add only two smallish packages, rest of the packages will be taken care of our Ansible roles when setting up the actual vagrant box instance.

### Remove unnecssary packages

1. Stop and remove Postfix __[optional]__

	`$ systemctl stop postfix
	$ yum remove postfix`

### Install other packages   

1. Install packages _nano_ and _wget_ (not required by any means, just a personally preferred tools over other options) __[optional]__

	`$ yum install nano bash-completion wget`

1. Ensure networking in on at boot

	`$ grep -Hn '^ONBOOT' /etc/sysconfig/network-scripts/*`
	
	This should result to 2 files witn `ONBOOT="yes"` -values.	If you see line `/etc/sysconfig/network-scripts/ifcfg-enp0s3:ONBOOT="no"`, edit the file:  
	`$ nano /etc/sysconfig/network-scripts/ifofg-enp0s3`

	..and check again that the value is now changed.  
	`$ grep -Hn '^ONBOOT' /etc/sysconfig/network-scripts/*`

1. Pass Grub 2 bootloader faster __[optional]__

	Set timeout 1 second, as default seems to be 5 seconds. This is changed only to make booting 4 seconds faster.  
	`$ sed -i 's/GRUB_TIMEOUT=[\d]{1,2}/GRUB_TIMEOUT=1/' /etc/default/grub`
	
	Compile Grub config using `grub2-mkconfig`tool:
	`$ grub2-mkconfig -o /boot/grub2/grub.cfg`

1. Turn SELinux off __[optional but recommended]__

	 As this box is a base for a local development vagrant box, there is no need to have SELinux enabled. It will most probably just make your life more difficult.  
	 Make sure to **reboot** after this; `init 6` will do it for you, though.  
	`$ sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
	$ init 6`

1. Add sudo-permission to vagrant user 

	`$ echo 'vagrant ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/vagrant`

1. Update sshd-configuration where needed

	`$ nano /etc/ssh/sshd_config`
	
	`Port 22 (or have Port -setting disabled by adding a # in front of the line)
	PubKeyAuthentication yes
	AuthorizedKeysFile .ssh/authorized_keys
	PermitEmptyPasswords no`

	Restart ssh daemon:  
	`$ service sshd restart`

1. Install vagrant insecure key

	`$ mkdir -p /home/vagrant/.ssh
	$ wget --no-check-certificate https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub -O /home/vagrant/.ssh/authorized_keys`
	
	Ensure we have the correct permissions set  
	`$ chmod 0700 /home/vagrant/.ssh
	$ chmod 0600 /home/vagrant/.ssh/authorized_keys
	$ chown -R vagrant:vagrant /home/vagrant/.ssh`

	Test ssh'ing as vagrant-user from you _current_ root session (create keys, test access, delete keys and known_hosts etc. after testing):
	
	`$ ssh -v vagrant@localhost`
	
	After succesful login exit and remove (all) keys from root's known_hosts -file:  
	`$ echo > /root/.ssh/known_hosts`
	
1. Disbable tty -requirement for sudoers

	`$ visudo`
	
	Comment out requiretty -setting changing this line  
	`Defaults requiretty`  
	to  
	`#Defaults requiretty`

	**NOTE** You may use simpler text editor _nano_ instad of _vim_ if you change your EDITOR value with:
	
	`$ EDITOR=$(which nano)
	export EDITOR`
	
	This change is temporary and is valid only within current shell session.  

1. Install DKMS -package required by VirtualBox Guest additions

	..as described in [http://www.linuxquestions.org/questions/linux-software-2/installing-dkms-on-rhel-7-0-a-4175510666/#post5239266](here)

	Closest download mirrors can be found in
	* [http://rpmfind.net//linux/RPM/epel/7/x86_64/d/dkms-2.2.0.3-30.git.7c3e7c5.el7.noarch.html]().
	
	Downlaod DKMS pacakge:  
	`$ wget ftp://rpmfind.net/linux/epel/7/x86_64/d/dkms-2.2.0.3-30.git.7c3e7c5.el7.noarch.rpm`

	Install DKMS requirements (these will install also 30+ other packages):  
	`$ yum install gcc kernel-devel kernel-headers`
	 
	Install the local DKMS package you've just downloaded:  
	`$ yum localinstall dkms-2.2.0.3-30.git.7c3e7c5.el7.noarch.rpm --nogpgcheck`
	
	Install Development Tools (this will install also 60+ other packages):  
	`$ yum groupinstall 'Development Tools'`
		
	Add/update kernel for VBoxGuestAdditions. This should install kernel version 3.10.0-229 or newer:  
	`$ yum update kernel`

1. Install VirtualBox guest additions: 
	
	.. as described in [http://download.virtualbox.org/virtualbox/5.0.0/]()

	`$ wget http://download.virtualbox.org/virtualbox/5.0.0/VBoxGuestAdditions_5.0.0.iso
	$ sudo mkdir /media/VBoxGuestAdditions
	$ sudo mount -o loop,ro VBoxGuestAdditions_4.3.8.iso /media/VBoxGuestAdditions
	$ /media/VBoxGuestAdditions/VBoxLinuxAdditions.run`

1. Correct missing locale -settings __[optional]__
	
	Add locale -settings to avoid repeating error message when logging through ssh to your box (as in every single time you use `vagrant ssh` -command). Edit this file:  
	`$ nano /etc/environments`
	
	Add these lines:  
	`LANG=en_US.utf-8
	LC_ALL=en_US.utf-8`
	
	Save and exit.
	
1. Give box a hostname __[optional]__

	 Use `nmcli` tool to to set a more desctiptive hostname (default is localhost.localdomain)  
	`$ nmcli general hostname sl7-dev.local`
	 
	 Check that changes was made with  
	`$ nmcli general hostname`

	**REBOOT NOW**:  
	`$ init 6`

1. Finalize and cleanup

	Ensure all packages are up to date, once more.  
	`$ yum update`

	Clean up yum cache  
	`$ yum clean all`
 
1. Minimize the size of VirtualBox virtual machine file
	
	Run this command to 1st fill up the virtual drive with an empty file and then removing the empty file. It will help VirtualBox to minimize the virtual drive.
	`$ dd if=/dev/zero of=/EMPTY bs=1M
	$ rm -f /EMPTY`

1. Clean up bash history __[optional]__

	Empty bash history and exit. Note that last session history is saved when exiting to the history file, so you might need to do this twice (and log out in between) to get empty history file:  
	`$ echo > .bash_history
	$ exit`

## Package the virtual machine for vagrant -use

Package newly baked Scientific Linux ready for Vagrant and do a test run.  
**Assuming** VBox server name `SL7-server`, change accoringly

`$ vagrant box remove test-sl7 2&>/dev/null
$ rm package.box 2&>/dev/null
$ vagrant package --base SL7-server
$ vagrant box add test-sl7 package.box
$ mkdir test
$ cd test
$ vagrant init test-sl7
$ vagrant up
$ vagrant ssh`

## Credit

Putting the credit wher it belongs! Main sources used to get this stuff in order are (but here just as a list without any spesific order of imortance):

* [http://docs.vagrantup.com/v2/virtualbox/boxes.html]()
* [http://aruizca.com/steps-to-create-a-vagrant-base-box-with-ubuntu-14-04-desktop-gui-and-virtualbox/]()
* [http://www.tecmint.com/remove-unwanted-services-in-centos-7/]()
* [https://vishmule.wordpress.com/2015/02/17/how-to-change-the-grub-timeout-in-rhel-7/]()
* [https://xuri.me/2015/09/06/resolve-setting-locale-failed-on-linux.html]()
* [http://www.itzgeek.com/how-tos/linux/centos-how-tos/change-hostname-in-centos-7-rhel-7.html]()

# Found a typo?

If you've found a typo, a sentence that could be improved or anything else that should be updated on this blog post, you can access it through a git repository and make a pull request. Instead of posting a comment, please go directly to <REPO URL> and open a new pull request with your changes.
