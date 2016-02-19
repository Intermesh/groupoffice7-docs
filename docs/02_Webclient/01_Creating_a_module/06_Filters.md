We're going to add some filters for our list in a side navigation pane. Because 
we've created a filter collection on the server side the client implementation 
is very easy.

Simply add this html to the main view in 'ux/tutorial/modules/bands/views/main.html':

````````````````````````````````````````````````````````````````````````````````
<md-sidenav class="md-sidenav-left md-whiteframe-z2" md-component-id="left" md-is-locked-open="$mdMedia('gt-md')">
	<div go-filter-collection="bands/filters" go-store="store" flex></div>
</md-sidenav>
````````````````````````````````````````````````````````````````````````````````

Because the filter collection is responsible now of loading the store we must 
remove this line in the main controller 'ux/tutorial/modules/bands/controller/main.js':

````````````````````````````````````````````````````````````````````````````````
$scope.store.load();

````````````````````````````````````````````````````````````````````````````````

Now reload the client and notice that the "Genre" filter created on the server
is displayed.

The md-icon tags will get the class "Genre Genre-{{option}}". You can use this
for styling.

