# Client Libraries

EGF2 provides a set of client libraries that simplify working with the provided API. Mobile libraries provide the following features:

1. Graph based API methods
* Data caching
* Auth endpoints support
* Search endpoint support
* File operations support
* Model generation


## iOS

### Installation

1. Clone the latest release of [EGF2 from GitHub](https://github.com/EGF2/ios-client).
* Go to your Xcode project’s "General" settings. Drag EGF2.framework to the "Embedded Binaries" section. Make sure "Copy items if needed" is selected and click "Finish".
* Create a new "Run Script Phase" in your app’s target’s "Build Phases" and paste the following snippet in the script text field:
`bash "${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}/EGF2.framework/strip.sh"`.
* Go to your Xcode project’s "Capabilities" settings. Enable "Keychain Sharing".
* For Objective-C project only. Go to your Xcode project’s "Build Settings". Set "Always Embed Swift Standard Libraries" to "Yes".

### Model Generation

EGF2 model generator can be utilized to create model and other classes that simplify work with the EGF2 back-end.

Before generation please prepare a "settings.json" file in some folder with the following content:

```json
{
	"name": "string, your project name, is used to name Core Data file, among other things",
	"server": "string, back-end server URL",
	"model_prefix": "string, prefix that will be used for your models",
	"excluded_models": ["model type name", ...] // array of strings listing models that should be omitted in generation
}
```

Get a `"client-data/config.json"` file from your repository and copy it to the same folder you have your `"settings.json"` file in. Copy EGF2GEN file to the same folder. Run EGF2GEN.

Model generator is capable of producing Objective C and Swift code. Import generated files into your project.

### Error Handling

Almost all methods provided by EGF2 library end with the one of the following common blocks which return either a result of an operation or an error if something went wrong. You should always check if an error has happened to take an appropriate action.

```
ObjectBlock = (NSObject?, NSError?) -> Void
ObjectsBlock = ([NSObject]?, Int, NSError?) -> Void
Completion = (Any?, NSError?) -> Void
```

### Classes and APIs
#### EGF2Graph

EGF2Graph is main class of EGF2 library. It provides methods for authentication and operations on graph objects.

#### EGF2Graph Properties

All properties of main EGF2Graph instance are being set during model generation. While there is no need to edit these properties it is still possible to do so, for example in case URL of the back-end has changed.

<table>
	<thead>
		<tr>
			<th>Property</th>
			<th>Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>var serverURL: URL</td>
			<td>URL of server, for example TODO</td>
		</tr>
		<tr>
			<td>var maxPageSize: Int</td>
			<td>Max size of a page in pagination</td>
		</tr>
		<tr>
			<td>var isObjectPaginationMode: Bool</td>
			<td>true if current pagination mode is object oriented (otherwise index oriented)</td>
		</tr>
		<tr>
			<td>var idsWithModelTypes: [String : NSObject.Type]</td>
			<td>Contains objects' suffixes with appropriate classes</td>
		</tr>
	</tbody>
</table>

#### EGF2Graph Auth Methods

<table>
	<thead>
		<tr>
			<th>Methods</th>
			<th>Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>func register(withFirstName firstName: String, lastName: String, email: String, dateOfBirth: Date, password: String, completion: @escaping Completion)</td>
			<td>Register a new user. As well as ‘login’ method ‘register’ returns auth token, so you don’t need to call ‘login’ after ‘register’.</td>
		</tr>
		<tr>
			<td>func login(withEmail email: String, password: String, completion: @escaping Completion)</td>
			<td>Login a new user.</td>
		</tr>
		<tr>
			<td>func logout(withCompletion completion: @escaping Completion)</td>
			<td>Logout. EGF2 library will clear all user data even if logout with back-end was not successful.</td>
		</tr>
		<tr>
			<td>func change(oldPassword: String, withNewPassword newPassword: String, completion: @escaping Completion)</td>
			<td>Change password of logged in user.</td>
		</tr>
		<tr>
			<td>func restorePassword(withEmail email: String, completion: @escaping Completion)</td>
			<td>Initiate a restoring process. Send a message with a secret token to the specified email.</td>
		</tr>
		<tr>
			<td>func resetPassword(withToken token: String, newPassword: String, completion: @escaping Completion)</td>
			<td>Reset the password of logged in user. User must use the secret token which was sent before.</td>
		</tr>
		<tr>
			<td>verifyEmail(withToken token: String, completion: @escaping Completion)</td>
			<td>Verify an email which was used while registering a new user.</td>
		</tr>
	</tbody>
</table>


#### EGF2Graph Notification Methods

<table>
	<thead>
		<tr>
			<th>Methods</th>
			<th>Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>func notificationObject(forSource source: String) -> Any</td>
			<td>Create a notification object to be used with NotificationCenter in order to listen for changes in the source object.</td>
		</tr>
		<tr>
			<td>func notificationObject(forSource source: String, andEdge edge: String) -> Any</td>
			<td>Create a notification object to be used with NotificationCenter in order to listen for specified edge changes.</td>
		</tr>
	</tbody>
</table>


#### EGF2Graph Graph API Methods

<table>
	<thead>
		<tr>
			<th>Methods</th>
			<th>Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>func object(withId id: String, completion: ObjectBlock?)</td>
			<td>Get object with specific id. If the object has already been cached it will be loaded from cache otherwise it will be loaded from server.</td>
		</tr>
		<tr>
			<td>func object(withId id: String, expand: [String], completion: ObjectBlock?)</td>
			<td>Get object with id and expansion. See more about expansion in Edge Operations section.</td>
		</tr>
		<tr>
			<td>func refreshObject(withId id: String, completion: ObjectBlock?)</td>
			<td>Get object with id from server. In this case cache will not be used for retrieval but will be updated when data has arrived from the server side.</td>
		</tr>
		<tr>
			<td>func refreshObject(withId id: String, expand: [String], completion: ObjectBlock?)</td>
			<td>Get object with id and expansion from server. Retrieved data with all expanded objects will be cached. See more about expansion in Edge Operations section.</td>
		</tr>
		<tr>
			<td>func userObject(withCompletion completion: ObjectBlock?)</td>
			<td>Get User object for currently logged in user.</td>
		</tr>
		<tr>
			<td>func createObject(withParameters parameters: [String : Any], completion: ObjectBlock?)</td>
			<td>Create a new object with specific parameters.</td>
		</tr>
		<tr>
			<td>func updateObject(withId id: String, parameters: [String : Any], completion: ObjectBlock?)</td>
			<td>Update object with id according to values from parameters.</td>
		</tr>
		<tr>
			<td>func updateObject(withId id: String, object: NSObject, completion: ObjectBlock?)</td>
			<td>Update object with specific id according to values from object.</td>
		</tr>
		<tr>
			<td>func deleteObject(withId id: String, completion: Completion?)</td>
			<td>Delete an object with id.</td>
		</tr>
		<tr>
			<td>func createObject(withParameters parameters: [String : Any], forSource source: String, onEdge edge: String, completion: ObjectBlock?)</td>
			<td>Create a new object with specific parameters on a specific edge.</td>
		</tr>
		<tr>
			<td>func addObject(withId id: String, forSource source: String, toEdge edge: String, completion: @escaping Completion)</td>
			<td>Create an edge for an existing object with id.</td>
		</tr>
		<tr>
			<td>func deleteObject(withId id: String, forSource source: String, fromEdge edge: String, completion: @escaping Completion)</td>
			<td>Delete an object with id from an edge.</td>
		</tr>
		<tr>
			<td>func doesObject(withId id: String, existForSource source: String, onEdge edge: String, completion: @escaping (Bool, NSError?) -> Swift.Void)</td>
			<td>Check if an object with id exists on a specific edge.</td>
		</tr>
		<tr>
			<td>func objects(forSource source: String, edge: String, completion: ObjectsBlock?)<br>
			func objects(forSource source: String, edge: String, after: String?, completion: ObjectsBlock?)<br>
			func objects(forSource source: String, edge: String, after: String?, expand: [String], completion: ObjectsBlock?)<br>
			func objects(forSource source: String, edge: String, after: String?, expand: [String], count: Int, completion: ObjectsBlock?)</td>
			<td>Get edge objects. If edge data was cached it will be loaded from cache otherwise it will be loaded from server.</td>
		</tr>
		<tr>
			<td>func refreshObjects(forSource source: String, edge: String, completion: ObjectsBlock?)<br>
			func refreshObjects(forSource source: String, edge: String, after: String?, completion: ObjectsBlock?)<br>
			func refreshObjects(forSource source: String, edge: String, after: String?, expand: [String], completion: ObjectsBlock?)<br>
			func refreshObjects(forSource source: String, edge: String, after: String?, expand: [String], count: Int, completion: ObjectsBlock?)</td>
			<td>Get edge objects from back-end, bypassing the cache. Retrieved data will be cached.</td>
		</tr>
		<tr>
			<td>func uploadFile(withData data: Data, title: String, mimeType: String, completion: @escaping ObjectBlock)</td>
			<td>Upload file data to server.</td>
		</tr>
		<tr>
			<td>func uploadImage(withData data: Data, title: String, mimeType: String, kind: String, completion: @escaping ObjectBlock)</td>
			<td>Upload image data to server.</td>
		</tr>
		<tr>
			<td>func search(forObject object: String, after: Int, count: Int, expand: [String]? = default, fields: [String]? = default, filters: [String : Any]? = default, range: [String : Any]? = default, sort: [String]? = default, query: String? = default, completion: @escaping ObjectsBlock)<br>
			func search(withParameters parameters: EGF2SearchParameters, after: Int, count: Int, completion: @escaping ObjectsBlock)</td>
			<td>Search for specific objects according to parameters.</td>
		</tr>
	</tbody>
</table>


#### Expand Objects

EGF2 uses graph oriented approach to modeling data. In practice it means that data is represented as objects and edges, where edges are connections between objects. It is often convenient to be able to not only get an object or an edge from the back-end but get expanded portion of a graph. For example, when we get a list of favorite Posts for some user it is beneficial to get Post authors as well. We may also want to get Post comments in the same request. Expansion feature allows us to do this.


Relations between objects in EGF2 are modelled using two concepts:

1. An edge is a list of objects related to this one
* An object property is a connection from this object to another


Every object property is modelled with two properties within library object, one holds string object ID, and another holds a reference to a connected library object.


For example:

```js
class Post: GraphObject {
	var text: String?
	var image: String? // id of image object
}

class Post: GraphObject {
	var text: String?
	var image: String?		// id of image object
	var imageObject: File?	// imageObject is instance of image object
	var creator: String?		// id of user object
	var creatorObject: User?	// creatorObject is instance of user object
}
```

So if you want to get just a post object you use the code below:

```js
Graph.object(withId: "<post id>") { (object, error) in
    guard let post = object as? Post else { return }
    // post.image contains id of image object
    // post.imageObject is nil
}
```

But if you also want to get an image object you use the code below:

```js
Graph.object(withId: "<post id>", expand: ["image"]) { (object, error) in
    guard let post = object as? Post else { return }
    // now post.imageObject contains instance of image object
}
```

It is possible to expand several object properties of an object at once:

```js
Graph.object(withId: "<post id>", expand: ["image","creator"]) { (object, error) in
    guard let post = object as? Post else { return }
    // post.imageObject contains image object
    // post.userObject contains user object who has created the post
}
```

You can also specify multi level expand:

```js
Graph.object(withId: "<post id>", expand: ["image{user}"]) { (object, error) in
    guard let post = object as? Post else { return }
    // post.imageObject contains image object
    // post.imageObject.userObject contains user object who has uploaded the image
}
```

Edges are also expandabe. It’s useful when you need to get data from an edge in advance:

```js
Graph.object(withId: "<post id>", expand: ["comments"]) { (object, error) in
    // If there are objects on comments edge they will be downloaded and cached
    // Later you can get them using ‘objects’ method
}
```

You can specify how many objects should be taken while expanding:

```js
Graph.object(withId: "<post id>", expand: ["comments(5)"]) { (object, error) in
    // The first five comments will be taken from cache or downloaded from server
}
```

This feature also works for edges:

```js
Graph.objects(forSource: "<user id>", edge: "posts", after:nil, expand:["image"]) { (objects, count, error) in
    guard let posts = objects as? [Post] else { return }
    // use posts
}
```

Using expand you can get all information you need in one request:

```js
// You want to get all newest posts
// Also you want to know who created these posts (creator) and what image you should show
// Besides you want to show the last comment under each post
let expand = ["image","creator","comments(1)"]
Graph.objects(forSource: "<user id>", edge: "posts", after:nil, expand:expand) { (objects, count, error) in
    guard let posts = objects as? [Post] else { return }
    // use posts
}
```

### Auxiliary NSObject methods

EGF2 iOS framework provides an extension for NSObject class which contains several useful methods for working with graph objects

<table>
	<thead>
		<tr>
			<th>Methods</th>
			<th>Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>func copyGraphObject() -> Self</td>
			<td>Make a copy of graph object.</td>
		</tr>
		<tr>
			<td>func isEqual(graphObject: NSObject) -> Bool</td>
			<td>Checking if object is equal to another graph object.</td>
		</tr>
		<tr>
			<td>func changesFrom(graphObject: NSObject) -> [String : Any]?</td>
			<td>Get a dictionary of changed fields in comparison with another object.</td>
		</tr>
		<tr>
			<td>var idsWithModelTypes: [String : NSObject.Type]</td>
			<td>Contains objects' suffixes with appropriate classes</td>
		</tr>
	</tbody>
</table>

###### Example

```js
let currentUser: User = … // get user object where “name.given == Mark”
let changedUser: User = currentUser.copyGraphObject() // a copy of current user object
changedUser.name?.given = "Tom"


if currentUser.isEqual(graphObject: changedUser) {
    print("Objects are equal")
}
else {
    print("Objects are different")
}
var dictionary = currentUser.changesFrom(graphObject: changedUser)!
print(dictionary)  // ["name": ["given": "Mark"]]
or
var dictionary = changedUser.changesFrom(graphObject: currentUser)!
print(dictionary)  // ["name": ["given": "Tom"]]
```

You can use objects and dictionary while working with graph objects:

```js
Graph.updateObject(withId: "<user id>", parameters: dictionary) { (_, error) in
}
or
Graph.updateObject(withId: "<user id>", object: changedUser) { (_, error) in
}
```

### Saving Data

By default all changes are being saved every time your app receives the following notifications:

1. UIApplicationDidEnterBackground
2. UIApplicationWillTerminate

So if your app suddenly crashes all unsaved changes will be lost.

### Notification

EGF2 library posts the following notifications when appropriate actions happen:

<table>
	<thead>
		<tr>
			<th>Objective-C notification names</th>
			<th>Swift notification names</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>EGF2NotificationEdgeCreated</td>
			<td>EGF2EdgeCreated</td>
		</tr>
		<tr>
			<td>EGF2NotificationEdgeRemoved</td>
			<td>EGF2EdgeRemoved</td>
		</tr>
		<tr>
			<td>EGF2NotificationEdgeRefreshed</td>
			<td>EGF2EdgeRefreshed</td>
		</tr>
		<tr>
			<td>EGF2NotificationEdgePageLoaded</td>
			<td>EGF2EdgePageLoaded</td>
		</tr>
		<tr>
			<td>EGF2NotificationObjectCreated</td>
			<td>EGF2ObjectCreated</td>
		</tr>
		<tr>
			<td>EGF2NotificationObjectUpdated</td>
			<td>EGF2ObjectUpdated</td>
		</tr>
		<tr>
			<td>EGF2NotificationObjectDeleted</td>
			<td>EGF2ObjectDeleted</td>
		</tr>
	</tbody>
</table>

<table>
	<thead>
		<tr>
			<th>Notification</th>
			<th>Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>EdgeCreated</td>
			<td>A new edge was created</td>
		</tr>
		<tr>
			<td>EdgeRemoved</td>
			<td>An existing edge was removed</td>
		</tr>
		<tr>
			<td>EdgeRefreshed</td>
			<td>An edge was refreshed. All cached pages were dropped, the first page downloaded and cached.</td>
		</tr>
		<tr>
			<td>EdgePageLoaded</td>
			<td>The next page for some specific edge has been loaded.</td>
		</tr>
		<tr>
			<td>ObjectCreated</td>
			<td>The object has been created</td>
		</tr>
		<tr>
			<td>ObjectUpdated</td>
			<td>The object has been updated</td>
		</tr>
		<tr>
			<td>ObjectDeleted</td>
			<td>The object has been deleted</td>
		</tr>
	</tbody>
</table>

UserInfo object of each notification may contain different objects which can be obtained using the following keys:

<table>
	<thead>
		<tr>
			<th>Key name</th>
			<th>Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>EGF2EdgeInfoKey</td>
			<td>Name of edge</td>
		</tr>
		<tr>
			<td>EGF2SourceInfoKey</td>
			<td>Id of object</td>
		</tr>
		<tr>
			<td>EGF2ObjectInfoKey</td>
			<td>Current object</td>
		</tr>
		<tr>
			<td>EGF2EdgeObjectInfoKey</td>
			<td>Object on edge</td>
		</tr>
		<tr>
			<td>EGF2EdgeObjectsInfoKey</td>
			<td>Objects on edge</td>
		</tr>
		<tr>
			<td>EGF2EdgeObjectsCountInfoKey</td>
			<td>Count of objects on edge</td>
		</tr>
	</tbody>
</table>

###### Example

```js
// Get notification object for specific graph object
let object = Graph.notificationObject(forSource: id)

// Want to get notifications only for an appropriate graph object
// if object == nil then we will get notifications for all updated objects
NotificationCenter.default.addObserver(self, selector: #selector(didUpdateObject(notification:)), name: .EGF2ObjectUpdated, object: object)

…

func didUpdateObject(notification: NSNotification) {
    guard let user = notification.userInfo?[EGF2ObjectInfoKey] as? User else { return }
    // user - the updated graph object
}
```
