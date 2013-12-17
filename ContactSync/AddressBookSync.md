###Problem Statement
You work for a company that maintains user's address book on their server like Gmail-contacts. As an engineer you have to come up with a technical design/proposal to support two-way address-book syncing with user's personal devices such as android phone, iphone, etc. Ignore authentication, address-book-entry-schema, etc. for this discussion. Keep in mind contacts can be edited both on server and on device by the user.
###Algorithm Discussion

####Classes Used

#####User Class
* getUserContact()
* setUserContact()
* getDeviceList()
* setDeviceList()

#####Contact Class
* getRevisionHash()
* create() 
* edit()
* delete()

#####AddressBook Class
* sync()
* copy()
* remove()

#####Hash Utility Class
* latestHash()
* createHash()

For 2-way address book syncing, we can assume that a single person will be responsible for entering his own data. This ensures there is no conflict between two different data. Only the latest data entered/edited by the user is the correct data. All previous data is stale.

Implementation:
1. getUserContact
returns latestHash() from all the available hashes of contacts available for the given user id.
2. setUserContact
called when new user is created and it in turn creates instance of Contact and calls create().
1. create
When a user enters his contact details for the first time, an instance of Contact Class is created and create() is called to make the user enter his initial data.
The AddressBook Class stores mapping of Contact Class with User Class(Many-One). When create() is called, an instance of Contact is created.
The data is stored with a unique revision number created by hashing.
To differentiate between two hashes of a particular User, we may use current timestamp as the hash, createHash().
Create function before return, calls the sync() function which does the two-way syncing.

2. edit
edit() contact edits the user data and once entered, it syncs the data using sync() method.

3. delete
delete() method removes the contact instance from the records and then calls sync() to remove it from all other records.

4. getRevisionHash
returns the hash of that instance of Contact class.

4. sync()
create() or edit() function calls sync().
sync() function is responsible for the 2-way syncing.
AddressBook Class has a  Many-One Contact-User record.
Case 1: Device
When sync() is called on any device, it pushes the latest Contact instance to server.
At server, the already existing latest Contact is compared with the pushed Contact using the latestHash() method.
If the pushed Contact is latest, server data is updated and server calls sync for all the remaining devices.
else process ends here.
Case 2: Server
When sync() is called on the server, it pushes the latest Contact instance to all the devices listed for sync.
On each device, it checks if the hash pushed is latest, if yes, this process continues till all the devices have been pushed with this latest Contact.
If in any device while checking the latestHash() it's found that the device has a latestHash(), the process stops and instead sync() from that device is called to the server.

