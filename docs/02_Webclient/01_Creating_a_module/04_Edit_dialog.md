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

GO.module('UX.Tutorial.Modules.Bands').
				controller('UX.Tutorial.Modules.Bands.Controller.Main', [
					'$scope',
					'UX.Tutorial.Modules.Bands.Model.Band',

					//Add this dependency:
					'GO.Core.Components.Modal',
					function ($scope, Band, Modal) {

						$scope.band = new Band();

						$scope.store = $scope.band.getStore({
							returnProperties: "id,name"
						});

						$scope.store.load();


						//Add this edit function
						$scope.edit = function (band) {

							if (!band) {
								band = new Band();
								band.addStore($scope.store);
							}

							Modal.show({
								editModel: band,
								templateUrl: 'ux/tutorial/modules/bands/views/band-edit.html'
							});
						};

					}]);

````````````````````````````````````````````````````````````````````````````````

The edit() function takes the band model as argument. If it's not given it will
create a new model and it will connect it to the store so that the store is
updated when the model changes.

The band model is passed as "editModel" option to 'GO.Core.Components.Modal'. 
This will automatically add a "save()" and "cancel()" function to the modal 
controller.

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
					<input type="text" name="{{album.$formName}}" ng-model="album.name" required go-autofocus="($index > 0 || model.id) && !album.id">
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