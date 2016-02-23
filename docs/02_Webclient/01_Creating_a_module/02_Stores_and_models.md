GroupOffice uses stores and models in the controller to handle data. A model 
typically represents a record model on the server and a store is a collection of 
models.

In this example we'll create a list to show all bands from the server that uses 
a store. The server will need the bands module from the [tutorial](http://intermesh.io/index.php/REST_API/Create_a_module/Folders_and_files).


# Dependency and main controller
First we'll add that dependency to the client module. If the server module is 
not available this client module won't be available either. We'll also define
a controller for our main view too. Don't reload the web client until you have
created the controller. Otherwise you'll get an error.

Edit 'app/ux/tutorial/modules/bands/module.js':
````````````````````````````````````````````````````````````````````````````````
'use strict';

GO.module('UX.Tutorial.Modules.Bands', ['GO.Core']).
		config(['GO.Core.launcherProvider', function (launcherProvider) {		
				
				// Add the server module dependency here:
				
				launcherProvider.add('bands', 'Bands', ['UX\\Modules\\Bands\\Module']);
			}]).
		config(['$stateProvider', function($stateProvider) {

				$stateProvider
						.state('bands', {
							url: "/bands",
							templateUrl: 'ux/tutorial/modules/bands/views/main.html',


							//Add the controller here.
							controller: 'UX.Tutorial.Modules.Bands.Controller.Main',


							data: {
								noAuth: false //optional set to true to disable authentication
							}
						});
			}]);
```````````````````````````````````````````````````````````````````````````````

# Band model

Create the model for the Band. See the Webclient API documentation for the available
properties and methods. When a model is modified and saved it will send out the
modified properties to the server.

Create the file: 'app/ux/tutorial/modules/bands/model/band.js':

```````````````````````````````````````````````````````````````````````````````
'use strict';

angular.module('UX.Tutorial.Modules.Bands').
				factory('UX.Tutorial.Modules.Bands.Model.Band', [
						'GO.Core.Data.Model', 
						function (Model) {
						
						//Extend the base model and set default return proeprties
						var Band = GO.extend(Model, function (baseParams) {
														
							this.$parent.constructor.call(this, 'bands', baseParams || {
								returnProperties: "*,albums,customfields"
							});

						});

						//Set the relations with their primary keys so the model knows that 
						//this is a related model and only the modified properties and the 
						//primary keys should be sent.
						
						Band.prototype.$relationKeys = {
							albums: ['id', 'bandId'],
							customfields: ['id']
						};

						return Band;
					}]);

```````````````````````````````````````````````````````````````````````````````

# Main controller

Now create the main controller file 
'ux/tutorial/modules/bands/controller/main.js' for the main view:

```````````````````````````````````````````````````````````````````````````````
'use strict';

GO.module('UX.Tutorial.Modules.Bands').
				controller('UX.Tutorial.Modules.Bands.Controller.Main', [
					'$scope',
					'UX.Tutorial.Modules.Bands.Model.Band',
					function ($scope, Band) {

						$scope.band = new Band();

						$scope.store = $scope.band.getStore({
							returnProperties: "id,name"
						});

						$scope.store.load();
					}]);


```````````````````````````````````````````````````````````````````````````````

# Main view

We'll use the 'go-list' directive in the main view to present the data. Change 
'ux/tutorial/modules/bands/view/main.html' into:

```````````````````````````````````````````````````````````````````````````````

<div class="go-list" layout="column">
	<go-list-toolbar store="store">
		<div class="md-toolbar-tools">
			<md-button aria-label="{{::'Open side navigation'| goT}}" ng-click="toggleSidenav('left')" hide-gt-md class="md-icon-button">
				<md-icon class="mdi-menu"></md-icon>
			</md-button>

			<span flex></span>

			<go-search-button></go-search-button>

			<md-menu md-position-mode="target-right target">
				<md-button aria-label="{{::'More options'| goT}}" class="md-icon-button" ng-click="$mdOpenMenu($event)">
					<md-icon md-menu-origin class="mdi-dots-vertical"></md-icon>
				</md-button>
				<md-menu-content>
					<md-menu-item>
						<md-button ng-click="store.reload()">{{"Refresh"| goT}}</md-button>
					</md-menu-item>
				</md-menu-content>
			</md-menu>
		</div>			
	</go-list-toolbar>


	<go-list store="store" flex>

		<item index="model.name" ui-sref="bands.band({bandId: model.id})">
			<div class="md-list-item-text">
				<h3>{{model.name}}</h3>
			</div>
		</item>

		<empty-state>
			<md-icon class="mdi-account"></md-icon>
			<p>{{"No bands found"| goT}}</p>
		</empty-state>

	</go-list>
</div>


<div flex ui-view layout="column" class="go-info-panel"></div>


```````````````````````````````````````````````````````````````````````````````


Now reload the webclient and navigate to the bands module. It should show the 
list of bands created in the API.

The list uses an infinite scrolling feature to deal with large stores.