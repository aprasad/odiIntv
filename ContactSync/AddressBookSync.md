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
AddressBook.setContact() enters new Contact instance for given User into AddressBook after checking lateshHash() and then calls syncServerContact() on all devices based on Device.getRevisionHash().

AddressBook.getContact() returns Contact stored in AddressBook for given User.

AddressBook.removeContact()deletes Contact stored in AddressBook for given User and calls syncServerContact to delete the contacts on all devices. 

User.getUserContact() calls AddressBook.getContact with the User id.

User.setUserContact() calls AddressBook.setContact() with User id.

User.getDeviceList() returns list of devices registered with current User.

User.setDeviceList() registers new device for given User and calls syncServerContact() to that device after checking Device.getRevisionHash().

Device.getRevisionHash() returns the revisionHash a device is using based on latest records.

Device.setRevisionHash() updates revisionHash of device when syncServerContact returns success and when a device pushes 

Contact.getRevisionHash() returns revisionHash of Contact instance.  

Contact.create() creates a new Contact instance and sets Contact revisionHash and calls User.setUserContact().

Contact.edit() changes data in the instance and updates revisionHash and calls User.setUserContact().

Contact.delete() delete Contact instance and calls User.removeUserContact().

HashUtility.latestHash() compares two Hash values and returns the one with latest timestamp.

HashUtility.createHash() create Hash value for a Contact.

SyncUtility.syncServerContact() pushes Contact to the given device id. If push returns success, it updates the 

Device.setRevisionHash() to show the revision of the Contact that was pushed to the device. If push returns connection success, but lateshHash() failure, it ends else if it returns connection failure, it sets a event dispatcher to syncServerContact() on device connection.

###Client Methods:
SyncUtility.syncClientContact() pushes Contact to server. In server, on latestHash() if it is found that this is latest revision, server updates it's own records and pushes syncServerContact to all remaining devices.
