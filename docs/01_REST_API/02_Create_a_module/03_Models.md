GroupOffice has powerful ORM database models. You must create a model for each 
database table. Relations are also defined in the models. In this case a band **has
many** albums and an album **has one** band.

Read more about models and their relations in the API documentation:

http://intermesh.io/php/docs/class-IFW.Orm.Record.html


Create 'UX/Modules/Bands/Model/Band.php':

````````````````````````````````````````````````````````````````````````````````
<?php

namespace UX\Modules\Bands\Model;

use GO\Core\Auth\Model\User;
use IFW\Orm\Record;

/**
 * The Band model
 *
 * @property int $id
 * @property string $name
 * @property int $ownedBy
 * @property User $owner
 * @property \DateTime|string $createdAt
 * @property \DateTime|string $modifiedAt
 * 
 * @property Album[] $albums
 * @property BandCustomFields $customfields
 *
 * @copyright (c) 2015, Intermesh BV http://www.intermesh.nl
 * @author Merijn Schering <mschering@intermesh.nl>
 * @license http://www.gnu.org/licenses/agpl-3.0.html AGPLv3
 */
class Band extends Record {

	protected static function defineRelations() {
		self::hasMany('albums', Album::class, ['id' => 'bandId']);
		self::hasOne('owner', User::class, ['ownedBy' => 'id']);

		parent::defineRelations();
	}

}

````````````````````````````````````````````````````````````````````````````````

Create 'UX/Modules/Bands/Model/Album.php':

````````````````````````````````````````````````````````````````````````````````
<?php

namespace UX\Modules\Bands\Model;

use GO\Core\Auth\Model\User;
use IFW\Orm\Record;

/**
 * The Album model
 *
 * @property int $id
 * @property int $bandId
 * @property string $name
 * @property string $genre
 * @property int $ownerUserId
 * @property User $owner
 * @property \DateTime|string $createdAt
 * @property \DateTime|string $modifiedAt
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
		self::hasOne('owner', User::class, ['ownedBy' => 'id']);

		//Define this new relation in the user model too.
		User::hasMany('albums', Album::class, ['id' => 'ownedBy']);

		parent::defineRelations();
	}

}

````````````````````````````````````````````````````````````````````````````````

> ## TIP
> Open the route "/devtools/models" in your browser to generate the class 
> documentation block for your Record based on the database columns. It will 
> also generate a default controller to get started with.
