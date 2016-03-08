Modules often require settings pages. The core implements a settings page which 
can be extended by the modules. It also has an accounts page which can be extended
as well. Think of E-mail, Twitter and CalDAV accounts for example.

To add a settings page you can use the 'addSettingsOption' function of the 
client module.

Edit 'ux/tutorial/modules/bands/module.js':

````````````````````````````````````````````````````````````````````````````````
'use strict';

//Use GO.module instead of angular.module so it will be added to the app dependencies
GO.module('UX.Tutorial.Modules.Bands', ['GO.Core'])
	.config([
		'GO.Core.Providers.ClientModulesProvider',
		function (ClientModulesProvider) {

			var module = ClientModulesProvider.add('bands', ['UX\\Modules\\Bands\\Module']);
			module.addLauncher('Bands');
			module.addSettingsOption('settings.bands', 'Bands', 'mdi-music-note');

		}])
	.config([
		'$stateProvider',
		function ($stateProvider) {
			$stateProvider
							.state('bands', {
								url: "/bands",
								templateUrl: 'ux/tutorial/modules/bands/views/main.html',
								controller: 'UX.Tutorial.Modules.Bands.Controller.Main',
								data: {
									noAuth: false //optional, set to true to disable authentication. It defaults to false.
								}
							})
							.state('bands.band', {
								url: "/{bandId:[0-9]*}",
								templateUrl: 'ux/tutorial/modules/bands/views/band.html',
								controller: 'UX.Tutorial.Modules.Bands.Controller.Band'
							})
							.state('settings.bands', {
								url: "/bands",
								templateUrl: 'ux/tutorial/modules/bands/views/settings/form.html'
							})
							;
		}]);

````````````````````````````````````````````````````````````````````````````````

We've added a settings option that will route to the "settings.bands" state and
we've added a new state 'settings.bands'.

Now create the actual settings page 'ux/tutorial/modules/bands/views/settings/form.html':

````````````````````````````````````````````````````````````````````````````````
<form layout="column" name="bandForm" go-submit="save()" novalidate>
	
	<md-toolbar class="md-hue-1">
		<div class="md-toolbar-tools">
			<span flex></span>
			<md-button type="submit" class="md-icon-button">
				<md-tooltip>{{"Save"| goT}}</md-tooltip>
				<md-icon class="mdi-check"></md-icon>
			</md-button>
		</div>
	</md-toolbar>

	<md-content class="md-padding" flex>
		<md-input-container  class="md-block">
				<label>Example field</label>
				<input name="dummy" required go-autofocus>
		</md-input-container>
	</md-content>
	
</form>

````````````````````````````````````````````````````````````````````````````````

Now reload the webclient and check out this page by going to the settings in the
top right of the main menu.
