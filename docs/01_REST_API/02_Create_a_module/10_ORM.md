GroupOffice has a powerful Object Relational Mapping system. 

It uses an active record implementation that is populated from the database. In
other words database columns will become magic properties in the Record models.

It also supports relations to other models.

To learn more about the ORM of Group-Office please read these API pages:

- http://intermesh.io/php/docs/class-IFW.Db.Record.html
- http://intermesh.io/php/docs/class-IFW.Db.Record.html#_find
- http://intermesh.io/php/docs/class-IFW.Db.Record.html#_defineRelations
- http://intermesh.io/php/docs/class-IFW.Db.Record.html#_save

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
