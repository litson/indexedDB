indexedDB
=========

A wrapper for javascript's indexedDB

    Tested in: Chrome 21
               Firefox 14
               IE 10 / Windows 8 metro app js

Usage
=========

See full example at the bottom

##Init##

    var dbSchema = [{
        name: 'animals',
        key: {
            keyPath: 'id'
        },
        indexes: [{
            name: 'class',
            field: 'class',
            unique: false
        }]
    }];
    
    var db;
    
    var IDB = new DB({
        name: 'dbname',
        version: 1,
        schema: dbSchema,
        success: function (DBObject) {
            // DBObject is a wrapper object to make calls with
            // native IndexedDB object can be found at DBObject.db
            db = DBObject;
            // make calls with the db object here
        },
        error: function (supported, e) {
            if (supported) {
                console.log('indexedDB supported');
                console.log(e);
            } else {
                console.log('indexedDB not supported');
            }
        }
    });

##DBObject##

###Inserting###

    var data = [
        {id: 1, name: 'raccoon', class: 'mammalia'},
        {id: 2, name: 'octopus', class: 'cephalopoda'},
        {id: 3, name: 'ibex', class: 'mammalia'},
        {id: 4, name: 'ant', class: 'insecta'},
        {id: 5, name: 'whale', class: 'mammalia'},
        {id: 6, name: 'eagle', class: 'aves'},
        {id: 7, name: 'iguana', class: 'reptilia'},
        {id: 8, name: 'crab', class: 'malacostraca'}
    ];

    db.insert({
        store: 'animals',
        data: data,
        success: function (rowsAdded, rowsFailed, successEvent) { /* success */ },
        error: function (rowsAdded, rowsFailed, failEvent) { /* error */ }
    });

###Deleting###

####Delete by keypath (id in this example)####

Delete record with id = 7

    db.delete({
        store: 'animals', 
        id: 7,
        success: function (deletedID, successEvent) { /* deleted */ },
        error: function (failEvent) { /* error */ }
    });

####Delete all rows in object store####

    db.clear({
        store: 'animals',
        success: function (successEvent) { /* deleted all records */ },
        error: function (failEvent) { /* error */ }
    });

###Fetching###

####Get record by keypath (id in this example)####

Get record with id = 5

    db.get({
        store: 'animals', 
        id: 5,
        success: function (data, successEvent) { /* {id: 5, name: 'whale', class: 'mammalia'} */ },
        error: function (failEvent) { /* error */ }
    });

####Get all records in an object store####

    db.all({
        store: 'animals',
        success: function (data, successEvent) { /* Array all records in animals */ },
        error: function (failEvent) { /* error */ }
    });

####Filtering and ordering data####

Get all animals from class 'mammilia' in ascending order (column of ordering is name) limit 3

    db.filter({
        store: 'animals', 
        key: 'class', 
        value: 'mammilia', 
        orderCol: 'name', 
        order: 'asc', 
        limit: 3,
        success: function (data, successEvent) { /* first 3 animal with class mammilia in asc order by name */ },
        error: function (failEvent) { /* error */ }
    });

###Counting records###

OptionalKey will constrain result to items matching that key at value optionalKeyValue

    db.count({
        store: 'animals',
        success: function (count, successEvent) { /* total number of records */ },
        error: function (failEvent) { /* error */ }
    });


Documentation
=========

##Init##

###Options####

When initing the DB function it needs some options. It supports an option object with 4 parameters:

* name: `<string>` the name of your database
* version: `<int>` the version number, a new version number will trigger an upgrade (change schema)
* schema: `<object>` an object as described below
* debug: `<bool>` turn on off console logs
* success: `<function>` [see below for arguments]
* error: `<function>` [see below for arguments]


######success arguments######

* `<object>` DBObject with methods: insert, delete, count, ...etc
* `<event>` request success event

######fail arguments######

* `<bool>` whether indexedDB has support in the browser
* `<event>` request fail event



###Schema###

The schema takes an array of objects. Each object represents an objectStore.

Each objectStoreObject has 3 properties:

    name: <string> the name you will reference the objectStore by when requesting data
    
    key: <object> an object describing the key. Properties: 
                  
        keyPath: <string> the objectStore's key
        autoIncrement: <bool> if key auto increments (default false)
        
    indexes: <array> an array of objects that will be indexes in your objectStore. 
                     Each object has 3 properties:
        name: <string> the name of your index
        field: <string> the actual field name in your object 
        unique: <bool> whether it is a unique index (default false)

##DBObject##

On successful DB.init the success function is called with an argument of DBObject which is our custom object with several methods described below.

###Properties###

    db: <IDBObject> the actual object created by a successful `indexedDB.open().onsuccess` call

###Methods###

###insert (<object> options)###
Insert data into objectStore

######arguments######

* store: `<string>` objectStore name [required],
* data: `<array>` array of objects that you want to insert - not required, but pointless to leave out...
* replace: `<bool>` default false. true will replace any records with the same key that already exist [optional]
* success: `<function>` [see below for arguments]
* error: `<function>` [see below for arguments]

######success arguments######

* `<array>` array of successfully added items
* `<array>` array of failed items
* `<string>` the objectStore option passed
* `<event>` request success event

