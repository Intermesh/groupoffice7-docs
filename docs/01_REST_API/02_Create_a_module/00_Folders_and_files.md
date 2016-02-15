This chapter is intended for PHP developers who wish to extend the REST API with
own modules.

In this example we're going to create a very simple "Bands" module. It will have 
bands with their albums.

## Naming conventions

| Type              | Convention                                                                                                                           |
|-------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| Namespaces        | PascalCase                                                                                                                           |
| Classes           | PascalCase. Use short and simple names and only append Interface for interface. Don't use Abstract in the name for abstract classes. |
| Variables         | camelCase                                                                                                                            |
| Methods           | use camelCase and https://www.cwu.edu/~gellenbe/javastyle/method.html                                                                |
| Folder/File names | PSR-4 standard. Use case sensitive names equal to the namespace/class name.                                                          |
| Database tables   | For compatibility reasons use underscores. PascalCase = pascal_case.                                                                 |
| Database columns  | camelCase                                                                                                                            |


## Folder structure

The GroupOffice modules that are provided in the packages are located in 
"GO/Modules". For custom modules that you create yourself we've created an 
additional folder/namespace called "UX/Modules". In this example we'll use that
User Extension namespace.



Create the folder **UX/Modules/Bands**.

Inside this folder create the following sub folders:

- Bands
	- Controller
	- Install
		- Database
	- Model

## Module manager file
Each module has a module manager file. In this case:

UX/Modules/Bands/Module.php.

Create this file and put in this minimal code:


``````````````````````````````````````````````
<?php

namespace UX\Modules\Bands;

use UX\Core\Modules\Model\InstallableModule;
use IFW\Http\Router;

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

	}
}


``````````````````````````````````````````````

## Install the module

Install the module by doing a POST request to the route:

/modules

With request payload:

``````````````````````````````````````````````
{
	"data": {
		"name": "UX\\Modules\\Bands\\Module"
	}
}
``````````````````````````````````````````````

This will add the module to the "modules_module" table.

If you don't know how to do the request please read [Using the REST API](http://intermesh.io/index.php/REST_API/Usage).