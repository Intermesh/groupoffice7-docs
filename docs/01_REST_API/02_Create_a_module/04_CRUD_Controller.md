Now that we have the models in place we must create a [RESTFul CRUD controller](http://intermesh.io/php/docs/class-GO.Core.Controller.Controller.html) to
be able to Create, Read, Update and Delete models through the REST API. 


## Controller actions
A controller action is a method that is prefixed with the word "action".

The router will automatically map query string parameters of the HTTP Request to
the controller action method arguments. In the example action below:

````````````````````````````````````````````````````````````````````````````````

public function actionExample($bar, $foo="default") {

}

````````````````````````````````````````````````````````````````````````````````

you can pass the arguments in your request like this: "/example?foo=value&bar=value"

If the method arguments don't have a default value then they are required in the
URL. So in the example above the "bar" query parameter is required.

Note that the order of the arguments is not important.

## Band Controller

Create the file "UX/Modules/Bands/Controller/BandController.php":

````````````````````````````````````````````````````````````````````````````````
<?php

namespace UX\Modules\Bands\Controller;

use GO\Core\Controller;
use UX\Modules\Bands\Model\Band;
use IFW;
use IFW\Exception\NotFound;
use IFW\Orm\Query;

/**
 * The controller for bands. Admin group is required.
 * 
 * Uses the {@see Band} model.
 *
 * @copyright (c) 2014, Intermesh BV http://www.intermesh.nl
 * @author Merijn Schering <mschering@intermesh.nl>
 * @license http://www.gnu.org/licenses/agpl-3.0.html AGPLv3
 */
class BandController extends Controller {

	/**
	 * Fetch bands
	 *
	 * @param string $orderColumn Order by this column
	 * @param string $orderDirection Sort in this direction 'ASC' or 'DESC'
	 * @param int $limit Limit the returned records
	 * @param int $offset Start the select on this offset
	 * @param string $searchQuery Search on this query.
	 * @param array|JSON $returnProperties The attributes to return to the client. eg. ['\*','emailAddresses.\*']. See {@see IFW\Db\ActiveRecord::getAttributes()} for more information.
	 * @param string $where {@see \IFW\Db\Criteria::whereSafe()}
	 * @return array JSON Model data
	 */
	protected function actionStore($orderColumn = 'name', $orderDirection = 'ASC', $limit = 10, $offset = 0, $searchQuery = "", $returnProperties = "", $where = null) {

		$query = (new Query())
						->orderBy([$orderColumn => $orderDirection])
						->limit($limit)
						->offset($offset);

		if (!empty($searchQuery)) {
			$query->search($searchQuery, ['t.name']);
		}

		if (!empty($where)) {

			$where = json_decode($where, true);

			if (count($where)) {
				$query
								->groupBy(['t.id'])
								->whereSafe($where);
			}
		}

		$bands = Band::find($query);
		$bands->setReturnProperties($returnProperties);

		$this->renderStore($bands);
	}

	/**
	 * GET a list of bands or fetch a single band
	 *
	 * 
	 * @param int $bandId The ID of the group
	 * @param array|JSON $returnProperties The attributes to return to the client. eg. ['\*','emailAddresses.\*']. See {@see IFW\Db\ActiveRecord::getAttributes()} for more information.
	 * @return JSON Model data
	 */
	protected function actionRead($bandId = null, $returnProperties = '*,albums') {

		$band = Band::findByPk($bandId);

		if (!$band) {
			throw new NotFound();
		}

		$this->renderModel($band, $returnProperties);
	}

	/**
	 * Get's the default data for a new band
	 * 
	 * @param $returnProperties
	 * @return array
	 */
	protected function actionNew($returnProperties = '*,albums') {

		//Check edit permission		
		$band = new Band();

		$this->renderModel($band, $returnProperties);
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
	 * @param array|JSON $returnProperties The attributes to return to the client. eg. ['\*','emailAddresses.\*']. See {@see IFW\Db\ActiveRecord::getAttributes()} for more information.
	 * @return JSON Model data
	 */
	public function actionCreate($returnProperties = '*,albums') {

		$band = new Band();
		$band->setValues(IFW::app()->getRequest()->body['data']);
		$band->save();

		$this->renderModel($band, $returnProperties);
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
	 * @param array|JSON $returnProperties The attributes to return to the client. eg. ['\*','emailAddresses.\*']. See {@see IFW\Db\ActiveRecord::getAttributes()} for more information.
	 * @return JSON Model data
	 * @throws NotFound
	 */
	public function actionUpdate($bandId, $returnProperties = '*,albums') {

		$band = Band::findByPk($bandId);

		if (!$band) {
			throw new NotFound();
		}

		$band->setValues(IFW::app()->getRequest()->body['data']);
		$band->save();

		$this->renderModel($band, $returnProperties);
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

		$band->delete();

		$this->renderModel($band);
	}

}


````````````````````````````````````````````````````````````````````````````````

## Add the controller routes

Add the route to the module manager file "UX/Modules/Bands/Module.php":

````````````````````````````````````````````````````````````````````````````````
<?php
namespace UX\Modules\Bands;

use GO\Core\Modules\Model\InstallableModule;
use UX\Modules\Bands\Controller\HelloController;

use IFW\Http\Router;

//Use the new controller
use UX\Modules\Bands\Controller\BandController;

/**
 * The bands module
 * 
 * A module for the tutorial.
 *
 * @copyright (c) 2015, Intermesh BV http://www.intermesh.nl
 * @author Merijn Schering <mschering@intermesh.nl>
 * @license http://www.gnu.org/licenses/agpl-3.0.html AGPLv3
 */
class Module extends InstallableModule {

	public static function defineWebRoutes(Router $router) {

		$router->addRoutesFor(HelloController::class)
						->get('bands/hello', 'name');
		
		//Add the new routes:
		$router->addRoutesFor(BandController::class)
						->get('bands', 'store')
						->get('bands/0', 'new')
						->get('bands/:bandId', 'read')
						->put('bands/:bandId', 'update')
						->post('bands', 'create')
						->delete('bands/:bandId', 'delete');
	}
}


````````````````````````````````````````````````````````````````````````````````

Now you should be able to perform a GET request on the API route "/bands" and 
the server should respond with an empty data array because there are no bands 
yet:

````````````````````````````````````````````````````````````````````````````````
{
  "data": [],
  "success": true
}
````````````````````````````````````````````````````````````````````````````````
