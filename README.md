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

### Removing a Space

Although a Space record can be removed, this does not (at the moment) remove the images belonging to that space.

```
curl -X DELETE -u <api id>:<api secret> https://api.dlc.services/customers/<customer id>/spaces/<space id>
```

## Images



register: some images with the DLCS and use named queries to create a IIIF manifest

update some images

delete some images

