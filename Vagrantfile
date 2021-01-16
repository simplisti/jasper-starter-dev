Vagrant.configure(2) do |config|
  config.vm.box = "debian/contrib-stretch64"

  config.vm.network :forwarded_port, guest: 3306, host: 3306 # sql

  config.vm.provider "virtualbox" do |v|
    v.memory = 512 # Bump up for initial composer update
  end

  config.vm.provision "shell", inline: <<-SHELL
        export DEBIAN_FRONTEND="noninteractive"

        # Upgrade sources for PHP 7.4+
		apt-get update && apt-get -y install apt-transport-https lsb-release ca-certificates &&
		wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg &&
		echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list &&

        # Install PHP 7.4+
		apt-get update && apt-get -y install php7.4-cli php7.4-gd php7.4-mysql php7.4-xdebug php7.4-zip php7.4-xml php 7.4-dom php7.4-curl php7.4-intl php7.4-mbstring &&
		cat /vagrant/.vagrant/config/php >> /etc/php/7.4/cli/php.ini

		# Install MariaDB
		apt-get -y install mysql-server php7.4-mysql libmysql-java &&
		cat /vagrant/.vagrant/config/mysql >> /etc/mysql/my.cnf &&
		service mysql restart &&
		mysql -u root -e "CREATE USER 'user'@'%' IDENTIFIED BY ''; GRANT ALL PRIVILEGES ON *.* TO 'user'@'%'; FLUSH PRIVILEGES;"

		# Tools
		apt-get -y install git curl unzip

		# Install composer package manager
		curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

		# Install Symfony binary globally
        wget https://get.symfony.com/cli/installer -O - | bash && mv /root/.symfony/bin/symfony /usr/local/bin/symfony
    SHELL

    config.vm.provision "shell", inline: <<-JASPER
        sed -i "s/stretch main/stretch main non-free contrib/g" /etc/apt/sources.list && apt-get update && apt-get -y install msttcorefonts

        cd /tmp
        cp /vagrant/.java/jasperstarter-3.0.0.zip	jasperstarter-3.0.0.zip
        unzip jasperstarter-3.0.0.zip
        mv jasperstarter /opt;
        cd /opt/jasperstarter/bin
        chmod 777 *
        ln -s  /usr/share/java/mysql.jar  /opt/jasperstarter/jdbc/mysql.jar
        apt-get install -y default-jre
        # JasperStarter end

        echo "export PATH=$PATH:/opt/jasperstarter/bin"
    JASPER

	config.vm.synced_folder ".", "/vagrant", type: "virtualbox"
	config.ssh.extra_args = ["-t", "cd /vagrant/project; bash --login"]
end
