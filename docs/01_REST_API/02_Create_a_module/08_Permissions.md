
See also the API docs: http://intermesh.io/php/docs/class-GO.Core.Auth.Model.RecordPermissionTrait.html

GroupOffice also comes with a role based permissions feature. Let's say we want to restrict access to our bands. We'll have to create a table for it with some permission level booleans:

We'll call this table bandsBandRole because it links the bandsBand table to the user roles. We will implement a readAccess and an editAccess level.
The table structure will then be like this:

``````````````````````````````````````````````````````````````````````````````````````````````````

CREATE TABLE IF NOT EXISTS `bandsBandRole` (
	`bandId` int(11) NOT NULL,
	`roleId` int(11) NOT NULL,
	`readAccess` tinyint(1) NOT NULL DEFAULT '0',
	`editAccess` tinyint(1) NOT NULL DEFAULT '0',
	PRIMARY KEY (`bandId`,`roleId`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

ALTER TABLE `bandsBandRole` ADD FOREIGN KEY ( `bandId` ) REFERENCES `bandsBand` (
`id`
) ON DELETE CASCADE ON UPDATE RESTRICT ;

ALTER TABLE `bandsBandRole` ADD FOREIGN KEY ( `roleId` ) REFERENCES `authRole` (
`id`
) ON DELETE CASCADE ON UPDATE RESTRICT;


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
 * @var int $roleId
 * @var int $moduleId
 * @var bool $readAccess 
 * @var bool $editAccess
 */
class BandRole extends AbstractRole{	
	
	/**
	 * The column pointing to the bands table
	 * 
	 * @return string
	 */
	public static function resourceKey() {
		return 'bandId';
	}	
	
	protected static function defineRelations(\GO\Core\Db\RelationFactory $r) {
		$relations = parent::defineRelations($r);		
		$relations[] = $r->belongsTo('band', Band::className(), 'bandId');
		
		return $relations;
	}
}

``````````````````````````````````````````````````````````````````````````````````````````````````

In the band model we need to use the **RecordPermissionTrait** to add the permission functions to the model.
We must also define a **roles** relation that points to the BandRole model.
Our band model will now look like this:


``````````````````````````````````````````````````````````````````````````````````````````````````
<?php
namespace GO\Modules\Bands\Model;

use GO\Core\Db\AbstractRecord;
use GO\Core\Db\RelationFactory;
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
	
	protected static function defineRelations(RelationFactory $r) {
		return [
			$r->hasMany('albums', Album::className(), 'bandId'),
			$r->hasOne('customfields', BandCustomFields::className(), 'id'),
			
			//add this role for permissions
			$r->hasMany('roles', BandRole::className(), 'bandId')
			];
	}
}
``````````````````````````````````````````````````````````````````````````````````````````````````

Now we've extended the model with permission functionality but we still need to implement this functionality in the controller.

We can use the "findPermitted" function in the store action to fetch only the allowed bands. We can use "checkPermission" to check read and write access.

Change the BandController like this:

``````````````````````````````````````````````````````````````````````````````````````````````````
<?php

namespace GO\Modules\Bands\Controller;

use GO\Core\App;
use GO\Core\Controller\AbstractCrudController;
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
class BandController extends AbstractCrudController {

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
		if(!$band->checkPermission('readAccess')) {
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
		
		//Check the module createAccess boolean here
		if (BandsModule::model()->checkPermission('createAccess')) {
			throw new Forbidden();
		}

		$band = new Band();

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
		
		//Check the module createAccess boolean here
		if (BandsModule::model()->checkPermission('createAccess')) {
			throw new Forbidden();
		}

		$band = new Band();
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
		if(!$band->checkPermission('editAccess')) {
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
		
		//Check edit permission
		if(!$band->checkPermission('editAccess')) {
			throw new Forbidden();
		}

		$band->delete();

		return $this->renderModel($band);
	}

}

``````````````````````````````````````````````````````````````````````````````````````````````````

Note that I've used editAccess for delete as well. Perhaps you'd like a deletePermission boolean in real life as well.
When new items are created I check the module's permission on createAccess.

## Use the API to test it

Now we're going to test this with Postman. Make sure you're logged in and request 
the bands on route /bands. It will now show an empty result because we don't have permission yet.

We can use a POST API call to /bands/1 to grant some permissions on the admins role:

````````````````````````````````````````````````````````````````````````````````
{"data": {"roles" :  [{"roleId": 1, "readAccess": true, "editAccess": true}]}}
````````````````````````````````````````````````````````````````````````````````

Now the GET on /bands will return the band again and it will also have a **permissions** property that shows the permissions for the current user.
Note that the admin user will always have permissions.
