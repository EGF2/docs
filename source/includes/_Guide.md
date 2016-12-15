# Guide

In this tutorial we will build a system pretty similar in functionality to instagram. The following features will be implemented:

* User registration, email verification, login and password reset.
* Every user will be able to upload an image with a text, we will call it a post. Creator of the post will be able to edit and delete it.
* Registered users will be able to comment on other users' posts. Comment creator will be able to edit and remove her/his comment.
* It will be possible for any user to see other users' collection of posts.
* Registered users will be able to follow other users
* User will be able to see a timeline - a collection of posts created by users that this user follows, arranged by time of creation.
* Any user will be able to search for other users and posts using free text query
* Registered user will be able to mark a post as offensive
* Admin users will be able to see a list of posts marked as offensive and delete such posts

While pretty compact, this system will allow us to work with the following aspects of the EGF2:

* User authentication
* Different user roles
* File operations
* Email notifications
* Business logic rules
* Domain modelling

We will start with domain modelling, proceed with EGF2 deployment and necessary back-end changes and will create web, iOS and Android apps that will work with the system.

Full implementation of the back-end, iOS and Android apps are [available at GitHub](https://github.com/egf2-guide)

## Domain Modeling

I think it makes sense to start with a User model and proceed from there.

EGF2 comes with a `User` object pre-defined. We can expand it with more fields and edges if necessary, but we should not remove it unless we don't want to expose any user related APIs.

Here is how it looks, in pseudo JSON:

```json
User
{
    "object_type": "user",
    "name": HumanName,
    "email": "<string>",
    "system": "<string, SystemUser object ID>",
    "verified": Boolean
}
```

I think it is enough for a start. Looking at the specification of features it is clear that we need to support the following user roles:

* Regular user, let's call the role `CustomerRole`
* Admin user - `AdminRole`

Let's define the roles.

```json
CustomerRole
{
    "object_type": "customer_role",
    "user": User
}
```

```json
AdminRole
{
    "object_type": "admin_role",
    "user": User
}
```

User's roles should be stored using `roles` edge of a `User` object. We usually refer to an edge using a simple notation `<Object type>/<edge name>`, for example `User/roles`. ACL support assumes that roles are store using this edge. We will use ACL in order to control access to objects and edges so we will use `User/roles` edge to store user's roles.

While we are at the user's part of it let's also add edges for the following bit - `User/followers` and `User/follows`.

That's enough for the users part, let's get to the meat of the system - Posts.

```json
Post
{
    "object_type": "post",
    "creator": User,
    "image": File,
    "description": "<string>"
}
```

As you can see, we specify `image` property as a `File`. This object type is supported by EGF2 out of the box.

Users create posts, so we need an edge to store a list of posts created by a `User`. Edge `User/posts` should do just fine.

We will use `AdminRole/offending_post` edge to keep track of posts that were found offensive. `Post/offended` edge will hold a list of users that found this Post offensive.

Registered users can post comments for posts. Comment can be modelled as:

```json
Comment
{
    "object_type": "comment",
    "creator": User,
    "post": Post,
    "text": "<string>"
}
```

We will connect `Posts` to `Comments` using `Post/comments` edge.

One more thing. We need to decide how user's timeline should behave. There are at least two ways we can approach this:

* For a particular user we can calculate timeline dynamically based on the list of users this particular user follows. This is feasible in case a single user follows not a large number of other users. If we get in to hundreds of follows this approach will be problematic due to performance issues.
* We can add an edge `User/timeline` and create an edge every time a user that is being followed creates a `Post`. This approach will work well for GET requests but will require resources in case we get into situation when there are users with large number of followers.

We will take the second route as it scales better. There are a couple of consequences of this decision:

* Timeline will only show posts that were added after a user has started following another user
* Old posts will not be removed from the timeline when a user stops following another user.

Both consequences can be avoided at the cost of additional processing, but I don't think it is really necessary from the business logic standpoint.

Info about object and edges can (and I think should) be summarised in Model section of the system documentation, for more info see [Suggested Documentation Format](http://doc.eigengraph.com/#suggested-documentation-format) section of the EGF2 documentation.

That's it for domain modelling, at least for now. In the next section I will show you how what needs to be done to implement this model with EGF2.


## Configuring Objects and Edges

In the previous section I outlined what objects and edges we need to have in the system. Now we are ready to actually add them!

Open `config.json` file from `client-data` service. First we need to adjust `User` object declaration. We have added some edges so we need to let the system know about them. After the changes `User` object declaration should look like:

```json
    "user": {
        "code": "03",
        "GET": "self",
        "PUT": "self",
        "fields": {
            "name": {
                "type": "struct",
                "schema": "human_name",
                "required": true,
                "edit_mode": "E"
            },
            "email": {
                "type": "string",
                "validator": "email",
                "required": true,
                "unique": true
            },
            "system": {
                "type": "object_id",
                "object_types": ["system_user"],
                "required": true
            },
            "verified": {
                "type": "boolean",
                "default": false
            },
            "date_of_birth": {
                "type": "string",
                "edit_mode": "E"
            }
        },
        "edges": {
            "roles": {
                "contains": ["customer_role", "admin_role"],
                "GET": "self"
            },
            "posts": {
                "contains": ["post"],
                "GET": "self",
                "POST": "self",
                "DELETE": "self"
            },
            "timeline": {
                "contains": ["post"],
                "GET": "self"
            },
            "follows": {
                "contains": ["user"],
                "GET": "self",
                "POST": "self",
                "DELETE": "self"
            },
            "followers": {
                "contains": ["user"],
                "GET": "self"
            }
        }
    }
```

As you can see if you compare original `User` declaration with the changed version I made the following changes:

* In `"edges"` section for `"roles"` edge I specified what target objects this edge can take. We know what roles we have in the system now.
* Edge declarations for `posts`, `timeline`, `follows` and `followers` edges were added. As you can see, edges `posts` and `follows` are fully editable by users who own them (edge that have authorised user as a source are owned by this user). Edges `followers` and `timeline` are read-only - the system will populate them automatically as a result of other users' actions.

With `User` object out of the way we can start adding objects that are totally new to the system. Let's start with Roles objects:

```json
    "customer_role": {
        "code": "10",
        "GET": "self",
        "fields": {
            "user": {
                "type": "object_id",
                "object_types": ["user"],
                "required": true
            }
        }
    }

    "admin_role": {
        "code": "11",
        "GET": "self",
        "fields": {
            "user": {
                "type": "object_id",
                "object_types": ["user"],
                "required": true
            }
        },
        "edges": {
            "offending_posts": {
                "contains": ["post"],
                "GET": "self",
                "DELETE": "self"
            }
        }
    }
```

These two objects are pretty straightforward, the only thing of note here is that `AdminRole/offending_posts` edge is readable and deletable by an admin, but admins can't create them directly. The system will create these edges automatically.

Next object is `Post`, declaration:

```json
    "post": {
        "code": "12",
        "GET": "any",
        "PUT": "self",
        "DELETE": "self",
        "fields": {
            "creator": {
                "type": "object_id",
                "object_types": ["user"],
                "required": true,
                "auto_value": "req.user"
            },
            "image": {
                "type": "object_id",
                "object_types": ["file"],
                "required": true,
                "edit_mode": "NE"                
            },            
            "desc": {
                "type": "string",
                "required": true,
                "edit_mode": "E",
                "min": 2,
                "max": 4096                
            },
        },
        "edges": {
            "comments": {
                "contains": ["comment"],
                "GET": "any",
                "POST": "registered_user"
            },
            "offended": {
                "contains": ["user"],
                "GET": "admin_role",
                "POST": "registered_user"
            }
        }
    }
```

Our Post objects will be accessible by anybody, even users who have not registered with the system. Only registered users can comment and mark posts as offensive. Only admins can see who finds a particular post offensive. Please note that once a `Post` is created fields `creator` and `image` are not editable, thus we have `"edit_mode": "NE"` for them. In case `"edit_mode"` is ommitted it means that a field can not be specified when an object is created and it can not be changed. Usually such fields are set automatically by the system.

Another thing of note with `Post` is `"auto_value"` parameter in `"creator"` field declaration. As you can see, `"creator"` field can't be affected by a user at all. `"auto_value": "req.user"` parameter specifies that this field should be populated with `User` object of the currently authenticated user.

And the last object we will define is `Comment`:

```json
    "comment": {
        "code": "13",
        "GET": "any",
        "PUT": "self",
        "DELETE": "self",
        "fields": {
            "creator": {
                "type": "object_id",
                "object_types": ["user"],
                "required": true,
                "auto_value": "req.user"
            },
            "post": {
                "type": "object_id",
                "object_types": ["post"],
                "required": true,
                "edit_mode": "NE"                
            },            
            "text": {
                "type": "string",
                "required": true,
                "edit_mode": "E",
                "min": 2,
                "max": 2048                
            },
        }
    }
```

We've got no edges defined for Comment objects. Comments are deletable and editable by creators.

With the prepared configuration we can proceed to deploying the system!


## Deployment Part 1

We've got **client-data** config ready, let's start deploying EGF2. I will explain how we can deploy framework and all necessary tools on a single Amazon Linux instance in AWS. Get an instance with at least 2GB of RAM, we are going to have a lot of stuff here. Also, please create an S3 bucket that will be used by the file service. An IAM role that allows full access to this S3 bucket should be configured and assigned to this instance.

But before we get to the console we need to fork necessary framework repositories. We will start with the following services (repositories):

* client-data
* client-api
* auth

Once you've got them forked please apply the changes to the `client-data/config.json`.

Before we deploy EGF2 services we need to install and configure RethinkDB.

### RethinkDB
As we are going to a single instance deployment it makes sense to use RethinkDB. Cassandra in this situation seems like a bit of an overkill :-)

To install RethinkDB from the development repo please run the following commands:

* ``wget "http://download.rethinkdb.com/centos/7/`uname -m`/rethinkdb.repo" -O /etc/yum.repos.d/rethinkdb.repo``

* `yum install -y rethinkdb`

In order to configure RethinkDB please create a file `/etc/rethinkdb/instances.d/guide-db.conf` with the following content:

```
bind=all
server-tag=default
server-tag=us_west_2
server-tag=guide_db
```

Please note that `server-tag=us_west_2` means that we are using an instance started in US-WEST-2 region of AWS.

And now you can start it:

* `chkconfig --add rethinkdb`
* `chkconfig rethinkdb on`
* `service rethinkdb start`

RethinkDB is now ready.

### Deploying Services

Please install the latest stable version of the Node.js.

Clone the services that we decided to deploy in `/opt` directory. Run `npm install` in every service directory.

In order to run **client-data** service we need to specify what port we want it to run on and configure RethinkDB parameters:

```json
{
    ...
    "port": 8000,
    "storage": "rethinkdb",
    "log_level": "debug",
    "rethinkdb": {
        "host": "localhost",
        "db": "guide"
    }
    ...
}
```

You can pick another port, if you'd like to. We just need to remember it as we will point other services to **client-data** using it.

To configure **auth** service please set the following parameters:

```json
{
    ...
    "port": 2016,
    "log_level": "debug",
    "client-data": "http://localhost:8000"
    ...
}
```

And **client-api** is configured with:

```json
{
    ...
    "port": 2019,
    "log_level": "debug",
    "client-data": "http://localhost:8000",
    "auth": "http://localhost:2016"
    ...
}
```

As you can see, **client-data** is the only service that talks to RethinkDB. Other services are using it for all data needs.

Before we start services we need to initialise DB. We can do it easily:

`node index.js  --config /opt/client-data/config.json --init`

Output of this task will include a line:

```
SecretOrganization = "<SecretOrganization object ID>"
```

Please use the SecretOrganization object ID string in:

```json
"graph": {
...
    "objects": {
        "secret_organization": "<object ID>"
    }
...
}
```

This will let **client-data** know object ID of `SecretOrganization` singleton.

In order to start a service please do the following:

* `cd /opt/<service>`
* `node index.js --config /opt/<service>/config.json`

Where `<service>` takes values "client-data", "client-api", "auth". Please start **client-data** first, before other services.

### NGINX

With our services started we need to do one more thing - install and configure NGINX with `yum install -y nginx`.

Create a file called `api-endpoints` in `/etc/nginx` folder. Add the following text to it:

```
#Client API
location /v1/graph {
    proxy_pass http://127.0.0.1:2019;
}

#Auth
location /v1/register {
    proxy_pass http://127.0.0.1:2016;
}
location /v1/verify_email {
    proxy_pass http://127.0.0.1:2016;
}
location /v1/login {
    proxy_pass http://127.0.0.1:2016;
}
location /v1/logout {
    proxy_pass http://127.0.0.1:2016;
}
location /v1/forgot_password {
    proxy_pass http://127.0.0.1:2016;
}
location /v1/change_password {
    proxy_pass http://127.0.0.1:2016;
}
location /v1/resend_email_verification {
    proxy_pass http://127.0.0.1:2016;
}
```

We also need to update `/etc/nginx/nginx.conf` file, add line in the following section:

```
server {
        listen       80;
        server_name  localhost;
        include api-endpoints;
...
```

We are adding one line - `include api-endpoints;`.

After that we are ready to start NGINX:

* `chkconfig nginx on`
* `service nginx start`

That's it - the system is up!

What have we got:

1. Full Graph API - all endpoints are functional and know about our designed model.
2. Registration, login, logout (without verification emails).

I will continue with deployments in the next post - we will add ElasticSearch, **sync**, **pusher** and **file** services to the system.


## Deployment Part 2
In the previous section we've got part of the services deployed along with RethinkDB and NGINX.

Now we need to finalise our works with the addition of ElasticSearch and the rest of the services. We will have search and file related endpoints powered as a result, email notifications will be sent for the email verification and forgot password features.

Let's start with ElasticSearch.

### ElasticSearch

First do RPM import:
```
rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
```

Create file `/etc/yum.repos.d/elasticsearch.repo` with content:

```
[elasticsearch-2.x]
name=Elasticsearch repository for 2.x packages
baseurl=http://packages.elastic.co/elasticsearch/2.x/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
```

Install package:

```
yum install -y elasticsearch
```

And to start ElasticSearch please do:

* `chkconfig elasticsearch on`
* `service elasticsearch start`

We need to add info on ES to the config files of our deployed and running services.

This line should be added to the end of **client-api** and **auth** service configs:

```json
"elastic": { "hosts": ["localhost:9200"] }
```

Restart **client-api** and **auth** services.

**client-api** is using ES to power the search endpoint. **auth** needs ES to lookup users by email.

We also need to configure one additional endpoint with our NGINX, please add the following to the `/etc/nginx/api-endpoints` file:

```
location /v1/search {  
    proxy_pass http://127.0.0.1:2019;
}
```

Now we have search endpoint ready!

### Deploying Services

Please fork and then clone **file**, **sync** and **pusher** services to the `/opt` folder.

#### sync

To configure **sync** service please change the following parameters:

```json
{
    ...
    "log_level": "debug",
    "client-data": "http://localhost:8000",
    "queue": "rethinkdb",
    "consumer-group": "sync",
    "rethinkdb": {
        "host": "localhost",
    	"port": "28015",
    	"db": "guide",
    	"table": "events",
    	"offsettable": "event_offset"
    }
    ...
}
```

With the single instance deployment we are using RethinkDB changes feed feature as a system event bus. While it works great for development and testing please note that it is not a scalable solution. For production systems please use `"queue": "kafka"` setup instead. `"rethinkdb"` parameter holds config necessary for **sync** to connect to RethinkDB directly.

The rest of the config we will leave intact for now. It contains system ES indexes that are used by some of the services we have. I will do a separate post on **sync** and ES indexes shortly.

We are ready to start **sync**:

* `cd /opt/sync`
* `npm install`
* `node index.js --config /opt/sync/config.json`

**sync** is running!

#### file

Let's configure file service:

```json
{
    "log_level": "debug",
    "port": 2018,
    "auth": "http://localhost:2016",
    "client-data": "http://localhost:8000",
    "s3_bucket": "egf2-guide-images",
    "queue": "rethinkdb",
    "consumer-group": "file",
    "rethinkdb": {
		"host": "localhost",
		"port": "28015",
		"db": "guide",
		"table": "events",
		"offsettable": "event_offset"
    }
}
```

Please note parameter `"s3_bucket"` - you need to use a name of an AWS S3 bucket here. Your instance should have an IAM role that allows full access to this bucket.

We will leave parameter called `"kinds"` intact for now. What it allows us to do is to have predefined image resize groups. When a new image is created a kind can be specified, **file** service will then create necessary resizes automatically.

In order to be able to resize images file service needs ImageMagick. To install the package please do `yum install ImageMagick`

And start the service:

* `cd /opt/file`
* `npm install`
* `node index.js --config /opt/file/config.json`

Let's allow **file** service endpoints with our NGINX, please add the following to the `/etc/nginx/api-endpoints` file:

```
location /v1/new_image {
    proxy_pass http://127.0.0.1:2018;
}

location /v1/new_file {
    proxy_pass http://127.0.0.1:2018;
}
```

#### pusher

Let's configure **pusher** service:

```json
{
    "log_level": "debug",
    "port": 2017,
    "auth": "http://localhost:2016",
    "client-data": "http://localhost:8000",
    "web_socket_port": 80,
    "email_transport": "sendgrid",
    "template_host": "localhost",
    "ignored_domains": [],
    "queue": "rethinkdb",
    "consumer-group": "pusher",
    "rethinkdb": {
        "host": "localhost",
        "port": "28015",
        "db": "guide",
        "table": "events",
        "offsettable": "event_offset"
    }   
}
```

`"email_transport": "sendgrid"` option specifies that we will use SendGrid to send our email notifications.

It is important to note that there is no SendGrid API key in config file here. In order to simplify config file deployment we assume that configs are not private, thus we don't store any sensitive information in configs. Instead, we have a singleton object of type `SecretOrganization` that stores API keys etc.

Please set value for key "sendgrid_api_key" with your SendGrid API key. It can be done as follows:

```
curl -XPUT -H "Content-Type: application/json" http://localhost:8000/v1/graph/<SecretOrganization object ID> -d '{"secret_keys":[{"key":"sendgrid_api_key","value":"<sendgrid_api_token>"}]}'
```

Where `<SecretOrganization object ID>` can be found in **client-data** config, `"graph"/"objects"/"secret_organization"` field.

After that we can start the service:

* `cd /opt/file`
* `npm install`
* `node index.js --config /opt/file/config.json`

We now have all the services and tools in place that are necessary for the Guide system at this stage. The only services that were not deployed yet are:

* **scheduler** - internal scheduling service
* **job** - large task async handling

Both of them are not necessary at the moment.

The system we have deployed so far can be utilised by web and mobile system developers to create and test client apps. We find it a great help to have good part of back-end in place as soon as possible, it helps client app development a lot.

We will continue with adding ElasticSearch indexes in the next section.


## Adding Search

Now that we've got the system operational it is time to add search functionality to our back-end.

We will add search for `Posts` and for `Users`. Let's start with separate indexes and then we can do a custom index that works for both `Users` and `Posts` simultaneously.

### Automatic Indexes

In order to add `Post` index please add the following text to the sync service config (can be found at `/opt/sync/config.json`), `"elastic"/"indices"` section:

```json
...
"post": {
    "object_type": "post",
    "mapping": {
        "id": {"type": "string", "index": "not_analyzed"},
        "description": {"type": "string"},
        "created_at": {"type": "date"}
    }
}
...
```

Please check out [ElasticSearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html) for supported mapping options.

`User` index is already partially supported for us. We need to add some fields here though. Here is the way `User` index should look like:

```json
...
"user": {
    "object_type": "user",
    "mapping": {
        "id": {"type": "string", "index": "not_analyzed"},
        "first_name": {"type": "string", "field_name": "name.given"},
        "last_name": {"type": "string", "field_name": "name.family"},
        "email": {"type": "string", "index": "not_analyzed"}
    }
}
...
```
As you can see, we have added `"first_name"` and `"last_name"` fields to the `User` index declaration.

Without writing a line of code we have just supported search for Users on name and for Posts on description.

Let's see what we can do with custom index handlers.

### Custom Index

Let's add the following text to the config, `"elastic"/"indices"` section:

```json
...
"post_user": {
    "mapping": {
        "id": {"type": "string", "index": "not_analyzed"},
        "combo": {"type": "string"}
    }
}
...
```

We need to implement the handler for our new index. Let's start with adding a file `"post_user.js"` to the `"extra"` folder of the **sync** service:


```js
"use strict";

const config = require("../components").config;
const elastic = require("../components").elasticSearch;

const indexName = "post_user";

function isUser(event) {
    let obj = event.current || event.previous;
    return obj.object_type === "user";
}

function onPost(event) {
    return elastic.index({
        index: indexName,
        type: indexName,
        id: event.object,
        body: {
            id: event.current.id,
            combo: isUser(event) ?
                `${event.current.name.given} ${event.current.name.family}` :
                event.current.description
        }
    });
}

function onPut(event) {
    let combo;
    if (isUser(event)) {
        if (event.current.name.given !== event.previous.name.given ||
            event.current.name.family !== event.previous.name.family) {
            combo = `${event.current.name.given} ${event.current.name.family}`;
        }
    } else if (event.current.description !== event.previous.description) {
        combo = event.current.description;
    }
    if (combo) {
        return elastic.update({
            index: indexName,
            type: indexName,
            id: event.object,
            body: {combo}
        });
    }
    return Promise.resolve();
}

function onDelete(event) {
    return elastic.delete({
        index: indexName,
        type: indexName,
        id: event.object
    });
}

module.exports = {
    onPost,
    onPut,
    onDelete
};
```

This code reacts to events related to `User` and `Post` and stores data in a combined ES index.

Last thing we need to do is let the system know about our new handler, please add these lines to the `"extra/index.js"`, somewhere in `handleRegistry` map:

```
...
"POST user": require("./post_user").onPost,
"PUT user": require("./post_user").onPut,
"DELETE user": require("./post_user").onDelete,
"POST post": require("./post_user").onPost,
"PUT post": require("./post_user").onPut,
"DELETE post": require("./post_user").onDelete
...
```

Restart **sync**. We are all set!

We can now do searches for users, posts and for both as follows:

* `GET /v1/search?q=john&object=user&fields=first_name,last_name` to get all users with name or surname "john"
* `GET /v1/search?q=waterfall&object=post&fields=description` to get all posts that have "waterfall" in description
* `GET /v1/search?q=helen&object=post_user&fields=combo` to get users and posts related to "helen"

In the next post we will add several **logic** rules.


## Adding Logic

There are situations when some actions need to be taken in response to a particular change in data. Change can be initiated by a user request, by some service working on a schedule or by any other source. In case actions that need to be done deal with data modifications (not sending notifications which we deal with in **pusher**) we implement such actions (called "rules") in **logic** service.

We will add rules for the following situations:

1. When a new `User/follows` edge is created **logic** should create a corresponding edge `User/followers`.
2. When a `User/follows` edge is removed **logic** should remove corresponding `User/followers` edge.
3. When a new `User/posts` edge is created **logic** should create `User/timeline` edge for each `User` that follows the author of this `Post` (e.g. `Users` from `User/followers` edge). Here we have an example of "fan-out on write" approach.
4. When a `User/posts` edge is removed logic should remove `User/timeline` edge for all followers of this User.
5. When a new `Post/offended` edge is created **logic** needs to find all users in the system that have `AdminRole` and create a new edge `AdminRole/offending_posts`. It should check if an edge was already created before creation though - we may have multiple users offended by a single `Post`.

In our system documentation we will have all rules listed in a table in `Logic Rules` section.

Let's start with adding files with rules' implementations and then proceed with configuring the registry.

### Rule 1, 2

Please create file `logic/extra/user_follows.js` with the following content:

```js
"use strict";

const clientData = require("../components").clientData;

// rule #1
function onCreate(event) {
    return clientData.createEdge(event.edge.dst, "followers", event.edge.src);
}

// rule #2
function onDelete(event) {
    return clientData.deleteEdge(event.edge.dst, "followers", event.edge.src);
}

module.exports = {
    onCreate,
    onDelete
};
```

Please note the use of `clientData`. `clientData` module provides easy and convenient way to interface with **client-data** service.

### Rule 3, 4

File name: `logic/extra/user_posts.js`, content:

```js
"use strict";

const clientData = require("../components").clientData;

// Rule #3
function onCreate(event) {
    return clientData.forEachPage(
        last => clientData.getEdges(event.edge.src, "followers", {after: last}),
        followers => Promise.all(followers.results.map(follower =>
            clientData.createEdge(follower.id, "timeline", event.edge.dst)
        ))
    );
}

// Rule #4
function onDelete(event) {
    return clientData.forEachPage(
        last => clientData.getEdges(event.edge.src, "followers", {after: last}),
        followers => Promise.all(followers.results.map(follower =>
            clientData.createEdge(follower.id, "timeline", event.edge.dst)
        ))
    );
}

module.exports = {
    onCreate,
    onDelete
};

```

In rules 3 and 4 we have an example of paginating over pages of an edge with `clientData.forEachPage`.

### Rule 5
This rule is a bit more interesting. We need to get a list of all admins in the system. Graph API has no way to get a list of objects that are not connected to some edge.

Let's define a new automatic ES index with **sync** and use search endpoint for this.

Add the following text to the `/opt/sync/config.json`:

```json
...
"admin_role": {
    "object_type": "admin_role",
    "mapping": {
        "id": {"type": "string", "index": "not_analyzed"}
    }
}
...
```

Restart **sync** - we are ready for getting admins.

Now we need to implement the rule, file name: `logic/extra/post_offended.js`, content:

```js
"use strict";

const clientData = require("../components").clientData;
const searcher = require("../components").searcher;
const log = require("../components").logger;

// Rule #5
function onCreate(event) {
    return clientData.getGraphConfig().then(config =>
        clientData.forEachPage(
            last => searcher.search({object: "admin_role", count: config.pagination.max_count, after: last}),
            admins => Promise.all(admins.results.map(admin =>
                clientData.createEdge(admin, "offending_posts", event.edge.src)
                    .catch(err => {
                        log.error(err);
                    })
                ))
        )
    );
}

module.exports = {
    onCreate
};
```

Please note here the way `searcher` module is used. `searcher` provides a way to interact with ElasticSearch without the need to work with ES driver.

We have prepared logic rule handlers, now we need to let **logic** know about them. Add the following lines in `logic/extra/index.js` file, in the `handleRegistry` map:

```
...
    "POST user/follows": require("./user_follows").onCreate,
    "DELETE user/follows": require("./user_follows").onDelete,
    "POST user/posts": require("./user_posts").onCreate,
    "DELETE user/posts": require("./user_posts").onDelete,
    "POST post/offended": require("./post_offended").onCreate
...
```

Restart **logic** service.

Even though EGF2 can not provide implementation for custom business rules for all systems and domains it does offer structure and a clear path for adding such rules. It suggests where implementation should live, how it should be connected to the rest of the system and it gives a recommendation on the way rules should be documented.

In the next post we will look into web app creation.
