# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "hiklasltd/RPiDesktopNov2017"

  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "forwarded_port", guest: 443, host: 8443

  config.vm.network "private_network", ip: "192.168.33.10"

  config.vm.synced_folder ENV["HOME"], "/vagrant_home", owner: "pi", group: "pi"

  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI - since we're using this as our desktop 
    vb.gui = true
 
    # Customize the amount of memory on the VM:
    vb.memory = "1024"
  end
  

  config.vm.provision "shell", inline: <<-SHELL

    # Set the PI password to something simple
    echo "pi:wibble" | chpasswd

    # Create a work directory
    if [ ! -e /vagrant_home/GitWork ] ; then mkdir /vagrant_home/GitWork ; else echo "GitWork directort already there" ; fi
    ln -s /vagrant_home/GitWork /home/pi/work
    chown pi:pi -h /home/pi/work

    # Install packages
    apt-get update
    apt-get install -y netbeans
    apt-get install -y apache2 
    apt-get install -y mariadb-server mariadb-client
    apt-get install -y php php-cli libapache2-mod-php php-xdebug
    apt-get install -y autoconf

    # Start processes
    systemctl enable apache2
    systemctl start apache2
  SHELL
end
