Modules often require settings pages. The core implements a settings page which 
can be extended by the modules. It also has an accounts page which can be extended
as well. Think of E-mail, Twitter and Caldav accounts for example.

To add a settings page you can use the 'GO.Core.settingsTemplatesProvider'.

Edit 'ux/tutorial/modules/bands/module.js':

````````````````````````````````````````````````````````````````````````````````
'use strict';

GO.module('UX.Tutorial.Modules.Bands', ['GO.Core']).		
		config([
			'GO.Core.launcherProvider',			
			'GO.Core.settingsTemplatesProvider',
			function (launcherProvider, settingsTemplates) {								
				launcherProvider.add('bands', 'Bands', ['UX\\Modules\\Bands\\Module']);

				settingsTemplates.add('ux/tutorial/modules/bands/views/settings/navigation.html');
			}]).
		config([
			'$stateProvider', 
			function($stateProvider) {
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

We've added the template for the navigation panel in the settings page and we've
added a new state 'settings.bands'.

Now add the view 'ux/tutorial/modules/bands/views/settings/navigation.html':

````````````````````````````````````````````````````````````````````````````````
<md-list-item ui-sref="settings.bands" ui-sref-active="selected">
	<md-icon class="mdi-music-note"></md-icon>
	<p>{{"Bands" | goT}}</p>
</md-list-item>					
````````````````````````````````````````````````````````````````````````````````

And the actual settings page 'ux/tutorial/modules/bands/views/settings/form.html':

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