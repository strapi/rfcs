- Start Date: 2023-09-05
- RFC PR: (leave this empty)

# Summary

V5 is in preparation and will require making changes to the Content APIs (REST & GraphQL).

# Motivation

V5 Content Engine requires some breaking changes to the API, It is also the opportunity to simplify the API Response format following user feedbacks.

> The main goal of this RFC is to propose a simplified and v5 compatible format while offering a smoother migration path with a legacy format.

# Detailed design

### Endpoints

Based on the [Database changes planned](https://github.com/strapi/rfcs/pull/52),we will now mainly work with the `documentId` within the API.

**Collection Types**

| Method   | Url                                             | Desc                                                                    |
| -------- | ----------------------------------------------- | ----------------------------------------------------------------------- |
| `GET`    | `/api/:contentType``                            | Find a list of documents                                                |
| `POST`   | `/api/:contentType`                             | Create a document                                                       |
| `GET`    | `/api/:contentType/:documentId`                 | Find a document                                                         |
| `PUT`    | `/api/:contentType/:documentId`                 | Update a document                                                       |
| `DELETE` | `/api/:contentType/:documentId`                 | Delete a document                                                       |
| `POST`   | `/api/:contentType/actions/:action`             | Actions on the collection of documents (bulk actions, custom action...) |
| `POST`   | `/api/:contentType/:documentId/actions/:action` | Actions on a specific document                                          |

**Single Types**

| Method   | Url                               | Desc                           |
| -------- | --------------------------------- | ------------------------------ |
| `GET`    | /api/:contentType                 | Find document                  |
| `PUT`    | /api/:contentType                 | Create or Update the document  |
| `DELETE` | /api/:contentType                 | Delete document                |
| `POST`   | /api/:contentType/actions/:action | Actions on the single document |

### Add `documentId`

Introduction of the `documentId` field and renaming of the old `id` to `entryId` or `entityId`

**Example**

```tsx
{
  "data": {
    "documentId": "clkgylmcc000008lcdd868feh",
    "entryId": "clkgylmcc000008lcdd868feh"
  },
  "meta": {
    "pagination": {
      "page": 1,
      "pageSize": 10
    }
  }
}
```

### Introduce a simplified format

We are planning to introduce a new version of the API response that flattens the structure and removes some of the complex nesting from v4.

To avoid conflicts between features & Content Type attributes we will keep the `meta` object but completely flatten the `attributes` & `data.attributes` sub paths.

**Before**

```json
{
  "data": {
    "id": 1,
    "attributes": {
      "title": "Article A",
      "locale": "en",
      "createdAt": "2023-01-01T00:00:00.000Z",
      "updatedAt": "2023-01-01T00:00:00.000Z",
      "publishedAt": "2023-01-01T00:00:00.000Z",
      "relation": {
        "data": {
          "id": 1,
          "attributes": {
            "name": "Category A",
            "locale": "en",
            "createdAt": "2023-01-01T00:00:00.000Z",
            "updatedAt": "2023-01-01T00:00:00.000Z",
            "publishedAt": "2023-01-01T00:00:00.000Z"
          }
        }
      }
    }
  },
  "meta": {
    "pagination": {
      "page": 1,
      "pageSize": 10
    }
  }
}
```

**After**

```json
{
  "data": {
    "documentId": "clkgylmcc000008lcdd868feh",
    "entryId": "cpmnyztbcc964008lcft812feo",
    "title": "Article A",
    "relation": {
      "documentId": "clkgylw7d000108lc4rw1bb6s",
      "entryId": 1,
      "name": "Category A",
      "meta": {
        "locale": "en",
        "createdAt": "2023-01-01T00:00:00.000Z",
        "updatedAt": "2023-01-01T00:00:00.000Z",
        "publishedAt": "2023-01-01T00:00:00.000Z"
      }
    },
    "meta": {
      "locale": "fr",
      "createdAt": "2023-01-01T00:00:00.000Z",
      "updatedAt": "2023-01-01T00:00:00.000Z",
      "publishedAt": "2023-01-01T00:00:00.000Z"
    }
  },
  "meta": {
    "pagination": {
      "page": 1,
      "pageSize": 10
    }
  }
}
```

### Separating meta attributes to avoid any current & future name conflicts

```json
{
  "data": {
    "documentId": "clkgylmcc000008lcdd868feh",
    "title": "Article A",
    "description": "My description",
    "meta": {
      "locale": "",
      "publishedAt": "",
      "createdAt": "",
      "createdBy": ""
    }
  }
}
```

Pros:

- No conflicts between user & system attributes
- Clear separation in different objects
- Easier extraction of all meta fields in one go

Cons

- Access is nested.
- Lesser discoverability (need to know it’s in meta) ⇒ the _SDK and/or Typescript typings would make the discoverability issue irrelevant_

```tsx
const locale = data.meta.locale;
const { locale } = data.meta;

// better separation
const { meta, ...attributes } = data;
```

### Low level **relational support**

In order to keep the old relational system, we will return the `entryId` to allow managing low level relations.

```json
{
  "data": {
    "id": "clkgylmcc000008lcdd868feh",
    "entryId": 1,
    "title": "..."
  }
}
```

# Release plan

Doing theses changes would break all the existing API Calls.

In order to reduce migration cost we are considering support a _deprecated_ response format:

- Keep the "id" name for the entry id
- Support fetching the API with the old IDs
- Keep the nested `data.attributes` format of v4

```tsx
{
  "data": {
    "documentId": "clkgylmcc000008lcdd868feh"
    "id": "clkgylmcc000008lcdd868feh",
    "attributes": {},
  }
}
```

- This format could be configured to be the default one during migration.
- We would support a query parameter to switch to the new format incrementally.

> The breaking changes are still forcing users to switch from calling the api with `id` to `documentId`

# Unresolved questions

- Should we use `id` or `documentId` in the new response format.
- Should we use `entryId` or `entityId` to represent the underlying Ids to use for low level relations
