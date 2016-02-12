Deleting stuff in GroupOffice can be dangerous. That's why we support soft deletion.

Now create a database patch file in Install/Database/20150115-1513.sql and add the deleted column:

````````````````````````````````````````````````````````````````````````````````
ALTER TABLE `bands_band` ADD `deleted` BOOLEAN NOT NULL DEFAULT FALSE , ADD INDEX (`deleted`) ; 
````````````````````````````````````````````````````````````````````````````````

Now run the API route /system/upgrade to update the database.

Now use POSTMan to do a DELETE request to /bands/1. Notice that it just changes
the deleted boolean:

````````````````````````````````````````````````````````````````````````````````
{
  "data": {
    "id": 1,
    "name": "Pearl Jam",
    "ownedBy": 1,
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

Now undelete the band because we need it later in the tutorial. Do a PUT request
to /bands/1 with the deleted property set to false:

````````````````````````````````````````````````````````````````````````````````
{
	"data": {		
	    "deleted": false
	}

}
````````````````````````````````````````````````````````````````````````````````
