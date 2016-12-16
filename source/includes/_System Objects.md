# System Objects

This part contains description of objects, relations between objects and reusable data structures (shown in *italic*) that are present in all EGF deployments. Objects will be named with bold font, reusable data structures will be named using *italic*.
All objects will have `"created_at"` and `"modified_at"` fields set by server side automatically. All objects will have `"id"` field, not shown here for the brevity sake.

Fields that are not allowed to be changed via user requests will be marked with NE. In case a field can not be set at creation time it will be marked with NC. The same notation will be used for objects as well. Please note that this restriction is only related to user initiated requests. Server side will still be able to set fields and do other changes.

In case API is consumed with the help of EGF2 mobile client libraries the following keywords should not be used as object types, edge and field names:  
"description", "class" and all names that start with "new", "abstract", "assert", "boolean", "break", "byte", "case", "catch", "char", "class", "const", "continue", "default", "do", "double", "else", "enum", "extends", "final", "finally", "float", "for", "goto", "if", "implements", "import", "instanceof", "int", "interface", "long", "native", "new", "package", "private", "protected", "public", "return", "short", "static", "strictfp", "super", "switch", "synchronized", "this", "throw", "throws", "transient", "try", "void", "volatile", "while", "repeat".


## KeyValue

```json
{
	"key": "<string>",
	"value": "<string>"
}
```

<u>Field validations:</u>

<table>
	<thead>
		<tr>
			<th>Field</th>
			<th>Restrictions</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>"key"</td>
			<td>required, string</td>
		</tr>
		<tr>
			<td>"value"</td>
			<td>required, string</td>
		</tr>
	</tbody>
</table>


## HumanName 

```json
{
	"use": "usual | official | temp | nickname | anonymous | old | maiden",
	"family": "<string, surname>",
	"given": "<string, first name>",
	"prefix": "<string>",
	"suffix": "<string>",
	"middle": "<string>"
}
```

<u>Field validations:</u>

<table>
	<thead>
		<tr>
			<th>Field</th>
			<th>Restrictions</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>"use"</td>
			<td>optional, string, value set "usual | official | temp | nickname | anonymous | old | maiden". Defaults to "official".</td>
		</tr>
		<tr>
			<td>"family"</td>
			<td>required, string, length: 1..512</td>
		</tr>
		<tr>
			<td>"given"</td>
			<td>required, string, length: 1..512</td>
		</tr>
		<tr>
			<td>"prefix"</td>
			<td>optional, string, length: 1..64</td>
		</tr>
		<tr>
			<td>"suffix"</td>
			<td>optional, string, length: 1..64</td>
		</tr>
		<tr>
			<td>"middle"</td>
			<td>optional, string, length: 1..64</td>
		</tr>
	</tbody>
</table>


## Event

```json
{
	"object_type": "event",
	"object": "<string, object ID>",
	"user": "<string>",
	"edge": {
		"src": "<string>",
		"dst": "<string>",
		"name": "<string>"
	},
	"method": "<string, POST, PUT, DELETE>",
	"current": {current state of an object or edge},
	"previous": {previous state of an object or edge},
	"crearted_at": <number, unix timestamp>
}
```
Object Code: 04

<u>Field validations:</u>

<table>
	<thead>
		<tr>
			<th>Field</th>
			<th>Restrictions</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>"object_type"</td>
			<td>required, string</td>
		</tr>
		<tr>
			<td>"object"</td>
			<td>optional, either this or "edge" field should be present, object ID</td>
		</tr>
		<tr>
			<td>"user"</td>
			<td>optional, User object ID</td>
		</tr>
		<tr>
			<td>"edge"</td>
			<td>optional, either this or "object" field should be present, struct</td>
		</tr>
		<tr>
			<td>"edge.src"</td>
			<td>required, object ID</td>
		</tr>
		<tr>
			<td>"edge.dst"</td>
			<td>required, object ID</td>
		</tr>
		<tr>
			<td>"edge.name"</td>
			<td>required, string, can be one of a set of values for edge names</td>
		</tr>
		<tr>
			<td>"method"</td>
			<td>required, string, can be "POST", "DELETE", "PUT"</td>
		</tr>
		<tr>
			<td>"previous"</td>
			<td>previous state of an object or edge</td>
		</tr>
		<tr>
			<td>"current"</td>
			<td>current state of an object or edge</td>
		</tr>
	</tbody>
