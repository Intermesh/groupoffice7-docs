The webclient is built using mainly AngularJS and Angular Material.

## Getting started
1. To get started with development you will minimally need to install:
	* The GroupOffice server (Do this first if you haven't done this yet)
  * [NPM](https://www.npmjs.org/)
  * [bower](http://bower.io)
  * [sass](http://sass-lang.com/)	

  On Ubuntu:

  Install git and npm with apt:

  `````````````````````````
  $ sudo apt-get install git npm
  `````````````````````````

  I had to work around a bug with (see https://github.com/joyent/node/issues/3911):

  ``````````````````````````````````````````
  $ sudo ln -s /usr/bin/nodejs /usr/bin/node
  ``````````````````````````````````````````

2. Clone the repository:

	First navigate into the document root:

	``````````````````````````````````````````````````````````````````````
	$ cd /var/www/html
	``````````````````````````````````````````````````````````````````````

  ``````````````````````````````````````````````````````````````````````
  $ git clone git@git.intermesh.nl:groupoffice-webclient.git
  ``````````````````````````````````````````````````````````````````````

3. Get all the required NPM modules by running (I had to clear the tmp folder in my home directory becasue they are owned by root now):

  ``````````````````````````````
  $ sudo rm -Rf /home/mschering/tmp/
  $ cd groupoffice-webclient
  $ npm install
  ``````````````````````````````

  This will automatically run "bower install" too. This will install all bower
  components required.

Now navigate to the "app" folder with your browser.

4. Install SASS for the stylesheets

	Installing ruby
	````````````````````````````````````````````````
	$ sudo apt-get install ruby-full build-essential
	````````````````````````````````````````````````

	Installing rubygems
	```````````````````````````````````````````
	$ sudo apt-get install rubygems-integration
	```````````````````````````````````````````

	Install sass
	`````````````````````
	$ sudo gem install sass
	`````````````````````

	check if sass is working
	`````````
	$ sass -v
	`````````

	Run gulp sass script to create app/css/app.css:
	
	``````````````````````````
	$ cd groupoffice-webclient
	$ gulp sass
	```````````````````````````
	
	I personally configure Netbeans to generate app/css/app.css for me but you can 
	also use "gulp sass:watch" to watch for changes on the *.scss files.

5. Configure
	Copy app/config.js.example to app/config.js and edit the API URl inside if 
	necessary.
	

6. Launch your browser and open http://localhost/groupoffice-webclient/app/