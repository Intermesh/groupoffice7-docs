Now that we've got the band view it's time for an edit dialog. We already put
the edit button in the band view but it's not functional yet:

````````````````````````````````````````````````````````````````````````````````
<md-button class="md-icon-button" ng-disabled="!band.permissions.update" ng-click="edit(band)">
	<md-tooltip>
		{{"Edit"| goT}}					
	</md-tooltip>
	<md-icon class="mdi-pencil"></md-icon>
</md-button>
````````````````````````````````````````````````````````````````````````````````

Because we also want to use the same edit() function for adding new band we'll
put the edit function in the main controller.

Edit 'ux/tutorial/modules/bands/controller/main.js':


````````````````````````````````````````````````````````````````````````````````

'use strict';

GO.module('GO.Modules.Tutorial.Bands').
				controller('GO.Modules.Tutorial.Bands.Controller.Main', [
					'$scope',
					'GO.Modules.Tutorial.Bands.Model.Band',
					'GO.Core.Widgets.Dialog',
					'$state',
					function ($scope, Band, Dialog, $state) {

						$scope.band = new Band();

						$scope.store = $scope.band.getStore({
							returnProperties: "id,name"
						});

						$scope.store.load();

						$scope.edit = function (band) {

							if (!band) {
								band = new Band();
								band.addStore($scope.store);
							}

							Dialog.show({
								editModel: band,
								templateUrl: 'ux/tutorial/modules/bands/views/band-edit.html'
							}).then(function (dialog) {
								dialog.close.then(function (band) {
									if (band) {
										$state.go("bands.band", {bandId: band.id});
									}
								});
							});
						};

					}]);


````````````````````````````````````````````````````````````````````````````````

The edit() function takes the band model as argument. If it's not given it will
create a new model and it will connect it to the store so that the store is
updated when the model changes.

The band model is passed as "editModel" option to 'GO.Core.Components.Modal'. 
This will automatically add a "save()" and "cancel()" function to the modal 
controller. It will also automatically do a GET request to the server to "read"
the model.

Create the band edit view 'ux/tutorial/modules/bands/views/band-edit.html':

````````````````````````````````````````````````````````````````````````````````
<form layout="column" name="bandForm" go-submit="save()" novalidate>
	
	<md-toolbar>
		<div class="md-toolbar-tools">
			<md-button type="button" class="md-icon-button" ng-click="cancel()">
				<md-tooltip>{{"Close"| goT}}</md-tooltip>
				<md-icon class="mdi-chevron-left"></md-icon>
			</md-button>

			<span flex></span>
			<md-button type="submit" class="md-icon-button">
				<md-tooltip>{{"Save"| goT}}</md-tooltip>
				<md-icon class="mdi-check"></md-icon>
			</md-button>
		</div>
	</md-toolbar>

	<md-content class="md-padding" flex>

		<md-input-container  class="md-block">
			<label>{{"Name"| goT}}</label>
			<input ng-model="model.name" name="name" required go-autofocus>
			<div ng-messages="modelForm.name.$error" role="alert">
				<div ng-message="required">
					{{"This field is required"| goT}}
				</div>
			</div>
		</md-input-container>
		
		
		<div class="go-input-multiple">
			<div layout ng-init="album.$formName = 'album_' + $index" ng-repeat="album in model.albums">
				<md-input-container flex>							
					<label>{{"Album name"| goT}}</label>
					<input type="text" name="{{album.$formName}}" ng-model="album.name" required go-autofocus="!album.id">
					<div ng-messages="bandForm[album.$formName].$error" role="alert">
						<div ng-message="required">
							{{"This field is required" | goT}}
						</div>
					</div>
				</md-input-container>
				
				<md-input-container flex>							
					<label>{{"Genre"| goT}}</label>
					<input type="text" name="{{album.$formName}}" ng-model="album.genre" required>
					<div ng-messages="bandForm[album.$formName].$error" role="alert">
						<div ng-message="required">
							{{"This field is required" | goT}}
						</div>
					</div>
				</md-input-container>

				<md-button aria-label="{{::'Delete' | goT}}" class="md-icon-button" type="button"  ng-click="model.albums.splice($index, 1);">
					<md-icon class="mdi-delete"></md-icon>				
				</md-button>
			</div>
			
			<md-button type="button" ng-click="model.albums.push({})"><md-icon class="mdi-plus"></md-icon>	 {{"Add album"| goT}}</md-button>
		</div>

	</md-content>

</form>



````````````````````````````````````````````````````````````````````````````````

This view has the name field in it and uses an "ng-repeat" to render the band 
input fields.


# Add button
Add a button to add new bands to the main view. Edit

'ux/tutorial/modules/bands/main.html' and add the following code right after
the '</go-list>' tag:

````````````````````````````````````````````````````````````````````````````````
<div class="go-fab-list">
	<md-button class="md-fab"  ng-click='edit()'>
		<md-icon class="mdi-plus"></md-icon>
		<md-tooltip md-direction="left">{{"Add"| goT}}</md-tooltip>
	</md-button>
</div>
````````````````````````````````````````````````````````````````````````````````

Now reload the angular view and a green circular button should appear in the 
bottom right corner of the list.


= Dialog controller

If you want to add extra functionality to the controller then you can create
a dialog controller for it.


Change the edit function in the main controller into:

````````````````````````````````````````````````````````````````````````````````
$scope.edit = function (band) {

	if (!band) {
		band = new Band();
		band.addStore($scope.store);
	}

	Dialog.show({
		editModel: band,
		templateUrl: 'ux/tutorial/modules/bands/views/band-edit.html',

		//add controller here
		controller: 'GO.Modules.Tutorial.Bands.Controller.BandEdit'

	}).then(function (dialog) {
		dialog.close.then(function (band) {
			if (band) {
				$state.go("bands.band", {bandId: band.id});
			}
		});
	});
};
````````````````````````````````````````````````````````````````````````````````


Create the file 'ux/tutorial/modules/bands/controller/band-edit.js':

````````````````````````````````````````````````````````````````````````````````
'use strict';

GO.module('GO.Modules.Tutorial.Bands').
				controller('GO.Modules.Tutorial.Bands.Controller.BandEdit', [
					'$scope',
					'close', // You can inject the 'close' function. When called the dialog closes.
					'read', // You can inject the 'read' promise. This is resolved when the passed 'editModel' is done with it's read request to the server.
					function ($scope, close, read) {
						
						read.then(function(result) {
							
							if(result.model.isNew()) {
								result.model.setAttributes({
									name: 'Default'
								});
							}
						});
						
					}]);
````````````````````````````````````````````````````````````````````````````````

This controller doesn't do anything useful. It set's the band name to 'Default'
if the model is new and after the read of the model is finished. The read 
promise of the model is injectable because the read request is made by the
edit dialog. You can also inject a 'close' function.