# Micro Services

We will list all services that are present in the framework. For each service we will show public and internal endpoints, config parameters and extension points. Not all of the services will have something in all of the subsections. By "extension points" we mean an intended way of extending framework functionality. For example, all **logic** rules will be provided by clients using extension point. Internal endpoints are to be used by services, they are not exposed to the Internet, public endpoints are exposed to the Internet.

## Client-data

This service will serve as a focal point for data exchange in the system. As all events in the system will have to go through client-data it should be fast, simple and horizontally scalable. It should be able to support all graph operations from either cache or data store. 

### Internal Endpoints

`GET /v1/graph` - return graph config in JSON.

`GET /v1/graph/<object_id>` - to get an object by ID, returns object JSON

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

<style type="text/css">
   tab { 
	margin-left: 30px;
       }
   tab2 { 
	margin-left: 60px;
        }
   tab3 { 
	margin-left: 90px;
        }
   tab4 { 
	margin-left: 120px;
        }
   tab5 { 
	margin-left: 150px;
        }
</style>

`{`</br>
<tab>`"port": 8000,`  // port that service will listen to </br>
<tab>`"log_level": "debug | info | warning",` </br>
<tab>`"storage": "rethinkdb | cassandra"` // storage system, currently supported are RethinkDB and Cassandra </br>
<tab>`"cassandra": {` // Cassandra config, should be present in case "storage" is equal to "cassandra". For more info on supported parameters please see [Cassandra driver config specification](https://www.npmjs.com/package/cassandra-driver). Parameters specified here are passed to the driver as is, without modifications.</br>
	<tab2>`"contactPoints": ["localhost"], `</br>
	<tab2>`"keyspace": "eigengraph"`</br>
<tab>`},`</br>
<tab>`"rethinkdb": {` // RethinkDB config, should be present in case "storage" is equal to "rethinkdb". For more info on supported parameters please see [RethinkDBDash driver docs](https://www.npmjs.com/package/rethinkdbdash). Parameters specified here are passed to the driver as is, without modifications.</br>
	<tab2>`"host": "localhost",`</br>
	<tab2>`"db": "eigengraph"`</br>
<tab>`}`</br>
<tab>`"graph": {` // graph config section, contains info on all objects, edges, ACLs and validations in the system</br>
	<tab2>`"custom_schemas": {`</br>
		<tab3>`"address": { `</br>
      			<tab4>`"use": { "type": "string", "enum": ["official", "shipping", "billing", "additional"], "default": "additional" },`</br>
      			<tab4>`"label": { "type": "string" },`</br>
      			<tab4>`"lines": { "type": "array:string", "min": "1" },`</br>
      			<tab4>`"city": { "type": "string" },`</br>
      			<tab4>`"state": { "type": "string" },`</br>
      			<tab4>`"country": { "type": "string" }`</br>
    		<tab3>`},`</br>
    		<tab3>`"human_name": {`</br>
      			<tab4>`"use": { "type": "string", "enum": ["usual", "official", "temp", "nickname", "anonymous", "old", "maiden"], "default": "official" },`</br>
      			<tab4>`"family": { "type": "string", "min": 1, "max": 512, "required": true },`</br>
      			<tab4>`"given": { "type": "string", "min": 1, "max": 512, "required": true },`</br>
      			<tab4>`"middle": { "type": "string", "min": 1, "max": 64 }`</br>
    		<tab3>`}`
  	<tab2>`},`</br>
 	<tab2>`"common_fields": { `</br>
    		<tab3>`"object_type": { "type": "string", "required": true },`</br>
    		<tab3>`"created_at": { "type": "date", "required": true },`</br>
    		<tab3>`"modified_at": { "type": "date", "required": true },`</br>
    		<tab3>`"deleted_at": { "type": "date" }`</br>
  	<tab2>`},`</br>
	<tab2>`"system_user": {`</br>
		<tab3>`"code": "01",` // object type code </br>
		<tab3>`"suppress_event": true` // in case "no_event" is present and set to true **client-data** will not produce any events related to this object </br>
		<tab3>`"validations": {` // for more info on validations please see [Validations](#field-validations) subsection</br>
		<tab3>`}`</br>
	<tab2>`},`</br>
	<tab2>`"session": {`</br>
		<tab3>`"code": "02",`</br>
        	<tab3>`"suppress_event": true`</br>
    	<tab2>`},`</br>
    	<tab2>`"user": {`</br>
        	<tab3>`"code": "03",`</br>
        	<tab3>`"edges": {` // edges allowed for this object type</br>
            		<tab4>`"roles": {` // here "roles" is edge name</br>
               			<tab5>`"contains": [""]` // array of strings with object type names that are allowed to be added to this edge</br>
            		<tab4>`}`</br>
        	<tab3>`},`</br>
		<tab3>`"fields": {`</br>
			<tab4>`"name": {`</br>
				<tab5>`"type": "struct",`</br>
				<tab5>`"schema": "human_name",`</br>
				<tab5>`"required": true,`</br>
				<tab5>`"edit_mode": "E",` // this field can take values: "NC | NE | E", NC means that this field can not be set at creation time, NE means that this field can not be changed, E means field can be edited </br>
				<tab5>`"default": "",`</br>
				<tab5>`"auto_value": ""`</br>
			<tab4>`}`</br>
		<tab3>`}`</br>
    	<tab2>`},`</br>
    	<tab2>`"event": {`</br>
        	<tab3>`"code": "04"`</br>
    	<tab2>`},`</br>
    	<tab2>`"job": {`</br>
        	<tab3>`"code": "05"`</br>
    	<tab2>`},`</br>
    	<tab2>`"file": {`</br>
        	<tab3>`"code": "06"`</br>
    	<tab2>`},`</br>
    	<tab2>`"schedule": {`</br>
        	<tab3>`"code": "07"`</br>
    	<tab2>`},`</br>
    	<tab2>`"schedule_event": {`</br>
        	<tab3>`"code": "08"`</br>
    	<tab2>`}`</br>
  <tab>`}`</br>
