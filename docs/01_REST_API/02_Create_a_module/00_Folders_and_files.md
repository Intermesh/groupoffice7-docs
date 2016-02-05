This chapter is intended for PHP developers who wish to extend the REST API with
own modules.

In this example we're going to create a very simple "Bands" module. It will have 
bands with their albums.

## Naming conventions

# Namespaces: use PascalCase
# Classes: use PascalCase. Use short and simple names and only append Interface for interface. Don't use Abstract in the name for abstract classes.
# Variables: use camelCase
# Methods: use camelCase and https://www.cwu.edu/~gellenbe/javastyle/method.html
# Folder/File names: PSR-4 standard. Use case sensitive names equal to the namespace/class name.
# Database tables: For compatibility reasons use underscores. PascalCase = pascal_case.
# Database columns: use camelCase


## Folder structure

Create the folder **GO/Modules/Bands**.

Inside this folder create the following sub folders:

- Controller
- Install/Database
- Model

## Module manager file
Each module has a module manager file. In this case:

GO/Modules/Bands/BandsModule.php.

Create this file and put in this minimal code:


``````````````````````````````````````````````
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
	
}

``````````````````````````````````````````````

## Install the module

Install the module by doing a POST request to the route:

/modules

With request payload:

``````````````````````````````````````````````
{
	"data": {
		"name": "GO\\Modules\\Bands\\BandsModule"
	}
}
``````````````````````````````````````````````

This will add the module to the "modules_module" table.