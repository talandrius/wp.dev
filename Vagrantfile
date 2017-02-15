# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

    config.vm.box = "scotch/box"
    config.vm.network "private_network", ip: "192.168.33.10"
    config.vm.synced_folder ".", "/var/www", :mount_options => ["dmode=777", "fmode=666"]
    config.vm.hostname = (ENV['hostname'] if ENV['hostname'] != nil) || (File.read('hostname').strip if File.file?('hostname')) || "wp.dev"
    File.write "hostname", "#{config.vm.hostname}\n"

    config.vm.provision "shell", inline: <<-SHELL

        # Install Composer.
        echo 'Installing composer'
        curl -Ss https://getcomposer.org/installer | php
        mv composer.phar /usr/bin/composer

        # Create the directory.
        mkdir -p /var/www/public/
        cd /var/www/public/

        # Download adminer.
        echo 'Downloading adminer'
        wget www.adminer.org/static/download/4.2.5/adminer-4.2.5-mysql-en.php -O adminer.php > /dev/null 2>&1

        # Create the database.
        echo 'Creating the database'
        mysql -u root -proot -e 'CREATE DATABASE IF NOT EXISTS `#{config.vm.hostname}`'

        # Update WP-CLI.
        echo 'Updating WP-CLI'
        wp cli update --allow-root --yes

        # Download WordPress.
        echo 'Downloading WordPress'
        wp core download --allow-root

        # Create WordPress configuration file.
        echo 'Creating WordPress configuration file'
        wp core config --allow-root               \
            --dbname=#{config.vm.hostname}        \
            --dbuser=root                         \
            --dbpass=root                         \
            --dbhost=localhost                    \
            --dbprefix=wp_

        # Run the installation.
        echo 'Running the installation'
        wp core install --skip-email --allow-root \
            --url="http://#{config.vm.hostname}"  \
            --title="Box"                         \
            --admin_user="admin"                  \
            --admin_password="password"           \
            --admin_email="admin@#{config.vm.hostname}"

        # Uninstall Hello and Akismet.
        echo 'Uninstalling Hello and Akismet'
        wp plugin uninstall --allow-root hello akismet

        # Done.
        echo 'Done!'

    SHELL

end