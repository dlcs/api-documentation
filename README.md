# api-documentation

DLCS API documentation

# First steps

Once you have your customer API details, you will be able to create and manipulate items using the DLCS API.

This guide gives some basic examples of creating and updating resources.

## Spaces

Images are organised using a loose organisational unit called a Space.

- A Space has a numeric ID and a name
- You can create as many Spaces as you want
- An Image can only exist within a Space
- Any number of images can exist within a Space

### Creating a Space

```
curl -X POST -u <api id>:<api secret> --data '{ "name": "<space name>" }' https://api.dlc.services/customers/<customer id>/spaces
```

This will return a piece of JSON-LD that describes the Space, e.g.:
```
{
  "@context": "https://api.dlc.services/contexts/Space.jsonld",
  "@id": "https://api.dlc.services/customers/2/spaces/1",
  "@type": "vocab:Space",
  "defaultRoles": "https://api.dlc.services/customers/2/spaces/1/defaultRoles",
  "images": "https://api.dlc.services/customers/2/spaces/1/images",
  "metadata": "https://api.dlc.services/customers/2/spaces/1/metadata",
  "storage": "https://api.dlc.services/customers/2/spaces/1/storage",
  "name": "demo",
  "created": "0001-01-01T00:00:00+00:00",
  "imageBucket": "",
  "defaultTags": [],
  "keep": false,
  "transform": false,
  "maxUnauthorised": -1,
  "approximateNumberOfImages": 0
}
```

### Fetch API information about a Space

```
curl -u <api id>:<api secret> https://api.dlc.services/customers/<customer id>/spaces/<space id>
```

### Fetch list of Images within a Space

```
curl -u <api id>:<api secret> https://api.dlc.services/customers/<customer id>/spaces/<space id>/images
```

The collection of Images is paged with a page size of 100 items. When there are more Images than the page size then a PartialCollectionView element will be in the resulting JSON-LD:

```
  ...
  {
    "@id": "https://api.dlc.services/customers/2/spaces/1/images?page=1",
    "@type": "PartialCollectionView",
    "next": "https://api.dlc.services/customers/2/spaces/1/images?page=2",
    "last": "https://api.dlc.services/customers/2/spaces/1/images?page=340"
  }
```

Please use the `?page=<x>` query string to page through the list of results which will be ordered by the ASCII value of the Image's ID.

### Updating a Space

There are a limited set of properties of a Space which may be modified using the PATCH HTTP verb.

```
curl -X PATCH -u <api id>:<api secret> --data '{ "maxUnauthorised": 400 }' https://api.dlc.services/customers/<customer id>/spaces/<space id>
```

### Deleting a Space

Although a Space record can be removed, this does not (at the moment) remove the images belonging to that space.

```
curl -X DELETE -u <api id>:<api secret> https://api.dlc.services/customers/<customer id>/spaces/<space id>
```

## Images

### Registering an Image, Batches and Queue activity

The API can be called to register one or multiple Images by sending some JSON-LD in the body of a POST to your customer Queue endpoint.

The JSON-LD itself is of the following format:

```
{
    "@context": "http://www.w3.org/ns/hydra/context.jsonld",
    "@type": "Collection",
    "member": [
        {
            "space": <space id>,
            "origin": "<public-facing URL of master image>"
        },
        ...
    ]
}
```

The following command will enqueue the ingestion of a collection of Images. One Image is used here, but you can include multiple Images in the `member` JSON-LD array.

```
curl -X POST -u <api id>:<api secret> \
  --data '{ "@context": "http://www.w3.org/ns/hydra/context.jsonld", "@type": "Collection", "member": [ { "space": <space id>, "origin": "<public-facing URL of master image>" } ] }' \
  https://api.dlc.services/customers/<customer id>/queue
```

This will return information about the Batch that the Images have been collected together in. The current activity can be fetched by the following command which will give a list of the unfinished, active Batches for your user:

```
curl -u <api id>:<api secret> https://api.dlc.services/customers/<customer id>/queue/active
```

If there is currently an active Batch then the output may look something like this:

```
{
  "@context": "http://www.w3.org/ns/hydra/context.jsonld",
  "@id": "https://api.dlc.services/customers/2/queue/active",
  "@type": "Collection",
  "totalItems": 1,
  "pageSize": 100,
  "member": [
    {
      "@context": "https://api.dlc.services/contexts/Batch.jsonld",
      "@id": "https://api.dlc.services/customers/2/queue/batches/429449",
      "@type": "vocab:Batch",
      "errorImages": "https://api.dlc.services/customers/2/queue/batches/429449/errorImages",
      "images": "https://api.dlc.services/customers/2/queue/batches/429449/images",
      "completedImages": "https://api.dlc.services/customers/2/queue/batches/429449/completedImages",
      "test": "https://api.dlc.services/customers/2/queue/batches/429449/test",
      "submitted": "2017-12-05T16:01:24.271591+00:00",
      "count": 64,
      "completed": 46,
      "errors": 0,
      "finished": "0001-01-01T00:00:00",
      "superseded": false
    }
  ]
}
```

If there are no currently active Batches then the `member` collection will be empty. Overall ingest activity can be checked using the following command:

```
curl https://api.dlc.services/queue
```

Which will return the global ingest queue activity counts:

