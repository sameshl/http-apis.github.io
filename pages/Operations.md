---
layout: page
title: Operations | Hydraecosystem.org
permalink: /Operations
---

# Operations

*hydrus* provides an easy way for an user to implement functions like CRUD operations and searching on the resources the user has defined in the ApiDoc.

This page explains how a user can interact with an hydrus interface for executing these functions on to any hydrus-deployed API.

### How to set up the *hydrus* server?

After following the instructions on installing *hydrus* from [here](/hydrus-demo/#demo), you would just need to execute:

```bash
hydrus serve --no-auth
```
- For the purpose of this tutorial, the ApiDoc we will be using is the default *hydrus* uses is which is an example Drone ApiDoc which can be found [here](https://github.com/HTTP-APIs/hydrus/blob/master/hydrus/samples/hydra_doc_sample.py).
- Also, the `--no-auth` flag is to disable authentication on our *hydrus* server, as the purpose of this tutorial is to understand the interface between *hydrus* and the user, and not get into how authentication works in *hydrus*. For more information on how authentication works in *hydrus*, check out [Authentication](/Auth).

If the above command executed successfully, then you should have a *hydrus* server running on port 8080 of your local machine. Now, fetching `http://localhost:8080/serverapi` on a browser or an tool like Postman should show you:
```json
{
    "@context": "/serverapi/contexts/EntryPoint.jsonld",
    "@id": "/serverapi",
    "@type": "EntryPoint",
    "Area": "/serverapi/Area",
    "CommandCollection": "/serverapi/CommandCollection",
    "DatastreamCollection": "/serverapi/DatastreamCollection",
    "DroneCollection": "/serverapi/DroneCollection",
    "LogEntryCollection": "/serverapi/LogEntryCollection",
    "MessageCollection": "/serverapi/MessageCollection",
    "StateCollection": "/serverapi/StateCollection"
}
```
 
### Supported operations on a Resource
The supported operations (GET/PUT/POST/DELETE) can be found at by *supportedOperation* key in the ApiDoc for the required resource.
For example, the operations supported by **MessageCollection** resource can be found [here](https://github.com/HTTP-APIs/hydrus/blob/72b4cda49ab9cfe0fb775146e8d5113f5d6869e0/hydrus/samples/hydra_doc_sample.py#L780-L807).
```python
...
"@id": "vocab:MessageCollection",
"@type": "hydra:Class",
"description": "A collection of message",
"subClassOf": "http://www.w3.org/ns/hydra/core#Collection",
"supportedOperation": [
    {
        "@id": "_:message_collection_retrieve",
        "@type": "http://schema.org/FindAction",
        "description": "Retrieves all Message entities",
        "expects": "null",
        "method": "GET",
        "returns": "vocab:MessageCollection",
        "expectsHeader": [],
        "returnsHeader": [],
        "possibleStatus": []
    },
    {
        "@id": "_:message_create",
        "@type": "http://schema.org/AddAction",
        "description": "Create new Message entity",
        "expects": "vocab:Message",
        "method": "PUT",
        "returns": "vocab:Message",
        "expectsHeader": [],
        "returnsHeader": [],
        "possibleStatus": [
            {
                "title": "If the Message entity was created successfully.",
                "statusCode": 201,
                "description": ""
            }
        ]
    }
],
"supportedProperty": [
    {
        "@type": "SupportedProperty",
        "description": "The message",
        "property": "http://www.w3.org/ns/hydra/core#member",
        "readable": "true",
        "required": "false",
        "title": "members",
        "writeable": "true"
    }
],
"title": "MessageCollection"
...
```
From the above part of the [ApiDoc](https://github.com/HTTP-APIs/hydrus/blob/master/hydrus/samples/hydra_doc_sample.py), we can see that only retrieving(GET) instances and adding instances(PUT) to the database is allowed for the **MessageCollection** resource.

### Retrieving resource instances from database
The above response shows all the resources with a collection endpoint served by *hydrus*. To explore more about any resource, visit the resource endpoint as described in the above reponse.
For example, for more information on the **MessageCollecion** resource, visit `http://localhost:8080/serverapi/MessageCollection`.
Therefore, making a **GET** request to `http://localhost:8080/serverapi/MessageCollection` endpoint, leads to response:
```json
{
    "@context": "/serverapi/contexts/MessageCollection.jsonld",
    "@id": "/serverapi/MessageCollection/",
    "@type": "MessageCollection",
    "hydra:totalItems": 0,
    "hydra:view": {
        "@id": "/serverapi/MessageCollection?page=1",
        "@type": "hydra:PartialCollectionView",
        "hydra:first": "/serverapi/MessageCollection?page=1",
        "hydra:last": "/serverapi/MessageCollection?page=1"
    },
    "members": [],
    "search": {
        "@type": "hydra:IriTemplate",
        "hydra:mapping": [
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "http://schema.org/Text",
                "hydra:required": false,
                "hydra:variable": "MessageString"
            },
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "pageIndex",
                "hydra:required": false,
                "hydra:variable": "pageIndex"
            },
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "limit",
                "hydra:required": false,
                "hydra:variable": "limit"
            },
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "offset",
                "hydra:required": false,
                "hydra:variable": "offset"
            }
        ],
        "hydra:template": "/serverapi/Message(MessageString, pageIndex, limit, offset)",
        "hydra:variableRepresentation": "hydra:BasicRepresentation"
    }
}
```
Some important points, to understand from the response here are:
- The `hydra:totalItems` key has value zero (`0`). This means that there are no instances of this resource in the database yet. This makes sense as till now, nothing is *added* into the database through hydrus. We have only retrieved(GET) data from the server.
- The `members` key is an empty array. This would show all the relvant details of members in the database if they had been added.

### Adding resource instance to database

To add an instance of a resource served by hydrus, we need to make a PUT request to the corresponding endpoint with some specific data in the body of the request.
For example, to add a *Message* object of the given *MessageCollection* resource to the database, we need to make a PUT request to `http://localhost:8080/serverapi/MessageCollection` with the body of the request as:
```json
{
"@type": "Message",
"MessageString": "Test Message"
}
```
- Here `@type` dictates to what resource should *hydrus* try to add the given data to the database, thus making it a necessary field.
- `MessageString` is just the `hydra:variable` name for a necessary field for an object to belong to the *Message* class.

If the above request succeeded, then you should get a response along the lines of:
```json
{
    "@context": "http://www.w3.org/ns/hydra/context.jsonld",
    "@type": "Status",
    "description": "Object with ID b7551d4f-cc91-484e-91b1-3b527b4899e0 successfully added",
    "statusCode": 201,
    "title": "Object successfully added"
}
```
- The response has a `title` key with value "Object successfully added" meaning that our request was successful and we added a Message object to the database.
- The long string of alpha-numeric characters is the unique ID of the object we just added to the database.

### Retrieving a single resource instance from the database
To get more details of the object stored in the database, make a GET request to `endpoint_on_which_object_added/uniqueID`.
Therefore, to get more details on this specific Message just inserted into the database, to make a GET request to `http://localhost:8080/serverapi/MessageCollection/b7551d4f-cc91-484e-91b1-3b527b4899e0`.
Then the response is:
```json
{
    "@context": "/serverapi/contexts/MessageCollection.jsonld",
    "@id": "/serverapi/MessageCollection/b7551d4f-cc91-484e-91b1-3b527b4899e0",
    "@type": "Message",
    "MessageString": "Test Message"
}
```

Here, we can see the Message object with the *MessageString* we put while adding the object to the database.

#### **NOTE**
Now, if a request is made to `http://localhost:8080/serverapi/MessageCollection` to view all the contents of the database on that `MessageCollection` resource, the response would contain:
```json
{
    "@context": "/serverapi/contexts/MessageCollection.jsonld",
    "@id": "/serverapi/MessageCollection/",
    "@type": "MessageCollection",
    "hydra:totalItems": 1,
    "hydra:view": {
        "@id": "/serverapi/MessageCollection?page=1",
        "@type": "hydra:PartialCollectionView",
        "hydra:first": "/serverapi/MessageCollection?page=1",
        "hydra:last": "/serverapi/MessageCollection?page=1"
    },
    "members": [
        {
            "@id": "/serverapi/MessageCollection/b7551d4f-cc91-484e-91b1-3b527b4899e0",
            "@type": "Message"
        }
    ],
    "search": {
        "@type": "hydra:IriTemplate",
        "hydra:mapping": [
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "http://schema.org/Text",
                "hydra:required": false,
                "hydra:variable": "MessageString"
            },
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "pageIndex",
                "hydra:required": false,
                "hydra:variable": "pageIndex"
            },
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "limit",
                "hydra:required": false,
                "hydra:variable": "limit"
            },
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "offset",
                "hydra:required": false,
                "hydra:variable": "offset"
            }
        ],
        "hydra:template": "/serverapi/Message(MessageString, pageIndex, limit, offset)",
        "hydra:variableRepresentation": "hydra:BasicRepresentation"
    }
}
```
- The key `hydra:totalItems` has value is now one(`1`) which shows that database has `1` item(which was inserted with the PUT request).
- The key `members` is does not have an empty array as value. It now contains information(`@id`) fn the members of that resource in the database.

### Updating a single resource instance from the database
Updating a single object from the database is very similar to the way to retrieve that single resource instance.
We need to make a POST request to `endpoint_on_which_object_added/uniqueID`.
Therefore, to update this specific Message which that was just inserted into the database, make a POST request to `http://localhost:8080/serverapi/MessageCollection/b7551d4f-cc91-484e-91b1-3b527b4899e0`.
You should get response as:
```json
{
    "message": "The method is not allowed for the requested URL."
}
```
This is not wrong. As we had seen earlier that only GET and PUT operation is allowed on MessageCollection. But to update a resource which has POST operation allowed, we will need to make a exactly similar request as we did for adding the object(PUT) but with just the body of the request with updated values. For example, if we did this on a Datastream object, we would get a response similar to:
```json
{
    "@context": "http://www.w3.org/ns/hydra/context.jsonld",
    "@type": "Status",
    "description": "Object with ID 933d72ee-1cf9-4245-9a94-80cbcb548539 successfully updated",
    "statusCode": 200,
    "title": "Object updated"
}
```

To confirm that the object was updated, making a GET request to `http://localhost:8080/serverapi/DatastreamCollections/933d72ee-1cf9-4245-9a94-80cbcb548539` returns the object details with updated values, which shows that the object from the database has been updated.

### Deleting a single resource instance from the database
Deleting a single object from the database is very similar to the way to retrieve that single resource instance.
We need to make a DELETE request to `endpoint_on_which_object_added/uniqueID`.
Therefore, to delete this specific Message which that was just inserted into the database, make a DELETE request to `http://localhost:8080/serverapi/MessageCollection/b7551d4f-cc91-484e-91b1-3b527b4899e0`.
You should get response as:
```json
{
    "@context": "http://www.w3.org/ns/hydra/context.jsonld",
    "@type": "Status",
    "description": "Object with ID b7551d4f-cc91-484e-91b1-3b527b4899e0 successfully deleted",
    "statusCode": 200,
    "title": "Object successfully deleted."
}
```

To confirm that the object was deleted, making a GET request to `http://localhost:8080/serverapi/MessageCollection` returns `hydra:totalItems` as 0, which shows that the object from the database has been deleted.

### Searching for an instance using hydrus
*hydrus* provides an interface to search for instances saved in the database out of the box.
For the sake of using searching feature, three instances of **Message** class have been inserted in the database using the PUT request with the *MessageString* property as "First", "Second" and "Third".
Therefore a GET request to `http://localhost:8080/serverapi/MessageCollection`, responds with
```json
{
    "@context": "/serverapi/contexts/MessageCollection.jsonld",
    "@id": "/serverapi/MessageCollection/",
    "@type": "MessageCollection",
    "hydra:totalItems": 3,
    "hydra:view": {
        "@id": "/serverapi/MessageCollection?page=1",
        "@type": "hydra:PartialCollectionView",
        "hydra:first": "/serverapi/MessageCollection?page=1",
        "hydra:last": "/serverapi/MessageCollection?page=1"
    },
    "members": [
        {
            "@id": "/serverapi/MessageCollection/720dfcb6-8283-4486-82bf-115c687700af",
            "@type": "Message"
        },
        {
            "@id": "/serverapi/MessageCollection/20cfd926-5b56-468f-a4ab-01c0941453ae",
            "@type": "Message"
        },
        {
            "@id": "/serverapi/MessageCollection/7cd4e1a0-9311-472b-bfba-0859cfffca6e",
            "@type": "Message"
        }
    ],
    "search": {
        "@type": "hydra:IriTemplate",
        "hydra:mapping": [
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "http://schema.org/Text",
                "hydra:required": false,
                "hydra:variable": "MessageString"
            },
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "pageIndex",
                "hydra:required": false,
                "hydra:variable": "pageIndex"
            },
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "limit",
                "hydra:required": false,
                "hydra:variable": "limit"
            },
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "offset",
                "hydra:required": false,
                "hydra:variable": "offset"
            }
        ],
        "hydra:template": "/serverapi/Message(MessageString, pageIndex, limit, offset)",
        "hydra:variableRepresentation": "hydra:BasicRepresentation"
    }
}
```
We can clearly see that there are three instances of **Message** class in the database.

*hydrus* uses query parameters to search for instances in the database.
Now to search for the instance of the **Message** class with *MessageString* as "First", we need to make a GET request to
`http://localhost:8080/serverapi/MessageCollection?MessageString=First`, and then we get the response as
```json
{
    "@context": "/serverapi/contexts/MessageCollection.jsonld",
    "@id": "/serverapi/MessageCollection/",
    "@type": "MessageCollection",
    "hydra:totalItems": 1,
    "hydra:view": {
        "@id": "/serverapi/MessageCollection?MessageString=First&page=1",
        "@type": "hydra:PartialCollectionView",
        "hydra:first": "/serverapi/MessageCollection?MessageString=First&page=1",
        "hydra:last": "/serverapi/MessageCollection?MessageString=First&page=1"
    },
    "members": [
        {
            "@id": "/serverapi/MessageCollection/720dfcb6-8283-4486-82bf-115c687700af",
            "@type": "Message"
        }
    ],
    "search": {
        "@type": "hydra:IriTemplate",
        "hydra:mapping": [
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "http://schema.org/Text",
                "hydra:required": false,
                "hydra:variable": "MessageString"
            },
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "pageIndex",
                "hydra:required": false,
                "hydra:variable": "pageIndex"
            },
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "limit",
                "hydra:required": false,
                "hydra:variable": "limit"
            },
            {
                "@type": "hydra:IriTemplateMapping",
                "hydra:property": "offset",
                "hydra:required": false,
                "hydra:variable": "offset"
            }
        ],
        "hydra:template": "/serverapi/Message(MessageString, pageIndex, limit, offset)",
        "hydra:variableRepresentation": "hydra:BasicRepresentation"
    }
}
```
Now, in the response, the number of members is just one. It is the instance of the database with MessageString as "First".
To confirm this, we can make a GET request to `http://localhost:8080/serverapi/MessageCollection/720dfcb6-8283-4486-82bf-115c687700af` to dereference that instance, and we get the response as:
```json
{
    "@context": "/serverapi/contexts/MessageCollection.jsonld",
    "@id": "/serverapi/MessageCollection/720dfcb6-8283-4486-82bf-115c687700af",
    "@type": "Message",
    "MessageString": "First"
}
```
We can clearly see that the "MessageString" value of this instance in "First", confirming that *hydrus* as filtered the correct instance.

**NOTE**: Searching in *hydrus* is case **sensitive**.

Therefore, a GET request to `http://localhost:8080/serverapi/MessageCollection?MessageString=first` would lead to zero matches.