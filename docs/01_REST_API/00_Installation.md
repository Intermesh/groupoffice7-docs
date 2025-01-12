Installation for development
============================

In this instructions we've used Ubuntu 15.10. Other platforms can work as well 
but you have to figure it out yourself ;)

In this example we're going to install it at /var/www/groupoffice-server. 
The API URL will be http://localhost/api

To install the GroupOffice server follow these steps:

1. Make sure you have the required software:

	* [GIT](https://git-scm.com/)
	* Apache2, PHP5.6+ and MySQL5.5+
	* curl (Just required to install composer)
	* [Composer](https://getcomposer.org)

	Install Apache, PHP, MySQL and sudo. We also need some specific extensions:
	````````````````````````````````````````````````````````````````````````````
	$ sudo apt-get install mysql-server libapache2-mod-php php-mysql php-curl php-zip
	````````````````````````````````````````````````````````````````````````````

	Install GIT and curl.
	````````````````````````````````````````````````````````````````````````````
	$ sudo apt-get install git curl
	````````````````````````````````````````````````````````````````````````````

	Install composer:
	````````````````````````````````````````````````````````````````````````````
	 $ curl -sS https://getcomposer.org/installer | php
	 $ sudo mv composer.phar /usr/local/bin/composer
	````````````````````````````````````````````````````````````````````````````

2. clone the repository:

	It's recommended to put the server outside the web server document root for
	security reasons.		

	````````````````````````````````````````````````````````````````````````````
	$ cd /var/www
	$ git clone git@github.com:Intermesh/groupoffice-server.git
	````````````````````````````````````````````````````````````````````````````

3. Run composer in the working directory. This will install all required PHP 
	libraries:

	````````````````````````````````````````````````````````````````````````````
	$ cd /var/www/groupoffice-server
	$ composer install
	````````````````````````````````````````````````````````````````````````````
4. You must create the web accessible access point and we do this with an apache
	Alias.

	Create a file /etc/apache2/conf-available/groupoffice-server.conf:

	````````````````````````````````````````````````````````````````````````````
		Alias /api /var/www/groupoffice-server/public/index.php

		<Directory /var/www/groupoffice-server/public>
		require all granted
		</Directory>	
	````````````````````````````````````````````````````````````````````````````

	Enable this config:
	````````````````````````````````````````````````````````````````````````````
	$ sudo a2enconf groupoffice-server
	$ sudo service apache2 reload
	````````````````````````````````````````````````````````````````````````````

5. Create a MySQL database and user for GroupOffice. For example named "go7".

	Here are some example commmands. If you use a root password then add -p.
	````````````````````````````````````````````````````````````````````````````
	$ mysql -u root -e "CREATE DATABASE go7"
	$ mysql -u root -e "GRANT ALL PRIVILEGES ON go7.* TO 'go7'@'localhost' IDENTIFIED BY 'secret' WITH GRANT OPTION"
	````````````````````````````````````````````````````````````````````````````

6. Create the data folder where Group-Office can store files.

	````````````````````````````````````````````````````````````````````````````
	$ mkdir /var/www/groupoffice-server/data
	$ sudo chown www-data:www-data /var/www/groupoffice-server/data
	````````````````````````````````````````````````````````````````````````````
7. Create config.php configuration file by copying the defaults:

	````````````````````````````````````````````````````````````````````````````
	$ cd /var/www/groupoffice-server
	$ cp config.php.example config.php
	````````````````````````````````````````````````````````````````````````````

	Set the correct database parameters and data storage path created in step 6 + 7.

	Now it should work. Launch the API:

	http://localhost/api

	This will redirect to a system test page:

	http://localhost/api/system/check

	It should output that all is OK ;). It doesn't look pretty but it's not meant to
	be because it's just an API.

8. Open http://localhost/api/system/install to install the database.

9. Install command line wrapper script
	Create the file /usr/local/bin/groupoffice and put this in:

	```````````````````````````````````````````````````````````````````````
	#!/bin/bash
	sudo -u www-data /var/www/groupoffice-server/bin/groupoffice $@	
	```````````````````````````````````````````````````````````````````````

	Make this file executable:

	```````````````````````````````````````````````````````````````````````
	sudo chown +x /usr/local/bin/groupoffice
	```````````````````````````````````````````````````````````````````````

	This allows you to run command line actions easily. For example:

	```````````````````````````````````````````````````````````````````````
	groupoffice system/upgrade
	```````````````````````````````````````````````````````````````````````

	```````````````````````````````````````````````````````````````````````
	groupoffice accounts/sync
	```````````````````````````````````````````````````````````````````````

	```````````````````````````````````````````````````````````````````````
	groupoffice cron/run
	```````````````````````````````````````````````````````````````````````
10. Install cron job in /etc/cron.d/groupoffice-server :

	````````````````````````````````````````````````````````````````````````````
	* * * * * www-data /usr/local/bin/groupoffice cron/run
	````````````````````````````````````````````````````````````````````````````

11. Read the [Usage chapter](http://groupoffice.io/index.php/REST_API/Usage) about 
	how to login and use the API.

	The default login is:

	Username: admin
	Password: Admin1!

12. Now [install the Web client](http://groupoffice.io/index.php/Webclient/Installation)

13. Updating, When you want to update use these commands:

	````````````````````````````````````````````````````````````````````````````
	$ git pull
	$ composer update
	$ groupoffice system/upgrade
	````````````````````````````````````````````````````````````````````````````

14. Create a second command to run CLI as the current user:

	```
	ln -s /var/www/groupoffice-server/bin/groupoffice /usr/local/bin/sugroupoffice
	```