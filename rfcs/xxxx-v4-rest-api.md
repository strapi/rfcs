- Start Date: 2021-05-12
- RFC PR: (leave this empty)

# Summary

Here is a first draft of what we would like the REST API to look like in v4.

# Example

**url**: `GET /api/articles`

**Response**

```json
{
  "data": [
    {
      "id": 1,
      "attributes": {
        "title": "My Title",
        "description": "My description"
      },
      "meta": {}
    },
    {
      "id": 2,
      "attributes": {
        "title": "My Title",
        "description": "My description"
      },
      "meta": {}
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "pageSize": 2,
      "pageCount": 10,
      "total": 20
    }
  }
}
```

# Motivation

The main motivation for this change is to make the API more flexible for future development while making it more consistent and easier to build with.

> We are taking some important concept from [json:api](https://jsonapi.org/) that we really like in general.

# Detailed design

## Endpoints

We would like to prefix the `Content Api` routes with `/api` to make it easier to differentiate & allow setting middlewares directly on the entire `Content API`.
This should also prevent use from creating unintended conflicts in the route names between the `Content API` & the `Admin Panel API`.

**Glossary:**

- `pluralApiId`: the plural version of the content type name (e.g: `articles` and not `article`)
- `documentId`: the id of the one document

**Collection Types**

| Method   | Url                                             | Desc                                                                    |
| -------- | ----------------------------------------------- | ----------------------------------------------------------------------- |
| `GET`    | `/api/:pluralApiId`                             | Find a list of documents                                                |
| `POST`   | `/api/:pluralApiId`                             | Create a document                                                       |
| `GET`    | `/api/:pluralApiId/:documentId`                 | Find a document                                                         |
| `PUT`    | `/api/:pluralApiId/:documentId`                 | Update a document                                                       |
| `DELETE` | `/api/:pluralApiId/:documentId`                 | Delete a document                                                       |
| `POST`   | `/api/:pluralApiId/actions/:action`             | Actions on the collection of documents (bulk actions, custom action...) |
| `POST`   | `/api/:pluralApiId/:documentId/actions/:action` | Actions on a specific document                                          |

**Single Types**

| Method   | Url                                 | Desc                                          |
| -------- | ----------------------------------- | --------------------------------------------- |
| `GET`    | `/api/:pluralApiId`                 | Find document                                 |
| `PUT`    | `/api/:pluralApiId`                 | Set / Update document                         |
| `DELETE` | `/api/:pluralApiId`                 | Delete document                               |
| `POST`   | `/api/:pluralApiId/actions/:action` | Actions on the single type (custom action...) |

### Actions

In the endpoints list you can see a few routes called **Actions**.

The goal is to normalize how we can extend a `Content Type API` with new actions that don't fit the basic `CRUD` REST API & that aren't REST compliant or making them compliant would make the developer experience worse.

Actions have a name and you use an HTTP `POST` request with a `JSON` body to send the required information for the action to run.

Here are a few examples with this convention:

#### Bulk Create

**url**: `POST /api/articles/actions/bulkCreate`

**Request**

```json
{
  "data": [
    {
      "title": "test"
    }
  ]
}
```

#### Publish

**url**: `POST /api/articles/1/actions/publish`

## Retrieving Data

### Fetching Entities

#### Fetching a collection of entities

**url**: `GET /api/:pluralApiId`

**Response**

```json
{
  "data": [
    {
      "id": 1,
      "attributes": {
        "entityId": 1,
        "title": "Mon titre",
        "description": "Ma super description"
      },
      "meta": {
        "availableLocales": []
      }
    }
  ],
  "meta": {
    "pagination": {}
  }
}
```

#### Fetching one entity

**url**: `GET /api/:pluralApiId/:documentId`

**Response**

```json
{
  "data": {
    "id": 1,
    "attributes": {
      "entityId": 1,
      "title": "Mon titre",
      "description": "Ma super description"
    },
    "meta": {
      "availableLocales": []
    }
  },
  "meta": {}
}
```

### Sorting

**Parameter** `sort`

You can prefix the field with `-` to make the order descending.

**Examples**

```
GET /api/:pluralApiId?sort=title
```

```
GET /api/:pluralApiId?sort=-title
```

```
GET /api/:pluralApiId?sort=title,-price
```

```
GET /api/:pluralApiId?sort=title,author.name
```

We also would allow arrays that the `qs` library can parse.

```
GET /api/:pluralApiId?sort=title&sort=author.name
```

```
GET /api/:pluralApiId?sort[]=title&sort[]=author.name
```

```
GET /api/:pluralApiId?sort[0]=title&sort[1]=author.name
```

**Example with qs**

```js
qs.stringify({
  sort: "title,-price",
});

// or a
qs.stringify({
  sort: ["title", "-price"],
});
```

### Pagination

**Parameter** `pagination`

**Paginate by page**

- `pagination[page]`: page number (default: 0)
- `pagination[pageSize]`: page size (default: 25)

```
GET /api/:pluralApiId?pagination[page]=0&pagination[pageSize]=10
```

**Response**

```json
{
  "data": [],
  "meta": {
    "pagination": {
      "page": 0,
      "pageSize": 10,
      "pageCount": 5,
      "total": 48
    }
  }
}
```

**Paginate by offset**

- `pagination[start]`: offset value (default: 0)
- `pagination[limit]`: number of entities to return (limit: 25)

```
GET /api/:pluralApiId?pagination[start]=20&pagination[limit]=30
```

**Response**

```json
{
  "data": [],
  "meta": {
    "pagination": {
      "start": 0,
      "limit": 10
    }
  }
}
```

**Example with qs**

```js
qs.stringify({
  pagination: {
    page: 0,
    pageSize: 10,
  },
});

// or a
qs.stringify({
  pagination: {
    start: 10,
    limit: 5,
  },
});
```

**Choosing to return the total number of documents**

As returning counts has a cost we think we might need to provide either a way to remove them or add them depending on the default.

Options (Please give us your feedback in the comments :D)

1. Never return counters for page/pageSize & start/limit & add `pagination[withCount]=true` to return them
2. Always return counters for page/pageSize & start/limit & add `pagination[withCount]=false` to remove them
3. For start/limit make `withCount` `false` by default & for page/pageSize make it `true` by default
4. Always return count for page/pageSize & never for start/limit

### Filtering

- We want to put all the filters in one query parameter so there are not conflicts with root level parameters.
- We want to change the syntax to be more flexible and separating field name from the operator with nested objects.
- We are keeping the existing filters and we will be able to add new ones easily once we release the first version.

**Parameter** `filters`

**Examples**

```
GET /api/:pluralApiId?filters[title][$eq]="Hello"
```

Using `qs`

```js
qs.stringify({
  filters: {
    $and: [{}],
    $or: [{}],
    title: {
      $eq: "hello",
    },
    date: {
      $eq: "2020-01-01",
    },
    name: {
      $eq: "Hello",
    },
    price: {
      $gte: 12,
    },
    author: {
      name: {
        $eq: "Kai doe deep filter",
      },
    },
  },
});
```

### Selecting fields

You can select the fields you want the API to return. You can either pass a comma separated list of fields or an array of fields.

When you populate a relation you can also select the relation fields you want.

**Parameter** `fields`

**Examples**

```
GET /api/:pluralApiId?fields=firstname,lastname
```

```
GET /api/:pluralApiId?fields=firstname&fields=lastname
```

```
GET /api/:pluralApiId?fields[0]=firstname&fields[1]=lastname
```

**Example with qs**

```js
qs.stringify({
  fields: ["title", "author.name", "author.lastname", "created_at"],
});

// or to check
qs.stringify({
  fields: ["title", "author", "created_at"],
});
```

### Populating relations

By default we will not populate relation. We want to make the API more responsive by default.

You will be able to select what relations you want to populate with a new `populate` query parameter. You can either pass a comma separated list of fields or an array of fields. This supports nested relations too.

**Parameter** `populate`

**Examples**

```
GET /api/:pluralApiId?populate=author
```

```
GET /api/:pluralApiId?populate=author.address
```

```
GET /api/:pluralApiId?populate=friends,comments.author
```

```
GET /api/:pluralApiId?populate[]=friends&populate[]=comments.author
```

**Wildcards**

Maybe not in the MVP but could be added quickly after

populate all relations of 1st level

```
GET /api/:pluralApiId?populate=*
```

load friends and it relations

```
GET /api/:pluralApiId?populate=friends.*
```

**Example with qs**

**We have a lot of options**

```js
qs.stringify({
  populate: ["friends", "comments.author"],
});

// or
qs.stringify({
  populate: "friends,comments.author",
});
```

## Response format

- `data`: Data returns either:
  - An [entity](#entity)
  - A list of [entities](#entity)
  - A custom response
- `error`: [Error](#errors)
- `meta`: Response metadata (pagination)

## Document format

- `id`: document id
- `attributes`: document attributes
- `meta`: document metadata

### Creating

**url**: `POST /api/:pluralApiId`

**Request**

```json
{
  "data": {
    "title": "Hello",
    "relation": 2,
    "relations": [2, 4]
  }
}
```

**Response**

```json
{
  "data": {
    "id": 1,
    "attributes": {},
    "meta": {}
  },
  "meta": {}
}
```

### Updating

**url**: `PUT /api/:pluralApiId/:documentId`

**Request**

```json
{
  "data": {
    "title": "Hello",
    "relation": 2,
    "relations": [2, 4]
  }
}
```

**Response**

```json
{
  "data": {
    "id": 1,
    "attributes": {},
    "meta": {}
  },
  "meta": {}
}
```

### Deleting

**url**: `DELETE /api/:pluralApiId/:documentId`

**Response**

```json
{
  "data": {
    "id": 1,
    "attributes": {},
    "meta": {}
  },
  "meta": {}
}
```

## Errors

To be defined in another RFC
