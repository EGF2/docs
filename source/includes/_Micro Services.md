# Micro Services

We will list all services that are present in the framework. For each service we will show public and internal endpoints, config parameters and extension points. Not all of the services will have something in all of the subsections. By “extension points” we mean an intended way of extending framework functionality. For example, all **logic** rules will be provided by clients using extension point. Internal endpoints are to be used by services, they are not exposed to the Internet, public endpoints are exposed to the Internet.

## Client-data

This service will serve as a focal point for data exchange in the system. As all events in the system will have to go through client-data it should be fast, simple and horizontally scalable. It should be able to support all graph operations from either cache or data store. 

### Internal Endpoints

`GET /v1/graph` - return graph config in JSON.

`GET /v1/graph/<object_id>` - to get an object by ID, returns object JSON

`GET /v1/graph/<object_id1>,<object_id2>, …` - to get several objects at once, returns JSON structured in the same way as paginated result in graph API.

To create a new object use `POST /v1/graph` with JSON body of the new object. Newly created object will be returned as a response, along with `“id”` and `“created_at”` fields set automatically. Object IDs are generated in the format: `“<UUID>-<object type code>”`.

To update an object use `PUT /v1/graph/<object_id>` with JSON body containing modified fields. In order to remove a field use `“delete_fields”: [“<field_name1>”, “<field_name2>”, ...]` field in the JSON body. Updated object will be returned.

To delete an object use `DELETE /v1/graph/<object_id>`. Objects are not deleted from the data store, `“deleted_at”` field is added instead. Returns the following JSON: `{“deleted_at”: “<date and time, RFC3339>”}`.

In order to get objects that listed on an edge use `GET /v1/graph/<src_object_id>/<edge_name>` with pagination parameters. Extension is not supported by **client-data** service but is supported by the “client-data” commons utility.

In order to check for an existence of an edge use `GET /v1/graph/<src_object_id>/<edge_name>/<dst_object_id>`. Service will return an object if the edge exists. 404 will be returned otherwise.

To create a new edge use `POST /v1/graph/<src_object_id>/<edge_name>/<dst_object_id>`. Returns JSON: `{“created_at”: “<date and time of creation for this edge>”}`.

To delete an edge use `DELETE /v1/graph/<src_object_id>/<edge_name>/<dst_object_id>`. Returns the following JSON: `{“deleted_at”: “<date and time of deletion, RFC3339>”}`.

### Public Endpoints

None

###

