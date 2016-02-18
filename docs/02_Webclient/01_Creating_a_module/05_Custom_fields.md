In the server tutorial we've also added a custom field. The webclient API has some directives to use those customfields.


# Band view
edit the band view in 'ux/tutorial/modules/bands/views/band.html' and add right before '</md-content>':

````````````````````````````````````````````````````````````````````````````````
<go-custom-fields-detail ng-model="band.customfields" server-model="UX\Modules\Bands\Model\BandCustomFields"></go-custom-fields-detail>
````````````````````````````````````````````````````````````````````````````````

Now reload the client and it should show the value.

# Band edit dialog

A similar directive is available for the edit dialog. 

Edit the view 'ux/tutorial/modules/bands/views/band-edit.html' and add right before '</md-content>':

````````````````````````````````````````````````````````````````````````````````
<go-custom-fields-edit form-controller="bandForm" ng-model="model.customfields" server-model="UX\Modules\Bands\Model\BandCustomFields"></go-custom-fields-edit>			
````````````````````````````````````````````````````````````````````````````````

NOw reload the client and try the edit dialog.