GroupOffice has a [RESTFul](http://en.wikipedia.org/wiki/Representational_state_transfer)
API. It uses JSON as data format. To learn how the API works we're going to use
the tool [POSTMan](https://www.getpostman.com/).

## Routes
You've installed the API somewhere. For example on "http://localhost/api". 
When I refer to a route in this document, for example route "/auth" then this means:

http://localhost/api/auth

You can find a list of available routes here:

http://groupoffice.io/php/docs/class-GO.Core.Web.App.html

## Authenticate

The authentication route is "/auth" so you can POST this JSON in the body to 
obtain the access token and the XSRFToken in a cookie. In POSTman you can set 
the method to POST and the body to "raw" with the content type at 
"application/json":

```````````````````````````````````````````
{
    "data" : {
        "username" : "admin",
        "password" : "Admin1!"
    }
}
```````````````````````````````````````````

**Note:** The cookies are not visible inside postman but added by chrome in the 
background. The XSRFToken must be sent in a header "X-XSRFToken" or GET 
parameter "XSRFToken" on each request. 

More technical details can be found in the API documentation:

http://groupoffice.io/php/docs/class-GO.Core.Auth.Browser.Model.Token.html

The server should respond with the token object this:

``````````````````
{
  "data": {
    "user": {
      "id": 1,
      "deleted": false,
      "enabled": true,
      "username": "admin",
      "createdAt": "2014-07-21T14:01:17Z",
      "modifiedAt": "2016-02-11T11:01:28Z",
      "loginCount": 83,
      "lastLogin": "2016-02-11T12:01:28Z",
      "isAdmin": true,
      "permissions": {
        "create": true,
        "read": true,
        "update": true,
        "delete": true,
        "changePermissions": true
      },
      "validationErrors": [],
      "className": "GO\Core\Auth\Model\User",
      "currentPassword": null,
      "markDeleted": false
    },
    "accessToken": "97e52a77985e1420ece3ee4ea228fd9e75ecba28",
    "XSRFToken": "d9f035533b0600710abf7051894a78ec39bb1663",
    "userId": 1,
    "expiresAt": "2016-02-12T12:01:28Z",
    "permissions": {
      "create": true,
      "read": true,
      "update": true,
      "delete": true,
      "changePermissions": true
    },
    "validationErrors": [],
    "className": "GO\Core\Auth\Browser\Model\Token",
    "checkXSRFToken": false,
    "markDeleted": false
  },
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
  "data": [
    {
      "id": 1,
      "deleted": false,
      "enabled": true,
      "username": "admin",
      "createdAt": "2014-07-21T14:01:17Z",
      "modifiedAt": "2016-02-11T11:01:28Z",
      "loginCount": 83,
      "lastLogin": "2016-02-11T12:01:28Z",
      "isAdmin": true,
      "permissions": {
        "create": true,
        "read": true,
        "update": true,
        "delete": true,
        "changePermissions": true
      },
      "validationErrors": [],
      "className": "GO\Core\Auth\Model\User",
      "currentPassword": null,
      "markDeleted": false
    }
  ],
  "success": true
}
````````````````````````````````````````````````````````````````````````````````

A GET request that lists objects will return a "data" array with models in them.

All the routes link to their controller class where you can read more about the GET 
parameters they support.

In this case the controller class is:

http://groupoffice.io/php/docs/class-GO.Core.Auth.Controller.UserController.html

You can read more about this [AbstractController](http://groupoffice.io/php/docs/class-GO.Core.Controller.AbstractController.html) 
in the API documenation.

## Data format
Each controller usually works with a model. The model should be mentioned in the
controller documentation. In this case it's the [User](http://groupoffice.io/php/docs/class-GO.Core.Auth.Model.User.html) 
model. To see **which properties** the user has look it up in the API 
documentation and take a look at the **magic properties**.

Note that some of the properties are read only like the primary key, className 
and validationErrors. You'll get an error if you try to set these.

You can view a single user by doing a GET request to the route "/auth/users/1".
This will return the user:

````````````````````````````````````````````````````````````````````````````````
{
  "data": {
    "id": 1,
    "deleted": false,
    "enabled": true,
    "username": "admin",
    "createdAt": "2014-07-21T14:01:17Z",
    "modifiedAt": "2016-02-11T11:01:28Z",
    "loginCount": 83,
    "lastLogin": "2016-02-11T12:01:28Z",
    "isAdmin": true,
    "permissions": {
      "create": true,
      "read": true,
      "update": true,
      "delete": true,
      "changePermissions": true
    },
    "validationErrors": [],
    "className": "GO\Core\Auth\Model\User",
    "currentPassword": null,
    "markDeleted": false
  },
  "success": true
}
````````````````````````````````````````````````````````````````````````````````

When saving a user you must PUT (for update) or POST (for create) this user model 
in the same format. The server will respond with the success boolean. On failure 
the model will have one or more "validationErrors".

In the PHP API the user is a Model based on [Record](http://groupoffice.io/php/docs/class-IFW.Orm.Record.html).


## Debugging

Debugging can be enabled in the config.php file or by sending the HTTP header
X-Debug=1. You will get extra debug output in the JSON responses.

In our webclient you can turn it on and off by pressing CTRL + F7.

## XML
The API defaults to JSON but also supports XML. If you want XML format you can 
set a header in the HTTP requests:

````````````````````````````````````````````````````````````````````````````````
Accept: application/xml
````````````````````````````````````````````````````````````````````````````````

or:

````````````````````````````````````````````````````````````````````````````````
Content-Type: application/xml
````````````````````````````````````````````````````````````````````````````````

The root element of the XML body is always “message”.
So for example the login XML request looks like this:

````````````````````````````````````````````````````````````````````````````````
<?xml version="1.0"?>
<message>
	<data>
		<username>admin</username>
		<password>Admin1!</password>
	</data>
</message>
````````````````````````````````````````````````````````````````````````````````

And the response looks like:

````````````````````````````````````````````````````````````````````````````````
<?xml version="1.0"?>
<message>
	<data>
		<user>
			<id type="integer">1</id>
			<deleted type="boolean">false</deleted>
			<enabled type="boolean">true</enabled>
etc.
````````````````````````````````````````````````````````````````````````````````