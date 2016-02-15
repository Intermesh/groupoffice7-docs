Each module can add routes. To read more about how this works read the API documentation:

http://intermesh.io/php/docs/class-IFW.Modules.Module.html#_defineWebRoutes

A route is URL path behind the API URL. For example:

http://localhost/api/system/check

In this example *http://localhost/api* is the API URL and */system/check* is the
route.

A route leads to a controller class. Let's start with the simplest example.

Add the controller HelloController.php in the controller folder and enter:

````````````````````````````````````````````````````````````````````````````````
<?php

namespace UX\Modules\Bands\Controller;

use UX\Core\Controller;

class HelloController extends Controller {

	public function actionName($name = "human") {
		$this->render(['data' => 'Hello ' . $name]);
	}

}
````````````````````````````````````````````````````````````````````````````````

The router can map HTTP requests to a controller action. 

In this example we've implemented the GET request to return a JSON object that 
greets with hello. It has one GET query parameter that defaults to the string
"human".

But we can't reach this controller method yet! We need to add a route to it in 
the module manager file. Add the route **bands/hello** like this in 
UX/Modules/Bands/Module.php:

````````````````````````````````````````````````````````````````````````````````
<?php

namespace UX\Modules\Bands;

use GO\Core\Modules\Model\InstallableModule;
use IFW\Http\Router;
use UX\Modules\Bands\Controller\HelloController;

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
	}
}

````````````````````````````````````````````````````````````````````````````````

Now do a GET request to the route "/bands/hello" (In our example the full URL is: http://localhost/api/bands/hello). It will output:

````````````````````````````````````````````````````````````````````````````````
{ 
	"data": "Hello human" 
}
````````````````````````````````````````````````````````````````````````````````

Make it say a name by going to "/bands/hello?name=Merijn"
