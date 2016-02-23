Creating a module in the AngularJS client
-----------------------------------------

The following third party libraries are available in the webclient. Use them
where you can:

1. [Angular Material](https://material.angularjs.org/latest/)
2. [Material Design Icons](https://materialdesignicons.com/) we use the font for icons.


### Create folder structure

The root folder "ux" is reserved for user extensions so you won't have to put
any code in the core modules folder.

Create the module folder "app/ux/tutorial/modules/bands".

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
Create "ux/tutorial/modules/bands/module.js" that will initialize the module.

In this file we'll add the states of the module and we can configure it before 
the application is running. We've used [UI Router](https://angular-ui.github.io/ui-router/site/#/api/ui.router) 
to route URL's to the controllers and views.

Example:

```````````````````````````````````````````````````````````````````````````````````````````````````````````````````````
'use strict';

//Use GO.module instead of angular.module so it will be added to the app dependencies
GO.module('UX.Tutorial.Modules.Bands', ['GO.Core']).
		//Create a launcher
		config(['GO.Core.launcherProvider', function (launcherProvider) {								
				launcherProvider.add('bands', 'Bands', ['UX\\Modules\\Bands\\Module']);
			}]).
		config(['$stateProvider', function($stateProvider) {

				// Now set up the states
				$stateProvider
						.state('bands', {
							url: "/bands",
							templateUrl: 'ux/tutorial/modules/bands/views/main.html',
							data: {
								noAuth: false //optional set to true to disable authentication
							}
						});
			}]);
```````````````````````````````````````````````````````````````````````````````````````````````````````````````````````

**Note** that you must use "GO.module" instead of "angular.module". "GO.module" also
adds the module to the main module's dependencies in app.js.


### Create the main view

Now create the main view 'ux/tutorial/modules/bands/views/main.html' that's defined in the 
$stateProvider above:

```````````````````````````````````````````````````````````````````````````````````````````````````````````````````````
<h1>Hello World!</h1>
```````````````````````````````````````````````````````````````````````````````````````````````````````````````````````

### Done!
Now refresh the Angular app and it should have the hello world launcher available.
