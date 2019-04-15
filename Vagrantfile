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
  config.vm.synced_folder "~/var/www/mysql-test", "/var/lib/mysql", 
    nfs: true, 
    linux__nfs_options: ["no_root_squash"]
    # @TODO: This bit fails on initial build, as the 'mysql' user doesn't exist yet. It didn't seem to make a difference anyway...
    # owner: "mysql",
    # group: "mysql",
    # mount_options: ["dmode=755,fmode=644"]
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

    echo " "
    echo "########## Updating packages ##########"
    echo " "
    export DEBIAN_FRONTEND=noninteractive
    echo "Updating packages"
    sudo apt-get update
    

    echo " "
    echo "########## Upgrading packages ##########"
    echo " "
    sudo apt-get -y upgrade
    

    # Set up some swapdisk space, to allow for low RAM limits on VM. This prevents crashes when using Composer
    echo " "
    echo "########## Creating swapdisk ##########"
    echo " "
    sudo /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
    sudo /sbin/mkswap /var/swap.1
    sudo /sbin/swapon /var/swap.1


    # Install some handy utilities
    echo " "
    echo "########## Installing ZIP and PipeViewer ##########"
    echo " "
    apt-get install -y zip unzip


    echo " "
    echo "########## Installing PPA (apache2) ##########"
    echo " "
    # apache2 is added here to recognise the 'a2dismod' and 'a2enmod' commands below
    # sudo apt-get install -y apache2
    sudo add-apt-repository ppa:ondrej/php
    sudo add-apt-repository ppa:ondrej/apache2
    echo "Copying in 000-default.conf site configuration to /etc/apache2/sites-available/"
    sudo cp /home/vagrant/transfer/000-default.conf /etc/apache2/sites-available/
    sudo cd /etc/apache2 && mkdir logs
    sudo service apache2 restart
    sudo apt-get update


    # Install PHP
    echo " "
    echo "########## Installing PHP ##########"
    echo " "
    sudo apt-get -y install php libapache2-mod-php php-mysql php-mbstring php-common php-xml php-gd php-curl


    # Install MySQL Server
    echo " "
    echo "########## Preparation for MySQL ##########"
    echo " "
    echo "If you get MySQL errors in the out, check that there isn't another VM already occupying the synced ~/var/lib/mysql directory."
    echo "If there is, save a copy of the directory so you don't destroy another VM's databases, and empty it."
    debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
    debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'
    sudo apt-get install -y mysql-server
    sudo mysqld --initialize
    

    # Tweak mods
    echo " "
    echo "########## Tweaking apache and php setup ##########"
    echo " "
    sudo a2dismod mpm_event
    sudo a2enmod actions mpm_prefork rewrite php7.3
    sudo phpenmod mbstring
    sudo service apache2 restart


    # Install Composer
    # echo " "
    # echo "########## Installing Composer ##########"
    echo " "
    curl -sS https://getcomposer.org/installer | php
    mv composer.phar /usr/local/bin/composer


    # Install Drush
    echo " "
    echo "########## Installing Drush 8.x ##########"
    echo " "
    sudo composer global require drush/drush:8.*
    cd && composer require drush/drush:8.*
    echo "export PATH='~/.config/composer/vendor/bin:$PATH'" >> /home/vagrant/.profile
    source /home/vagrant/.profile
    

    # Install WP CLI
    echo " "
    echo "########## Installing WP CLI ##########"
    echo " "
    curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
    chmod +x wp-cli.phar
    mv wp-cli.phar /usr/local/bin/wp
    

    # Install NodeJs
    echo " "
    echo "########## Installing NodeJS ##########"
    echo " "
    curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash - > /dev/null 2>&1
    sudo apt-get install -y nodejs > /dev/null 2>&1
    

    # Install Astrum (for Pattern Library building)
    echo " "
    echo "########## Installing Astrum ##########"
    echo " "
    sudo npm install -g astrum


    # Add some aliases to make the VM shell much sexy
    # echo " "
    # echo "########## Adding aliases ##########"
    echo " "
    alias aa="ls -alF"
    alias cd="cd .."
    alias cc="clear"
    alias cc="exit"


    echo " "
    echo "########## Allow the Vagrant to use Drupal's temporary file directory ##########"
    echo " "
    echo 'Changing ownership of /var/tmp'
    sudo chown vagrant:vagrant /var/tmp


    echo " "
    echo "########## Source Acquia site aliases ##########"
    echo " "
    # You must download them into ~/var/www/transfer first, for the correct version of Drush,
    # and also import/set up SSH keys for them to work
    runuser -l vagrant -c 'tar -C $HOME -xf $HOME/transfer/acquia-cloud.drush-8-aliases.tar.gz'

  SHELL

  # Always Start Apache and MySQL
  config.vm.provision "shell", inline: "echo '########## Starting apache ##########' && sudo service apache2 start",
    run: "always"
  config.vm.provision "shell", inline: "echo '########## Starting mysql ##########' && sudo service mysql start",
    run: "always"

  config.vm.provision "shell", run: 'always', inline: <<-SHELL
    echo " "
    echo "#### Restarting Apache"
    echo " "
    sudo service apache2 restart
    echo "########## VM provisioning complete! ##########"
  SHELL

  # Auto-create the databases listed stored in /transfers/databases.txt
  # config.vm.provision "shell", inline: <<-SHELL
  #   runuser -l vagrant -c 'bash ~/scripts/db-create.sh'
  # SHELL

end

# @TODO:
# 
# 1. Get Drush installing the correct version - installs 5.x, not 8.x - is Composer required? - YES, DONE
# 2. Auto-create relevant databases - DB list in transfers/databases.txt - DONE
# 3. Auto-download latest PROD backup from live hosting
# 4. Auto-import latest PROD backup into matching database in VM. Maybe store the databases inside folders matching the DB name? - DONE, see scripts/db-import.sh
# 5. Auto-add and configure stage_file_proxy
# 6. Set up a local dev modules folder and auto-enable them
# 