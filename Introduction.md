

# Introduction to the Google APIs Client Library for Objective-C #

Google APIs allow client software to access and manipulate data hosted by Google services.

The Google APIs Client Library for Objective-C is a Cocoa framework that enables developers for Mac OS X and iOS to easily write native applications using Google's JSON-RPC APIs.  The framework handles

  * JSON parsing and generation
  * Networking
  * Sign-in for Google accounts
  * Service-specific protocols and query generation


## Preparing to Use the Library ##

### Requirements ###

The Google APIs Client Library for Objective-C requires iOS 3 or later, or Mac OS X 10.5 or later.

To download the project via Subversion, install Xcode command-line tools so the `svn` command is available. In Xcode, choose `Xcode > Preferences > Downloads`, and click the Install button for Command Line Tools

### Example Applications ###

The Examples directory contains example applications showing typical interactions with Google services using the framework.  The applications act as simple browsers for the data classes for each service.  The WindowController source files of the samples were written with typical Cocoa idioms to serve as quick introductions to use of the APIs.

The example applications run on Mac OS X, but the library does not provide any user interface support apart from authentication, so use of the library APIs is the same for Mac and iOS applications.

In order to use the example applications that require authentication, you must create **client ID** and **client secret** strings for an **installed application** (not a web application) type **other** (not iOS) using the [API Console](https://console.developers.google.com/), as explained in the [OAuth 2 controller documentation](http://code.google.com/p/gtm-oauth2/wiki/Introduction).

Google APIs which do not require authentication may require an **API key**, also obtained from the [API Console](https://console.developers.google.com/).

### Adding the Library to a Project ###

BuildingTheLibrary explains how to add the library to a Mac or iPhone application project.

### Subsidiary Libraries ###

The Google APIs Library for Objective-C uses the prefix `GTL`.

The library incorporates other Google libraries with the `GTM` prefix. Those provide http handling ([gtm-http-fetcher](http://code.google.com/p/gtm-http-fetcher/)) and OAuth 2 authentication ([gtm-oauth2](http://code.google.com/p/gtm-oauth2/)).  For systems earlier than iOS 5 and Mac OS X 10.7, the library uses the [SBJSON](http://code.google.com/p/json-framework/) library for JSON parsing and generation.

All external libraries needed are checked out automatically with the main API library using Subversion's externals facility.

### Authentication and Authorization ###

**Authentication** is the confirmation of a user's identity using her username, password, and possibly other data, such as captcha answers or 2-step codes provided via mobile phone. Authentication is required for access to non-public data.

Google APIs rely on OAuth 2 for user sign-in. The library includes the [GTM OAuth 2 controllers](http://code.google.com/p/gtm-oauth2/) for iOS and Mac OS X to handle the sign-in sequence. A `GTMOAuth2Authentication` object encapsulates the resulting access tokens.

**Authorization** is the use of access tokens to allow specific requests. Applications pass the OAuth 2 authorization object to a service class's `setAuthorizer:` method to authorize queries.

Be sure to read the introduction wiki page for [gtm-oauth2](http://code.google.com/p/gtm-oauth2/), as adding authentication support to your project is likely to require more effort than is using the API client library.

**Note:** The [GTM OAuth 2 controllers](http://code.google.com/p/gtm-oauth2/) support signing in from within a Mac or iOS app. Alternatively, iOS apps may offer sign-in to Google services using a [Google+ Platform sign-in](https://developers.google.com/+/mobile/ios/) button. Google+ Platform sign-in brings Safari to the foreground while the user is signing in.


### API Quotas ###

For each API used by your application, enable the API and check its default **quota** in the Services section of the [API Console](https://console.developers.google.com/).

Applications which make frequent API requests or have many users may need more than the default limit. The specific API documentation or the service's control in the API Console will explain how to request a larger quota.

**Note:** Be sure to estimate your application's total queries per day for all users and request an appropriate quota _before_ releasing your application to a large audience. An inadequate quota may lead to errors on API requests after your application ships.

## Basics ##

### Objects and Queries ###

Servers respond to client API requests with an object.  The object is typically either an individual item, or a collection with an `items` property that accesses an `NSArray` of items.

For example, a search for products would return a `GTLShoppingProducts` collection, where the items of the collection are the individual product results found by the search.

The individual items are derived from **GTLObject**. The collection is derived from **GTLCollectionObject**, which is a `GTLObject` that also provides for indexed access to items via subscripts and an `itemAtIndex:` method. `GTLCollectionObject` also supports `for` loops over the collection items by implementing the `NSFastEnumeration` protocol, as shown in the next code snippet.

Each request to the server is a **query**. Server interactions in the library are handled by **service objects**.  A single transaction with a service is tracked with a **service ticket**.

For example, here is how to use the API library to execute a search for hiking boots.
```
// Create a service object for executing queries
GTLServiceGoogleShopping *service = [[GTLServiceGoogleShopping alloc] init];

// Services which do not require sign-in may need an API key from the
// API Console
service.APIKey = @"AIzaSyDpzrGImFGQPOw";

// Create a query
GTLQueryShopping *query = [GTLQueryShopping queryForProductsListWithSource:@"public"];
query.q = @"hiking boots";

// Execute the query
GTLServiceTicket *ticket = [service executeQuery:query
                               completionHandler:^(GTLServiceTicket *ticket, id object, NSError *error) {
  // This callback block is run when the fetch completes
  if (error == nil) {
    GTLShoppingProducts *products = object;

    // GTLShoppingProducts derives from GTLCollectionObject, so it supports
    // iteration of items and subscript access to items.
    for (GTLShoppingProduct *item in products) {
      // Print the name of each product result item
      NSLog(@"%@", item.product.title);
    }
  }
};
```

Unseen by the application, the server is returning a JSON tree in response to queries.  Each `GTLObject`, such as `GTLShoppingProducts` in the example above, is a wrapper for a tree of JSON data. The `GTLObject` allows the JSON data to be treated like a first-class Objective-C object, using normal Objective-C property notation. This use of properties is visible in the code snippet above, where each product item result name is accessible as `item.product.title`.

`GTLObject`'s support for Objective-C properties allows compile-time syntax checking and enables Xcode's autocompletion for each object.  The header files for each object class clearly define the fields of each object. For example, the shopping products object interface looks in part like this:

```
@interface GTLShoppingProducts : GTLCollectionObject
@property (retain) NSString *identifier;
@property (retain) NSArray *items;  // of GTLShoppingProduct
@property (retain) NSNumber *currentItemCount;  // intValue
@property (retain) GTLShoppingProductsSpelling *spelling;
...
@end
```

Each object property returns either a standard Objective-C type (NSString, NSNumber, NSArray), other `GTLObjects`, or `GTLDateTime`.

When your application gets a property from a `GTLObject`, the library converts the property name to the JSON key string to get or set the result in the underlying JSON tree.  Subtrees of the JSON are returned wrapped in a new `GTLObject`s. To reduce memory overhead, the `GTLObjects` are not created for inner trees of the JSON until they are needed by the application.

Normally, applications will use `GTLObject` properties and so will not need to access the plain JSON tree, but it is available for each `GTLObject` as a dictionary with the property `JSON`.

For compatibility with iOS 3 and Mac OS X 10.5, queries may also be executed with a delegate and selector for the callback:
```
GTLServiceTicket *ticket = [service executeQuery:query
                                        delegate:self
                                finishedSelector:@selector(serviceTicket:finishedWithObject:error:)];
```
The delegate is retained until the query has completed, or until the ticket has been canceled.

### Services and Tickets ###

Service objects maintain cookies and track other persistent data across queries. Typically, an application will create a single instance of a service object to use for executing all queries.

The service object makes a copy of each query for execution, so changes to a query object made after calling `executeQuery:` will not affect the request.

Query execution by the service is inherently asynchronous. There is no need to create an operation queue or to use GCD to run queries on other threads. Generally, it's best to execute queries on the main application thread. Any number of queries may be executed either concurrently or sequentially, subject to rate limits shown in the [API Console](https://console.developers.google.com/). Additional information about threading support is listed below in the section on Performance Optimizations.

On iOS, query fetches will normally be stopped once the user puts the application into the background. Fetches and callbacks can be continued in the background, up to the operating system time limit, by setting the property `shouldFetchInBackground`:
```
// Enable fetches to continue in the background on iOS
service.shouldFetchInBackground = YES;
```

A new ticket is created each time a query is executed by the service. When a ticket is created, many of the ticket properties, such as retry settings and surrogates (both described below), are initialized from the service's properties.

The application may choose to retain the ticket after a query starts executing, allowing the user to cancel the service request.

Once either a query's callback has been invoked or the ticket is canceled, the ticket is no longer useful and may be released. To cancel a query in progress, call `[ticket cancelTicket]`.

The query being executed by a ticket is available as `ticket.executingQuery`. The `GTMHTTPFetcher` object used to execute the query is accessible as `ticket.objectFetcher`.

### Result Pages ###

A query may return an object with only a subset of the results in the items array. The object is considered one of several **pages**. When a result object includes a **nextPageToken** string, then the query can be executed again with the token provided as the **pageToken** property of the new query, fetching the next set of results.
```
if (object.nextPageToken) {
  // Manually make a query to fetch the next page of results by reusing the previous query
  GTLQueryTasks *nextQuery = previousQuery;
  nextQuery.pageToken = object.nextPageToken;
  GTLServiceTicket *nextTicket = [service executeQuery:nextQuery ...
}
```

Some APIs instead support a **nextStartIndex** in result objects, and a **startIndex** property for queries, but using those is similar.

For APIs that provide a `nextPageToken` or `nextStartIndex` property, the library can automatically fetch all pages, and return an object whose items array includes the items of all pages (up to 25 pages). This can be turned on by setting the `shouldFetchNextPages` property of a service or a ticket:
```
// Turn on automatic page fetches
service.shouldFetchNextPages = YES;
```

Note, however, that results spread over many pages may take a long time to be retrieved, as each page fetch will lead to a new http request.  The server can be told to use a larger page size (that is, more items in each page returned) by fetching a query for the feed with a `maxResults` value:

```
// Specify a large page size to reduce the need to fetch additional result pages
GTLQueryTasks *query = [GTLQueryTasks queryForTasklistsList];
query.maxResults = 1000;
ticket = [service executeQuery:query ...
```

Ideally, maxResults will be large enough that, for typical user data, all results will be returned in a single page.

For queries that search public data, with potentially a very large number of items resulting, `shouldFetchNextPages` should not be enabled for the service or the ticket. Additional pages of large data sets can be fetched manually.

### Query Operations ###

Query objects typically implement these basic operations:
  * **list** - fetch a list of items
  * **insert** - add an item to a list
  * **update** - replace an entire item
  * **patch** - replace fields of an item
  * **delete** - remove an item

There may be custom query operations as well.  To find the Objective-C query class for a documented API operation, search for the **method name** shown in the JSON-RPC documentation for the API.  It is listed before each method name in the query class interface, as shown here for the method named "shopping.products.list":
```
// Method: shopping.products.list
// Returns a list of products and content modules
// Fetches a GTLShoppingProducts.
+ (id)queryForProductsListWithSource:(NSString *)source;
```
The comments for each method also specify the type of object passed to the callback if the query succeeds. In this example, the callback object class is `GTLShoppingProducts`.

### Creating GTLObjects from Scratch ###

Typically, `GTLObject`s are created by the library from JSON returned from a server, but occasionally it is useful to create one from scratch, such as when inserting a new item or patching an existing item.  The `+object` method creates an empty instance of a `GTLObject`.

This snippet shows how to create a new task list for the user's Google Tasks account:
```
- (void)changeATaskListTitle:(NSString *)title {
  GTLTasksTaskList *tasklist = [GTLTasksTaskList object];
  tasklist.title = title;

  GTLQueryTasks *query = [GTLQueryTasks queryForTasklistsInsertWithObject:tasklist];

  [service executeQuery:query
      completionHandler:^(GTLServiceTicket *ticket, id item, NSError *error) {
        // Callback
        if (error == nil) {
          // Succeeded
          GTLTasksTaskList *tasklist = item;
        }
      }];
}
```

### Uploading Files ###

Queries that can upload files will take a `GTLUploadParameters` object. The upload parameters object requests a MIME type describing the file data, and either an `NSData` with the file's contents, or an `NSFileHandle` for reading from the file.

Typically, the query will also take an object with additional metadata for the upload file, such as the file's title.

```
NSFileHandle *fileHandle = [NSFileHandle fileHandleForReadingAtPath:path];
if (fileHandle) {
  NSString *mimeType = @"image/jpeg";
  GTLUploadParameters *uploadParameters = [GTLUploadParameters uploadParametersWithFileHandle:fileHandle
                                                                                     MIMEType:mimeType];
  GTLDriveFile *fileObj = [GTLDriveFile object];
  fileObj.title = [path lastPathComponent];

  GTLQueryDrive *query = [GTLQueryDrive queryForFilesInsertWithObject:fileObj
                                                     uploadParameters:uploadParameters];

  GTLServiceTicket *ticket = [service executeQuery:query ... 
```

The library uses Google's resumable (chunked) upload protocol for transferring the file to the server.

#### Progress Monitoring ####

The application can supply a block to be called for displaying progress during uploads.
```
ticket.uploadProgressBlock = ^(GTLServiceTicket *ticket,
                               unsigned long long numberOfBytesRead,
                               unsigned long long dataLength) {
      [progressIndicator setMaxValue:(double)dataLength];
      [progressIndicator setDoubleValue:(double)numberOfBytesRead];
};
```

#### Pause and Resume ####

Uploads in progress can be paused and resumed.

```
if ([ticket isUploadPaused]) {
  [ticket resumeUpload];
} else {
  [ticket pauseUpload];
}
```

Uploads can be cancelled with the ticket's `cancelTicket` method.

### Downloading Files ###

The library includes `GTMHTTPFetcher`, a class that supports convenient http downloading and uploading.

Download of any individual user’s data from Google services requires that the request be authorized.

```
// Fetchers can be created from an NSURLRequest, an NSURL, or an NSString.
NSString *urlString = [NSURL URLWithString:@"http://example.com/file.bin"];
GTMHTTPFetcher *fetcher = [GTMHTTPFetcher fetcherWithURLString:urlString];

// For downloads requiring authorization, set the authorizer.
fetcher.authorizer = self.authorizer;

// The fetcher can optionally download directly to a file. When downloading to a file, 
fetcher.downloadPath = savePath;

// Applications may also specify a receivedDataBlock to be notified as data is received.

// Fetcher http logs can include useful comments.
[fetcher setCommentWithFormat:@"Downloading \"%@\"", [urlString lastPathComponent]];

[fetcher beginFetchWithCompletionHandler:^(NSData *data, NSError *error) {
  // Callback
  if (error == nil) {
    // Successfully received the file data.
    //
    // Since a downloadPath property was specified, the data argument is
    // nil, and the file data has been written to disk.
  } else {
    // Error downloading or saving file.
  }
}];
```

## Performance and Memory Optimizations ##
### Partial Responses ###

Applications can avoid fetching unneeded data by setting a query's `fields` property. Google APIs support a syntax similar to XPath for selecting fields to be included in the response.

For example, to request only the IDs and author email addresses for each item in the collection, use this request:
```
query.fields = @"items(id,author/email,kind),kind,nextPageToken";
```

The library may use the `kind` fields of responses to choose the proper object classes, so requests for partial responses should _always_ include the `kind` fields for both items and collections, as shown above. Otherwise, the library may be forced to incorrectly instantiate the `GTLObject` base class for the response.

Queries for collections should also accept the `nextPageToken` or `nextStartIndex` field, as one of those is returned when an additional result page should be fetched. The service API documentation should specify which type of paging field is returned with collection responses.

Refer to the [documentation](http://code.google.com/apis/calendar/v3/performance.html#partial) for details on the fields parameter.

`GTLObject` provides a `fieldsDescription` method which returns a string describing all of the fields in an object.  The string may be useful as a starting point when making a query to request specific fields.

### Partial Updates ###

An update query will replace an entire item of a collection. Typically, it is more useful to replace only one or a few fields of an item. A patch query accomplishes that: it replaces only the fields specified by the patch object.

Here is how to use a partial update to change only the title of a task list:
```
GTLTasksTaskList *patchObject = [GTLTasksTaskList object];
patchObject.title = title;

GTLQueryTasks *query = [GTLQueryTasks queryForTasklistsPatchWithObject:patchObject
                                                              tasklist:taskList.identifier];
GTLServiceTicket *ticket = [service executeQuery:query ...
```

An explicit **null value** indicates that a field should be deleted. For example, removing just the notes from a task looks like this:

```
GTLTasksTask *patchObject = [GTLTasksTask object];
patchObject.notes = [GTLObject nullValue];

GTLQueryTasks *query = [GTLQueryTasks queryForTasksPatchWithObject:patchObject
                                                          tasklist:tasklist.identifier
                                                              task:task.identifier];
GTLServiceTicket *ticket = [service executeQuery:query ...
```

An array in a patch object field always replaces the entire array for the object on the server, as described in the [documentation](http://code.google.com/apis/calendar/v3/performance.html#patch) for partial updates.

`GTLObject` includes a helpful method `patchObjectFromOriginal:` for making a patch object that contains just the changes from a previous version of that object.

### Batch Operations ###

Several unrelated queries may be executed together in a **batch**. Batch execution is faster than is executing queries individually.

There are two ways to obtain the results of batch queries: the unified completion handler callback (or delegate and selector) for the batch query execution, with results for all queries, and individual completion blocks for each query.

Typically, it is easier to use individual query completion blocks for unrelated queries, and the unified completion handler for batches of related queries.

Both types of callbacks may be used; the individual query completion blocks are called before the completion handler for the batch query execution. Individual query completion blocks are optional, and may be omitted.

#### Individual Query Completion Blocks ####

A completion block can be specified for each query, as shown here:
```
GTLQueryPlus *query = [GTLQueryPlus queryForPeopleGetWithUserId:@"me"];
query.completionBlock = ^(GTLServiceTicket *ticket, id object, NSError *error) {
  if (error == nil) {
    // This query succeeded
  } else {
    // This query failed
  }
};

GTLBatchQuery *batchQuery = [GTLBatchQuery batchQuery];
[batchQuery addQuery:query];
...
```

Errors passed to the query's completion block will have an underlying `GTLErrorObject` when execution succeeded but the server returned an error for this specific query:
```
GTLErrorObject *errorObj = [GTLErrorObject underlyingObjectForError:error];
if (errorObj) {
    // The server returned this error for this specific query
  } else {
    // This error occurred because the batch execution failed
}
```

#### Unified Batch Completion Handler ####

The unified result of executing a batch query is a `GTLBatchResult` object with two dictionaries, one for the results of successful queries, and one for the error objects returned by unsuccessful queries.

Note that in addition to the dictionary of error results, there is also an `NSError` passed to the completion handler which indicates if the execution did not succeed.

```
GTLBatchQuery *batchQuery = [GTLBatchQuery batchQuery];
[batchQuery addQuery:query1];
[batchQuery addQuery:query2];

[service executeQuery:batchQuery
    completionHandler:^(GTLServiceTicket *ticket, id object, NSError *error) {
    if (error == nil) {
      // Execute succeeded: step through the query successes
      // and failures in the result
      GTLBatchResult *batchResult = object;

      NSDictionary *successes = batchResult.successes;
      for (NSString *requestID in successes) {
        GTLObject *result = [successes objectForKey:requestID];
      }

      NSDictionary *failures = batchResults.failures;
      for (NSString *requestID in failures) {
        GTLErrorObject *errorObj = [failures objectForKey:requestID];
      }
    } else {
      // Here, error is non-nil so execute failed: no success or failure
      // results were obtained from the server
    }
  }];
```

Each query object is created with a unique `requestID`, though your application may set a custom `requestID` string for the query prior to execution. The `requestID` string must be non-empty, and all queries in a batch must have unique `requestID`s.

### Threading ###

Typically, applications will execute queries on the main thread, and the query completion handler will be called back on the main thread.

Alternatively, an application can execute a query on a background thread and will be called back on that same background thread, though the application must run an `NSRunLoop` on that thread for the callbacks to occur.

Beginning with iOS 6 and Mac OS X 10.7, a query may be executed on either the main thread or a background thread, and be called back on either the main thread or a background thread.  To have callbacks occur on a different thread than the query was executed on, specify an `NSOperationQueue` (not a dispatch queue) for the service object's `delegateQueue` property.

For example, queries made from any thread can be called back on the main thread by using the main queue:
```
service.delegateQueue = [NSOperationQueue mainQueue];
```
Queries made from any thread can be called back on a background thread by providing a background queue, as in this example:
```
service.delegateQueue = [[NSOperationQueue alloc] init];
```
When a delegate queue is specified, there is no requirement for a run loop to be running on the thread that executes the query.

## Convenience Features ##

### Saving Objects (Serialization) ###

A `GTLObject` has an underlying NSDictionary accessible through its `JSON` property. The dictionary may be saved as a property list:
```
// Saving to disk
NSMutableDictionary *dict = tasklist.JSON;
BOOL didSave = [dict writeToFile:path atomically:YES];
```
An object's JSON can also be expressed as a string with the `JSONString` method.

The object may later be recreated with the class method `+objectWithJSON:` The JSON dictionary should be constructed with mutable containers:
```
// Reading from disk
NSData *data = [NSData dataWithContentsOfFile:path
                                      options:0
                                        error:&error];
if (data) {
  NSMutableDictionary *dict = [NSPropertyListSerialization propertyListWithData:data
                                                                        options:NSPropertyListMutableContainers
                                                                         format:NULL
                                                                          error:&error];
  if (dict) {
    GTLTasksTaskList *tasklist = [GTLTasksTaskList objectWithJSON:dict];
  }
}
```

### Adding Custom Data ###

Often it is useful to add data locally to a `GTLObject`. For example, an entry used to represent a photo being uploaded would be more convenient if it also carried a path to the photo's file on the local disk.

Your application can add data to any instance of a `GTLObject` in three ways. These three techniques only add data to objects _locally_ for the Objective-C code; the data will not be retained on the server.

#### UserData ####

Each `GTLObject` has a `userData` property to set and retrieve a single NSObject. Adding a local path string to a photo entry `userData` would look like this:

```
GTLPhoto *newPhoto = [GTLPhoto object];
newPhoto.userData = localPathString;
```

#### User Properties ####

An application can set and retrieve multiple objects as named properties of any `GTLObject` instance with the methods `setProperty:forKey:` and `propertyForKey:`. Unlike `userData`, this is useful for attaching multiple properties.

```
GTLPhoto *newPhoto = [GTLPhoto object];
[newPhoto setProperty:localPathString forKey:@"myPath"];
[newPhoto setProperty:thumbnailImage forKey:@"myThumbnail"];
```

Property keys beginning with an underscore are reserved by the library and should not be used by applications.

#### Subclassing GTLObjects ####

Finally, applications may subclass `GTLObject`s to add ivars and methods.  To have your subclasses be instantiated in place of the standard object class during the parsing of JSON as part of query execution, set the `surrogates` property of the service:

```
GTLServicePhotos *service = [[GTLServicePhotos alloc] init];

service.surrogates = @{ 
  [GTLPhoto class] : [MyPhoto class],
  [GTLAlbum class] : [MyAlbum class]
};
```
### Passing Data to Query Callbacks ###

It is often useful to pass an NSObject to a callback, particularly when using selector callbacks rather than blocks.

To retain an object from query execution until its callback is invoked, use `GTLServiceTicket`'s `setProperty:forKey:` methods.  For example:

```
GTLServiceTicket *ticket = [service executeQuery:...];
[ticket setProperty:callbackData forKey:@"myCallbackData"];
```

The callback can then access the data:
```
- (void)ticket:(GTLServiceTicket *)ticket finishedWithObject:(GTLObject *)object error:(NSError *)error {

  id myCallbackData = [ticket propertyForKey:@"myCallbackData"];
  ...
}
```

`GTLServiceTicket` also supports a `userData` property, which is a single object to be retained, without an explicit key.

### Automatic Retry ###

GTL service classes and the `GTMHTTPFetcher` class provide a mechanism for automatic retry of a few common network and server errors, with appropriate increasing delays between each attempt.  You can turn on the automatic retry support for a GTL service by setting the `retryEnabled` property.

```
// Turn on automatic retry of some common error results
service.retryEnabled = YES;
```

The default errors retried are http status 408 (request timeout), 503 (service unavailable), and 504 (gateway timeout), `NSURLErrorNetworkConnectionLost`, and `NSURLErrorTimedOut`.  You may specify a maximum retry interval other than the default of 1 minute, and can provide an optional retry selector to customize the criteria for each retry attempt.

### Testing ###

For unit tests, queries can be executed without any network activity. When testing, set the `testBlock` property of a query or service object. The `testBlock` should call its response parameter with an object or error.

```
  query.testBlock = ^(GTLServiceTicket *ticket, GTLQueryTestResponse testResponse) {
    // The query is available from the ticket.
    GTLQuery *testQuery = ticket.originalQuery;

    // The testBlock can create a GTLObject or GTLBatchResult, or an NSError.
    //
    // Here, we will make a GTLObject as the test response.
    GTLTasksTask *task = [GTLTasksTask object];
    task.title = @"My Fake Task";

    GTLTasksTasks *tasks = [GTLTasksTasks object];
    tasks.items = @[ task ];

    NSError *testError = nil;

    testResponse(tasks, testError);
  };
```

When a service or query has a `testBlock`, that will be used instead of the normal network activity. Since the `executeQuery:` invocation is asynchronous, the unit test should use either `GTLService` `waitForTicket:timeout:fetchedObject:error:` or with `XCTestCase` `waitForExpectationsWithTimeout:` to wait for the completion handler to be called.

### Using APIs Without Generated Classes ###

The Google APIs Client Library for Objective-C includes generated service, query and data classes. These are Objective-C files constructed by processing the output of the [Google APIs Discovery Service](https://code.google.com/apis/discovery/). The generated classes are derived from `GTLService`, `GTLQuery`, and `GTLObject`.

The library may also be used for Google and non-Google APIs without generated classes. Methods for fetching objects via REST and RPC requests are listed in the "REST Fetch Methods" and "RPC Fetch Methods" sections of `GTLService.h`.

For use with RPC queries, instances of `GTLService` should specify the request URL and API version, such as
```
GTLService *service = [[GTLService alloc] init];
service.rpcURL = [NSURL URLWithstring:@"https://www.example.com/rpc"];
service.apiVersion = @"v1";
```

## Logging HTTP Server Traffic ##

Debugging query execution is often easier when you can browse the JSON and headers being sent back and forth over the network.  To make this convenient, the framework can save copies of the server traffic, including http headers, to files in a local directory.  Your application should call

```
[GTMHTTPFetcher setLoggingEnabled:YES]
```

to turn on logging.  The project building the fetcher class must also include `GTMHTTPFetcherLogging.h` and `.m`.

Normally, logs are written to the directory GTMHTTPDebugLogs in the logs directory. The logs directory is in the current user's Desktop folder for Mac applications. In the iPhone simulator, the default logs location is the user's home directory. On the iPhone device, the default location is the application's documents folder on the device.

To locate the folder with the Mac Finder, search for the file name GTMHTTPDebugLogs with the search option "System files are included".

The path to the logs folder can be specified with the `+setLoggingDirectory:` method.

To view the most recently saved logs, use a web browser to open the symlink named MyAppName\_log\_newest.html (for whatever your application's name is) in the logging directory.

For each executed query, when logging is enabled, the http log is also available as the property `ticket.objectFetcher.log`

**_iOS Note:_** Logging support is stripped out in non-DEBUG builds by default.  This can be overridden by explicitly setting STRIP\_GTM\_FETCH\_LOGGING=0 for the project.

**_Tip:_** Providing a convenient way for your users to enable logging is often helpful in diagnosing problems when using the API.

## Questions and Comments ##

**If you have any questions or comments** about the library or this documentation, please join the [discussion group](http://groups.google.com/group/google-api-objectivec-client).