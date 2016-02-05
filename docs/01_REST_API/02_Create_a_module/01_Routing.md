Each module can add routes. To read more about how this works read the API documentation:

http://intermesh.io/php/docs/class-IFW.Modules.Module.html#_defineHttpRoutes

A route leads to a controller class. Let's start with the simplest example.

Add the controller HelloController.php in the controller folder and enter:

````````````````````````````````````````````````````````````````````````````````
<?php
namespace GO\Modules\Bands\Controller;

use GO\Core\Controller;

class HelloController extends Controller {
	public function actionName($name = "human"){
		return ['data' => 'Hello '.$name];
	}
}
````````````````````````````````````````````````````````````````````````````````

The router can map HTTP requests to a controller action. 

In this example we've implemented the GET request to return a JSON object that 
greets with hello. It has one GET query parameter that defaults to the string
"human".

But we can't reach this controller method yet! We need to add a route to it in 
the module manager file. Add the route **bands/hello** like this in 
GO/Modules/Bands/BandsModule.php:

````````````````````````````````````````````````````````````````````````````````
<?php
namespace GO\Modules\Bands;

use GO\Core\Modules\Model\InstallableModule;
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
class BandsModule extends InstallableModule {

	public static function defineHttpRoutes(Router $router) {		
		$router->addRoutesFor(HelloController::class)
				->get('bands/hello', 'name');
	}
}

````````````````````````````````````````````````````````````````````````````````

Now request this route by going to the route "/bands/hello". It will output:

````````````````````````````````````````````````````````````````````````````````
{ 
	"data": "Hello human" 
}
````````````````````````````````````````````````````````````````````````````````

Make it say a name by going to "/bands/hello?name=Merijn"
