
See also the API docs: http://intermesh.io/php/docs/class-GO.Core.Auth.Model.RecordPermissionTrait.html

GroupOffice also comes with a role based permissions feature. Let's say we want 
to restrict access to our bands. We'll have to create a table for it:

We'll call this table bandsBandRole because it links the bandsBand table to the 
user roles.
The table structure will then be like this:

``````````````````````````````````````````````````````````````````````````````````````````````````

CREATE TABLE IF NOT EXISTS `bandsBandRole` (
  `bandId` int(11) NOT NULL,
  `roleId` int(11) NOT NULL,
  `permissionType` int(11) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

ALTER TABLE `bandsBandRole`
 ADD PRIMARY KEY (`bandId`,`roleId`,`permissionType`), ADD KEY `roleId` (`roleId`);

ALTER TABLE `bandsBandRole`
ADD CONSTRAINT `bandsBandRole_ibfk_2` FOREIGN KEY (`roleId`) REFERENCES `authRole` (`id`) ON DELETE CASCADE,
ADD CONSTRAINT `bandsBandRole_ibfk_1` FOREIGN KEY (`bandId`) REFERENCES `bandsBand` (`id`) ON DELETE CASCADE;



``````````````````````````````````````````````````````````````````````````````````````````````````

You can put this in a database patch file and run the route /modules/upgrade.

We'll also have to create a model for it too:


``````````````````````````````````````````````````````````````````````````````````````````````````
<?php
namespace GO\Modules\Bands\Model;

use GO\Core\Auth\Model\AbstractRole;

/**
 * The band roles model
 * 
 * @property int $roleId
 * @property int $bandId
 * @property int $permissionType
 */
class BandRole extends AbstractRole {
	protected static function defineResource(){
		return self::belongsTo('band', Band::className(), 'bandId');
	}
}

``````````````````````````````````````````````````````````````````````````````````````````````````

We can change the Band model to extends the AbstractProtectedCRUDRecord model to 
add standard read, write and delete permissions to the model.
We must also define a **roles** relation that points to the BandRole model.

The permissions system is very flexible. Any constant prefixed with "PERMISSION_" will
automatically become a permission type.

The AbstractProtectedCRUDRecord already has these defined:

- PERMISSION_READ
- PERMISSION_WRITE
- PERMISSION_DELETE

The API will have a permissions object in the band with keys "read", "write" and
"delete". The contacts will have PERMISSION_ stripped off and they are converted 
to lower case.

We must also implement the abstract function "hasCreatePermission". It should
return true when the current user has permission to create new bands.

In most cases it's logical to implement a permission type at the module level
to define this. In this case we will add the constant "PERMISSION_CREATE" to
the BandsModule class.

Our band model will now look like this:

`````````````````````````````````````````````````````````````````````````````````````````````````
<?php
namespace GO\Modules\Bands\Model;

use GO\Core\Auth\Model\AbstractProtectedCRUDRecord;
use GO\Core\Auth\Model\User;
use GO\Modules\Bands\BandsModule;

/**
 * The Band model
 *
 * @property int $id
 * @property string $name
 * @property int $ownerUserId
 * @property User $owner
 * @property string $createdAt
 * @property string $modifiedAt
 * 
 * @property Album[] $albums
 * @property BandCustomFields $customfields
 * @property BandRole[] $roles
 *
 * @copyright (c) 2015, Intermesh BV http://www.intermesh.nl
 * @author Merijn Schering <mschering@intermesh.nl>
 * @license http://www.gnu.org/licenses/agpl-3.0.html AGPLv3
 */
class Band extends AbstractProtectedCRUDRecord{	
	
	use \GO\Core\Db\SoftDeleteTrait;
	
	protected static function defineRelations() {
		
			self::hasMany('albums', Album::className(), 'bandId');
			self::hasOne('customfields', BandCustomFields::className(), 'id');
			
			//add this role for permissions
			self::hasMany('roles', BandRole::className(), 'bandId');		
	}
	
	/**
	 * Allowed when user has create permission on the module permissions
	 * 
	 * @return boolean
	 */
	public function hasCreatePermission() {
		return BandsModule::model()->permissions->has(BandsModule::PERMISSION_CREATE);
	}
}
``````````````````````````````````````````````````````````````````````````````````````````````````

And our BandsModule class looks like this now:

``````````````````````````````````````````````````````````````````````````````````````````````````
<?php
namespace GO\Modules\Bands;

use GO\Core\AbstractModule;
use GO\Modules\Bands\Controller\BandController;
use GO\Modules\Bands\Controller\HelloController;

/**
 * The bands module
 * 
 * A module for the tutorial.
 *
 * @copyright (c) 2015, Intermesh BV http://www.intermesh.nl
 * @author Merijn Schering <mschering@intermesh.nl>
 * @license http://www.gnu.org/licenses/agpl-3.0.html AGPLv3
 */
class BandsModule extends AbstractModule {
	
	/**
	 * Controls if users are allowed to create new bands
	 */
	const PERMISSION_CREATE = 1; //Use 1 as 0 is already defined in the module.
	
	public function routes() {
		BandController::routes()
				->get('bands', 'store')
				->get('bands/0','new')
				->get('bands/:bandId','read')
				->put('bands/:bandId', 'update')
				->post('bands', 'create')
				->delete('bands/:bandId','delete');
		
		HelloController::routes()
				->get('bands/hello', 'name');
	}	
}
``````````````````````````````````````````````````````````````````````````````````````````````````

We don't need to change anything in the controller as everything is implemented 
at the model level now.

``````````````````````````````````````````````````````````````````````````````````````````````````

## Use the API to test it

Now we're going to test this with Postman. Make sure you're logged in and request 
the bands on route /bands. It will now show an empty result because we don't 
have permission yet.

We can use a PUT API call to /bands/1 to grant some permissions on the admins role:

````````````````````````````````````````````````````````````````````````````````
{"data": {"roles" :  [{"roleId": 1, "permissionType" : "read"}, {"roleId": 1, "permissionType" : "write"}, {"roleId": 1, "permissionType" : "delete"}]}}
````````````````````````````````````````````````````````````````````````````````

Now the GET on /bands will return the band again and it will also have a 
**permissions** property that shows the permissions for the current user.
Note that the admin user will always have permissions.