```
{
  "@context": "https://api.dlc.services/contexts/Queue.jsonld",
  "@id": "https://api.dlc.services/queue",
  "@type": "vocab:Queue",
  "incoming": 0,
  "success": 3873,
  "failed": 0,
  "priority": 0
}
```

### A Note regarding Image IDs

In the API examples, an Image is referred to by its fully qualified API ID, which is of the form:

```
https://api.dlc.services/customers/<customer id>/spaces/<space id>/images/<image id>
```

But when dealing with the same Image for IIIF purposes, using the apex endpoint `dlc.services`, the Image will be referred to using its contracted form of ID:

```
https://dlc.services/iiif-img/<customer id>/<space id>/<image id>
```

In the underlying storage, an Image has an ID of simply `<customer id>/<space id>/<image id>` which forms the unique identifier within the database.

The `<image id>` portion of the Image ID will generally be a GUID, however it is allowed that the value of `<image id>` can be set during ingest, so this should not be relied upon to always be of that form.

### Fetch API information about an Image

```
curl -u <api id>:<api secret> https://api.dlc.services/customers/<customer id>/spaces/<space id>/images/<image id>
```

***Please note that the `<image id>` is the last part of the Image's ID property***

A typical Image may look like the following JSON-LD:

```
{
  "@context": "https://api.dlc.services/contexts/Image.jsonld",
  "@id": "https://api.dlc.services/customers/2/spaces/1/images/92065289-545c-4a58-9f7e-a1d405a487b7",
  "@type": "vocab:Image",
  "created": "2016-09-07T12:19:49.351+00:00",
  "origin": "http://dlcs-dlcservices-storage-origin.s3.amazonaws.com/2/1/92065289-545c-4a58-9f7e-a1d405a487b7",
  "tags": [],
  "roles": [],
  "preservedUri": "",
  "string1": "abcd1",
  "string2": "",
  "string3": "",
  "maxUnauthorised": -1,
  "number1": 0,
  "number2": 0,
  "number3": 0,
  "width": 3680,
  "height": 2456,
  "duration": 0,
  "error": "",
  "batch": 0,
  "finished": "2016-09-07T12:19:30.51+00:00",
  "ingesting": false,
  "imageOptimisationPolicy": "https://api.dlc.services/imageOptimisationPolicies/fast-higher",
  "thumbnailPolicy": "https://api.dlc.services/thumbnailPolicies/default",
  "family": "I",
  "mediaType": "image/jp2",
  "storage": "https://api.dlc.services/customers/2/spaces/1/images/92065289-545c-4a58-9f7e-a1d405a487b7/storage",
  "metadata": "https://api.dlc.services/customers/2/spaces/1/images/92065289-545c-4a58-9f7e-a1d405a487b7/metadata"
}
```

### Updating an Image

Some properties of an Image may be updated using the PATCH HTTP verb:

```
curl -X PATCH -u <api id>:<api secret> \
  --data '{ "string1": "abcd1", "number1": 1 }' \
  https://api.dlc.services/customers/<customer id>/spaces/<space id>/images/<image id>
```

### Deleting an Image

Images may be deleted using the DELETE HTTP verb:

```
curl -X DELETE -u <api id>:<api secret> https://api.dlc.services/customers/<customer id>/spaces/<space id>/images/<image id>
```

***Please note that the `<image id>` is the last part of the Image's ID property***

### Fetch a single Image as a IIIF Manifest

A quick and dirty way to see an Image in a IIIF viewer is to ask the DLCS for a basic IIIF Manifest (outside of the context of Named Queries, examined below).

```
curl https://dlc.services/iiif-manifest/<customer id>/<space id>/<image id>
```

The IIIF Manifest that is produced is extremely basic but will carry a single Thumbnail Image API endpoint.

## Named Queries

Named Queries are ways of selecting Images in your Spaces and projecting them as some kind of resource - typically a IIIF Manifest.

***Please note that the API for creating named queries is buggy so Named queries should be added by DLCS support***

An example Named Query might be something like this:

```
{
  "@context": "https://api.dlc.services/contexts/NamedQuery.jsonld",
  "@id": "https://api.dlc.services/customers/2/namedQueries/abcdef10-016f-4873-8baa-b4fa40b85bd0",
  "@type": "vocab:NamedQuery",
  "name": "example",
  "global": false,
  "template": "manifest=s1&sequence=0&canvas=n1&s1=p1"
}
```

The `template` property is a set of shorthand commands that mean the following:

| Element       | Description                                                                           |
|---------------|---------------------------------------------------------------------------------------|
| `manifest=s1` | Project a IIIF Manifest from the results of selecting based on metadata field String1 |
| `sequence=0`  | All projected Images will belong to the first Sequence in the IIIF Manifest           |
| `canvas=n1`   | Canvases in the Sequence will be ordered by metadata field Number1                    |
| `s1=p1`       | The value for String1 that we will select upon is equal to the value of parameter 1   |

The URL for the Named Query, which should yield a IIIF Manifest as JSON-LD, will be something like:

```
https://dlc.services/iiif-resource/<customer id>/<query name>/<parameter 1>[/<parameter 2>...]
```

***Note that these results are NOT emitted from the API, but rather from the apex domain `dlc.services`***

e.g.

```
https://dlc.services/iiif-resource/2/example/abcd1
```

Which would pass the value `abcd1` as parameter 1, and would therefore select Images that had a matching String1 metadata field.

The IIIF Manifest that is produced from a Named Query will contain Thumbnail Image API endpoints as appropriate.
