The three column view that we've created here is typical in GroupOffice.

We're going to apply the following rules:

1. The side nav on the left collapses on smaller devices like tablets and smartphones.
2. On smart phones and the list and info view will be full screen cards.
3. The edit dialog will be full screen on smart phones (this is already built in).


# Side navigation

The side navigation panel on the left is already responsive. In the main view
we've put this attribute in the 'md-sidenav' element:

`````````````````````````````````````
md-is-locked-open="$mdMedia('gt-md')"
``````````````````````````````````````

This will enable the side panel on screens greater than tablet screens (For 
more info visit the Angular Material site).

We've also added a button in the list that shows on screens smaller than a desktop.
Try to make your browser window smaller and you'll see how it works.

# List and info view
What's not working yet is the list and info view.

Change the main view 'ux/tutorial/modules/bands/view/main.html':

````````````````````````````````````````````````````````````````````````````````
<md-sidenav class="md-sidenav-left md-whiteframe-z2" md-component-id="left" md-is-locked-open="$mdMedia('gt-md')">
	<div go-filter-collection="bands/filters" go-store="store" flex></div>
</md-sidenav>

<!-- Add the following line and add the next div element -->
<div class="go-cards-sm" flex layout="row">

	<div class=" go-card go-list" layout="column" ng-class="{'go-active' : $state.is('bands')}">
		<go-list-toolbar store="store">
			<div class="md-toolbar-tools">
				<md-button aria-label="{{::'Open side navigation'| goT}}" ng-click="toggleSidenav('left')" hide-gt-md class="md-icon-button">
					<md-icon class="mdi-menu"></md-icon>
				</md-button>

				<span flex></span>

				<go-search-button></go-search-button>

				<md-menu md-position-mode="target-right target">
					<md-button aria-label="{{::'More options'| goT}}" class="md-icon-button" ng-click="$mdOpenMenu($event)">
						<md-icon md-menu-origin class="mdi-dots-vertical"></md-icon>
					</md-button>
					<md-menu-content>
						<md-menu-item>
							<md-button ng-click="store.reload()">{{"Refresh"| goT}}</md-button>
						</md-menu-item>
					</md-menu-content>
				</md-menu>
			</div>			
		</go-list-toolbar>


		<go-list store="store" flex>

			<item index="model.name" ui-sref="bands.band({bandId: model.id})">
				<div class="md-list-item-text">
					<h3>{{model.name}}</h3>
				</div>
			</item>

			<empty-state>
				<md-icon class="mdi-account"></md-icon>
				<p>{{"No bands found"| goT}}</p>
			</empty-state>

		</go-list>
		
		<div class="go-fab-list">
			<md-button class="md-fab"  ng-click='edit()'>
				<md-icon class="mdi-plus"></md-icon>
				<md-tooltip md-direction="left">{{"Add"| goT}}</md-tooltip>
			</md-button>
		</div>
	</div>


	<!-- Change the following line -->
	<div flex ui-view layout="column" class="go-card go-info-panel" ng-class="{'go-active' : !$state.is('bands')}"></div>
</div>

````````````````````````````````````````````````````````````````````````````````

Check the lines that follow a comment. We've added a div element with class
'go-cards-sm' which will turn the child 'go-card' elements into a card layout
when viewed on small devices.

in the '$rootScope' we've added the '$state' so we can use that service in any
view. We're using the state 'bands' to toggle the list and info view.