Now your API is ready but the GET request on the /bands route returns a boring empty result. Let's insert a band!

Use POSTMan to add an new band with the API.

Do a GET request on /bands/0 to get an empty band model with default values:

````````````````````````````````````````````````````````````````````````````````
{
    "data": {
        "id": null,
        "name": null,
        "ownerUserId": 1,
        "createdAt": null,
        "modifiedAt": null,
        "isNew": true,
        "validationErrors": [],
        "className": "GO\\Modules\\Bands\\Model\\Band",
        "markDeleted": false,
				"albums": [],
        "isOwner": true
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
        "id": 3,
        "name": "Pearl Jam",
        "ownerUserId": 1,
        "createdAt": "2015-01-15T14:04:21Z",
        "modifiedAt": "2015-01-15T14:04:21Z",
        "isNew": false,
        "validationErrors": [],
        "className": "GO\\Modules\\Bands\\Model\\Band",
        "markDeleted": false,
        "albums": [
            {
                "id": 1,
                "bandId": 3,
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
                "bandId": 3,
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
        "isOwner": true
    },
    "success": true    
}
````````````````````````````````````````````````````````````````````````````````