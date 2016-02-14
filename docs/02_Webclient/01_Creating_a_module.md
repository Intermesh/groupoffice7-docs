Creating a module in the AngularJS client
-----------------------------------------

The following third party libraries are available in the webclient. Use them
where you can:

1. [Angular Material](https://material.angularjs.org/latest/)
2. [Material Design Icons](https://materialdesignicons.com/)


### Create folder structure

Create the module folder "app/customizations/modules/bands".

All JS files within this folder are loaded automatically by index.php or the
build script.

Optionally create these subfolders:

- controller: for the controller JS files.
- models: Model JS files
- views: HTML templates
- language: Localization JS files
- scss: For the SASS styles
- services: For angular services

### Create module.js
Create "app/modules/helloworld/module.js" that will initialize the module.

In this file we'll add the states of the module and we can configure it before the application is running.

Example:

```````````````````````````````````````````````````````````````````````````````````````````````````````````````````````
'use strict';

//Use GO.module instead of angular.module so it will be added to the app dependencies
GO.module('GO.Modules.HelloWorld', ['GO.Core']).
		//Create a launcher
		config(['GO.Core.launcherProvider', function (launcherProvider) {								
				launcherProvider.add('helloworld', 'Hello World', []);
			}]).
		config(['$stateProvider', function($stateProvider) {

				// Now set up the states
				$stateProvider
						.state('helloworld', {
							url: "/helloworld",
							templateUrl: 'modules/helloworld/views/main.html',
							data: {
								noAuth: false //optional set to true to disable authentication
							}
						});
			}]);
```````````````````````````````````````````````````````````````````````````````````````````````````````````````````````

### Create views/main.html

```````````````````````````````````````````````````````````````````````````````````````````````````````````````````````
<h1>Hello World!</h1>
```````````````````````````````````````````````````````````````````````````````````````````````````````````````````````


### Done!
Now refresh the Angular app and it should have the hello world launcher available.
