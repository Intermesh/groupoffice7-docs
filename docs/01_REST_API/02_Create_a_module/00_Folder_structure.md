This chapter is intended for PHP developers who wish to extend the REST API with
own modules.

In this example we're going to create a very simple "Bands" module. It will have bands with their albums.

Create the folder **lib/GO/Modules/Bands**.

Inside this folder create the following subfolders:

- Controller
- Model
- Install/Database

Each module has a module manager file. In this case:

lib/GO/Modules/Bands/BandsModule.php.

Create this file and put in this minimal code:


``````````````````````````````````````````````
<?php
namespace GO\Modules\Bands;

use GO\Core\AbstractModule;

class BandsModule extends AbstractModule {
	
}
``````````````````````````````````````````````

> ### Naming convention
> The folder and files must be named UpperCamelCase.

## Install the module

Install the module by running the route:

/modules/upgrade

This will add the module to the "modulesModule" table.