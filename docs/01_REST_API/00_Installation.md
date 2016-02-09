Installation for development
============================

To install the GroupOffice server follow these steps:

1. Make sure you have the required software:
   ``````````````````````````````````````````````````````````````````
   $ sudo apt-get install git curl php5-mcrypt php5-curl
   ``````````````````````````````````````````````````````````````````

2. clone the repository:

   It's recommended to put the server outside the webserver document root for
   security reasons.		

   ``````````````````````````````````````````````````````````````````
	 $ cd /var/www
   $ git clone git@git.intermesh.nl:groupoffice-server.git
   ``````````````````````````````````````````````````````````````````

3. Install composer if you haven't done that already. On Ubuntu do:

   ```````````````````````````````````````````````````
    $ curl -sS https://getcomposer.org/installer | php
    $ sudo mv composer.phar /usr/local/bin/composer
   ```````````````````````````````````````````````````
4. Run composer in the working directory:

   ``````````````````````````
   $ cd groupoffice-server
   $ composer install
   ``````````````````````````
5. Optional:
	 If you didn't put the library in the document root (recommended) then put 
	 the /public/index.php file (or symlink it) in a web server root and make sure 
	 the path to IFW.php is correct:

   ```````````````````````````````````
   require("../lib/IFW/IFW.php");
   ```````````````````````````````````
   Also adjust the path to config.php that you will create in step 7:
   `````````````````````````````````````````````````````
   $app = new App(require('../config.php'));
   `````````````````````````````````````````````````````
6. Create a MySQL database called "go7".

7. Create the data folder where Group-Office can store files.

   ``````````````````````````````````````````````````````
	 $ mkdir /var/www/groupoffice-server/data
   $ sudo chown www-data:www-data /var/www/groupoffice-server/data
   ``````````````````````````````````````````````````````
8. Copy config.php.example to config.php and adjust it with the correct database 
	parameters and data storage path configured in step 6 + 7.

	Now it should work. Launch index.php:

	/index.php

	This will redirect to a system test page:

	/index.php/system/check

	Optionally you can create an alias for apache:

	Alias /api /path/to/index.php

	This allows pretty URL's like /api/system/check

	It should output that all is OK ;). It doesn't look pretty but it's not meant to
	be because it's just an API.

9. Open /index.php/system/install to install the database.

10. Now you can login (See the docs on how to do it with POSTMan or install the web client).

	The default login is:

	Username: admin
	Password: Admin1!





