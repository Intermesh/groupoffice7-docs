GroupOffice has powerful ORM database models. You must create a model for each 
database table. Relations are also defined in the models. In this case a band **has
many** albums and an album **belongs to** a band.

Read more about models in the API documentation:

http://intermesh.io/php/docs/class-IFW.Orm.Record.html


Band:

````````````````````````````````````````````````````````````````````````````````
<?php
namespace GO\Modules\Bands\Model;

use GO\Core\Auth\Model\User;
use GO\Modules\Bands\BandsModule;
use IFW\Exception\Forbidden;
use IFW\Orm\Record;

/**
 * The Band model
 *
 * @property int $id
 * @property string $name
 * @property int $ownerUserId
 * @property User $owner
 * @property string $createdAt
 * @property string $modifiedAt
 * 
 * @property Album[] $albums
 *
 * @copyright (c) 2015, Intermesh BV http://www.intermesh.nl
 * @author Merijn Schering <mschering@intermesh.nl>
 * @license http://www.gnu.org/licenses/agpl-3.0.html AGPLv3
 */
class Band extends Record{	
	
	const PERMISSION_LISTEN = 3;
	
	protected static function defineRelations() {		
		self::hasMany('albums', Album::class, ['id'=>'bandId']);
		self::hasOne('owner', User::class, ['ownerUserId' => 'id']);	
		parent::defineRelations();
	}	
}



````````````````````````````````````````````````````````````````````````````````

Album:

````````````````````````````````````````````````````````````````````````````````
<?php
namespace GO\Modules\Bands\Model;

use GO\Core\Auth\Model\User;
use IFW\Orm\Record;

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
 * @property Band $band
 *
 * @copyright (c) 2015, Intermesh BV http://www.intermesh.nl
 * @author Merijn Schering <mschering@intermesh.nl>
 * @license http://www.gnu.org/licenses/agpl-3.0.html AGPLv3
 */
class Album extends Record {
	protected static function defineRelations() {
		self::hasOne('band', Band::class, ['bandId' => 'id']);
		self::hasOne('owner', User::class, ['ownerUserId' => 'id']);
		
		User::hasMany('albums', Album::class, ['ownerUserId' => 'id']);
		
		parent::defineRelations();
	}
}
````````````````````````````````````````````````````````````````````````````````

> ## TIP
> Use these API routes to generate the documentation blocks: 
>
> - /devtools/model/GO\Modules\Bands\Model\Band
> - /devtools/model/GO\Modules\Bands\Model\Album