</table>


## Job 

```json
{
	"object_type": "job", NE
	"user": User, NE
	"job_code": "<string>",
	"input": [KeyValue, ...],
	"output": [KeyValue, ...],
	"completed": Boolean,
	"read": Boolean,
	"delete_at": "<string>"
}
```
Object Code: 05

Note: this object is used by "job" service and can be omitted in case "job" service is not deployed.

<u>Field validations:</u>
<table>
	<thead>
		<tr>
			<th>Field</th>
			<th>Restrictions</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>"object_type"</td>
			<td>required, string</td>
		</tr>
		<tr>
			<td>"user"</td>
			<td>required, User object ID</td>
		</tr>
		<tr>
			<td>"job_code"</td>
			<td>required, string</td>
		</tr>
		<tr>
			<td>"input"</td>
			<td>optional, array of KeyValue</td>
		</tr>
		<tr>
			<td>"output"</td>
			<td>optional, array of KeyValue</td>
		</tr>
		<tr>
			<td>"completed"</td>
			<td>optional, Boolean, defaults to false</td>
		</tr>
		<tr>
			<td>"read"</td>
			<td>optional, Boolean, defaults to false</td>
		</tr>
		<tr>
			<td>"delete_at"</td>
			<td>optional, string, date RFC3339</td>
		</tr>
	</tbody>
</table>


## SystemUser

```json
{
       	// TODO: list all system fields
	"service_ids": ["string formatted as <service prefix, e.g. fb, google>:<service user ID>", …]
}
```
Object Code: 01


## Session

```json
{
	"user": "<string>",
	"token": "<string>",
	"expires_at": "<date>"
}
```
Object Code: 02


## User
```json
{
	"object_type": "user",
	"name": HumanName,
	"email": "<string>",
	"system": "<string, SystemUser object ID>",
	"verified": Boolean
}
```
Object Code: 03

<u>Field Validations:</u>
<table>
	<thead>
		<tr>
			<th>Field</th>
			<th>Restrictions</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>"object_type"</td>
			<td>required, string</td>
		</tr>
		<tr>
			<td>"name"</td>
			<td>required, HumanName</td>
		</tr>
		<tr>
			<td>"email"</td>
			<td>required, string, email</td>
		</tr>
		<tr>
			<td>"system"</td>
			<td>optional, SystemUser object ID</td>
		</tr>
		<tr>
        	<td>"verified"</td>
        	<td>optional, Boolean, defaults to false</td>
        </tr>
	</tbody>
</table>

Edges:

1. roles - role objects should be named *Role, for example CustomerRole, AdminRole.

Note: other fields can be added to this object as necessary.


## File

```json
{
	"object_type": "file", NE
	"mime_type": "<string, file type>", NE, NC
	"user": User, NE, NC
	"url": "<URL string>", NE, NC
	"upload_url": "<URL, string>", NE, NC
	"title": "<string>",
	"size": <file size in Kb>, NE, NC
	"dimensions": {
		"height": <int>,
		"width": <int>
	}, NE, NC
	"resizes": [{
		"url": "<URL string>",
		"dimensions": {
			"height": <int>,
			"width": <int>
		}
	}], NE, NC
	"hosted": Boolean, // true in case file is stored on MBP S3, NE, NC
	"uploaded": Boolean,
	"standalone": Boolean, NE, NC
}
```
Object Code: 06

