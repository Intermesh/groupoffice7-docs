The core has a built in comments feature.

To use this take the following steps. In this example we create comments for
contacts.

1. Create a link table between comments and your item.

For example:

```sql
CREATE TABLE `contacts_comment` (
  `commentId` int(11) NOT NULL,
  `contactId` int(11) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

ALTER TABLE `contacts_comment`
  ADD PRIMARY KEY (`commentId`),
  ADD KEY `contactId` (`contactId`);

ALTER TABLE `contacts_comment`
  ADD CONSTRAINT `contacts_comment_ibfk_1` FOREIGN KEY (`commentId`) REFERENCES `comments_comment` (`id`) ON DELETE CASCADE,
  ADD CONSTRAINT `contacts_comment_ibfk_2` FOREIGN KEY (`contactId`) REFERENCES `contacts_contact` (`id`) ON DELETE CASCADE;
```

Don't forget to put the SQL in an install patch file.

2. Generate a record and a controller for it:

```
cd GO/Modules/GroupOffice/Contacts
php ../../../../bin/groupoffice devtools/module/init --tablePrefix=contacts
```


2. Alter the generated record /GO/Modules/GroupOffice/Contacts/Model/Comment.php:

Please note that the commentId is not defined because it's required and already 
defined in the abstract core comment record.

```php

<?php

namespace GO\Modules\GroupOffice\Contacts\Model;

use GO\Core\Comments\Model\Comment as CoreComment;
use IFW\Auth\Permissions\ViaRelation;

/**
 * @property Contact $contact The contact this comment belongs to
 */
class Comment extends CoreComment {

	/**
	 *
	 * @var int 
	 */
	public $contactId;

	protected static function defineRelations() {
		parent::defineRelations();

		self::hasOne('contact', Contact::class, ['contactId' => 'id']);
	}

	/**
	 * Permissions via contact
	 * 
	 * @return \GO\Modules\GroupOffice\Contacts\Model\ViaRelation
	 */
	protected static function internalGetPermissions() {
		return new ViaRelation('contact');
	}

}

```

3. Modify the actionCreate in the generated controller in 
GO/Modules/GroupOffice/Contacts/Controller/CommentController.php. It needs to 
set contactId from the route parameters:

```
public function actionCreate($contactId, $returnProperties = "") {

	$comment = new Comment();
	$comment->setValues(GO()->getRequest()->body['data']);
	$comment->contactId = $contactId;
	$comment->save();

	$this->renderModel($comment, $returnProperties);
}
```


4. Create routes for the controller in the "defineWebRoutes" method inside /GO/Modules/GroupOffice/Contacts/Module.php:

```php
$router->addRoutesFor(Controller\CommentController::class)
						->crud('contacts/:contactId/comments', 'commentId');

```

5. Now the API for contact comments should work. Try it with postman.


6. Client side:

Inside the contacts view (app/modules/groupoffice/contacts/views/contact.html) add:

Inside the <md-tabs> element:
```
<md-tab ng-click="goto('comments')"><md-tab-label>{{::"Comments" | goT}}</md-tab-label></md-tab>		
```

Inside the <md-content> element:

```
<md-card  id="comments">
	<md-toolbar>
		<div class="md-toolbar-tools">
			{{"Comments"|goT}}
		</div>	
	</md-toolbar>

	<md-card-content>
		<go-comments ng-if="contact.id" go-route="contacts/{{contact.id}}/comments"></go-comments>
	</md-card-content>
</md-card>
```

7. Now try it!

