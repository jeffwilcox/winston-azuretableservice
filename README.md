# winston-azuretableservice

Windows Azure Table Service Transport for Winston (Node.js)

The `winston-azuretableservice` module for Node.js + Winston apps lets you write log messages to the cloud with Windows Azure Table Service as the provider. The table service is a cost-effective logging area as the costs are the same as blob storage.

This library is in 0.0.0 - I am just using in my own production code but nothing very official, so it is not yet on `npm`.

### Injection
``` js
var winston = require('winston');

// exposes the transport winston.transports.AzureTableService
require('winston-azuretableservice').AzureTableService; 

// you need to set credentials in options or use the standard 
// Azure environment variables with the storage key info
var options = { /* see docs */ };

winston.add(winston.transports.AzureTableService, options);
```

### Row keys

The RowKey used for entities needs to be unique to prevent runtime errors and make sure that all of your intended log messages are saved in the cloud. Though the library will emit the `error` event when a problem pops up, it will not try to dynamically select alternative row keys or anything like that.

Researching the various winston transports others typically use, there's a lot of personal preference when it comes to selecting a row key that will work depending on the scale of the service. Many people use UUIDs and at this time my implementation uses a timestamp + UUID formula that should insert logs in a way to maintain sorting on the table service side without having to use expensive sort methods when querying the data.

A default (and yes a little too lengthy) method of `timestamp-uuid` is used. To change, pass in a `rowKey` value to the `options` when intializing. You can also supply your own function.

Other supported `rowKey` values:

* uuid
* epoch
* timestamp
* supply your own function

### Let's talk about partition keys

The [Windows Azure Table Service][2] is designed to handle huge amounts of data and is super affordable, priced equivalent to blog storage (though you'll want to remember the storage transaction pricing which I believe is $0.01/US/100k REST requests if you have a super busy site). [Pricing details here][3].

Keep in mind that for long-lived services, this value may be dynamic and change over time if you want to use a date or time-based partition selection mechanism.

### All supported options

When initializing `winston.transports.AzureTableService`, consider these options:

* __level:__ Minimum level of log severity that the transport should log (defaults to `info`).
* __account:__ The name of the Azure storage account to use. Alternatively set the AZURE_STORAGE_ACCOUNT environment variable. Note that connection strings are currently not supported.
* __accessKey:__ The shared storage access key. Alternatively set the AZURE_STORAGE_ACCESS_KEY environment variable.
* __tableService:__ Advanced functionality, as an alternative to `account` and `accessKey` you can pass in an instance of an `azure` npm module `TableService`. This allows development storage use, connection strings, or other advanced scenarios where you inject your own table service instance.
* __table:__ The name of the table to use. It does not have to exist. The default table name is `logs`
* __createTableIfNotExists:__ Optional bool parameter indicating whether to check for and create the table the first time logging happens. Defaults to `true`
* __columns:__ `true` by default, any metadata logged with winston will be stored in unique column names. False will store a JSON string with the metadata in a column named `meta`
* __partition:__ `logs` by default, a function or string name. This can support but not scale a large amount of data. A better partition implementation would be to select and store in date buckets or year+date buckets, useful for high availability sites where querying may not be in real-time but over time a large amount of data will be stored. The `winston-azuretableservice` module exports a function `DatePartitionKeyFunction` (TBD: not yet implemented) that provides this functionality.
* __rowKey:__ `timestamp-uuid` by default, a function or string value to select the kind of RowKey selection method used. See the Row keys section above.

Note that the behavior of the logger to create the table by default is different than that of `winston-skywriter`; that default combined with my interest in just building on top of the vanilla `azure` npm module were my goals for this project.

#### What about the storage emulator?

You can pass your own `tableService` instance to this logger but at this time the default use case is live Windows Azure, and so the storage key and account name are the supported mechanism for access - not development/emulator storage or fancy connection strings.

#### Alternative and inspiration

Inspired during a port of some AWS projects to Azure where I had been using `winston-simpledb` as the transport for simple logging.

Also consider checking out the much cooler-named project [winston-skywriter][0] also which is built on top of the same `azure` npm module but also [bluesky][1] - an interop layer.

[0]: https://github.com/pofallon/winston-skywriter/
[1]: https://github.com/pofallon/node-bluesky
[2]: http://msdn.microsoft.com/en-us/library/dd179463.aspx
[3]: http://www.windowsazure.com/en-us/pricing/details/#header-4
