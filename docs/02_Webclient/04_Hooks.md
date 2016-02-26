There's a hook system to create tailor made modifications. It's possible to
modify the DOM structure of HTML templates and it's possible to override 
controller methods.

The root folder in the web application called "customizations" is loaded after
the core app is loaded and is the right place for hooks. The folder structure
should match the core app.

For example if you create an override for the "contacts" module and you do this 
for client "intermesh" you should put the hooks in the file 
'ux/intermesh/modules/contacts/hooks.js'.

If you look at the file 'app/modules/contacts/views/contact.html you'll find this
tag:
````````````````````````````````````````````````````````````````````````````````
<go-hook name="contact"></go-hook>
````````````````````````````````````````````````````````````````````````````````

You can hook into the contents of that tag and adjust the template just before
it's compiled.

Here's an example (The templates are written inline here but it's advisable to 
create HTML templates that are included with ng-include):

```````````````````````````````````````````````````````````````````````````````

//Adds an extra card with id info2 to the contacts info panel
GO.hooks.register('contacts.contact' , ['element', function(element) {
	var contents = element.find('md-content');
	contents.prepend('<md-card id="info2"><md-card-content>Naam: {{contact.name}}</md-card-content></md-card>');

	var toolbar = element.find('md-tabs');
	toolbar.prepend('<md-tab ng-click="goto(\'info2\')">Info 2</md-tab>');
}]);


//example controller override to put a confirm dialog before the original edit function of a contact
GO.hooks.overrideController("GO.Modules.Contacts.ContactController", ["ctrlLocals", "$mdDialog", "GO.Core.Translate", function(ctrlLocals, $mdDialog, Translate){
		
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

