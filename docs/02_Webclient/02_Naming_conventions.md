Namespaces
----------

AngularJS lacks namespace support but we've found a workaround. When declaring a 
controller for example we use the [Inline Array Annotation](https://docs.angularjs.org/guide/di).

In the strings we can use dots to mimick a namespace. In the variables we can just use the last part.
So the string is the full name space that we use as the variable name.

`````````````````````````````````````````````````````````````````````````````````````````````````
angular.module('GO.Modules.Email').
		controller('GO.Modules.Email.EmailController', ['$scope', 'GO.Modules.Email.Account', function($scope, Account) {
});

`````````````````````````````````````````````````````````````````````````````````````````````````