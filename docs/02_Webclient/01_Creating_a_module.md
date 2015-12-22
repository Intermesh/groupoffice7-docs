Creating a module in the AngularJS client
-----------------------------------------

### Create folder structure
1. Create folder "app/modules/helloworld"
2. Create folder "app/modules/helloworld/js" for the script files.
3. Create folder "app/modules/helloworld/views" for the HTML views.

### Create module.js
Create "app/modules/helloworld/module.js" that will initialize the module.

In this file we'll add the states of the module and we can configure it before the application is running.

Example:

```````````````````````````````````````````````````````````````````````````````````````````````````````````````````````
'use strict';

//Use GO.module instead of angular.module so it will be added to the app dependencies
GO.module('GO.helloworld').
		//Create a launcher
		config(['launcherProvider', function (launcherProvider) {								
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


### Script loading
Add this script to app/index.html

### Create views/main.html

```````````````````````````````````````````````````````````````````````````````````````````````````````````````````````
<h1>Hello World!</h1>
```````````````````````````````````````````````````````````````````````````````````````````````````````````````````````


### Done!
Now refresh the Angular app and it should have the hello world icon on the start screen!
