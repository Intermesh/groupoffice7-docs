In modules we often use filters. For example you can filter on tags or contact gender etc. We've created a filtering
mechanism that makes it easy to implement them.

We're going to implement a filter by album genre. There are various
predefined [filter types](http://intermesh.io/php/docs/namespace-IFW.Data.Filter.html)

We'll use the multi select filter.

Create UX/Modules/Bands/Filter/Genre.php:

````````````````````````````````````````````````````````````````````````````````
<?php
namespace UX\Modules\Bands\Filter;

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
		return Album::find(
						(new Query())
						->distinct()
						->fetchMode(PDO::FETCH_COLUMN,0)
						->select('genre')
						)->all();
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

````````````````````````````````````````````````````````````````````````````````


````````````````````````````````````````````````````````````````````````````````

