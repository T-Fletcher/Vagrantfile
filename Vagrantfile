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
  config.vm.synced_folder "~/Documents/repositories", "/var/www/html"
  config.vm.synced_folder "~/var/www/site-php", "/var/www/site-php"
  config.vm.synced_folder "~/var/www/configs", "/etc/apache2/sites-enabled"
  config.vm.synced_folder "~/var/www/logs", "/var/www/logs"
  config.vm.synced_folder "~/var/www/mysql", "/var/lib/mysql", nfs: true, :linux__nfs_options => ["no_root_squash"]
  config.vm.synced_folder "~/var/www/transfer", "/home/vagrant/transfer"
  config.vm.synced_folder "~/var/www/scripts", "/home/vagrant/scripts"

  # Shell commands to run on boot
  # NOTE that all commands are run as root!
  config.vm.provision "shell", inline: <<-SHELL
    
    # Update and Install Utils
    export DEBIAN_FRONTEND=noninteractive
    sudo apt-get update
    apt-get install -y zip unzip
    
    # Install Apache
    apt-get install -y apache2
    
    # Install PHP
    sudo apt-get -y install php libapache2-mod-php php-mysql php-mbstring php-common php-xml php-gd php-curl
    
    # Install MySQL
    debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
    debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'
    apt-get install -y mysql-server
    
    # Tweak mods
    sudo a2dismod mpm_event
    sudo a2enmod actions mpm_prefork rewrite php7.0
    sudo phpenmod mbstring

    # Composer and Drush - needs work, SSH in and do the commented out stuff manually.
    curl -sS https://getcomposer.org/installer | php
    mv composer.phar /usr/local/bin/composer
    
    # @TODO: Drush
    runuser -l vagrant -c 'composer global require drush/drush:8.x'
    echo "export PATH=\"$HOME/.composer/vendor/bin:$PATH\"" >> /home/vagrant/.profile
    source /home/vagrant/.profile
    
    # WP CLI
    curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
    chmod +x wp-cli.phar
    mv wp-cli.phar /usr/local/bin/wp
    
    # Install NodeJs - and Astrum (for Pattern Library building)
    curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash - > /dev/null 2>&1
    sudo apt-get install -y nodejs > /dev/null 2>&1
    sudo npm install -g astrum

    # Add some aliases, fix file ownership and permissions etc
    cat /home/vagrant/transfer/.bash_aliases > /home/vagrant/.bash_aliases
    chmod 644 /home/vagrant/.bash_aliases
    chown vagrant:vagrant /home/vagrant/.bash_aliases
    source /home/vagrant/.bash_aliases

    # Reboot apache, as sometimes it doesn't pick up the sites...
    sudo service apache2 restart
  SHELL

  # Always Start Apache and MySQL
  config.vm.provision "shell", inline: "sudo service apache2 start",
    run: "always"
  config.vm.provision "shell", inline: "sudo service mysql start",
    run: "always"
end

# @TODO:
# 
# 1. Get Drush working
# 2. Auto-create relevant databases - DB list in transfers/databases.txt
# 3. Auto-download latest PROD backup from live hosting
# 4. Auto-add and configure stage_file_proxy
# 