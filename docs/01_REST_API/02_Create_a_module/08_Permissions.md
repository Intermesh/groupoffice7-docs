
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

In the band model we need to use the **RecordPermissionTrait** to add the 
permission functions to the model.
We must also define a **roles** relation that points to the BandRole model.

Finally we need to define permission contants that describe the different permission types. They are prefixed with "PERMISSION_"

For example:

- PERMISSION_READ
- PERMISSION_WRITE
- PERMISSION_DELETE

The API will have a permissions object in the band with keys "read", "write" and
"delete". The contacts will have PERMISSION_ stripped off and they are converted 
to lower case.

Our band model will now look like this:

`````````````````````````````````````````````````````````````````````````````````````````````````
<?php
namespace GO\Modules\Bands\Model;

use GO\Core\Db\AbstractRecord;

use GO\Core\Auth\Model\User;

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
 * @copyright (c) 2015, Intermesh BV http://www.intermesh.nl
 * @author Merijn Schering <mschering@intermesh.nl>
 * @license http://www.gnu.org/licenses/agpl-3.0.html AGPLv3
 */
class Band extends AbstractRecord{	
	
	use \GO\Core\Db\SoftDeleteTrait;	
	
	//Add this line for permissions
	use \GO\Core\Auth\Model\RecordPermissionTrait;
	
	
	/**
	 * Allow read access to the role
	 */
	const PERMISSION_READ = 0;
	
	/**
	 * Allow write access to the role
	 */
	const PERMISSION_WRITE = 1;	
	
	/**
	 * Allow delete access to the role
	 */
	const PERMISSION_DELETE = 2;
	
	
	protected static function defineRelations() {
		
			self::hasMany('albums', Album::className(), 'bandId');
			self::hasOne('customfields', BandCustomFields::className(), 'id');
			
			//add this role for permissions
			self::hasMany('roles', BandRole::className(), 'bandId');		
	}
}

``````````````````````````````````````````````````````````````````````````````````````````````````

In the controller we need to check the permissions with $band->permissions->has(Band::PERMISSION_READ) for example.
The controller will look like this now:


``````````````````````````````````````````````````````````````````````````````````````````````````
<?php

namespace GO\Modules\Bands\Controller;

use GO\Core\App;
use GO\Core\Controller\AbstractController;
use GO\Core\Data\Store;
use GO\Core\Db\Query;
use GO\Core\Exception\Forbidden;
use GO\Core\Exception\NotFound;
use GO\Modules\Bands\BandsModule;
use GO\Modules\Bands\Model\Band;

/**
 * The controller for bands. Admin role is required.
 * 
 * Uses the {@see Band} model.
 *
 * @copyright (c) 2014, Intermesh BV http://www.intermesh.nl
 * @author Merijn Schering <mschering@intermesh.nl>
 * @license http://www.gnu.org/licenses/agpl-3.0.html AGPLv3
 */
class BandController extends AbstractController {

	/**
	 * Fetch bands
	 *
	 * @param string $orderColumn Order by this column
	 * @param string $orderDirection Sort in this direction 'ASC' or 'DESC'
	 * @param int $limit Limit the returned records
	 * @param int $offset Start the select on this offset
	 * @param string $searchQuery Search on this query.
	 * @param array|JSON $returnAttributes The attributes to return to the client. eg. ['\*','emailAddresses.\*']. See {@see GO\Core\Db\ActiveRecord::getAttributes()} for more information.
	 * @param string $where {@see \GO\Core\Db\Criteria::whereSafe()}
	 * @return array JSON Model data
	 */
	protected function actionStore($orderColumn = 'name', $orderDirection = 'ASC', $limit = 10, $offset = 0, $searchQuery = "", $returnAttributes = [], $where = null) {

		$query = Query::newInstance()
				->orderBy([$orderColumn => $orderDirection])
				->limit($limit)
				->offset($offset);

		if (!empty($searchQuery)) {
			$query->search($searchQuery, ['t.bandname']);
		}

		if (!empty($where)) {

			$where = json_decode($where, true);

			if (count($where)) {
				$query
						->groupBy(['t.id'])
						->whereSafe($where);
			}
		}

		//Use findPermitted for permissions
		$bands = Band::findPermitted($query);

		$store = new Store($bands);
		$store->setReturnAttributes($returnAttributes);

		return $this->renderStore($store);
	}

	/**
	 * GET a list of bands or fetch a single band
	 *
	 * 
	 * @param int $bandId The ID of the role
	 * @param array|JSON $returnAttributes The attributes to return to the client. eg. ['\*','emailAddresses.\*']. See {@see GO\Core\Db\ActiveRecord::getAttributes()} for more information.
	 * @return JSON Model data
	 */
	protected function actionRead($bandId = null, $returnAttributes = ['*','albums']) {

		$band = Band::findByPk($bandId);

		if (!$band) {
			throw new NotFound();
		}
		
		//Check read permission
		if(!$band->permissions->has(Band::PERMISSION_READ)){
			throw new Forbidden();
		}

		return $this->renderModel($band, $returnAttributes);
	}

	/**
	 * Get's the default data for a new band
	 * 
	 * @param array $returnAttributes
	 * @return array
	 */
	protected function actionNew($returnAttributes = ['*','albums']) {
				
		//Check edit permission		
		$band = new Band();	
		
		if(BandsModule::newInstance()->permissions->has(BandsModule::PERMISSION_CREATE)){
			throw new Forbidden();
		}
		
		return $this->renderModel($band, $returnAttributes);
	}

	/**
	 * Create a new band. Use GET to fetch the default attributes or POST to add a new band.
	 *
	 * The attributes of this band should be posted as JSON in a band object
	 *
	 * <p>Example for POST and return data:</p>
	 * <code>
	 * {"data":{"attributes":{"bandname":"test",...}}}
	 * </code>
	 * 
	 * @param array|JSON $returnAttributes The attributes to return to the client. eg. ['\*','emailAddresses.\*']. See {@see GO\Core\Db\ActiveRecord::getAttributes()} for more information.
	 * @return JSON Model data
	 */
	public function actionCreate($returnAttributes = ['*','albums']) {
		
		//Check edit permission
		$band = new Band();	
		if(BandsModule::newInstance()->permissions->has(BandsModule::PERMISSION_CREATE)){
			throw new Forbidden();
		}		
		
		$band->setAttributes(App::request()->payload['data']);
		$band->save();

		return $this->renderModel($band, $returnAttributes);
	}

	/**
	 * Update a band. Use GET to fetch the default attributes or POST to add a new band.
	 *
	 * The attributes of this band should be posted as JSON in a band object
	 *
	 * <p>Example for POST and return data:</p>
	 * <code>
	 * {"data":{"attributes":{"bandname":"test",...}}}
	 * </code>
	 * 
	 * @param int $bandId The ID of the band
	 * @param array|JSON $returnAttributes The attributes to return to the client. eg. ['\*','emailAddresses.\*']. See {@see GO\Core\Db\ActiveRecord::getAttributes()} for more information.
	 * @return JSON Model data
	 * @throws NotFound
	 */
	public function actionUpdate($bandId, $returnAttributes = ['*','albums']) {

		$band = Band::findByPk($bandId);

		if (!$band) {
			throw new NotFound();
		}
		
		//Check edit permission
		if(!$band->permissions->has(Band::PERMISSION_WRITE)){
			throw new Forbidden();
		}

		$band->setAttributes(App::request()->payload['data']);

		$band->save();


		return $this->renderModel($band, $returnAttributes);
	}

	/**
	 * Delete a band
	 *
	 * @param int $bandId
	 * @throws NotFound
	 */
	public function actionDelete($bandId) {
		$band = Band::findByPk($bandId);

		if (!$band) {
			throw new NotFound();
		}
		
		if(!$band->permissions->has(Band::PERMISSION_DELETE)){
			throw new Forbidden();
		}

		$band->delete();

		return $this->renderModel($band);
	}

}



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
