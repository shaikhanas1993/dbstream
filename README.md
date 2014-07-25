dbstream
========

DatabaseStream API for creating abstract, portable and functional Node streams for accessing databases.

> Due to the massive fragmentation of Node libraries for accessing different databases, it's difficult to write elegant code that is fully portable across database systems. This API has been designed to encourage similarity between Node modules that are used to access databases. Inspired by [PEP 249](http://legacy.python.org/dev/peps/pep-0249/)

### Usage

This example shows the effectiveness of using Node streams, and the functional, stream-lined API of the `dbstream` API.

```javascript
var db = require( "dbstream-somedb" );
var connection = db.connect( /* settings */ );

// write data
var cursor = new connect.Cursor(); 
cursor.write({ name: "Hello" }) // upsert where id == 1
cursor.write({ name: "World" } // insert
cursor.end();

// read data
new connect.Cursor()
  .find({ name: "Hello" })
  .limit(10)
  .on( "data", console.log ) 
```

###### Insert

Inserting objects involves calling the `write` for each object you wish to write to the database. Once done, call the `end()` method to indicate that no more writes are required. Once everything is saved, the `finish` event will be triggered:

```javascript
cursor.write({ name: "Hello" });
cursor.write({ name: "World" });
cursor.on("finish", function() {
  console.log( "Everything was saved" );
})
cursor.end()
```

###### Upsert

Updating objects is exactly the same as inserting, except that the object contains an `id` field which will cause the operation to be an upsert:

```javascript
cursor.write({ id: 1, name: "Hello" });
cursor.write({ id: 1, name: "World" });
cursor.end(); // just one object will be saved, "World" will override "Hello"
```

###### Remove

Similarily, the `remove` command can be used to remove objects:

```javascript
cursor.remove({ id: 1 });
cursor.on( "finish", function() {
  console.log( "ID: 1 was removed" );
});
cursor.end();
```

###### Read

Reading objects from the database involves defining the query parameters, and then reading the results:


```javascript
cursor.find({ name: "Hello" }).sort("id").skip(1).limit(1);
cursor.on("data", console.log);
cursor.on("end", function() {
  console.log("Done reading");
});
```

Because cursors are just Node Streams, you can pipe them together to construct functional data-processing pipelines:

```javascript
var es = require("event-stream");
cursor.find({ name: "Hello" })
  .pipe(es.map(function(obj, callback){
    obj.name += "!";
    callback(obj);
  })
  .pipe(new Cursor()) // write the modifications back to the database
  .pipe(process.stdout) // write the saved object to stdout
```


### Cursor

Cursors provide the core functionality of the API. They are simply [Node Streams](http://nodejs.org/api/stream.html#stream_class_stream_duplex) that expose an API for defining a Database operation in a DB-agnostic manner:

###### .find(query)

* `query` a key-value Object that defines the database query selection
* Returns the Cursor instance itself


###### .sort(key [, direction])

* `key` A String for the field-name to sort by
* `direction` An integer that defines the sort direction: 1 for ascending (default), -1 for decending
* Returns the Cursor instance itself

###### .skip(n)

* `n` Number of rows to skip
* Returns the Cursor instance itself


###### .limit(n)

* `n` Number of maximum rows to return
* Returns Cursor object itself

#### Connection

###### module.connect( settings )

* `settings` a configuration object defined by the concrete implementation
* Returns a Connection object 

High-level constructor of the Connection object. 

