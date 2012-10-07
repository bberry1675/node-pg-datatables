node-datatable
==============

Server-side processor for the JQuery Datatable plug-in.

This initial checkin is not ready for public use.

The node-datatable module provides backend SQL query generation and result parsing to support
[datatable](http://datatables.net/usage/server-side) server-side processing for SQL databases.
This module does not connect nor query a database, instead leaving this task to the calling application.
This has been done so that the caller can leverage his or her existing module choices for connection pools,
database interfaces, and the like. This module has been used with
both [node-mysql](https://github.com/felixge/node-mysql) and [sequelize](http://sequelizejs.com).

An incomplete code example:

```javascript
var QueryBuilder = require('datatable');

var tableDefinition = {
    sTableName: "Orgs",
    aoColumnDefs: [
        { mData: "o", bSearchable: true },
        { mData: "cn", bSearchable: true },
        { mData: "support" }
    ]};

var queryBuilder = new QueryBuilder( tableDefinition );

// requestQuery is normally provided by the datatable ajax call
var requestQuery = {
    iDisplayStart: 0,
    iDisplayLength: 5
};

// Build an array of queries
var queries = queryBuilder.buildQuery( requestQuery );

// Connect with and query the database.
// If you have turned on multipleStatements (e.g. node-mysql) then you may join the queries into one string.
// multipleStatements is normally turned off by default to prevent SQL injections.

var myDbObject = ...
myDbObject.query( queries, function( err, resultArray ) {
    var result = queryBuilder.parseResponse(resultArray);
    res.json(result)
});

```

## API ##

The source code contains additional comments that will help you understand this module.

### Constructor ###

#### Parameters ####

The node-datatable constructor takes an object parameter that has the following options:

- ```sTableName``` - The name of the table in the database where a JOIN is not used. If JOIN is used then set ```sSelectSql```.

- ```sCountColumnName``` For simple queries this is the name of the column for which to do a COUNT(). Defaults to ```id```.
For more complex queries, when sSelectSql is set, ```*``` will be used.

- ```sDatabase``` - If set then will add a SQL "USE sDatabase" statement as the first SQL query string.

- ```aoColumnDefs``` - An array of objects each containing ```mData``` and ```bSearchable``` properties.

- ```bSearchable``` default is false.

- ```aSearchColumns``` -

- ```sSelectSql``` - If set then this defines the columns that should be selected, otherwise ```*``` is used. This can be
used in combination with joins (see ```sFromSql```).

- ```sFromSql``` - If set then this is used as the FROM section for the SELECT statement. If not set then ```sTableName```
is used. Use this for more complex queries, for example when using JOIN. Example when using a double JOIN:

- ```javascript
"table1 LEFT JOIN table2 ON table1.errorId=table2.errorId LEFT JOIN table3 ON table1.sessionId=table3.sessionId"
```

- ```sWhereAndSql``` - Any arbitrary custom SQL to be joined to other WHERE clauses using AND.

- ```sDateColumnName``` - If this property and one of ```dateFrom``` or ```dateTo``` is set, a date range WHERE construct
will be added to the SQL query. This should be set to the name of the datetime column.

- ```dateFrom``` - If set then the query will filter for records greater then or equal to this date.

- ```dateTo``` - If set then the query will filter for records less then or equal to this date.

- ```fnRowFormatter``` - A row formatter function (more documentation to follow).

#### Returns #####

The query builder object.

Example:

```javascript
var queryBuilder = new QueryBuilder( {
    sTableName: 'user',
    aoColumnDefs: [
        { mData: 'username', bSearchable: true },
        { mData: 'email', bSearchable: true }
    ]});
```

### buildQuery ###

Builds an array containing between two and four SQL statements, in the following order:

1. _(Optional, if sDatabase is set)_ A USE statement that specifies which database to use.
2. _(Optional, if requestQuery.sSearch is set)_ A SELECT statement that counts the number of filtered entries.
This is used to calculate the ```iTotalDisplayRecords``` return value.
3. A SELECT statement that counts the total number of unfiltered entries in the database. This is used to calculate
the ```iTotalRecords``` return value.
4. A SELECT statement that returns the actual filtered records from the database. This will use LIMIT to limit the number
of entries returned.

Note that #2, #3 and #4 will include date filtering as well as any other filtering specified in ```sWhereAndSql```.

#### Parameters ####

- ```requestQuery```: An object containing the properties set by the client-side datatable library as defined in [Parameters sent to the server](http://datatables.net/usage/server-side).

#### Returns #####

The resultant array of query strings. The queries should be executed in order, and the result objects collected into an array which you later pass to the ```parseReponse``` function. 

Example:

```javascript
var queries = queryBuilder.buildQuery( oRequestQuery );
```

### parseResponse ###

Parses an array of response objects that were received in response to each of the queries generated by the ```buildQuery``` function. The order of responses must correspond with the query order.

#### Parameters ####

- __queryResult__: The ordered array of query response objects.

#### Returns #####

An object containing the properties defined in [Reply from the server](http://datatables.net/usage/server-side).

Example:

```javascript
var result = queryBuilder.parseResponse( queryResponseArray );
res.json(result);
```

## TODO ##

1. Add an additional parameter to allow more then the requested number of records to be returned. This can be used to reduce the
number of client-server calls (I think).
2. A more thorough SQL injection security review (volunteers?).

## References ##

[1](http://datatables.net/usage/server-side)
[2](http://datatables.net/forums/discussion/4214/solved-how-to-handle-large-datasets/p1)