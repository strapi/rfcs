- Start Date: 2023-09-05
- RFC PR: (leave this empty)

# Summary

V5 is in preparation and will require making changes to the Content APIs (REST & GraphQL).

# Motivation

V5 Content Engine requires some breaking changes to the API, It is also the opportunity to simplify the API Response format following user feedbacks.

> The main point of this RFC is to propose a simplified and v5 compatible format while offering a smoother migration path with a legacy format.

# Detailed proposal

## Use the internal `documentId` as the only identifier the ContentAPI knows about.

- We will name it `id` for simplicity sake.

**Collection Types**

| Method | Url                                     | Desc                                                                    |
| ------ | --------------------------------------- | ----------------------------------------------------------------------- |
| GET    | `/api/:contentType`                     | Find a list of documents                                                |
| POST   | `/api/:contentType`                     | Create a document                                                       |
| GET    | `/api/:contentType/:id`                 | Find a document                                                         |
| PUT    | `/api/:contentType/:id`                 | Update a document                                                       |
| DELETE | `/api/:contentType/:id`                 | Delete a document                                                       |
| POST   | `/api/:contentType/actions/:action`     | Actions on the collection of documents (bulk actions, custom action...) |
| POST   | `/api/:contentType/:id/actions/:action` | Actions on a specific document                                          |

**Single Types**

| Method | Url                                 | Desc                           |
| ------ | ----------------------------------- | ------------------------------ |
| GET    | `/api/:contentType`                 | Find document                  |
| PUT    | `/api/:contentType`                 | Set / Update document          |
| DELETE | `/api/:contentType`                 | Delete document                |
| POST   | `/api/:contentType/actions/:action` | Actions on the single document |

```json
{
  "data": [
    {
      "id": "clkgylmcc000008lcdd868feh"
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "pageSize": 10
    }
  }
}
```

## Introduce a simplified format

The second change is the introduction of a new version of the API response format that flattens the structure.

- No more `data.attributes`

```json
{
  "data": {
    "id": "clkgylmcc000008lcdd868feh",
    "entryId": 1,
    "title": "Article A",
    "relation": {
      "id": "clkgylw7d000108lc4rw1bb6s",
      "name": "Category A"
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

Simplifying this so much will have some downsides that can’t be obvious considering we haven’t used any of the advantages of the v4 format:

- This new format will not allow relation pagination without adding some nesting

### Dealing with Strapi attributes vs user attributes

### Option 1

```json
{
  "data": {
    "id": "clkgylmcc000008lcdd868feh",
    "title": "Article A",
    "description": "My description",
    "locale": "",
    "localizations": [],
    "publishedAt": "",
    "release": "",
    "publishedBy": "",
    "createdAt": "",
    "createdBy": "",
    "updatedAt": "",
    "updatedBy": ""
  }
}
```

Pros:

- The simplest API a user can consume. everything is at the root level the user just uses plain data
- Simpler API schema & GraphQL schema
- Same behaviour but with good validation as in v4 so easier migration as it would contain the same attributes as in v4 `data.attrbitues`
- Including the attributes visually in the CTB when creating a content-type could be a good way to show they are internals but are part of the content-type and explains why you can’t use them.

Cons

- We prevent the use of specific attribute names ⇒ **One could argue it is even better as it will avoid confusions better than allowing for example: `internals.locale` & `locale` to exist at the same time from Strapi & a user attribute**
- Adding new internal attributes could conflict with existing user attributes ⇒
  **2 options:**
  - We release major versions more often for these use-cases & add a deprecation warning before that for users to rename some properties.
  - We provide good migration paths for attributes (difficult & error prone + need manual labor from users)

### Option 2

```json
{
  "data": {
    "id": "clkgylmcc000008lcdd868feh",
    "title": "Article A",
    "description": "My description",
    "internals": {
      "locale": "",
      "localizations": [],
      "publishedAt": "",
      "release": "",
      "publishedBy": "",
      "createdAt": "",
      "createdBy": "",
      "updatedAt": "",
      "updatedBy": ""
    }
  }
}
```

Pros:

- No conflicts between user & system attributes
- Clear separation in different objects
- Easier extraction of all `internal` fields in one go

Cons

- Access is nested.

```js
const locale = data.internals.locale;
const { locale } = data.internals;

