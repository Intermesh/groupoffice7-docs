Now that we have a list of our bands it's time to add a band view. In the main 
view there's already an ui-sref directive that points to the "bands.band" state:

``````````````````````````````````````````
ui-sref="bands.band({bandId: model.id})"
``````````````````````````````````````````

Let's define that state in "ux/tutorial/modules/bands/module.js":

'use strict';

````````````````````````````````````````````````````````````````````````````````
GO.module('UX.Tutorial.Modules.Bands', ['GO.Core'])
	.config([
		'GO.Core.Providers.ClientModulesProvider',
		function (ClientModulesProvider) {

			var module = ClientModulesProvider.add('bands', ['UX\\Modules\\Bands\\Module']);
			module.addLauncher('Bands');
		
		}])
	.config(['$stateProvider', function($stateProvider) {
		$stateProvider
				.state('bands', {
					url: "/bands",
					templateUrl: 'ux/tutorial/modules/bands/views/main.html',
					controller: 'UX.Tutorial.Modules.Bands.Controller.Main',
					data: {
						noAuth: false //optional, set to true to disable authentication. It defaults to false.
					}
				})

				//Add this sub state:

				.state('bands.band', {
					url: "/{bandId:[0-9]*}",
					templateUrl: 'ux/tutorial/modules/bands/views/band.html',
					controller: 'UX.Tutorial.Modules.Bands.Controller.Band'
				});
	}]);
````````````````````````````````````````````````````````````````````````````````


# Create band view

The view template must be created in 'ux/tutorial/modules/bands/views/band.html':


````````````````````````````````````````````````````````````````````````````````
<go-hook name="bands.band" flex layout="column">
	<go-mask active="band.deleted">
		<md-button class="md-raised md-warn" ng-click="band.unDelete()">{{"Undo delete"| goT }}</md-button>		
	</go-mask>


	<md-toolbar class="md-hue-1" go-scroll-class>
		<div class="md-toolbar-tools">
			<md-button class="md-icon-button hide-gt-md" ng-click="$state.go('^')">
				<md-icon class="mdi-chevron-left"></md-icon>
			</md-button>

			<span flex>

				<span class="go-scroll-hidden">
					{{band.name}}
				</span>

			</span>

		</div>

		<div class="go-display-panel-header" layout="row" layout-align="start center">
			<span class="go-avatar"><md-icon class="mdi-music-note"></md-icon></span>
			<span>
				<strong>{{band.name}}</strong><br />
				<small>A music band</small>
			</span>

		</div>


	</md-toolbar>

	<div class="go-progress-container"><md-progress-linear md-mode="indeterminate" ng-if="band.$busy"></md-progress-linear></div>
	<md-tabs md-selected="0">
		<!-- goto is defined in $rootScope in app/controller/body-controller.js" -->
		<md-tab ng-click="goto('info')">Info</md-tab>
	</md-tabs>


	<md-content flex layout="column" class="go-grey">

		<md-card id="info">
			<md-toolbar class="md-hue-1">
				<div class="md-toolbar-tools">
					<h2 flex>Info</h2>
					<md-button class="md-icon-button" ng-disabled="!band.permissions.update" ng-click="edit(band)">
						<md-tooltip>
							{{"Edit"| goT}}					
						</md-tooltip>
						<md-icon class="mdi-pencil"></md-icon>
					</md-button>
				</div>
			</md-toolbar>

			<md-card-content>

				<md-list>
					<md-list-item class="md-2-line">
						<md-icon class="mdi-clock"></md-icon>
						<div class="md-list-item-text">
							<h3>{{::band.createdAt| date:"longDate"}} {{"at"| goT}} {{::band.createdAt| date:"shortTime"}}</h3>
							<p>{{"Created at"| goT}}</p>
						</div>
					</md-list-item>	

					<md-list-item class="md-2-line" ng-if="band.lastLogin">
						<div class="md-list-item-text md-offset">
							<h3>{{::band.lastLogin| date:"longDate"}} {{"at"| goT}} {{::band.lastLogin| date:"shortTime"}}</h3>
							<p>{{"Created at"| goT}}</p>
						</div>
					</md-list-item>

					<md-list-item class="md-2-line">
						<md-icon class="mdi-account-multiple"></md-icon>
						<div class="md-list-item-text">
							<h3>
								<ul class="comma-list">
									<li ng-repeat="album in band.albums">
										{{album.name}}
									</li>
								</ul>							
							</h3>
							<p>{{"Albums"| goT}}</p>
						</div>
					</md-list-item>			

				</md-list>


			</md-card-content>
		</md-card>

	</md-content>
</go-hook>




````````````````````````````````````````````````````````````````````````````````



# Create band controller

The controller will take the band ID from '$stateParams' and read it into the 
model.

Create 'ux/tutorial/modules/bands/controller/band.js':

````````````````````````````````````````````````````````````````````````````````
'use strict';

GO.module('UX.Tutorial.Modules.Bands').
				controller('UX.Tutorial.Modules.Bands.Controller.Band', [
					'$scope',
					'$stateParams',
					function ($scope, $stateParams) {
						$scope.band.readIf($stateParams.bandId);
					}]);

````````````````````````````````````````````````````````````````````````````````


# Reload

Reload the app and the subview will be loaded into this element of the main view:

````````````````````````````````````````````````````````````````````````````````
<div flex ui-view layout="column" class="go-info-panel"></div>
````````````````````````````````````````````````````````````````````````````````

You should be able to see the band with the albums. There's an edit button but
it doesn't function yet. We'll get to that in the next chapter.
