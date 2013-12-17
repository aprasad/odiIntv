#Problem Statement
You work for a company that maintains user's address book on their server like Gmail-contacts. As an engineer you have to come up with a technical design/proposal to support two-way address-book syncing with user's personal devices such as android phone, iphone, etc. Ignore authentication, address-book-entry-schema, etc. for this discussion. Keep in mind contacts can be edited both on server and on device by the user.
#Classes Used
##Server Classes
###AddressBook
* setContact()
* getContact()
* removeContact()

###User
* getUserContact()
* setUserContact()
* getDeviceList()
* setDeviceList()

###Device
* getRevisionHash()
* setRevisionHash()

###Contact
* getRevisionHash()
* create() 
* edit()
* delete()

###HashUtility
* latestHash()
* createHash()

###SyncUtility
* syncServerContact()

##Client Classes
###Contact
* getRevisionHash()
* create() 
* edit()
* delete()

###HashUtility
* latestHash()
* createHash()

###SyncUtility
* syncClientContact()

##Algorithm Discussion
For 2-way address book syncing, we can assume that a single person will be responsible for entering his own data. This ensures there is no conflict between two different data. Only the latest data entered/edited by the user is the correct data. All previous data is stale.

###Server Methods:
__AddressBook.setContact()__ enters new Contact instance for given User into AddressBook after checking _lateshHash()_ and then calls _syncServerContact()_ on all devices based on _Device.getRevisionHash()_.

__AddressBook.getContact()__ returns Contact stored in AddressBook for given User.

__AddressBook.removeContact()__ deletes Contact stored in AddressBook for given User and calls _syncServerContact()_ to delete the contacts on all devices. 

__User.getUserContact()__ calls _AddressBook.getContact()_ with the User id.

__User.setUserContact()__ calls _AddressBook.setContact()_ with User id.

__User.getDeviceList()__ returns list of devices registered with current User.

__User.setDeviceList()__ registers new device for given User and calls _syncServerContact()_ to that device after checking _Device.getRevisionHash()_.

__Device.getRevisionHash()__ returns the revisionHash a device is using based on latest records.

__Device.setRevisionHash()__ updates revisionHash of device when _syncServerContact()_ returns success and when a device pushes Contact instance to server. 

__Contact.getRevisionHash()__ returns revisionHash of Contact instance.  

__Contact.create()__ creates a new Contact instance and sets Contact revisionHash and calls _User.setUserContact()_.

__Contact.edit()__ changes data in the instance and updates revisionHash and calls _User.setUserContact()_.

__Contact.delete()__ delete Contact instance and calls _User.removeUserContact()_.

__HashUtility.latestHash()__ compares two Hash values and returns the one with latest timestamp.

__HashUtility.createHash()__ create Hash value for a Contact.

__SyncUtility.syncServerContact()__ pushes Contact to the given device id. If push returns success, it updates the _Device.setRevisionHash()_ to show the revision of the Contact that was pushed to the device. If push returns connection success, but _lateshHash()_ failure, it ends else if it returns connection failure, it sets a event dispatcher to _syncServerContact()_ on device connection.

###Client Methods:
__SyncUtility.syncClientContact()__ pushes Contact to server. In server, on _latestHash()_ if it is found that this is latest revision, server updates it's own records and pushes _syncServerContact()_ to all remaining devices.