// better separation
const { internals, ...attributes } = data;
```

- Confusion can happen between internal attribute and user attributes if we allow them to have the same names
- Lesser discoverability (need to know it’s in internals) ⇒ the _SDK and/or Typescript typings would make the discoverability issue irrelevant_
- This makes the DB & Entity service layers more complex

```js
entityService.find({
  filters: {
    locale: 'fr' // Would apply to the user attributes so we need a translation to something else.
    internal_locale: 'fr'
  },
  populate: {
    createdBy: true // same question and conflicts with properties
  }
})
// same issue for Database queries
```

### Option 3

The main alternative to consider is for meta attributes

```json
{
  "data": {
    "id": "clkgylmcc000008lcdd868feh",
    "title": "Article A",
    "description": "My description",

    "$locale": "",
    "$publishedAt": "",

    "_availableLocales": ["en", "fr"],
    "_createdAt": "1970-01-01T00:00:00.000Z"
  }
}
```

Pros:

- As flat as it gets
- Simple path access
- No conflicts between user & system attributes. But do we really want to allow that ?

```json
{
  "data": {
    "id": "kaozdianzoind",
    "locale": "myCustomAttributeLocale"
    "_locale": "fr",
  }
}
```

```js
// if so we also need the DB & Entity Service layers to accept this

entityService.find({
  filters: {
    locale: "heyo",
    _locale: "fr",
  },
});
```

Cons

- `_` has been long used for private fields and some eslint rules disallow it (https://eslint.org/docs/latest/rules/no-underscore-dangle). This can make destructuring a bit more annoying. ⇒ Using another special char would fix it and avoid any confusion with private fields (e.g `$`)
- No way to get all metas in one go without mapping over all fields and extracting based on prefix

```js
const { $locale } = document;
document.$locale

const meta = Object.entries(document).reduce((res, [key,value]) => key.startsWith('$') ? Object.assign(res, [key]: value) : res,{});
```

- Confusion can happen between internal attribute and user attributes if we allow them to have the same names
- _SDK & Typescript typings wouldn’t improve the cons that much except by adding more API surface for accessing the attributes_

We could consider using a more obvious prefix like `strapi` to reduce the potential confusions

```json
{
  "data": {
    "id": "kaozdianzoind",
    "locale": "myCustomAttributeLocale",
    "strapiLocale": "fr"
  }
}
```

```js
// if so we also need the DB & Entity Service layers to accept this

entityService.find({
  filters: {
    locale: "heyo",
    strapiLocale: "fr",
    strapiCreatedBy: "...",
  },
});
```

> This becomes less usable than option 2

## Metadata

The API can also return data that isn’t directly stored inside the document but attached to it. those metadata can be added in a sub `meta` key that will be used as.

```json
{
  "data": {
    "id": "clkgylmcc000008lcdd868feh",
    "title": "Article A",
    "description": "My description",
    "meta": {
      "localizations": {},
      "publicationState": "",
      "reviewWorkflow": {},
      "history": {}
    }
  }
}
```

`Metadata` are added information that you can’t work with directly through the main `content API` endpoints & don’t work the same way as simple attributes that you can select/filter/orderBy

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

### Release plan

Doing theses changes will break all existing clients application. In order to reduce migration cost we can offer an older version of the ContentAPI response format to ease the transition

```json
{
  "data": {
    "id": "entryId",
    "_documentId": "azodapzd",
    "attributes": {}
  }
}
```

- We can add a config to define the default format
- Add a query parameter to allow switch to the other format when needed to enable incremental migrations
