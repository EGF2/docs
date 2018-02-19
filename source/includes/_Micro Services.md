# Micro Services

We will list all services that are present in the framework. For each service we will show public and internal endpoints, config parameters and extension points. Not all of the services will have something in all of the subsections. By "extension points" we mean an intended way of extending framework functionality. For example, all **logic** rules will be provided by clients using extension point. Internal endpoints are to be used by services, they are not exposed to the Internet, public endpoints are exposed to the Internet.

All services might be configured with config.json file, which provides default config settings. Another way is to use environment variables (env vars). Please, pay attention to the fact that env vars have precedence over config.json.
Also, please note env vars prefix "egf_" which allows to isolate env vars of the particular service from variables of deployment environment.

## client-data

This service will serve as a focal point for data exchange in the system. As all events in the system will have to go through **client-data** it should be fast, simple and horizontally scalable. It should be able to support all graph operations from either cache or data store.

### Internal Endpoints

`GET /v1/graph` - return graph config in JSON.

`GET /v1/graph/<object_id>` - to get an object by ID, returns object JSON.

`GET /v1/graph/<object_id1>,<object_id2>, …` - to get several objects at once, returns JSON structured in the same way as paginated result in graph API.

To create a new object use `POST /v1/graph` with JSON body of the new object. Newly created object will be returned as a response, along with `"id"` and `"created_at"` fields set automatically. Object IDs are generated in the format: `"<UUID>-<object type code>"`.

To update an object use `PUT /v1/graph/<object_id>` with JSON body containing modified fields. In order to remove a field use `"delete_fields": ["<field_name1>", "<field_name2>", ...]` field in the JSON body. Updated object will be returned.

To delete an object use `DELETE /v1/graph/<object_id>`. Objects are not deleted from the data store, `"deleted_at"` field is added instead. Returns the following JSON: `{"deleted_at": "<date and time, RFC3339>"}`.

In order to get objects that listed on an edge use `GET /v1/graph/<src_object_id>/<edge_name>` with pagination parameters. Extension is not supported by **client-data** service but is supported by the "client-data" commons utility.

In order to check for an existence of an edge use `GET /v1/graph/<src_object_id>/<edge_name>/<dst_object_id>`. Service will return an object if the edge exists. 404 will be returned otherwise.

To create a new edge use `POST /v1/graph/<src_object_id>/<edge_name>/<dst_object_id>`. Returns JSON: `{"created_at": "<date and time of creation for this edge>"}`.

To delete an edge use `DELETE /v1/graph/<src_object_id>/<edge_name>/<dst_object_id>`. Returns the following JSON: `{"deleted_at": "<date and time of deletion, RFC3339>"}`.

### Public Endpoints

None

### Config