`}`

#### Field Validations

Object field validations can be added to field declarations inside of an object declaration as follows:

`"fields": {`</br>
  <tab>`"name": { "type": "struct", "schema": "human_name", "required": true },`</br>
`},`</br>


Another sample is for a hypothetical Account object:

`"fields": {`</br>
  <tab>`"user": { "type": "object_id:user" },`</br>
  <tab>`"balance": { "type": "string", "required": true },`</br>
  <tab>`"credit": { "type": "number", "default": 0 },`</br>
  <tab>`"tax_collected": { "type": "number", "default": 0 }`</br>
`}`</br>

Field validations are provided in the "custom_schemas", "common_fields" fields inside of "graph" section. "custom_schemas" section contains validations for common reusable data structures. "common_fields" section provides validations for fields that can be present in all object types. Validations for particular objects are stored in field declarations.

Validation specification for a particular field can contain the following fields:

1. "type" - required, can be one of the following strings: "string", "number", "date", "boolean", "object_id", "struct" and "array"
* "required" - optional boolean field, defaults to false
* "default" - optional field that contains default value for the field
* "enum" - optional array of strings. If present only values from this array can be assigned to the field
* "min" and "max" - optional fields that can be present for "string", "array" and "number" typed fields. For "string" these fields restrict string length. For "array" these fields restrict array size, for "number" - min and max values that can be stored.
* "schema" - required for fields with type "struct" or "array:struct". Contains nested field validation specification for the struct. Can also contain a name of one of declared custom schemas (section "custom_schemas")
* "edit_mode" - can take values "NC", "NE", "E". "NC" means field can not be set at creation time. "NE" means that field can not be changed. "E" means that the object is editable. In case "edit_mode" is not present it means that the field can’t be set at creation and also is not editable by users.

For fields with type "object_id" additional declaration of what types of objects are allowed to be referenced in this field can be added. For example: `"foo": { "type": "object_id:user,account" }` means that *foo* field must be object_id of User or Account objects.
`"bar" : {"type": "object_id"}` means that *bar* field should be object_id of any object.

Fields with type "array" should have additional declaration for the type of entities that can be stored within this array. Any type can be used in the declaration, except for "array". For example `"spam": {"type": "array:string"}` means that spam is an array of strings.

#### ACL

ACL related fields can be added on object definition and edge definition levels in "graph" section.

`"some_object": {`</br>
  <tab>`"GET": "<optional, comma separated ACL rules for GET method>",`</br>
  <tab>`"POST": "<optional, comma separated ACL rules for POST method>",`</br>
  <tab>`"PUT": "<optional, comma separated ACL rules for PUT method>",`</br>
  <tab>`"DELETE": "<optional, comma separated ACL rules for DELETE method>",`</br>
  <tab>`"edges": {`</br>
	<tab2>`"<edge name>": {`</br>
		<tab3>`"GET": "<optional, comma separated ACL rules for GET method>",`</br>
		<tab3>`"POST": "<optional, comma separated ACL rules for POST /v1/graph/<src>/<edge_name> with new destination object body>",`</br>
		<tab3>`"LINK": "<optional, comma separated ACL rules for POST /v1/graph/<src>/<edge_name>/<dst> (create edge for existing objects)>",`</br>
		<tab3>`"DELETE": "<optional, comma separated ACL rules for DELETE method>"`</br>
	<tab2>`},`</br>
	<tab2>`...`</br>
  <tab>`}`</br>
`}`</br>

Supported ACL rules:

- any - allow access to any request.
- registered_user - allow access to registered users.
- self - allow access to own objects and edges - objects that have field “user” pointing to the current user.
- \<role based access rule> - this rule is specified as a role name that user should have on User/roles edge. Example: CustomerRole, AdminRole, etc
- TODO: we need to provide info here on how clients will be able to specify custom ACL rules

### Extension Points

#### Custom validation handler

In case built-in validations are not sufficient client can implement custom validation logic for any field of any object. For example, email field may require custom validation.


In order to add custom validation please do the following:

1. Implement a module that will perform custom validation inside of "controllers/validation/" folder. This module should implement one or several functions with signature: `function (val)` {} where *val* is the value of the field. Function should return true in case validation succeeded and false otherwise. Function (or functions, in case single module implements multiple validators) should be exported from the module.
* Add a record of the form `"<type_name>": require("./<module_file_name>").<optional, function name in case module exports multiple functions>` to the *validationRegistry* map in "validation/index.js"
* In **client-data** config, "graph" section, specify \<type_name> in field validation declaration in “type” field.

Example of a custom validator for `"email"` field: 

`function CheckEmail(val) {`</br>
  <tab>`let re = new RegExp("^\S+@\S+$");`</br>
  <tab>`return re.test(val);`</br>
`}`</br>

Corresponding *validationRegistry* map will be:

`const validationRegistry = {`</br>
  <tab>`email: require(“./check_email”)`</br>
`}`</br>

And object validations will look like: 

`"validations": {`</br>
  <tab>`"user_email": { "type": "email" },`</br>
`}`</br>




