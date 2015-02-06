Deleting stuff in GroupOffice can be dangerous. That's why we support soft deletion.
You can implement a Trait in your model to enable it. You also need to add an 'deleted' boolean column to the database.

Add this trait to the Band model (don't put this in as a namespace but inside the class definition):

````````````````````````````````````````````````````````````````````````````````
use \GO\Core\Db\SoftDeleteTrait;
````````````````````````````````````````````````````````````````````````````````

Now create a database patch file in Install/Database/20150115-1513.sql and add the deleted column:

````````````````````````````````````````````````````````````````````````````````
ALTER TABLE `bandsBand` ADD `deleted` BOOLEAN NOT NULL DEFAULT FALSE , ADD INDEX (`deleted`) ; 
````````````````````````````````````````````````````````````````````````````````

Now run the API route /modules/upgrade to update the database.

Now use POSTMan to do a DELETE request on /bands/1. Notice that it just changes
the deleted boolean.
