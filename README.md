## Overview

Vagrant setup for the Getting Into Tech training course.

Rather than setup laptops with development environments (which could involve a wide variety of hardware/OS), just go for Vagrant VM to get a consistent set of tools.

## Pre-requistes

The following should be installed on the students laptops

- VirtualBox - current version at time of writing 5.2.2 [download from here](https://www.virtualbox.org/wiki/Downloads)
- Vagrant - current version is 2.0.1 [download here](https://www.vagrantup.com/downloads.html)

For the creation of the base box used for this Vagrant setup the following is needed

- Raspberry Pi Desktop - November 2017 release [iso from here](https://www.raspberrypi.org/downloads/raspberry-pi-desktop/)


## Installation of tools


### Mac

Installing VirtualBox and Vagrant on a Mac is fairly straight forward since they have packages to download, open and install.  


### Linux

<TBC>


### Windows

<TBC>


## Installing VM

Clone this git repository or simply copy the VagrantFile to a suitable location.  Using Terminal, Command Prompt or 
similar command line run the following command

```
vagrant up
```

The above command will download the base box (this could take some time over a slow internet connection) and will 
then start and configure the relevant packages.  Wait until the vagrant command has completed before attempting
to use the virtual machine (the GUI/desktop will appear immediately but it would be advisable to wait while 
packages are downloaded).

You can now use the virutal machine as normal and can also shutdown/pause using the VirtualBox GUI.  Once 
you are finished with the vagrant VM and wish to delete it use the following command

```
vagrant destroy
```

This will erase *everything* and you will need to run the "up" command again to recreate the virtual machine
from scratch again.

NOTE: Read the next section for tips on how to use the VM such that any work you do is preserved.


## Using the VM

The virutal machine is setup to allow you to see certain locations from the host (the physical 
computer on which you're running vagrant/VirtualBox) within the guest (the Raspberry Pi desktop
environment).  

Specifically, in this case, the mount point /vagarnt_home is setup to point to the HOME directory 
on the users host system, e.g. /Users/<user> on MacOS or c:\Users\<user> on Windows, etc.  The
VagrantFile provisioning script also creates a directory, GitWork under HOME on the host and 
links it to /home/pi/work.

This means that any files saved to /home/pi/work will be written to the host directory and will 
survive after the virtual machine is destroyed.

NOTE: Students MUST save work to /home/pi/work to have to persist after the VM is destroyed.


## Base Box

### Initial Creation

Creating the initial base box followed these steps.  

NOTE: ***You do NOT need to create the base box*** This work has already been carried out and the base box
is already available on [vagrant cloud](https://app.vagrantup.com/hiklasltd/boxes/RPiDesktopNov2017).  
This base box is free/open/public so should remain available, however if you depend on this heavily I
would *strongly* suggest that you take a copy under your own account on Vagrant Cloud.


* Start VirtualBox
* For me I had to install the latest VirtualBox essentials
* Create new VM with name "RPiDesktop", Type "Linux", version "Debian 32bit"
* Memory size 1Gb
* Create Virtual Disk now
* VDI (VirtualBox Disk Image)
* Dynamically allocated
* Change size to 16Gb (just in case)
* Create disk 
* Click on settings, navigate to storage
* Click on the optical disc on left, then the button to select an image on the right
* Locate the RPi Desktop image
* Start the VM
* From boot menu select "Install" (quicker than using graphical install)
* Select keyboard language (I'm using British English)
* Choose "Guided - use entire disk" partitioning 
* Choose "All files in one partition"
* Accept the partition scheme
* Start installation
* Accept GRUB setup on MBR, select the /dev/sda device
* The VM will reboot and startup in the new desktop
* Shutdown (from the Raspberry Pi menu at the top the left hand side)

### Important Changes

If you are on hardware that uses SSD/Flash drives then you *MUST* take precautions to reduce the amount of swapping that 
the Linux kernel will do.  Otherwise your SSD/Flash is going to wear out very very quickly

Start the VM and run the terminal, execute the following commands

```
sudo bash -l
echo 1 > /proc/sys/vm/swappiness
``` 

To make this setting permanent add the following to the /etc/sysctl.conf file

```
vm.swappiness=1
```

### Base Box Setup

Following the [Vagrant base box instructions](https://www.vagrantup.com/docs/virtualbox/boxes.html)

Adding the vagrant user

```
groupadd -g 1001 vagrant
useradd -g 1001 -u 1001 -G users,sudo -m -d /home/vagrant -c "Vagrant User" vagrant
```

Created the /home/vagrant/.ssh/authorized_keys file and added the [vagrant public key](https://github.com/mitchellh/vagrant/tree/master/keys)
Ensured that the permissions are correct on this file

```
chown vagrant.vagrant /home/vagrant/.ssh/authorized_keys 
chmod 700 /home/vagrant/.ssh/authorized_keys
```

Added a file /etc/sudoers.d/020_vagrant-nopasswd with the following contents

```
vagrant ALL=(ALL) NOPASSWD: ALL
```

Install SSH - this doesn't appear to be included by default for the Raspberry Pi desktop

```
sudo apt-get install ssh
sudo systemctl enable ssh
sudo systemctl start ssh
```

Install packages for building VirtualBox guest additions

```
sudo apt-get install linux-headers-$(uname -r) build-essential dkms
```

Inserted the VirtualBox additions CD and ran the following command

```
sudo sh /media/cdrom/VBoxLinuxAdditions.run
```

This should build the guest additions and install them.  Rebooted the virutal machine to make sure it picks up the correct kernel.
Shutdown the box again ready for packaging.


### Package the Base Box

The setup above has created a Vagrant Base Box which can be used for further development and customisation.  This now needs to be packaged
and uploaded to Vagrant Cloud.

The packaging command is as follows:

```
vagrant package --base RPiDesktop
```