######fail arguments######

* `<array>` array of successfully added items
* `<array>` array of failed items
* `<string>` the objectStore option passed
* `<event>` request fail event

###get (<object> options)###
Get one record from objectStore by it's key

######arguments######

* store: `<string>` objectStore name [required],
* id: `<mixed>` the id* of the row you are trying to retrieve, (*id as set by keyPath in schema) [required]
* success: `<function>` [see below for arguments]
* error: `<function>` [see below for arguments]

######success arguments######

* `<object>` data you are asking for
* `<string>` the objectStore option passed
* `<event>` success event

#####fail arguments#####

* `<mixed>` the id you tried to get
* `<string>` the objectStore option passed
* `<event>` fail event
    
###all (<object> options)###
Get all records from objectStore

######arguments######

* store: `<string>` objectStore name [required]
* success: `<function>` [see below for arguments]
* error: `<function>` [see below for arguments]

######success arguments######

* `<array>` array of all records in the objectStore
* `<string>` the objectStore option passed
* `<event>` success event

######fail arguments######

* `<string>` the objectStore option passed
* `<event>` fail event

###delete (<object> options)###
Delete one record from objectStore

######arguments######

* store: `<string>` objectStore name [required],
* id: `<mixed>` the key you are trying to delete, can be `<int>` or `<string>` [required]
* success: `<function>` [see below for arguments]
* error: `<function>` [see below for arguments]

######success arguments######

* `<mixed>` the key you deleted `<int>` or `<string>`
* `<string>` the objectStore option passed
* `<event>` success event

######fail arguments######

* `<string>` the objectStore option passed
* `<event>` fail event

###filter (<object> options)###
Filter records in an objectStore by an index/value and optional ordering by arbitrary column/limiting

######arguments######

* store: `<string>` objectStore name [required],
* key: `<string>` index you want to filter by, must be set up in the schema [required],
* value: `<mixed>` value at that index you want to filer for [required],
* orderCol: `<string>` column to order by [optional] * both orderColumn and order must be set for sorting,
* order: `<enum ('asc'|'desc')>` defaults to 'asc' for ascending order [optional],
* limit: `<int>` number of rows to limit to [optional]
* success: `<function>` [see below for arguments]
* error: `<function>` [see below for arguments]

######success arguments######

* `<array>` array of data you are asking for
* `<string>` the objectStore option passed
* `<event>` success event

######fail arguments######

* `<string>` the objectStore option passed
* `<event>` fail event

###clear (<object> options)###
Delete all rows from an objectStore

######arguments######

* store: `<string>` objectStore name [required]
* success: `<function>` [see below for arguments]
* error: `<function>` [see below for arguments]

######success arguments######

* `<string>` the objectStore option passed
* `<event>` success event

######fail arguments######

* `<string>` the objectStore option passed
* `<event>` fail event

###count (<object> options)###
Count total records in an objectStore, with optional filtering for an index/value*

######arguments######

* store: `<string>` objectStore name [required],
* key: `<string>` index to filter count to [optional]
* value: `<string>` value of index to filter count to [optional]
* success: `<function>` [see below for arguments]
* error: `<function>` [see below for arguments]

######success arguments######

* `<int>` count
* `<string>` the objectStore option passed
* `<event>` success event

######fail arguments######

* `<string>` the objectStore option passed
* `<event>` fail event


Full Example
=========

    // schema
    var dbSchema = [{
        name: 'animals',
        key: { keyPath: 'id'},
        indexes: [
            { name: 'class', field: 'class', unique: false }
        ]
    }];

    // dbobject store
    var db;

    // init
    var IDB = new DB({
        name: 'dbname',
        version: 1,
        schema: dbSchema
        success: function (DBObject) {
            db = DBObject;
            doStuff();
        },
        error: function (e) {
            console.log('error initing db');
        }
    });
    
    function doStuff () {
        // make calls with the db object here
        var data = [
            {id: 1, name: 'raccoon', class: 'mammalia'},
            {id: 2, name: 'octopus', class: 'cephalopoda'},
            {id: 3, name: 'ibex', class: 'mammalia'},
            {id: 4, name: 'ant', class: 'insecta'},
            {id: 5, name: 'whale', class: 'mammalia'},
            {id: 6, name: 'eagle', class: 'aves'},
            {id: 7, name: 'iguana', class: 'reptilia'},
            {id: 8, name: 'crab', class: 'malacostraca'}
        ];

        db.insert({
            store: 'animals', 
            data: data, 
            replace: true,
            success: function (rowsAdded, rowsFailed, e) {
	            // added
	            console.log('Added ' + rowsAdded.length + ' rows to animals');
	            
	            // get id = 5
	            db.get({
	                store: 'animals', 
	                id: 5,
	                success: function (data, e) {
	                    console.log(data.name + ' is in the class ' + data['class']);
	                },
	                error: function (id, e) {
	                    console.log('error getting row ' + id);
	                }
	            });
	        }, 
	        error: function (rowsAdded, rowsFailed, e) {
                // error
                console.log(rowsFailed.length + ' rows failed insert');
            }
        });
    }
