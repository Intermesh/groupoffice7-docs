GroupOffice has powerful ORM database models. You must create a model for each 
database table. Relations are also defined in the models. In this case a band **has
many** albums and an album **belongs to** a band.

Read more about models in the API documentation:

http://intermesh.io/php/docs/class-GO.Core.Db.AbstractRecord.html


Band:

````````````````````````````````````````````````````````````````````````````````
<?php
namespace GO\Modules\Bands\Model;

use GO\Core\Db\AbstractRecord;

/**
 * The Band model
 *
 * @property int $id
 * @property string $name
 * @property int $ownerUserId
 * @property \GO\Modules\Auth\Model\User $owner
 * @property string $createdAt
 * @property string $modifiedAt
 *
 * @copyright (c) 2015, Intermesh BV http://www.intermesh.nl
 * @author Merijn Schering <mschering@intermesh.nl>
 * @license http://www.gnu.org/licenses/agpl-3.0.html AGPLv3
 */
class Band extends AbstractRecord{
	protected static function defineRelations(\GO\Core\Db\RelationFactory $r) {
		return [$r->hasMany('albums', Album::className(), 'bandId')];
	}
}
````````````````````````````````````````````````````````````````````````````````

Album:

````````````````````````````````````````````````````````````````````````````````
<?php
namespace GO\Modules\Bands\Model;

use GO\Core\Db\AbstractRecord;
use GO\Core\Auth\Model\User;

/**
 * The Album model
 *
 * @property int $id
 * @property int $bandId
 * @property string $name
 * @property int $ownerUserId
 * @property User $owner
 * @property string $createdAt
 * @property string $modifiedAt
 *
 * @copyright (c) 2015, Intermesh BV http://www.intermesh.nl
 * @author Merijn Schering <mschering@intermesh.nl>
 * @license http://www.gnu.org/licenses/agpl-3.0.html AGPLv3
 */
class Album extends AbstractRecord {
	protected static function defineRelations(\GO\Core\Db\RelationFactory $r) {
		return [$r->belongsTo('band', Band::className(), 'bandId')];
	}
}
````````````````````````````````````````````````````````````````````````````````

> ## TIP
> Use these API routes to generate the documentation blocks: 
>
> - /devtools/model/GO\Modules\Bands\Model\Band
> - /devtools/model/GO\Modules\Bands\Model\Album
