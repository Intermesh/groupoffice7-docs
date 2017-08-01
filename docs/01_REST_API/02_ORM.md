GroupOffice has a powerful Object Relational Mapping system. 

It uses an active record implementation that is populated from the database. In
other words database columns will become magic properties in the Record models.

It also supports relations to other models.

To learn more about the ORM of Group-Office please read these API pages:

- http://groupoffice.io/php/docs/class-IFW.Orm.Record.html
- http://groupoffice.io/php/docs/class-IFW.Orm.Record.html#_find
- http://groupoffice.io/php/docs/class-IFW.Orm.Record.html#_defineRelations
- http://groupoffice.io/php/docs/class-IFW.Orm.Record.html#_save

## Relational saving

Relations can be saved directly on the models.

Here's some example code that demonstrates this:

````````````````````````````````````````````````````````````````````````````````
$contactAttr = ['firstName' => 'Test', 'lastName'=>'Has One'];		
$contact = Contact::find($contactAttr)->single();
if($contact){
	$contact->deleteHard();
}

$email = new \GO\Modules\Contacts\Model\ContactEmailAddress();
$email->email = 'test3@intermesh.nl';


$contactAttr['emailAddresses'] = [
		['email'=>'test1@intermesh.nl','type'=>'work'],
		['email'=>'test2@intermesh.nl','type'=>'work'],
		$email				
];

$contactAttr['phoneNumbers'] = [
		['number'=>'1234567890','type'=>'work']				
];

$contact = new Contact();
$contact->setValues($contactAttr);			

$success = $contact->save() !== false;		

$firstEmail = $contact->emailAddresses->single();
$firstEmail->type='home';

//update single email
$contact->emailAddresses[] = $firstEmail;
$contact->emailAddresses[] = ['email'=>'piet@intermesh.nl'];


$success = $contact->save() !== false;	
````````````````````````````````````````````````````````````````````````````````	

## Identifying relations
An identifying relation is like an attribute. For example email addresses of a 
contact are identifying related models. It's determined by the database primary
key. The to key in the related model must be included.

For example with contact.emailAddresses the primary key of the email address model must be 
['id', 'contactId']

See also:
http://stackoverflow.com/questions/762937/whats-the-difference-between-identifying-and-non-identifying-relationships

At the moment the only place where the difference is made is in the record 
copy function. Identifying relations are copied along by default.










TODO:

new implementation:

$contact = new Contact();
		
		$contact->customFields = [				
			'test' => 'test'	
		]; //creates new or updates existing because it's an identifying relation
		
		$contact->emailAddresses[] = [
				'id' => 1,
				"email" => 'test'
		]; //updates contact e-mail address
		
		$contact->emailAddresses[] = [
				"email" => 'test'
		]; //Creates new address
		
		
		
		$contact->emailAddresses[0]->email = $veld1;
		$contact->emailAddresses[1]->email = $test2;
		
		
		$contact->owner = [
				'id' => 1,
				'name' =>  'Admins'
		]; //throwed exception. You can't change non identifying relations
		
		$contact->ownedBy = 1; //is the way to change owner

LATER: s possible but can only change the key The user object is never saved
		
		
		$admins = \GO\Core\Users\Model\Group::findAminGroup();
		
		$contact->owner = $admins; //throws exception 
		
		
		$contact->owner->name="Superadmins";
		$contact->save();
		//Doesn't work. It will not save owner
		
		
		$contact->organizations[] = ['id' => 1]; //creates via record
		
		$contact->organizations[] = ['id' => 1, 'markDeleted' => true]; //deletes via record
		
		
		$user->tasks[] = new Task(); //not identifying throws exception
		