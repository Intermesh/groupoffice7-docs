Creating a module in the AngularJS client
-----------------------------------------

The following third party libraries are available in the webclient. Use them
where you can:

1. [Angular Material](https://material.angularjs.org/latest/)
2. [Google Material Design Icons](https://material.io/icons/) we use the font for icons.


### Create folder structure

We want to separate code per owner. So inside the "app/modules" there's the 
default "groupoffice" folder for the standard modules. Each vendor should create
it's own folder there. We'll use "tutorial".

Create the module folder "app/modules/tutorial/modules/bands".

From now on all paths will be relative to the path above.

All JS files within this folder are loaded automatically by index.php or the
build script.

Optionally create these subfolders:

- controller: for the controller JS files.
- model: Model JS files
- view: HTML templates
- language: Localization JS files
- scss: For the SASS styles
- service: For angular services

### Create module.js
Create "module.js" that will initialize the module.

In this file we'll add the states of the module and we can configure it before 
the application is running. We've used [UI Router](https://angular-ui.github.io/ui-router/site/#/api/ui.router) 
to route URL's to the controllers and views.

Example:

```````````````````````````````````````````````````````````````````````````````````````````````````````````````````````
'use strict';

//Use GO.module instead of angular.module so it will be added to the app dependencies
GO.module('GO.Modules.Tutorial.Bands', ['GO.Core'])
	.config([
		'GO.Core.Providers.ClientModulesProvider',
		function (ClientModulesProvider) {

			var module = ClientModulesProvider.add('bands', []);
			module.addLauncher('Bands');
		}])
	.config([
		'$stateProvider',
		function ($stateProvider) {
			$stateProvider
							.state('bands', {
								url: "/bands",
								templateUrl: 'ux/tutorial/modules/bands/views/main.html',								
								data: {
									noAuth: false //optional, set to true to disable authentication. It defaults to false.
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
