Group-Office also comes with a per record permissions feature. 
By default Record models implement [admins only permissions](http://intermesh.io/php/docs/class-IFW.Auth.Permissions.AdminsOnly.html) 
which means only admins may read and write them.

Record has a property permissions that will return all allowed actions:

````````````````````````````````````````
"permissions": {
		"create": true,
		"read": true,
		"update": true,
		"delete": true,
		"changePermissions": true
}
````````````````````````````````````````


To implement your own specific permissions you must override the function 
createPermissions in your record:

````````````````````````````````````````
protected function createPermissions() {
	return new ContactPermissions($this);
}
````````````````````````````````````````

There are some default permissions objects that you can use:

See: http://intermesh.io/php/docs/namespace-IFW.Auth.Permissions.html

1. AdminsOnly: Only admins can use the models (default)
2. Owner: Only the owner can use the model. It looks at the database field "ownedBy"
3. ReadOnly: It's read only for everyone except admins.
4. ViaRelation: You can tell the model to look at a related model's permissions 
   object. For example when saving an e-mail address of a contact it checks the 
   permissions of the contact.

## Stores
To avoid permissions problems in stores you must implement correct find queries 
yourself. There's no magic in the find function to automatically detect permissions.
If you return record's that are not allowed the API will throw a Forbidden error.

It's possible to optimize performance to tell the permissions object not to check
read permissions on each model when you already know you have executed the right
query. The [ContactPermission](http://intermesh.io/php/docs/class-GO.Modules.Contacts.Model.ContactPermissions.html)
object implements this for example.


## Band permissions
We're going to restrict access