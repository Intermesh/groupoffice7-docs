To implement tailor made customizations GroupOffice has an events system. You can
attach listeners to events to execute extra code.

Attached listeners are persistent. If you attach a listener it's cached and 
always executed.

In this example we're going to set a debug line on every contact save.

You can put this code in any class but it's recommended to put it in a module 
manager class:

```````````````````````````````````````````````````````````````````````````````
public static function defineEvents() {		
	parent::defineEvents();

	\GO\Modules\Contacts\Model\Contact::on(\GO\Modules\Contacts\Model\Contact::EVENT_AFTER_SAVE, static::class, 'onContactSave');
}

public static function onContactSave($contact, $success, $isNew, $modifiedAttributes) {
	\IFW::app()->debug("Contact with ID ".$contact->id." was modified: ".var_export($modifiedAttributes, true));
}
````````````````````````````````````````````````````````````````````````````````

Now modify a contact (Make sure debugging is enabled with CTRL+F7 in the web 
client) and notice the debug message.