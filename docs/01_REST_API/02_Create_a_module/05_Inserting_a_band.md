Now your API is ready but the GET request on the /bands route returns a boring 
empty result. So let's insert a band!

Use POSTMan to add an new band with the API.

Do a GET request on /bands/0 to get an empty band model with default values:

````````````````````````````````````````````````````````````````````````````````
{
  "data": {
    "albums": [],
    "id": null,
    "name": null,
    "ownedBy": 1,
    "createdAt": null,
    "modifiedAt": null,
    "deleted": false,
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

Now we can POST a new band with albums. GroupOffice can handle relations in one 
request too. We only need to post the required values name and albums. All the 
other attributes are determined by GroupOffice automatically.

````````````````````````````````````````````````````````````````````````````````
{
    "data": {
        "name": "Pearl Jam",
				"albums": [{
					"name": "Ten"
				},{
					"name": "Backspacer"
				}]
    }
}
````````````````````````````````````````````````````````````````````````````````

If successful it responds with the new band:

````````````````````````````````````````````````````````````````````````````````
{
  "data": {
    "albums": [
      {
        "id": 1,
        "bandId": 1,
        "name": "Ten",
        "ownedBy": 1,
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
        "ownedBy": 1,
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
    "ownedBy": 1,
    "createdAt": "2016-02-12T10:12:15Z",
    "modifiedAt": "2016-02-12T10:12:15Z",
    "deleted": false,
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