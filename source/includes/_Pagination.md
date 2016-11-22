# Pagination

Graph API and other endpoints may return a set of results. In this case pagination will be provided with the help of the following query parameters:

- `"after"` – return results after an index
- `"count"` – return this number of objects, defaults to 25

Endpoints that return paginated results will use the following JSON wrapper:

```
{
       	"results": [],
       	"first": "<index or object ID of the first entry, string>",
       	"last": "<index or object ID of the last entry, string>", // this field will not be populated in case this page is the last one
       	"count": <total number of objects, int>
}
```
