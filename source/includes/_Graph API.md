# Graph API

EGF will provide a graph based API (similar to Facebook) to facilitate web and mobile clients:

1. `GET`, `POST`, `PUT`, `DELETE` `/v1/graph/<object ID>` to work with objects
* `GET /v1/graph/<object ID>/<edge name>` to get paginated list of object’s edges
* `POST /v1/graph/<object ID 1>/<edge name>/<object ID 2>` to create new edge between existing objects
* `POST /v1/graph/<object ID 1>/<edge name>` JSON body of target object to create new target object and an edge in one request
* `DELETE /v1/graph/<object ID 1>/<edge name>/<object ID 2>` to delete an edge
* `GET /v1/graph/me` will return User object for currently authenticated user.
* `GET /v1/graph/<object ID>/<edge name>/<object ID 2>` to get object if edge exists.

Most client app needs will be covered by Graph API endpoint. In case a separate endpoint is required it will be documented explicitly (e.g. auth related endpoints).


When an object or an edge is requested via GET client apps can use "expand" option. Expand is a comma separated list of fields and edge names which has the next format: `expand=<field1>{<nested field2>,<edge 2>},<edge1>(<count>){<field3>}`.

- `<field|edge>{...}` - next level expand (max: 4).
- `<edge>(<count>)` - number of objects in returned page.