```
{
	"port": 8000,  // port that service will listen to
	"log_level": "debug | info | warning",
	"storage": "rethinkdb | cassandra" // storage system, currently supported are RethinkDB and Cassandra
	"cassandra": { // Cassandra config, should be present in case "storage" is equal to "cassandra". For more info on supported parameters please see Cassandra driver config specification. Parameters specified here are passed to the driver as is, without modifications.
		"contactPoints": ["localhost"],
		"keyspace": "eigengraph"
	},
	"rethinkdb": { // RethinkDB config, should be present in case "storage" is equal to "rethinkdb". For more info on supported parameters please see RethinkDBDash driver docs. Parameters specified here are passed to the driver as is, without modifications.
		"host": "localhost",
		"db": "eigengraph"
	},
	"queue": "rethinkdb", // For single instance only! For scalable solutions use kafka.
	"graph": { // graph config section, contains info on all objects, edges, ACLs and validations in the system
		"custom_schemas": {
			"address": {
      				"use": { "type": "string", "enum": ["official", "shipping", "billing", "additional"], "default": "additional" },
      				"label": { "type": "string" },
      				"lines": { "type": "array:string", "min": "1" },
      				"city": { "type": "string" },
      				"state": { "type": "string" },
      				"country": { "type": "string" }
    			},
    			"human_name": {
      				"use": { "type": "string", "enum": ["usual", "official", "temp", "nickname", "anonymous", "old", "maiden"], "default": "official" },
      			 	"family": { "type": "string", "min": 1, "max": 512, "required": true },
      				"given": { "type": "string", "min": 1, "max": 512, "required": true },
      				"middle": { "type": "string", "min": 1, "max": 64 }`
    			}
  		},
 		"common_fields": {
    			"object_type": { "type": "string", "required": true },
    			"created_at": { "type": "date", "required": true },
    			"modified_at": { "type": "date", "required": true },
    			"deleted_at": { "type": "date" }
  		},
		"system_user": {
			"code": "01", // object type code
			"suppress_event": true, // in case "no_event" is present and set to true client-data will not produce any events related to this object
			"back_end_only": true,
			"validations": { // for more info on validations please see Validations subsection
			}
		},
		"session": {
			"code": "02",
        	"suppress_event": true,
			"back_end_only": true
    		},
    		"user": {
        		"code": "03",
        		"edges": { // edges allowed for this object type
            			"roles": { // here "roles" is edge name
               				"contains": [""] // array of strings with object type names that are allowed to be added to this edge
            			}
        		},
			"fields": {
				"name": {
					"type": "struct",
					"schema": "human_name",
					"required": true,`
					"edit_mode": "E", // this field can take values: "NC | NE | E", NC means that this field can not be set at creation time, NE means that this field can not be changed, E means field can be edited
					"default": "",
					"auto_value": ""
				}
			}
    		},
    		"event": {"code": "04"},
    		"job": {"code": "05"},
    		"file": {"code": "06"},
    		"schedule": {"code": "07", "back_end_only": true},
    		"schedule_event": {"code": "08", "back_end_only": true},
    		"objects": {
            	"secret_organization": "<object ID for the SecretOrganization object, string>"
            }

  	}
}
```
List of vars with default values is as follows:

```
egf_port = 8000
egf_log_level = info
egf_storage = rethinkdb
egf_cassandra = { "contactPoints": ["localhost"], "keyspace": "eigengraph"}
egf_rethinkdb = { "host": "localhost", "db": "eigengraph" }
egf_queue = rethinkdb
egf_kafka = { "hosts": ["localhost:9092"], "client-id": "client-data", "topic": "events" }
```

Please note that for brevity sake we are not showing the full client-data configuration here, <a href="https://github.com/egf2">see it on GitHub</a>.

Object declaration can contain “volatile” boolean field. In case it is set to true objects of this type will be physically removed from a DB upon deletion. Otherwise DELETE requests mark objects with “deleted_at”, objects are not physically removed from the DB.

#### Field Validations

Object field validations can be added to field declarations inside of an object declaration as follows:

```js
"fields": {
	"name": { "type": "struct", "schema": "human_name", "required": true }
}
```

Another sample is for a hypothetical Account object:

```js
"fields": {
  	"user": { "type": "object_id:user" },
  	"balance": { "type": "string", "required": true },
  	"credit": { "type": "number", "default": 0 },
  	"tax_collected": { "type": "number", "default": 0 }
}
```
Field validations are provided in the "custom_schemas", "common_fields" fields inside of "graph" section. "custom_schemas" section contains validations for common reusable data structures. "common_fields" section provides validations for fields that can be present in all object types. Validations for particular objects are stored in field declarations.

Validation specification for a particular field can contain the following fields:

1. `"type"` - required, can be one of the following strings: "string", "number", "integer", "date", "boolean", "object_id", "struct" and "array".
* `"validator"` - optional, custom validation handler name.
* `"required"` - optional boolean field, defaults to false.
* `"default"` - optional field that contains default value for the field.
* `"enum"` - optional array of strings. If present only values from this array can be assigned to the field.
* `"min"` and `"max"` - optional fields that can be present for "string", "array" and "number" typed fields. For "string" these fields restrict string length. For "array" these fields restrict array size, for "number" - min and max values that can be stored.
* `"schema"` - required for fields with type "struct" or "array:struct". Contains nested field validation specification for the struct. Can also contain a name of one of declared custom schemas (section "custom_schemas").
* `"edit_mode"` - can take values "NC", "NE", "E". "NC" means field can not be set at creation time. "NE" means that field can not be changed. "E" means that the object is editable. In case "edit_mode" is not present it means that the field can’t be set at creation and also is not editable by users.
* `"object_types"` - array of strings, required in case "type" is equal to "object_id"
* `"auto_value"` - string, can take values: `"req.user"`, `"src.<field_name>"`. `"req.user"` will set the field with User object corresponding to the currently authenticated user. `"src.<field_name>"` is handy when an object is created on edge. client-api will take value of a field `"<field_name>"` and set this value into the field with `"auto_value"`.

For fields with type "object_id" field `"object_types"` should be provided. In case this field can hold any object type please use `"object_types": ["any"]`.

Fields with type "array" should have additional declaration for the type of entities that can be stored within this array. It should be specified in `"schema"` field.

#### ACL

ACL related fields can be added on object definition and edge definition levels in "graph" section.

```js
"some_object": {
  	"GET": "<optional, comma separated ACL rules for GET method>",
 	"POST": "<optional, comma separated ACL rules for POST method>",
  	"PUT": "<optional, comma separated ACL rules for PUT method>",
  	"DELETE": "<optional, comma separated ACL rules for DELETE method>",
  	"edges": {
		"<edge name>": {
			"GET": "<optional, comma separated ACL rules for GET method>",
			"POST": "<optional, comma separated ACL rules for POST /v1/graph/<src>/<edge_name> with new destination object body>",
			"LINK": "<optional, comma separated ACL rules for POST /v1/graph/<src>/<edge_name>/<dst> (create edge for existing objects)>",
			"DELETE": "<optional, comma separated ACL rules for DELETE method>"
		},
		...
  	}
}
```
Supported ACL rules:

- any - allow access to any request.
- registered_user - allow access to registered users.
- self - allow access to own objects and edges - objects that have field `"user"` pointing to the current user.
- \<role based access rule> - this rule is specified as a role name that user should have on User/roles edge. Example: customer_role, admin_role, etc.

### Extension Points

#### Custom validation handler

In case built-in validations are not sufficient client can implement custom validation logic for any field of any object. For example, `email` field may require custom validation.


In order to add custom validation please do the following:

1. Implement a module that will perform custom validation inside of "controllers/validation/" folder. This module should implement one or several functions with signature: `function (val) {}`` where *val* is the value of the field. Function should return true in case validation succeeded and false otherwise. Function (or functions, in case single module implements multiple validators) should be exported from the module.
* Add a record of the form `"<type_name>": require("./<module_file_name>").<optional, function name in case module exports multiple functions>` to the *validationRegistry* map in `"validation/index.js"`.
* In **client-data** config, "graph" section, specify \<type_name> in field validation declaration in `"validation"` field.

Example of a custom validator for `"email"` field:

```js
function CheckEmail(val) {
	let re = new RegExp("^\S+@\S+$");
	return re.test(val);
}
```

Corresponding *validationRegistry* map will be:

```js
const validationRegistry = {
  	email: require("./check_email")
}
```

And object validations will look like:

```js
"validations": {
  	"user_email": { "type": "email" }
}
```

## client-api
### Internal Endpoints

None

### Public Endpoints
#### Object Operations
In order to **get** an object use `GET <gateway URL>/v1/graph/<object ID>`. I will omit `<gateway URL>` going forward for brevity sake.


To **create** an object use `POST /v1/graph` with JSON body containing object fields. As a result full object JSON will be returned, including field `"id"`.


To **change** object use `PUT /v1/graph/<object ID>` with JSON body containing only the fields that need to be modified. Unchanged fields can be sent as well but it is not required. In order to remove a field from an object completely you need to send `"delete_fields": ["field names", ...]` field inside of the JSON. We don’t support removing fields from within nested structures at the moment.


To **delete** an object use `DELETE /v1/graph/<object ID>`.

#### Edge Operations
To verify **existence** of an edge please use `GET /v1/graph/<src object ID>/<edge name>/<dst object ID>`. This request will return edge’s target object in case such an edge exists. 404 will be returned otherwise.


To get a **list** of objects that belong to an edge use `GET /v1/graph/<src object ID>/<edge name>`. The list can be paginated with `"after"`, and `"count"` query fields. Request will return JSON of the following format:

```
{
    "results": [],
    "first": <index or object ID of the first entry, int>,
    "last": <index or object ID of the last entry, int>,
    "count": <total number of objects, int>
}
```

Please note that `"first"` and `"last"` field values can vary depending on the DB that is used for a deployment. For Cassandra these fields will contain object IDs, for RethinkDB they will contain indices of objects within a collection. We recommend to not try interpreting `"first"` and `"last"` field values, just use `"last"` as a parameter for `"after"` in case next page should be retrieved. In case the returned page is the last one in the collection field “last” will be omitted.

EGF2 supports expansion on edges for:

- Other edges, e.g. `GET /v1/graph/<Post object ID>?expand=comments` will return a list of Post objects. Post objects that have comments will contain `Post.comments` field populated with a list of related comments along with pagination related info. Only the first page can be retrieved this way. Page size can be specified as follows: `GET /v1/graph/<Post object ID>?expand=comments(10)` - return 10 objects (if exist). Default page size is 25, max amount of objects that can be specified is 50.
- Object fields, for example, `GET /v1/graph/<Post object ID>?expand=user,comments`. `Post.user` field will contain embedded User object.
- Object fields within nested structures, example: `GET /v1/graph/<Post object ID>?expand=foo.bar`.

Nested expand up to 4 levels can be used as follows: `GET /v1/graph/<Post object ID>?expand=user{roles{favorites}},comments`. `{}` is used to go one level down in expansion for the field on the left of curly braces.

To **create** a new edge please use `POST /v1/graph/<src object ID>/<edge name>/<dst object ID>` in case target object already exists, or use `POST /v1/graph/<src object ID>/<edge name>` with JSON body that contains fields to be used for the new target object. In this case target object and edge will be created simultaneously.


To **delete** an edge use `DELETE /v1/graph/<src object ID>/<edge name>/<dst object ID>`.

#### Search Endpoint

**client-api** provides a `GET /v1/search endpoint for searching`. The following parameters are supported:

- `"q"` - textual query, optional. In case q is not specified all objects from an index will returned (paginated, of course).
- `"fields"` - comma separated list of fields. Here you can list fields that the q query should be searched in.
- `"object"` - object type name. This parameter identifies ES index that will be used for searching. In most cases ES index name will correspond to an object type name, e.g. "user" index for User objects. It is also possible to have a non trivial index spanning several objects / edges. Those indexes named in an arbitrary manner, preferably using object type names for clarity sake.
- `"filters"` - comma separated key value pairs, e.g. `filters=category:123,size:12`. In case `!` sign is added in front of a value server will return results that do not contain this value. Example `filters=manufacturer:!23423435` - return only results that do not have manufacturer field with specified value.
- `"sort"` - comma separated list of fields to be used for sorting, with optional direction specifier in parenthesis, i.e. `gender(DESC)`.
- `"range"` - comma separated list of ranges, format: `range=<field_name>:<min value of the field or “min” string>:<max value of the field or “max” string>`. For example, in order to get all Files that are older than a particular date: `range=created_at:min:<date TODO>`.

Results are returned in paginated format, the same as results for getting edges. Full expansion functionality is supported for search requests as well, the same as for regular graph API requests.

ACL rules are applied to the search results in the same way as they are applied to the results of graph API get edge requests.

##### Miscellaneous

[TO BE IMPLEMENTED IN V0.2.0]

Mobile client applications can use app version endpoint by calling GET /v1/version?app_id=<some app ID>&version=<current installed version of the app>. Server will respond with JSON:

{
	“latest”: “<string>”, // latest available version of the app
	“update_mode”: “now | period | recommended”,
	“force_update_at”: “<string, RFC3339>”
}

This endpoint can be called by client apps once per day. Received JSON should be interpreted as follows:
In case “latest” string is equal to the current installed version of the app fields “update_mode” and “force_update_at” will not be present. No action is required
In case “update_mode” is present and equal to “now” the app should display a message to the user. When message is read the app should quit
In case “update_mode” is equal to “period” the app should display a message informing the user that the app should be updated before “force_update_at” date. After message is presented the app should continue as usual.
In case “update_mode” is “recommended” the app should display a message and them continue as usual.

**client-api** should use “graph/objects” section of client-data config in order to obtain references to MobileClient objects. These objects should be used in order to answer client requests.

### Config

```
{
   "port": 2019,
   "log_level": “debug | info | warning”,
   "auth": "http://localhost:2016",
   "client-data": "<URL pointing to client-data service>"
   "elastic": { // ElasticSearch parameters. Passed to the ElasticSearch driver without modifications.
    	"hosts": ["localhost:9200"]
   }
}
```

List of vars with default values is as follows:

```
egf_port = 2019
egf_auth = "http://localhost:2016"
egf_client-data = "http://localhost:8000"
egf_elastic = { "hosts": ["localhost:9200"] }
```

### Extension Points
#### Custom ACL rule

ACL handling covers basic needs with regards to restricting access to objects and edges. In case more elaborate control is needed custom ACL handler can be implemented.

In order to create a new custom ACL handler please do the following:

1. Implement a module that will handle the request inside of `"acl/extra/"` folder. This module should implement one or several functions with signature: `function (user, id, body) {}` that will perform the processing of the request. Function should return a promise with the result. Function (or functions, in case single module implements multiple handlers) should be exported from the module.
* Add a record of the form `"<custom ACL name>": require("./<module_file_name>").<optional, function name in case module exports multiple functions>` to the rules map in `"acl/rules.js"`.

Example of a custom ACL handler:

```javascript
"use stict";

const clientData = required("./client-data");

function reviewOwner(user, id) {
	return clientData.getObject(id).then(review => user === review.from)
}

module.exports = reviewOwner;
```
#### Custom endpoint handler

There can be situations when simple processing of a request to the graph API is not sufficient and this extended processing should be performed synchronously within request (not asynchronously in logic service). For example, client-api should set some fields in a newly created or modified object that are based on fields sent in by a client. Such cases can be accommodated for by creating a custom endpoint handler.

Custom endpoint handlers can be assigned to any object type, any edge and any HTTP method. In order to create a new custom handler please do the following:

1. Implement a module that will handle the request inside of `"controllers/extra/"` folder. This module should implement one or several functions with signature: `function (req) {}` that will perform the processing of the request. Function should return a promise with the result. Function (or functions, in case single module implements multiple handlers) should be exported from the module.
* Add a record of the form `"<method, e.g. POST, GET, PUT, DELETE> <object type or edge>": require("./<module_file_name>").<optional, function name in case module exports multiple functions>` to the *handleRegistry* map in `"controllers/extra/index.js"`.

Example of a custom handler module that creates and removes a `"followers"` edge for creation and removal of `"follows"` edge:

```javascript
"use strict";

const clientData = require("../clientData");

function Follow(req) {
    return clientData.createEdge(req.params.src, req.params.edge_name, req.params.dst)
        .then(follow =>
            clientData.createEdge(req.params.dst, "followers", req.params.src).then(() => follow)
        );
}
module.exports.Follow = Follow;

function Unfollow(req) {
    return clientData.deleteEdge(req.params.src, req.params.edge_name, req.params.dst)
    .then(deleted =>
        clientData.deleteEdge(req.params.dst, "followers", req.params.src).then(() => deleted)
    );
}
module.exports.Unfollow = Unfollow;
```

## sync
In small and scalable modes this service is responsible for updating ES according to the changes happening in the system.

### Internal Endpoints
None

### Public Endpoints
None

### Config

```
{
	"log_level": "debug | info | warning",
	"elastic":{
		"hosts": ["localhost:9200"],
		"settings": { // ES settings, used as is with ES, no changes made
   			"filter": {
        			"autocomplete": {
       					"type": "edgeNGram",
           				"min_gram": 2,
            				"max_gram": 30,
       					"token_chars": ["letter", "digit", "symbol", "punctuation"]
       				}
    			},
			"analyzer": {
        			"autocomplete_index": {
          				"type": "custom",
            				"tokenizer": "standard",
           				"filter": ["autocomplete", "lowercase"]
        			},
       				"autocomplete_search": {
         				"type": "custom",
            				"tokenizer": "standard",
            				"filter": "lowercase"
        			}
    			}
			"indices": {
				"file": { // index name, to be used with search endpoint
					"settings": {} // ES settings local for this index
					"object_type": "file",
					"index": "file", // ES index name
					"mapping": {
						"id": {"type": "string", "index": "not_analyzed"},
						"standalone": {"type": "boolean"},
						"created_at": {"type": "date"}
					}
				},
			"schedule": { // index name, to be used with search endpoint
				"settings": {} // ES settings local for this index
				"object_type": "schedule",
				"index": "schedule", // ES index name
				"mapping": {
					"id": {"type": "string", "index": "not_analyzed"}
				}
			}
		}
	},
	"client-data": "<URL pointing to client-data service>",
	"queue": "kafka | rethinkdb",
	"consumer-group": "sync",
	"rethinkdb": {
		"host": "localhost",
		"port": "28015",
		"db": "eigengraph",
		"table": "events",
		"offsettable": "event_offset"
	},
	"kafka": {} // TODO Kafka parameters
}
```
List of vars with default values is as follows:

```
egf_log_level = info
egf_client-data = "http://localhost:8000"
egf_queue = rethinkdb
egf_rethinkdb = { "host": "localhost", "port": "28015", "db": "eigengraph", "table": "events", "offsettable": "event_offset" }
egf_kafka = { "hosts": ["localhost:9092"], "client-id": "sync", "topic": "events" }
```

"elastic" section contains info on ES indexes and global ES settings. It is possible to specify ES settings on the index level using "settings" field. We take "settings" content without modifications and apply settings to ES.

**sync** supports automatic and custom index processing, for the brevity sake we will use "automatic index" and "custom index" terms to identify type of processing going forward.

Adding custom indexes is described in Extension Points section below.

**sync** will react to events related to an object specified in `"object_type"` field for automatic indexes (this field is ignored for custom indexes). Automatic handler presumes that object field names correspond directly to field names in ES index. I.e `File.created_at` field is mapped into `"created_at"` field in ES index. It is possible to override this by providing `"field_name"` parameter in field declaration. This feature is useful to support nested structures, for, example: `"state": {"type": "string", "index": "not_analyzed", "field_name": "address.state"}` will populate ES index field "state" using data from `"address.state"` nested object field.

### Extension Points
#### Custom Index Handler

There can be situations when automatic ES index handling is not sufficient. For example, if client wants to build an index for an edge, which is currently not supported by automatic handling.

In order to create a new custom index handler please do the following:

1. Implement a module that will handle events related to the index inside of `"extra/"` folder (in **sync** service). This module should implement one or several functions with signature: `function (event) {}` that will perform the processing of the event. Function should return a promise with operation result. Function (or functions, in case single module implements multiple handlers) should be exported from the module.
* Add a record of the form `"<method, e.g. POST, GET, PUT, DELETE> <object type or edge>": require("./<module_file_name>").<optional, function name in case module exports multiple functions>` to the *handleRegistry* map in `"extra/index.js"`. Handlers from a single module will usually be specified for several records related to the events of interest for the custom index.

Example of a custom index handler that performs some pre-processing for an organization title:

```js
function processOrgTitle(title) {
	// some custom processing
}

function onPostOrganization(event) {
    var doc = {
        title: processOrgTitle(event.current.title.toLowerCase())
    };

    return elastic.client.index({
        index: elastic.indices.organization.index,
        type: elastic.indices.organization.type,
        id: event.object,
        body: doc
    });
}
```

## logic
This service contains business logic rules that can and should be executed in asynchronous mode.

### Internal Endpoints
None

### Public Endpoints
None

### Config

```
{
	"log_level": "debug | info | warning",
	"client-data": "<URL pointing to client-data service>",
	"scheduler": "<URL pointing to scheduler service>", // we presume that logic may have some recurrent tasks that need scheduling
	"queue": "kafka | rethinkdb",
	"consumer-group": "logic",
	"rethinkdb": {
		"host": "localhost",
		"port": "28015",
		"db": "eigengraph",
		"table": "events",
		"offsettable": "event_offset"
	},
	"kafka": {} // TODO Kafka parameters
}
```
List of vars with default values is as follows:

```
egf_log_level = info
egf_client-data = "http://127.0.0.1:8000/"
egf_queue = rethinkdb
egf_rethinkdb = { "host": "localhost", "port": "28015", "db": "eigengraph", "table": "events", "offsettable": "event_offset" }
egf_kafka = { "hosts": ["localhost:9092"], "client-id": "logic", "topic": "events" }
```

### Extension Points

Logic rules are implemented as handlers in **logic** service. In order to implement a rule:

1. Implement a module that will handle events related to the rule inside of `"extra/"` folder (in **logic** service). This module should implement one or several functions with signature: `function (event) {}` that will perform the processing of the event. Function should return a promise with operation result. Function (or functions, in case single module implements multiple handlers) should be exported from the module.
* Add a record of the form `"<method, e.g. POST, GET, PUT, DELETE> <object type or edge>": require("./<module_file_name>").<optional, function name in case module exports multiple functions>` to the *handleRegistry* map in `"extra/index.js"`. Handlers from a single module will usually be specified for several records related to the events of interest for the custom index.

## job
This service will execute long and/or heavy tasks asynchronously. It will listen to the system event queue and react to creation of Job objects. Number of simultaneous jobs that can be handled is specified in config option `"max_concurrent_jobs"`.

### Internal Endpoints
None

### Public Endpoints
None

### Config

```
{
	"log_level": "debug | info | warning",
	"max_concurrent_jobs": 5,
	"client-data": "<URL pointing to client-data service>",
	"queue": "kafka | rethinkdb",
	"consumer-group": "job",
	"rethinkdb": {
		"host": "localhost",
		"port": "28015",
		"db": "eigengraph",
		"table": "events",
		"offsettable": "event_offset"
	},
	"kafka": {} // TODO Kafka parameters
}
```

### Extension Points

Jobs are implemented as handlers in **logic** service. In order to implement a rule:

1. Implement a module that will handle a job inside of `"extra/"` folder (in **job** service). This module should implement one or several functions with signature: `function (jobCode) {}` that will perform the job. Function should return a promise with operation result. Function (or functions, in case single module implements multiple handlers) should be exported from the module.
* Add a record of the form `"<job code>”: require("./<module_file_name>").<optional, function name in case module exports multiple functions>` to the *handleRegistry* map in `"extra/index.js"`. Handlers from a single module will usually be specified for several records related to the events of interest for the custom index.


## pusher
Is responsible for reacting to events by sending notifications using various mechanisms, be it WebSockets, emails, SMS, native mobile push notifications etc.

[TO BE SUPPORTED IN V0.2.0]

Client applications can subscribe for notifications based on:
Object ID
Particular edge, represented as source object ID / edge name pair

In order to subscribe for notifications please send the following JSON via WebSockets connection:

```js
{
		“subscribe”: [
				{ “object_id”: “<string>” },
				{ “edge”:
						{
						“source”: “<string>”,
								“name”: “<string>”
						}
				}				
		]			
}
```
Subscription that is mentioned in the message will be added to the list of subscriptions for this client.

It is also possible to cancel subscription for an object or an edge:

```js
{
	“unsubscribe”: [
			{ “object_id”: “<string>” },
			{ “edge”:
					{
					“source”: “<string>”,
						“name”: “<string>”
					}
			}
	]
}
```

Note: In case connection is dropped all subscriptions are lost. When connection is restored client should renew subscriptions.

### Internal Endpoints

To **send** out an email please use `POST /v1/internal/send_email` with JSON:

```
{
	"template": "<template ID, string>",
	"to": "email address, string",
	"from": "email address, string",
	"params": {
		// template parameters
	}
}
```

We currently only support sending emails via SendGrid. SecretOrganization object is used to store SendGrid API token as follows: `"sendgrid_api_key": "<key value>"`.

### Public Endpoints

pusher exposes a WebSocket endpoint `/v1/listen`. Connection to the endpoint has to be authorized using Authorization header with bearer token, e.g. `"Authorization Bearer <token>"`. All connections are pooled within the service and are available for custom event handlers. We use Primus and Primus Rooms to allow a user to be connected from multiple devices.

### Config

```
{
	"port": 2017,
	"web_socket_port": 2000,
	"email_transport": "sendgrid",
	"log_level": "debug | info | warning",
	"client-data": "<URL pointing to client-data service>",
	"auth": "<URL pointing to auth service>",
 	"template_host": "host": "<URL to the host to be used in templates>",
 	"ignored_domains": ["<domain name>"], // list of domains for which emails will not be sent, for debug purposes
	"queue": "kafka | rethinkdb",
	"consumer-group": "pusher",
	"rethinkdb": {
		"host": "localhost",
		"port": "28015",
		"db": "eigengraph",
		"table": "events",
		"offsettable": "event_offset"
	},
	"kafka": {} // TODO Kafka parameters
}
```
List of vars with default values is as follows:

```
egf_port = 2017
egf_web_socket_port = 2000
egf_email_transport = sendgrid
egf_log_level = info
egf_client-data = "http://localhost:8000"
egf_auth = "http://127.0.0.1:2016",
egf_queue = rethinkdb
egf_rethinkdb = { "db": "eigengraph", "table": "events", "offsettable": "event_offset" }
egf_kafka = { "hosts": ["localhost: 9092"], "client-id": "pusher", "topic": "events" }
```

### Extension Points
#### Email Template

Email templates can be added as follows:

1. Add [MJML](https://mjml.io/) template file to the `"config/templates/mjml"` folder. Template variables can be added in the template using double curly brackets: `{{<parameter_name>}}`.
* Add template description to the `"config/templates/config.json"` file. Specify a path to the MJML template using "template" property. Email subject should be specified using `"subject"` field. Use `"params"` field to list template parameters.

`"sendEmail"` function from `"controller/email"` module in **pusher** should be used to send templated emails.

#### Custom Event Handler

In order to implement a handler:

1. Implement a module that will handle events related to the handler inside of `"extra/"` folder (in **pusher** service). This module should implement one or several functions with signature: `function (event) {}` that will perform the processing of the event. Processing usually means sending out a notification using some supported transport. Function should return a promise with operation result. Function (or functions, in case single module implements multiple handlers) should be exported from the module.
* Add a record of the form `"<method, e.g. POST, GET, PUT, DELETE> <object type or edge>": require("./<module_file_name>").<optional, function name in case module exports multiple functions>` to the *handleRegistry* map in `"extra/index.js"`. Handlers from a single module will usually be specified for several records related to the events of interest for the custom index.

## auth
### Internal Endpoints
Services will be able to get **session info** using `GET /v1/internal/session?token=<string, token value>`. **Auth** server will respond with Session object in case it can find one, 404 otherwise.

### Public Endpoints

This service is responsible for user related features, e.g. login, logout, password reset, email verification, registration, etc.

**"auth"** service will handle requests listed below.

To **register** `POST /v1/register` the following JSON:

```json
{
	"first_name": "<string>",
	"last_name": "<string>",
	"email": "<string>",
	"date_of_birth": "<string>",
	"password": "<string>"
}
```
Service will return `{"token": "<string>"}` JSON in case of success. In other words, user will be authenticated as a result of successful registration.

Server side should check if an email is taken yet. Upon registration the following will happen:

- **pusher** service will send an email with account verification ID.
- User object will be created with `User.verified = false`.
- New session will be created for the user.

To **verify email** `GET /v1/verify_email?token=<secret verification token>`, service will respond with 200 in case of success.

**Login** using `POST /v1/login` with the following JSON:

```json
{
	"email": "<email string>",
	"password": "<string>"
}
```
Server will return `{"token": "<string>"}` that will be used to access API.

**Logout** using `POST /v1/logout` with token specified as usual when accessing protected APIs. Server will respond with 200 in case logout was successful.

To **restore password** use `GET /v1/forgot_password?email="<string>"`. Server will send out an email with password reset instructions and respond with 200 in case operation was successful.
To reset password `POST /v1/reset_password` with JSON:

```json
{
	"reset_token": "<string>",
	"new_password": "<string>"
}
```
Service will respond with 200 in case of success.

In order to **change password** `POST /v1/change_password` with JSON:

```
{
	"old_password": "<string>", // not required in case User.no_password = true
	"new_password": "<string>"
}
```
This call should be authorized (accompanied with a token). Service will respond with 200 in case operation was successful.

To **access** protected endpoints either:

1. Specify `"token"="<string, token value>"` in query parameters
2. Add `Authorization: Bearer <string, token value>` to request headers

In order to **resend** email with user’s email address verification `POST /v1/resend_email_verification`. User has to be logged in in order to use this endpoint. Verification email will be sent to User.email address. Service will respond with 200 in case of success.

### Config

```
{
  	"port": 2016,
  	"session_lifetime": 86400,
  	"log_level": "debug | info | warning",
	"pusher": "http://localhost:2017",
	"client-data": "<URL pointing to client-data service>",
  	"email_from": "<email from which notifications should be sent>",
  	"elastic": {
    	"hosts": ["localhost:9200"]
  	}
}
```
List of vars with default values is as follows:

```
egf_port = 2016
egf_session_lifetime = 86400,
egf_log_level = "debug"
egf_client-data = "http://127.0.0.1:8000/"
egf_pusher = "http://localhost:2017"
egf_email_from = ""
egf_elastic =  { "hosts": ["localhost:9200"] }
```

### Extension Points
None

## file

Service responsible for file management features, e.g. file upload, download, removal, etc

Service creates a new S3 bucket once per month to store uploaded files. Buckets have LIST operation disabled. File names are formed using UUID.

File uploads are partitioned in S3 as follows:

1. There is a root bucket for uploads (configurable)
* A new bucket is created each week inside the root bucket, uploads are stored in this bucket (e.g. egf-uploads/2015-23 where 23 is the number of the week within a year)

This partitioning allows us to split all uploads more or less evenly without the need to create huge amount of buckets.

### Internal Endpoints
I think we have internal

### Public Endpoints

**file** server will handle requests listed below.

In order to **upload** a file to EGF S3 first call `GET /v1/new_image or GET /v1/new_file` passing `"mime_type"`, `"title"` and `"kind"` in query. Request will return File object JSON with a link that should be used for uploading image in `File.upload_url` field.

Client / server interactions:

1. Client sends GET request
* Server creates new File object, sets `"mime_type"` and `"title"`. It prepares a short lived S3 link for file uploading and sets it to `File.upload_url` and also sets `File.url` link.
* Server sets `File.resizes` based on `"kind"` parameter
* Client uploads file using `File.upload_url` link
* Client sets `File.uploaded = true` and updates the object using regular graph PUT.
* Server listens to the `File.uploaded = true` event and
   * Sets `File.url` to a permanent S3 GET link, saves the File object
   * Schedules resizing jobs for all `File.resizes` entries. Jobs will upload resizing results to S3 and update File object

### Config

```
{
	"standalone_ttl": 24, // time to live for stand alone files, in hours; when elapsed and file was not connected to any other object such file will be deleted
	"port": 2018,
  	"auth": "<URL to the auth service>",
	"client-data": "<URL pointing to client-data service>",
  	"s3_bucket": "test_bucket",
	"queue": "kafka | rethinkdb",
	"consumer-group": "pusher",
	"rethinkdb": {
		"host": "localhost",
		"port": "28015",
		"db": "eigengraph",
		"table": "events",
		"offsettable": "event_offset"
	},
	"kafka": {}, // TODO Kafka parameters
	"elastic": { // ElasticSearch parameters. Passed to the ElasticSearch driver without modifications.
        	"hosts": ["localhost:9200"]
      	},
  	"kinds": {
    		"avatar": [
      			{"height": 200, "width": 200},
      			{"height": 300, "width": 400}
    		],
    		"image": [
      			{"height": 200, "width": 200},
      			{"height": 300, "width": 400}
    		]
  	}
}
```

List of vars with default values is as follows:

```
egf_port = 2018
egf_auth = "http://127.0.0.1:2016"
egf_client-data = "http://127.0.0.1:8000/"
egf_queue = rethinkdb
egf_rethinkdb = { "host": "localhost", "port": "28015", "db": "eigengraph", "table": "events", "offsettable": "event_offset" }
egf_kafka = { "hosts": ["localhost:9092"], "client-id": "file", "topic": "events" }, "elastic": { "hosts": ["localhost:9200"] }
egf_elastic = { "hosts": ["localhost:9200"] }
```

### Extension Points
None

## scheduler
This service creates ScheduleEvent objects as specified by Schedule objects. Services can create ScheduleEvent objects and listen to the system object queue to get notified when scheduled actions should be performed.

### Internal Endpoints
None

### Public Endpoint
None

### Config

```
{
	"log_level": "debug | info | warning",
	"client-data": "<URL pointing to client-data service>",
	"elastic": { // ElasticSearch parameters. Passed to the ElasticSearch driver without modifications.
    		"hosts": ["localhost:9200"]
  	}
	"queue": "kafka | rethinkdb",
	"consumer-group": "scheduler",
	"rethinkdb": {
		"host": "localhost",
		"port": "28015",
		"db": "eigengraph",
		"table": "events",
		"offsettable": "event_offset"
	},
	"kafka": {} // TODO Kafka parameters
}
```
### Extension Points
None
