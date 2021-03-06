# The issue

Patient apps (eg Apple Health) get data from FHIR end points. Periodically they ask for new / changed data - 
relying on Meta.lastUpdated or searching by _lastUpdated.

If the patient record within the provider is the result of a merge and that gets undone - or if a new merge happen - 
the data will change in a way that the system above would not detect: a bunch of old data points disappear (split) or appear (merge). 
The data in the app will be out of sync - and in the case of a split in a very bad way: data of what is now known to the provider to be 
a _different_ patient is still in the personal data of someone else inside the app.

# Possible solution

Servers implement an additional `$isDataValid` operation:

- applieas to a Patient instance

- has one single required parameter `since` of type Instant

- returns a single bool: `true` is the patient data is still OK, `false` if an event that invalidated the data (merge, split, 
or some other drastic change) happend after the specified date-time

Every time apps fetch new patient data they first call `.../Patient/id/$isDataValid?since=xxxx` - where `xxxx` is the date-time of 
the last successfull data fetch for that patient - and if the server response is `false` the app removes all the data it previously 
fetched and start from scratch.

