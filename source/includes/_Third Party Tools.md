# Third Party Tools

## Spark Streaming

Code will have to be prepared that will read event stream from a queue solution and update master datastore as well as all batch views incrementally. Within batch in Spark Streaming we may want to attempt at restoring proper event ordering based on event timestamp.

## Spark Batch

This layer will contain all batch jobs we may have, including but not limited to:

- Full seed jobs necessary to populate all batch views
- Statistics jobs
- ML related jobs
- etc


## Cache

Caching solution is utilized by client-data.

## Data Store

We currently support RethinkDB, need to support Cassandra.

## RethinkDB Data Structures
Objects table holds all objects documents. Each document contains `"id"`, `"object_type"`, `"created_at"`, `"modified_at"` and `"deleted_at"` system fields. RethinkDB document example:

```
{
	id: "<UUID with object code suffix>",
	object_type: "<string, object type>",
	created_at: "<date>",
	modified_at: "<date>",
	...
}
```

Edges table has the next structure:

```
{
	id: "<source object ID>_<edge name>_<destination object ID>",
	src: "<source object ID>",
	edge_name: "<string, edge name>",
	dst: "<destination object ID>",
	sort_by: "date, string date created",
	created_at: "<date>"
}
```
`edge_sorting` index contains `[src, edge_name, sort_by, dst]`.


Event table contains events for objects and edges.

```
{
	id: "<timebased UUID with event object suffix>"
	// either object or edge fields must be filled
	object: "<optional, object id associated with the event>",
	edge: {
		src: "<string, source object id>",
		dst: "<string, destination object id>",
		name: "<string, edge name>"
	}, // optional, edge identificator associated with the event
	method: "<string, "POST", "PUT" or "DELETE" value>",
	previous: {
		<optional, object or edge body for previous state>
	},
	current: {
		<optional, object or edge body for current state>
	},
	created_at: "<date>"
}
```

## Cassandra Data Structures

### Storing Objects with Cassandra

```sql
CREATE TABLE objects (
    id text,
    type text,
    fields map<text, text>,
    PRIMARY KEY (type, id)
);
```
### Storing Edges with Cassandra

Edges are stored in the following table:

```sql
CREATE TABLE edges (
    src text,
    name text,
    dst text,
    sort_value text,
    PRIMARY KEY ((name, src), sort_value, dst)
)
WITH CLUSTERING ORDER BY (sort_value DESC);
CREATE INDEX ON edges (dst);
```

Notes:

1. `"sort_value"` contains edge creation date, is sorted in DESC order
* We don’t need an index on `"sort_value"` because we have it in the clustering fields

### Storing Events with Cassandra

```sql
CREATE TABLE events (
    id text,
    object_type text,
    method text,
    object text,
    edge: {
	src: text,
	name: text,
	dst: text
    },
    current map<text, text>,
    previous map<text, text>,
    created_at bigint,
    PRIMARY KEY (id)
);
```


