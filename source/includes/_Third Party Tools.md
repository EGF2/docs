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

Data store is responsible for persistently and reliably storing and retrieving all system data.


