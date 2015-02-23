GroupOffice has a [RESTFul](http://en.wikipedia.org/wiki/Representational_state_transfer)
API. It uses JSON as data format. To learn how the API works we're going to use the Postman REST Client. I assume you've already got the server up and running.

## Routes
You've installed the API somewhere. For example on http://example.com/api/index.php. 
When I refer to a route in this document, for example route "/auth" then this means:

http://example.com/api/index.php/auth

You can find a list of available routes here:

http://intermesh.io/php/docs/class-GO.Core.Http.Router.html

## Authenticate

The authentication route is /auth so you can post this JSON object to obtain the 
access token and the XSRFToken in a cookie:

```````````````````````````````````````````
{
	"username": "admin",
	"password": "Admin1!"
}
```````````````````````````````````````````

**Note:** The cookies are not visible inside postman but added by chrome in the 
background. The XSRFToken must be sent in a header "X-XSRFToken" or GET 
parameter "XSRFToken" on each request. 

More technical details can be found in the API documentation:

http://intermesh.io/php/docs/class-GO.Core.Auth.Browser.Model.Token.html

The server should respond with:

``````````````````
{
	"XSRFToken: "abcdefg",
	"success": true
}
``````````````````

Use POSTMan to authenticate with a POST request.

**Note:** OAuth2 will also be supported in the future.

## Listing users

In the routes list you will also find "/auth/users". Now that you're logged in as
admin you are able to list the users:

Use postman to do a GET request to route "/auth/users". It will return something like this:

`````````````````````````````````````````````````````````````````````````````````
{
	"success": true,
	"results": [{
		"id": 1,
		"deleted": false,
		"enabled": true,
		"username": "admin",
		"password": "",
		"digest": "",
		"createdAt": "2014-07-21T14:01:17Z",
		"modifiedAt": "2015-01-15T10:58:32Z",
		"loginCount": 84,
		"lastLogin": "2015-01-15T11:58:32Z",
		"validationErrors": [ ],
		"className": "GO\\Modules\\Auth\\Model\\User",
		"currentPassword": null,
		"markDeleted": false
		}]
}
````````````````````````````````````````````````````````````````````````````````

A GET request that lists objects will return a "results" array with models in them.

All the routes link to their controller class where you can read more about the GET 
parameters they support.

In this case the controller class is:

http://intermesh.io/php/docs/class-GO.Core.Auth.Controller.UserController.html

You can read more about this [AbstractController](http://intermesh.io/php/docs/class-GO.Core.Controller.AbstractController.html) 
in the API documenation.

## Data format
Each controller usually works with a model. The model should be mentioned in the
controller documentation. In this case it's the [User](http://intermesh.io/php/docs/class-GO.Core.Auth.Model.User.html) 
model. To see which properties the user has look it up in the API documentation and take a look
at the magic properties.

Note that some of the properties are read only like the primary key, className 
and validationErrors. You'll get an error if you try to set these.

You can view a single user by doing a GET request to the route /auth/users/1.
This will return the user:

````````````````````````````````````````````````````````````````````````````````
{
    "data": {
        "id": 1,
        "deleted": false,
        "enabled": true,
        "username": "admin",
        "password": "",
        "digest": "",
        "createdAt": "2014-07-21T14:01:17Z",
        "modifiedAt": "2015-01-15T10:58:32Z",
        "loginCount": 84,
        "lastLogin": "2015-01-15T11:58:32Z",
        "validationErrors": [],
        "className": "GO\\Modules\\Auth\\Model\\User",
        "currentPassword": null,
        "markDeleted": false
    },
    "success": true
}
````````````````````````````````````````````````````````````````````````````````

When saving a user you must PUT (for update) or POST (for create) this user model 
in the same format. The server will respond with the success boolean. On failure 
the model will have one or more "validationErrors".

In the PHP API the user is a Model based on [AbstractRecord](http://intermesh.io/php/docs/class-GO.Core.Db.AbstractRecord.html).
