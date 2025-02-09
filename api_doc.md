## Api Document


### Connection objects have server wide methods.
------------

#### ok,err = conn:connect(host, port)
Default host and port is: `localhost` and `27017`.

#### ok,err = conn:set_timeout(msec)
Sets socket connecting, reading, writing timeout value, unit is milliseconds.

In case of success, returns 1. In case of errors, returns nil with a string describing the error.

#### ok,err = conn:set_keepalive(msec, pool_size)
Keeps the socket alive for `msec` by ngx_lua cosocket.

In case of success, returns 1. In case of errors, returns nil with a string describing the error.

#### times,err = conn:get_reused_times()
Returns the socket reused times.

In case of success, returns times. In case of errors, returns nil with a string describing the error.

#### ok,err = conn:close()
Closes the connection.

In case of success, returns 1. In case of errors, returns nil with a string describing the error.

#### bool, hosts = conn:ismaster()
Returns a boolean indicating if this is the master server and a table of other hosts this server is replicating with
or `nil, err` on failure.

#### newconn = conn:getprimary ( [already_checked] )
Returns a new connection object that is connected to the primary server
or `nil , errmsg` on failure.

The returned connection object may be this connection object itself.


#### databases = conn:databases ( )
Returns a table describing databases on the server.

		databases.name: string
		databases.empty: boolean
		databases.sizeOnDisk: number

#### conn:shutdown()
Shutsdown the server. Returns nothing.

#### db = conn:new_db_handle(database_name)
Returns a database object, or nil.

### Database objects perform actions on a database
------------

#### db:list()

#### db:dropDatabase()

#### db:add_user(username, password)

#### ok, err = db:auth(username, password)
Returns 1 in case of success, or nil with error message.

#### ok, err = db:auth_scram_sha1(username, password)
Returns 1 in case of success, or nil with error message.

For authentication with MongoDB 2.8(3.0) or later.Authenticate using SCRAM-SHA-1 which is the default authentication mechanism supported by a cluster configured for authentication with MongoDB 2.8(3.0) or later.

#### col = db:get_col(collection_name)
Returns a collection object for more operations.

#### gridfs = db:get_gridfs(fs)

Collection objects
------------

#### n = col:count(query)

#### ok, err = col:drop()
Returns 1 in case of success, or nil with error message.

#### n, err = col:update(selector, update, upsert, multiupdate, safe)
Returns number of rows been updated or nil for error.

 - upsert, if set to `1`, the database will insert the supplied object into the collection if no matching document is found, default to `0`.
 - multiupdate, if set to `1`, the database will update all matching objects in the collection. Otherwise only updates first matching  doc, default to `0`. Multi update only works with $ operators.
 - safe can be a boolean or integer, defaults to `0`. If `1`, the program will issue a cmd `getlasterror` to server to query the result. If `false`, return value `n` would always be `-1`

#### n, err = col:insert(docs, continue_on_error, safe)
Returns 0 for success, or nil with error message.

 - continue_on_error, if set, the database will not stop processing a bulk insert if one fails (eg due to duplicate IDs).
 - safe can be a boolean or integer, defaults to `0` or `false`. If `1` or ``true`, the program will issue a cmd `getlasterror` to server to query the result. If `false`, return value `n` would always be `-1`

#### n, err = col:delete(selector, singleRemove, safe)
Returns number of rows been deleted, or nil with error message.

 - singleRemove if set to 1, the database will remove only the first matching document in the collection. Otherwise all matching documents will be removed. Default to `0`
 - safe can be a boolean or integer, defaults to `0`. If `1`, the program will issue a cmd `getlasterror` to server to query the result. If `false`, return value `n` would always be `-1`

#### r = col:find_one(query, returnfields)
Returns a single element array, or nil.

 - returnfields is the fields to return, eg: `{n=0}` or `{n=1}`

#### cursor = col:find(query, returnfields, num_each_query)
Returns a cursor object for excuting query.

 - returnfields is the fields to return, eg: `{n=0}` or `{n=1}`
 - num_each_query is the max result number for each query of the cursor to avoid fetch a large result in memory, must larger than `1`, `0` for no limit, default to `100`.

#### col:getmore(cursorID, [numberToReturn], [offset_i])
 - cursorID is an 8 byte string representing the cursor to getmore on
 - numberToReturn is the number of results to return, defaults to -1
 - offset_i is the number to start numbering the returned table from, defaults to 1

#### col:kill_cursors(cursorIDs)

### Cursor objects
--------------------

#### index, item = cursor:next()
Returns the next item and advances the cursor.

#### cursor:pairs()
A handy wrapper around cursor:next() that works in a generic for loop:

		for index, item in cursor:pairs() do

#### cursor:limit(n)
Limits the number of results returned.

#### cursor = cursor:sort(fields)
Returns an array with size `size` sorted by given field. 

 - field is an array by which to sort, and this array size _MUST be 1_. The element in the array has as key the field name, and as value either `1` for ascending sort, or `-1` for descending sort. 


###  Object id
#### objid:tostring()
#### objid:get_ts()
#### objid:get_pid()
#### objid:get_hostname()
#### objid:get_inc()

### Grid FS Object

_under developing_

#### gridfs_file = gridfs:find_one(fields)
Returns a gridfs file object.

#### gridfs_file = gridfs:remove(fields, continue_on_err, safe)
Returns number of files been deleted, or nil with error message.

 - singleRemove if set to 1, the database will remove only the first matching document in the collection. Otherwise all matching documents will be removed. Default to `0`
 - safe can be a boolean or integer, defaults to `0`. If `1`, the program will issue a cmd `getlasterror` to server to query the result. If `false`, return value `n` would always be `-1`

#### bool = gridfs:get(file_handler, fields)
Writes first object matchs fields into file_handler. This API will malloc a buffer in file size in memory.

#### n, err = gridfs:insert(file_handler, meta, safe)
Returns 0 for success, or nil with error message.

 - file_handler is file handler returned by io:open().
 - meta is a table include `_id, filename, chunkSize, contentType, aliases, metadata` or anything the user wants to store. Default meta.filename is the object id in string.
 - safe can be a boolean or integer, defaults to `0`. If `1`, the program will issue a cmd `getlasterror` to server to query the result. If `false`, return value `n` would always be `-1`

#### gridfs_file, err = gridfs:new(meta)
Returns a new gridfs file object, or nil with error message.

 - meta is a table include `_id, filename, chunkSize, contentType, aliases, metadata` or anything the user wants to store. Default meta.filename is the object id in string.


### Grid FS File Object
-------------------

#### n, err = gridfs_file:read(size, offset)
Returns number of bytes read from mongodb, or nil with error message.

 - offse start from 0

#### n, err = gridfs_file:write(buf, offset, size)
Returns number of bytes writen into mongodb, or nil with error message.

 - offset is the file offset(should not beyond the end of the file), starting from 0.
 - size is the number of bytes to be writen.

#### bool, err = gridfs_file:update_md5()
Hashs the file content and updates the md5 in file collection.
