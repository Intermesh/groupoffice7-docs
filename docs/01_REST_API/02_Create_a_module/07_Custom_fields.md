GroupOffice comes with a powerful custom fields module so users can extend models with their own custom fields.
We're going to make the Bands model extensible with custom fields in this example.

## Create the model BandCustomFields and make this extend the AbstractCustomfieldsRecord:

````````````````````````````````````````````````````````````````````````````````
<?php
namespace GO\Modules\Bands\Model;

use GO\Modules\CustomFields\Model\AbstractCustomFieldsRecord;

class BandCustomFields extends AbstractCustomFieldsRecord{	

}

````````````````````````````````````````````````````````````````````````````````

Add the database patch file Install/Database/20150115-1529.sql:

Note that's important to make it cascade on band deletion.

````````````````````````````````````````````````````````````````````````````````
CREATE TABLE `bands_band_custom_fields` (
  `id` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  CONSTRAINT `bands_band_custom_fields_ibfk_1` FOREIGN KEY (`id`) REFERENCES `bands_band` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

````````````````````````````````````````````````````````````````````````````````

And run the route /modules/update to apply it.


## Define relation in the Band model:
``````````````````````````````````````````````````````````````````
self::hasOne('customfields', BandCustomFields::class, ['id' => 'id']);	
```````````````````````````````````````````````````````````````````

## Add a custom field with the API

You can fetch a band now with custom fields by using the returnAttibutes query parameter:

/bands/1?returnAttributes=*,albums,customfields

The custom fields will have a null value. We must first add a custom field for the model.



POST this data to /customfields/fieldsets/GO%5CModules%5CBands%5CModel%5CBand

The model name must be URL encoded.

````````````````````````````````````````````````````````````````````````````````
{
    "data": {
        "modelName": "GO\\Modules\\Bands\\Model\\BandCustomFields",
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

Now we can set this field on the band model. Do a PUT request to update the
band:

PUT /bands/1?returnAttributes=*,albums,customfields

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
        "id": 1,
        "name": "Pearl Jam",
        "ownerUserId": 1,
        "createdAt": "2015-01-15T14:04:21Z",
        "modifiedAt": "2015-01-15T15:00:24Z",
        "deleted": false,
        "isNew": false,
        "validationErrors": [],
        "className": "GO\\Modules\\Bands\\Model\\Band",
        "markDeleted": false,
        "albums": [
            {
                "id": 1,
                "bandId": 1,
                "name": "Ten",
                "ownerUserId": 1,
                "createdAt": "2015-01-15T14:04:21Z",
                "modifiedAt": "2015-01-15T14:04:21Z",
                "isNew": false,
                "validationErrors": [],
                "className": "GO\\Modules\\Bands\\Model\\Album",
                "markDeleted": false,
                "isOwner": true
            },
            {
                "id": 2,
                "bandId": 1,
                "name": "Backspacer",
                "ownerUserId": 1,
                "createdAt": "2015-01-15T14:04:21Z",
                "modifiedAt": "2015-01-15T14:04:21Z",
                "isNew": false,
                "validationErrors": [],
                "className": "GO\\Modules\\Bands\\Model\\Album",
                "markDeleted": false,
                "isOwner": true
            }
        ],
        "customfields": {
            "id": 1,
            "textfield": "My text value",
            "isNew": false,
            "validationErrors": [],
            "className": "GO\\Modules\\Bands\\Model\\BandCustomFields",
            "markDeleted": false
        },
        "isOwner": true
    },
    "success": true
}

````````````````````````````````````````````````````````````````````````````````

As you can see the value is now stored in the custom fields model.