<u>Field validations:</u>
<table>
	<thead>
		<tr>
			<th>Field</th>
			<th>Restrictions</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>"object_type"</td>
			<td>required, string</td>
		</tr>
		<tr>
			<td>"mime_type"</td>
			<td>required, string, we should have a list of file types we want to allow</td>
		</tr>
		<tr>
			<td>"user"</td>
			<td>required, User object ID</td>
		</tr>
		<tr>
			<td>"url"</td>
			<td>required, string, URL format</td>
		</tr>
		<tr>
			<td>"upload_url"</td>
			<td>optional, string, URL format</td>
		</tr>
		<tr>
			<td>"title"</td>
			<td>optional, string, length: 1..256</td>
		</tr>
		<tr>
			<td>"size"</td>
			<td>required if File.hosted = true, int</td>
		</tr>
		<tr>
			<td>"dimensions"</td>
			<td>Struct, required if File.hosted = true and File is an image</td>
		</tr>
		<tr>
			<td>"dimensions.height"</td>
			<td>int, required</td>
		</tr>
		<tr>
			<td>"dimensions.width"</td>
			<td>int, required</td>
		</tr>
		<tr>
			<td>"resizes"</td>
			<td>optional, only available for image files</td>
		</tr>
		<tr>
			<td>"hosted"</td>
			<td>optional, Boolean, defaults to false</td>
		</tr>
		<tr>
			<td>"uploaded"</td>
			<td>optional, Boolean, defaults to false</td>
		</tr>
		<tr>
        	<td>"standalone"</td>
        	<td>optional, Boolean, defaults to false</td>
        </tr>
	</tbody>
</table>

Note: This object is used by "file" service and can be omitted in case "file" service is not 
deployed.


## Schedule

```json
{
"object_type": "schedule", NE
"listener": "<string, service name>", NE
"schedule_code": "<string>",
"time": {"hour": <int>, "minute": <int>, "second": <int>}, NE
"date": {"year": <int>, "month": <int>, "day": <int>, "day_of_week": <int>}, NE
"repeat": "daily | weekly | monthly | yearly", NE
}
```
Object Code: 07

<u>Field validations:</u>
<table>
	<thead>
		<tr>
			<th>Field</th>
			<th>Restrictions</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>"object_type"</td>
			<td>required, string</td>
		</tr>
		<tr>
			<td>"listener"</td>
			<td>required, string</td>
		</tr>
		<tr>
			<td>"schedule_code"</td>
			<td>required, string</td>
		</tr>
		<tr>
			<td>"time"</td>
			<td>required, struct</td>
		</tr>
		<tr>
			<td>"date"</td>
			<td>optional, struct</td>
		</tr>
		<tr>
			<td>"repeat"</td>
			<td>optional, string, value set</td>
		</tr>
	</tbody>
</table>

Note: This object is used by "scheduler" service and can be omitted in case "scheduler" service is not deployed.


## ScheduleEvent

```json
{
	"object_type": "schedule_event", NE
	"schedule": Schedule, NE
	"schedule_code": "<string>",
	"listener": "<string, service name>", NE
}
```

Object Code: 08

<u>Field validations:</u>
<table>
	<thead>
		<tr>
			<th>Field</th>
			<th>Restrictions</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>"object_type"</td>
			<td>required, string</td>
		</tr>
		<tr>
			<td>"schedule"</td>
			<td>required, Schedule object ID</td>
		</tr>
		<tr>
			<td>"schedule_code"</td>
			<td>required, string</td>
		</tr>
		<tr>
			<td>"listener"</td>
			<td>required, string</td>
		</tr>
	</tbody>
</table>

Note: This object is used by "scheduler" service and can be omitted in case "scheduler" service is not deployed.


## SecretOrganization 

```json
{
	"object_type": "secret_organization",
	"secret_keys": [{"key": "<string>", "value": "<string>"}, ...]
}
```
Object Code: 09

<u>Field validations:</u>
<table>
	<thead>
		<tr>
			<th>Field</th>
			<th>Restrictions</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>"object_type"</td>
			<td>required, string</td>
		</tr>
		<tr>
			<td>"secret_keys"</td>
			<td>optional, array of struct</td>
		</tr>
	</tbody>
</table>
   
