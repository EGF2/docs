# Pagination

Graph API and other endpoints may return a set of results. In this case pagination will be provided with the help of the following query parameters:

- "after" – return results after an index
- "count" – return this number of objects, defaults to 25

Endpoints that return paginated results will use the following JSON wrapper:

<style type="text/css">
   tab { 
	margin-left: 30px;
       }
</style>

    `{`<br>
        <tab>`"results": [],`<br>
       	<tab>`"first": <index of the first entry, int>,` // this field will not be populated if this is the first page<br>
        <tab>`"last": <index of the last entry, int>,` // this field will not be populated in case this page is the last one<br>
        <tab>`"count":<int, total number of objects>`<br>
    `}`
