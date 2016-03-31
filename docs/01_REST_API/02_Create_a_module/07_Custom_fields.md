GroupOffice comes with a powerful custom fields module so users can extend 
models with their own custom fields. We're going to make the Bands model 
extensible with custom fields in this example.

## Create the file "UX/Modules/Bands/Model/BandCustomFields.php" and make this 
model extend the model "GO\Core\CustomFields\Model\CustomFieldsRecord":

````````````````````````````````````````````````````````````````````````````````
<?php
namespace UX\Modules\Bands\Model;

use GO\Core\CustomFields\Model\CustomFieldsRecord;

class BandCustomFields extends CustomFieldsRecord {
	
}


````````````````````````````````````````````````````````````````````````````````

Add the database patch file "Install/Database/20160212-1118.sql":

Note that's important to make it cascade on band deletion.

````````````````````````````````````````````````````````````````````````````````
CREATE TABLE `bands_band_custom_fields` (
  `id` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  CONSTRAINT `bands_band_custom_fields_ibfk_1` FOREIGN KEY (`id`) REFERENCES `bands_band` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

````````````````````````````````````````````````````````````````````````````````

And do a GET request to the route "/system/update" to apply it.


## Define relation in the Band model:
````````````````````````````````````````````````````````````````````````````````
<?php

namespace UX\Modules\Bands\Model;

use GO\Core\Auth\Model\User;
use IFW\Orm\Record;

//add this use
use UX\Modules\Bands\Model\BandCustomFields;

/**
 * The Band model
 *
 * @property int $id
 * @property string $name
 * @property int $createdBy
 * @property User $owner
 * @property string $createdAt
 * @property string $modifiedAt
 * 
 * @property Album[] $albums
 * @property BandCustomFields $customfields
 *
 * @copyright (c) 2015, Intermesh BV http://www.intermesh.nl
 * @author Merijn Schering <mschering@intermesh.nl>
 * @license http://www.gnu.org/licenses/agpl-3.0.html AGPLv3
 */
class Band extends Record {

	protected static function defineRelations() {
		self::hasMany('albums', Album::class, ['id' => 'bandId']);
		self::hasOne('owner', User::class, ['createdBy' => 'id']);
		
		//add this custom field relation
		self::hasOne('customfields', BandCustomFields::class, ['id' => 'id']);

		parent::defineRelations();
	}

}

````````````````````````````````````````````````````````````````````````````````

## Add a custom field with the API

You can fetch a band now with custom fields by using the returnProperties query parameter:

/bands/1?returnProperties=*,albums,customfields

The custom fields will have a null value. We must first add a custom field for the model.

````````````````````````````````````````````````````````````````````````````````
{
  "data": {
    "customfields": null,
    "id": 1,
    "name": "Pearl Jam",
    "createdBy": 1,
    "createdAt": "2016-02-12T10:12:15Z",
    "modifiedAt": "2016-02-12T10:15:40Z",
    "deleted": true,
    "permissions": {
      "create": true,
      "read": true,
      "update": true,
      "delete": true,
      "changePermissions": true
    },
    "validationErrors": [],
    "className": "UX\Modules\Bands\Model\Band",
    "markDeleted": false
  },
  "success": true
}
````````````````````````````````````````````````````````````````````````````````

POST this data to the route "/customfields/fieldsets/UX%5CModules%5CBands%5CModel%5CBand"

The model name must be URL encoded.

````````````````````````````````````````````````````````````````````````````````
{
    "data": {
        "modelName": "UX\\Modules\\Bands\\Model\\BandCustomFields",
        "name": "More info",
				"fields": [{
					"type": "text",
					"name": "Text field",
					"databaseName": "textfield",
					"placeholder": "Type something",
					"required": false,
					"defaultValue": ""			
				}]
    } 
}
````````````````````````````````````````````````````````````````````````````````

Now we can set this field on the band model. Do a PUT request to route 
"/bands/1?returnProperties=*,albums,customfields" to update the band:


````````````````````````````````````````````````````````````````````````````````
{
	"data": {		
		"customfields": {
			"textfield": "My text value"
		}
	}

}

````````````````````````````````````````````````````````````````````````````````

This will return on success:

````````````````````````````````````````````````````````````````````````````````
{
  "data": {
    "customfields": {
      "id": 1,
      "textfield": "My text value",
      "permissions": {
        "create": true,
        "read": true,
        "update": true,
        "delete": true,
        "changePermissions": true
      },
      "validationErrors": [],
      "className": "UX\Modules\Bands\Model\BandCustomFields",
      "markDeleted": false
    },
    "albums": [
      {
        "id": 1,
        "bandId": 1,
        "name": "Ten",
        "createdBy": 1,
        "createdAt": "2016-02-12T10:12:15Z",
        "modifiedAt": "2016-02-12T10:12:15Z",
        "permissions": {
          "create": true,
          "read": true,
          "update": true,
          "delete": true,
          "changePermissions": true
        },
        "validationErrors": [],
        "className": "UX\Modules\Bands\Model\Album",
        "markDeleted": false
      },
      {
        "id": 2,
        "bandId": 1,
        "name": "Backspacer",
        "createdBy": 1,
        "createdAt": "2016-02-12T10:12:15Z",
        "modifiedAt": "2016-02-12T10:12:15Z",
        "permissions": {
          "create": true,
          "read": true,
          "update": true,
          "delete": true,
          "changePermissions": true
        },
        "validationErrors": [],
        "className": "UX\Modules\Bands\Model\Album",
        "markDeleted": false
      }
    ],
    "id": 1,
    "name": "Pearl Jam",
    "createdBy": 1,
    "createdAt": "2016-02-12T10:12:15Z",
    "modifiedAt": "2016-02-12T10:46:34Z",
    "deleted": true,
    "permissions": {
      "create": true,
      "read": true,
      "update": true,
      "delete": true,
      "changePermissions": true
    },
    "validationErrors": [],
    "className": "UX\Modules\Bands\Model\Band",
    "markDeleted": false
  },
  "success": true
}

````````````````````````````````````````````````````````````````````````````````

As you can see the value is now stored in the custom fields model.