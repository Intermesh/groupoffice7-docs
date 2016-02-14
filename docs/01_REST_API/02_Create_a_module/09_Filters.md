In modules we often use filters. For example you can filter on tags or contact gender etc. We've created a filtering
mechanism that makes it easy to implement them.

We're going to implement a filter by album genre. There are various
predefined [filter types](http://intermesh.io/php/docs/namespace-IFW.Data.Filter.html)

We'll use the multi select filter.

Create UX/Modules/Bands/Filter/Genre.php:

````````````````````````````````````````````````````````````````````````````````
<?php
namespace UX\Modules\Bands\Filter;

use IFW\Data\Filter\FilterOption;
use IFW\Data\Filter\MultiselectFilter;
use IFW\Orm\Query;
use PDO;
use UX\Modules\Bands\Model\Album;

class Genre extends MultiselectFilter {
	
	/**
	 * Applies the filter on a store query
	 * 
	 * @param Query $query
	 */
	public function apply(Query $query) {
		
		$selected = $this->getSelected();
		
		if(!empty($selected)) {
			$query->joinRelation('albums')
						->groupBy(['id'])
						->andWhere(['albums.genre'=>$selected]);
		}
	}

	/**
	 * Get's all genres
	 * 
	 * Uses a distinct select query on the album genre column
	 * 
	 * @return string[]
	 */
	public function getOptions() {
		$genres = Album::find(
						(new Query())
						->fetchMode(PDO::FETCH_COLUMN, 0)
						->select('genre')
						->distinct()
						);
		
		$options = [];
		foreach($genres as $genre) {
			$options[] = new FilterOption($this, $genre, $genre, $this->count($genre));
		}
		
		return $options;
		
	}
	
	/**
	 * Counts the number of occurenses but also applies all selected filter 
	 * options in this query.
	 * 
	 * @param string $genre
	 * @return int
	 */
	private function count($genre){
		
		$query = $this->collection->countQuery();		
		$this->collection->apply($query);		
		$query->where(['albums.genre' => $genre]);
		
		return (int) call_user_func([$this->collection->getModelClassName(), 'find'], $query)->single();
	}

}

````````````````````````````````````````````````````````````````````````````````


## Applying filters

Now we must attach this filter in the BandController. 
Edit UX/Modules/Bands/Filter/BandController.php and add an action to get the
filters:


Add these uses:

````````````````````````````````````````````````````````````````````````````````
use GO\Core\CustomFields\Model\Field;
use IFW\Data\Filter\FilterCollection;
use UX\Modules\Bands\Filter\Genre;
use UX\Modules\Bands\Model\BandCustomFields;
````````````````````````````````````````````````````````````````````````````````

Add this action and private function:

````````````````````````````````````````````````````````````````````````````````
public function actionFilters() {
	$this->render($this->getFilterCollection()->toArray());		
}

private function getFilterCollection() {
	$filters = new FilterCollection(Band::class);

	$filters->addFilter(Genre::class);

	//Adds custom field filters automatically
	Field::addFilters(BandCustomFields::class, $filters);

	return $filters;
}
````````````````````````````````````````````````````````````````````````````````

And finally modify actionStore to apply the filters to the store query. I've 
also add the debug() call to the query so we can see the SQL in our debug output:


````````````````````````````````````````````````````````````````````````````````
protected function actionStore($orderColumn = 'name', $orderDirection = 'ASC', $limit = 10, $offset = 0, $searchQuery = "", $returnProperties = "", $where = null) {

	$query = (new Query())
					->orderBy([$orderColumn => $orderDirection])
					->limit($limit)
					->offset($offset)
					->debug();

	if (!empty($searchQuery)) {
		$query->search($searchQuery, ['t.bandname']);
	}

	if (!empty($where)) {

		$where = json_decode($where, true);

		if (count($where)) {
			$query
							->groupBy(['t.id'])
							->whereSafe($where);
		}
	}

	//Add this line to apply the filters
	$this->getFilterCollection()->apply($query);		

	$bands = Band::find($query);
	$bands->setReturnProperties($returnProperties);

	$this->renderStore($bands);
}
````````````````````````````````````````````````````````````````````````````````


## Using the filters

To demonstrate the filters we must have band with a different genre. So do this
post request to /bands:
````````````````````````````````````````````````````````````````````````````````
{
    "data": {
        "name": "Katy Perry",
				"albums": [{
					"name": "Prism",
					"genre": "Pop"
				}]
    }
}

````````````````````````````````````````````````````````````````````````````````


Each filter has a name that defaults to the class name. The function getName()
can be overridden to change the name but in most cases this won't be necessary.
The name is also used as query paramter to send the filter value. Multiple values
are separated by a comma. So to filter on genre we can do this request:

GET http://localhost/api/bands?Genre=Rock,Pop

Remove Pop or Rock to see different results.


## Getting the filters

In the controller we've also created a new action to get the filter information.
First we must create a route to this action in UX/Modules/Bands/Module.php:

````````````````````````````````````````````````````````````````````````````````
public static function defineHttpRoutes(Router $router) {

	$router->addRoutesFor(HelloController::class)
					->get('bands/hello', 'name');

	$router->addRoutesFor(BandController::class)
					->get('bands', 'store')
					->get('bands/0', 'new')
					->get('bands/:bandId', 'read')
					->put('bands/:bandId', 'update')
					->post('bands', 'create')
					->delete('bands/:bandId', 'delete')
					
					//Add this route
					->get('bands/filters', 'filters');
}
````````````````````````````````````````````````````````````````````````````````


Now we can do a get request on /bands/filters. The response looks like this:

````````````````````````````````````````````````````````````````````````````````
{
  "filters": [
    {
      "name": "Genre",
      "label": null,
      "type": "multiselect",
      "clearFilters": [],
      "selected": [
        "Rock"
      ],
      "options": [
        {
          "count": 1,
          "label": "Rock",
          "value": "Rock",
          "selected": true,
          "className": "IFW\Data\Filter\FilterOption"
        },
        {
          "count": 1,
          "label": "Pop",
          "value": "Pop",
          "selected": false,
          "className": "IFW\Data\Filter\FilterOption"
        }
      ],
      "className": "UX\Modules\Bands\Filter\Genre"
    }
  ],
  "count": 1,
  "className": "IFW\Data\Filter\FilterCollection",
  "success": true
}
````````````````````````````````````````````````````````````````````````````````

## Passing filter options
Each filter option also returns the number of occurrences in the "count" property.

We can also pass the same get parameters to this action so the counts update
with that option enabled:

Do a get with Genre=Rock and you'll get count=0 for Pop in the result. This is
because there are zero Pop bands that also have the Rock genre.