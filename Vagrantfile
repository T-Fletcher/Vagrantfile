# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/xenial64"
  config.vm.network "private_network", ip: "192.168.33.10"

  # Synced folders
  # For some reason, absolute Host machine paths don't work, must use ~/.. 
  # In some cases, we need to enforce permissions and owner:user groups
  config.vm.synced_folder "~/Documents/repositories", "/var/www/html",
    owner: "vagrant",
    group: "vagrant",
    mount_options: ["dmode=755,fmode=644"]
  config.vm.synced_folder "~/var/www/site-php", "/var/www/site-php"
  config.vm.synced_folder "~/var/www/configs", "/etc/apache2/sites-enabled"
  config.vm.synced_folder "~/var/www/logs", "/var/www/logs"
  config.vm.synced_folder "~/var/www/mysql", "/var/lib/mysql", 
    nfs: true, 
    linux__nfs_options: ["no_root_squash"],
    owner: "mysql",
    group: "mysql",
    mount_options: ["dmode=755,fmode=644"]
  config.vm.synced_folder "~/var/www/transfer", "/home/vagrant/transfer",
    owner: "vagrant",
    group: "vagrant",
    mount_options: ["dmode=755,fmode=644"]
  config.vm.synced_folder "~/var/www/scripts", "/home/vagrant/scripts",
    owner: "vagrant",
    group: "vagrant",
    mount_options: ["dmode=755,fmode=644"]

  # Shell commands to run on boot
  # NOTE that all commands are run as root!
  config.vm.provision "shell", inline: <<-SHELL

    # Update and Install Utils
    export DEBIAN_FRONTEND=noninteractive
    echo "Updating packages"
    sudo apt-get update
    echo ""
    echo "#############################"
    echo "Installing ZIP"
    echo "#############################"
    apt-get install -y zip unzip
    
    # Install PPA (Apache2)
    echo ""
    echo "#############################"
    echo "Installing PPA"
    echo "#############################"    
    sudo add-apt-repository ppa:ondrej/php
    sudo add-apt-repository ppa:ondrej/apache2
    sudo apt-get update

    # Install PHP 7.2+
    echo ""
    echo "#############################"
    echo "Installing PHP"
    echo "#############################"
    sudo apt-get -y install php7.2 php7.2-common php7.2-cli php7.2-fpm
    
    # Install MySQL Server
    echo ""
    echo "#############################"
    echo "Installing MySQL"
    echo "#############################"
    debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
    debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'
    apt-get install -y mysql-server
    
    # Tweak mods
    sudo a2dismod mpm_event
    sudo a2enmod actions mpm_prefork rewrite php7.0
    sudo phpenmod mbstring

    # Install Composer
    echo ""
    echo "#############################"
    echo  "Installing Composer"
    echo "#############################"
    curl -sS https://getcomposer.org/installer | php
    mv composer.phar /usr/local/bin/composer
    # TODO: This does nothing as a project is not initialised at this point, so $HOME/.composer doesn't exist. 
    echo "export PATH=\"$HOME/.composer/vendor/bin:$PATH\"" >> /home/vagrant/.profile
    source /home/vagrant/.profile
    
    # @TODO: Drush sticks with v5.10, not 8.x
    echo ""
    echo "#############################"
    echo "Installing Drush 8.x"
    echo "#############################"
    # runuser -l vagrant -c 'composer global require drush/drush:8.*'
    runuser -l vagrant -c 'cd && composer require drush/drush:8.*'
    
    # WP CLI
    echo ""
    echo "#############################"
    echo "Installing WP CLI"
    echo "#############################"
    curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
    chmod +x wp-cli.phar
    mv wp-cli.phar /usr/local/bin/wp
    
    # Install NodeJs - and Astrum (for Pattern Library building)
    echo ""
    echo "#############################"
    echo "Installing NodeJS and Astrum"
    echo "#############################"
    curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash - > /dev/null 2>&1
    sudo apt-get install -y nodejs > /dev/null 2>&1
    sudo npm install -g astrum

    # Add some aliases, fix file ownership and permissions etc
    # echo ""
    # echo "#############################"
    # echo "Adding aliases"
    # echo "#############################"
    # runuser -l vagrant -c 'cat /home/vagrant/transfer/.bash_aliases > /home/vagrant/.bash_aliases'
    # chmod 644 /home/vagrant/.bash_aliases
    # chown vagrant:vagrant /home/vagrant/.bash_aliases
    # runuser -l vagrant -c 'source /home/vagrant/.bash_aliases'

    # Allow the Vagrant to use Drupal's temporary file directory
    chown vagrant:vagrant /var/tmp

  SHELL

  # Always Start Apache and MySQL
  config.vm.provision "shell", inline: "sudo service apache2 start",
    run: "always"
  config.vm.provision "shell", inline: "sudo service mysql start",
    run: "always"

  config.vm.provision "shell", run: 'always', inline: <<-SHELL
    # Reboot apache, as sometimes it doesn't pick up the sites...
    echo "#### Restarting Apache"
    sudo service apache2 restart
    echo "*** VM provisioning complete! ***"
  SHELL
end

# @TODO:
# 
# 1. Get Drush installing the correct version - installs 5.x, not 8.x - is Composer required?
# 2. Auto-create relevant databases - DB list in transfers/databases.txt
# 3. Auto-download latest PROD backup from live hosting
# 4. Auto-add and configure stage_file_proxy
# 