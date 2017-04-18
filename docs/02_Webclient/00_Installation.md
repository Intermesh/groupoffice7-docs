The webclient is built using mainly AngularJS and Angular Material.

In this instructions we've used Ubuntu 15.10. Other platforms can work as well 
but you have to figure it out yourself ;)

We assume the GroupOffice server has already been installed and in this example 
we're going to install it at /var/www/html/groupoffice-webclient. 
The URL will be http://localhost/groupoffice-webclient/app/

## Getting started

1. To get started with development you will minimally need to install:
	* [The GroupOffice server](http://groupoffice.io/index.php/REST_API/Installation) (Do this first if you haven't done this yet)
	* [GIT](https://git-scm.com/)
  * [NPM](https://www.npmjs.org/)
  * [bower](http://bower.io)
  * [sass](http://sass-lang.com/)	
	* [gulp](http://gulpjs.com/)

  Install git and npm with apt:

  `````````````````````````
  $ sudo apt-get install git npm
  `````````````````````````
	
	I had to install nodejs-legacy as well to avoid problems with a missing node binary:

  ``````````````````````````````````````````
  $ sudo apt-get install nodejs-legacy
  ``````````````````````````````````````````

2. Install gulp and bower globally:

	``````````````````````````````````````````````````````````````````````
	$ sudo npm -g install gulp bower
	``````````````````````````````````````````````````````````````````````

3. Clone the repository:

	First navigate into the document root:

	``````````````````````````````````````````````````````````````````````
	$ cd /var/www/html
	``````````````````````````````````````````````````````````````````````

  ``````````````````````````````````````````````````````````````````````
  $ git clone git@github.com:Intermesh/groupoffice-webclient.git
  ``````````````````````````````````````````````````````````````````````

4. Get all the required NPM modules by running (I had to clear the ~/tmp folder in my home directory because they are owned by root now):

  ``````````````````````````````````````````````````````````````````````
  $ sudo rm -Rf ~/tmp/
	$ sudo chown $USER -R ~/.npm
  $ cd groupoffice-webclient
  $ npm install
  ``````````````````````````````````````````````````````````````````````

  This will automatically run "bower install" too. This will install all bower
  components required.

5. Install SASS for the stylesheets

	Installing ruby
	``````````````````````````````````````````````````````````````````````
	$ sudo apt-get install ruby-full build-essential
	``````````````````````````````````````````````````````````````````````

	Installing rubygems
	``````````````````````````````````````````````````````````````````````
	$ sudo apt-get install rubygems-integration
	``````````````````````````````````````````````````````````````````````

	Install sass
	``````````````````````````````````````````````````````````````````````
	$ sudo gem install sass
	``````````````````````````````````````````````````````````````````````

	check if sass is working
	``````````````````````````````````````````````````````````````````````
	$ sass -v
	``````````````````````````````````````````````````````````````````````

	Run gulp sass script to create app/css/app.css:
	
	``````````````````````````````````````````````````````````````````````
	$ cd /var/www/html/groupoffice-webclient
	$ gulp sass
	``````````````````````````````````````````````````````````````````````
	
	You can use "gulp sass:watch" to watch for changes on the *.scss files.

	If you want to use Netbeans with the "sass" command then you'll need the 
	[sass-globbing](https://github.com/chriseppstein/sass-globbing) plugin:

	``````````````````````````````````````````````````````````````````````
	$ sudo gem install sass-globbing
	``````````````````````````````````````````````````````````````````````
	Then edit the project properties and set the command line arguments for 
	sass to "-r sass-globbing".

	We ran into an error with sass watch. We fixed this problem by running:
	``````````````````````````````````````````````````````````````````````
	$ echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
	``````````````````````````````````````````````````````````````````````

6. Configure
	Copy app/config.js.example to app/config.js and edit the API URl inside if 
	necessary.

	``````````````````````````````````````````````````````````````````````
	$ cd /var/www/html/groupoffice-webclient/app
	$ cp config.js.example config.js
	``````````````````````````````````````````````````````````````````````

7. Launch your browser and open http://localhost/groupoffice-webclient/app/

8. Updating

	If you want to update the repository you can do:

	``````````````````````````````````````````````````````````````````````
	$ cd /var/www/html/groupoffice-webclient/
	``````````````````````````````````````````````````````````````````````

	To update it:
	``````````````````````````````````````````````````````````````````````
	$ git pull
	``````````````````````````````````````````````````````````````````````

	Install possible new bower components:

	``````````````````````````````````````````````````````````````````````
	$ bower update
	``````````````````````````````````````````````````````````````````````

	Generate the CSS file when someone made changes to the SASS files:
	``````````````````````````````````````````````````````````````````````
	$ gulp sass
	``````````````````````````````````````````````````````````````````````

