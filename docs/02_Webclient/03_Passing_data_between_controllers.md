Sometimes you need to pass data objects between controllers without exposing them in the URL.
For example when creating a new contact from an e-mail message we go to the edit
contact page and we pass along some contact attributes to prefill the form with 
name an e-amil address.

To do this you can define a parameter in the module.js in the UI router state 
configuration. For example:

``````````````````````````````````````````````````````````````
.state('contacts.edit', {
							url: "/edit/{contactId:[0-9]*}",
							templateUrl: 'modules/contacts/views/contact-edit.html',
							controller: 'ContactEditController',
							params: {
								attributes: null
							}
						});
``````````````````````````````````````````````````````````````

Now this parameter can be passed in two ways. You can do it with the $state service:

``````````````````````````````````````````````````````````````
$state.go("contacts.edit", {contactId: 0, attributes: {name: this.from.personal, emailAddresses: [{email: this.from.email}]}});
``````````````````````````````````````````````````````````````

or in the view with the ui-sref directive:

``````````````````````````````````````````````````````````````
<a ui-sref="contacts.edit({contactId: 0, attributes: {name: from.personal, emailAddresses: [{email: from.email}]})">Create contact</a>
``````````````````````````````````````````````````````````````


Now you can use those attributes in the controller that's associated with this state:

``````````````````````````````````````````````````````````````
if($stateParams.attributes) {
	$scope.contact.setAttributes($stateParams.attributes);
}
``````````````````````````````````````````````````````````````