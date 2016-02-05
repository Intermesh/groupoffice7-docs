There's a hook system to create tailor made modifications. It's possible to
modify the DOM structure of HTML templates and it's possible to override 
controller methods.

The root folder in the web application called "customizations" is loaded after
the core app is loaded and is the right place for hooks. The folder scructure
should match the core app.

For example if you create an override for the "contacts" module and you do this 
for client "intermesh" you should put the hooks in:

customizations/intermesh/modules/contacts/hooks.js

Here's an example:

Note: The templates are written inline here but it's advisable to create HTML 
templates that are included with ng-include.

```````````````````````````````````````````````````````````````````````````````

//Adds an extra card with id info2 to the contacts info panel
GO.hooks.register('contact' , ['element', function(element) {
	var contents = element.find('md-content');
	contents.prepend('<md-card id="info2"><md-card-content>Naam: {{contact.name}}</md-card-content></md-card>');
}]);

//Adds an extra tab button with id info2 to the contacts info panel
GO.hooks.register('contact-toolbar' , ['element', function(element) {
	element.prepend('<md-tab ng-click="goto(\'info2\')">Info 2</md-tab>');
}]);

GO.hooks.overrideController("GO.Modules.Contacts.ContactController", ["ctrlLocals", "$mdDialog", "GO.Core.Translate", function(ctrlLocals, $mdDialog, Translate){
		
	//example hook to put a confirm dialog before the original edit function of a contact
		
	//copy the original edit function so we can use it later after the confirmation
	var origEdit = ctrlLocals.$scope.edit;

	//overwrite the edit function
	ctrlLocals.$scope.edit = function() {
		
		//copy function arguments to apply later
		var args = arguments;
		
		//create confirm dialog
		var confirm = $mdDialog.confirm()
												.title(Translate.t('A stupid question'))
												.textContent(Translate.t('Do you really want to edit this contact?'))
												.ariaLabel('Confirm')
												.ok(Translate.t("Yes"))
												.cancel(Translate.t("No, take me away!"));
								
								$mdDialog.show(confirm).then(function () {

									//call orignal edit function
									return origEdit.apply(null, args);
								}, function () {									
									close();
								});								
	};
}]);
```````````````````````````````````````````````````````````````````````````````

