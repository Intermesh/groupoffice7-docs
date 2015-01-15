# Routing

Each module can add routes. To read more about how this works read the API documentation:

http://localhost/groupoffice-server/docs/apigen/class-GO.Core.AbstractModule.html#_routes

A route leads to a controller class. Let's start with the simplest example.

Add the controller HelloController.php in the controller folder and enter:

````````````````````````````````````````````````````````````````````````````````
<?php
namespace GO\Modules\Bands\Controller;

use GO\Core\Controller\AbstractRESTController;

class HelloController extends AbstractRESTController {
	public function httpGet($name = "human"){
		return ['data' => 'Hello '.$name];
	}
}
````````````````````````````````````````````````````````````````````````````````

The REST controllers can implement the following methods:

1. httpGet
2. httpPost
3. httpPut
4. httpDelete

Each of these methods handles the corresponding request method.

In this example we've implemented the GET request to return a JSON object that 
greets with hello. It has one GET query parameter that defaults to the string
"human".

But we can't reach this controller method yet! We need to add a route to it in 
the module manager file. Add the route **bands/hello** like this in 
lib/GO/Modules/Bands/BandsModule.php:

````````````````````````````````````````````````````````````````````````````````
<?php
namespace GO\Modules\Bands;

use GO\Core\AbstractModule;
use GO\Modules\Bands\Controller\HelloController;

class BandsModule extends AbstractModule {
	public function routes() {
		return [
			'bands' => [
				'children' => [
					'hello' => [
						'controller' => HelloController::className()
					]
				]
			]
		];
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
