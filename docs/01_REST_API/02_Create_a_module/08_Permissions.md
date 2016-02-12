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
We're going to control access to our bands.  Now bands use the default 
permissions that allow only access to admins. We're going to allow everybody to
read and write to bands.

### Create a bands user

Do a POST to /auth/users:

```````````````````````````````````````````````````````````````````````````````
{
  "data": {
    "username": "bands",
    "password": "Bands123!"
  }
}
```````````````````````````````````````````````````````````````````````````````

Now login as that user by posting the same data as above to /auth.

If we do a get on /bands now we'll get an 403 Forbidden error because this user
is not allowed to access our earlier created band:

```````````````````````````````````````````````````````````````````````````````
{
  "success": false,
  "errors": [
    "You're not permitted to read UX\Modules\Bands\Model\Band array (\n  'id' => 1,\n)"
  ],
  "exception": [
    "exception 'IFW\Exception\HttpException' with message 'You're not permitted to read UX\Modules\Bands\Model\Band array (",
    "  'id' => 1,",
    ")' in /var/www/groupoffice-server/lib/IFW/Http/Router.php:596",
    "Stack trace:",
    "#0 /var/www/groupoffice-server/lib/IFW/App.php(266): IFW\Http\Router->run()",
    "#1 /var/www/groupoffice-server/public/index.php(13): IFW\App->run()",
    "#2 {main}"
  ]
}
```````````````````````````````````````````````````````````````````````````````

### Create the permissions model

Create UX/Modules/Bands/Model/BandPermissions.php:

```````````````````````````````````````````````````````````````````````````````

<?php
namespace UX\Modules\Bands\Model;

use IFW\Auth\Permissions\Base;

class BandPermissions extends Base {
	protected function checkAction($action) {
		
		return ($action == self::ACTION_READ || $action == self::ACTION_UPDATE);
		
	}
}


```````````````````````````````````````````````````````````````````````````````

This model will allow read and write to anyone.

We can use this model in the Bands record by overriding "createPermissions"
in UX/Modules/Bands/Model/Band.php:

```````````````````````````````````````````````````````````````````````````````
protected function createPermissions() {
	return new BandPermissions($this);
}
```````````````````````````````````````````````````````````````````````````````

If we do the GET request on route /bands now again we'll get the band. Notice
the permissions properties that has true for read and write.
```````````````````````````````````````````````````````````````````````````````
{
  "data": [
    {
      "id": 1,
      "name": "Pearl Jam",
      "ownedBy": 1,
      "createdAt": "2016-02-12T10:12:15Z",
      "modifiedAt": "2016-02-12T12:44:51Z",
      "deleted": false,
      "permissions": {
        "create": false,
        "read": true,
        "update": true,
        "delete": false,
        "changePermissions": false
      },
      "validationErrors": [],
      "className": "UX\Modules\Bands\Model\Band",
      "markDeleted": false
    }
  ],
  "success": true
}
```````````````````````````````````````````````````````````````````````````````

In real life you would probably want something more complex like allowing access
per group. This could be accomplished by creating a link table with permission
columns for it.