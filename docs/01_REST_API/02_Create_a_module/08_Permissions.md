
See also the API docs: http://intermesh.io/php/docs/class-GO.Core.Auth.Model.AbstractProtectedRecord.html

Group-Office also comes with a role based permissions feature. Let's say we want 
to restrict access to our bands. 
If we want basic permissions like read, write and delete we can make the Band 
model extend the [AbstractProtectedCrudRecord](http://intermesh.io/php/docs/class-GO.Core.Auth.Model.AbstractProtectedCrudRecord.html) model.

That's it! In the model read permissions will be checked in the contructor, 
write permissions in the save function and delete permissions in the delete 
function. No changes need to be made in the controller.

``````````````````````````````````````````````````````````````````````````````````````````````````

## Use the API to test it

Now we're going to test this with Postman. Make sure you're logged in and request 
the bands on route /bands. Note that admin users will always see everything.
Owners and admins may change permissions.

You can fetch the permissions with the following route:

/auth/permissions/:modelName/:modelId/roles

Note that the "\" char must be URL encoded as "%5C".

So if we do this get request: /auth/permissions/GO%5CModules%5CBands%5CModel%5CBand/1/roles

it will return something like this:

``````````````````````````````````````````````````````````````````````````````````````````````````

{
  "success": true,
  "results": [
    {
      "id": 2,
      "name": "Everyone",
      "permissionTypes": null,
      "isOwner": null,
      "validationErrors": [],
      "className": "GO\\\\Core\\\\Auth\\\\Model\\\\Role",
      "markDeleted": false,
      "permissions": {
        "read": false,
        "write": false,
        "delete": false
      }
    }
  ],
  "permissionTypes": [
    {
      "name": "read",
      "value": 0
    },
    {
      "name": "write",
      "value": 1
    },
    {
      "name": "delete",
      "value": 2
    }
  ]
}
``````````````````````````````````````````````````````````````````````````````````````````````````

It lists the available permissionTypes "read","write" and "delete".


We can use a PUT API call to /auth/permissions/GO%5CModules%5CBands%5CModel%5CBand/1/roles/2 to grant some permissions on the "Everyone" role:

````````````````````````````````````````````````````````````````````````````````
{"data":{"permissions":{"read":true,"write":true,"delete":false}}}
````````````````````````````````````````````````````````````````````````````````

Now the GET on /bands will return the band again and it will also have a 
**permissions** property that shows the permissions for the current user.
Note that the admin user will always have permissions.
