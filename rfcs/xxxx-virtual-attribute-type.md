- Start Date: 2020-08-16
- RFC PR: (leave this empty)

# Summary

It would be very useful to have "read only" attributes that are generated during "read" lifecycles (find, findOne)
using developer-defined functions. Currently it is possible to do so for the API (by extending the result in the
lifecycle functions). However, those attributes are not be available in any plugins (most importantly Content Manager
and Content Type Builder), since they are not defined in the model. By adding a "virtual" attribute type, you could
add attributes to a model without them being saved to the database, which would make them available to plugins too.

Related issues: [#3131](https://github.com/strapi/strapi/issues/3131)

Related PRs: [#2712](https://github.com/strapi/strapi/pull/2712), [#6433](https://github.com/strapi/strapi/pull/6433), 
[#7221](https://github.com/strapi/strapi/pull/7221)

Related discussions: [#7173](https://github.com/strapi/strapi/discussions/7173),
[#7465](https://github.com/strapi/strapi/discussions/7465)

# Example

## model.settings.json

```json
{
  ...
  "attributes": {
    "first_name": {
      "type": "string",
    },
    "last_name": {
      "type": "string",
    },
    "full_name": {
      "type": "virtual",
    }
  }
}
```

## model.js

```js
module.exports = {
  virtualAttributes: {
    full_name(entity) {
      return `${entity.first_name} ${entity.last_name}`;
    }
  }
}
```

# Motivation

Currently it is impossible to add custom fields to admin panel that are not written to the database. As shown in this
simple example, a user might want to show the full name as a field in the admin panel, while having two separate
fields for first and last names. Right now the only way of doing this is adding an attribute to the model and modifying
`create` and `update` lifecycles. This works, however it duplicates the data in the database, which is a bad solution.

This would also solve issue [#3131](https://github.com/strapi/strapi/issues/3131) in a much more practical way (in my
opinion), as this provides much much more freedom than simple formulas.

# Detailed design

The example shown before is pretty straightforward and should be self explanatory, but I will elaborate further. Before
Strapi returns an entity it should run a function defined in `virtualAttributes` for each attribute of virtual type.
Something akin to this (pseudo code):

```js
// attributes - model type attribute definitions
// entity - model entity object with populated attributes from the database
// virtualAttributes - object with virtual attribute functions of the model

for (const key in attributes) {
  if (attributes[key].type === 'virtual') {
    entity[key] = virtualAttributes[key](entity);
  }
}
```

Things that would need to be implemented (likely missing some):

+ Either Database Manager plugin (ideally) or both Database Connector Bookshelf & Mongoose plugins should not save virtual attributes to the database
+ Database Manager plugin should execute `virtualAttributes` functions after `find` and `findOne` queries
+ Virtual attribute type should be implemented in Content Type Builder plugin

# Tradeoffs

Personally I cannot see any real tradeoffs. This seems relatively not too difficult to implement, while adding a very
much needed feature to Strapi. It does not require any migration or interfere with existing concepts and features of
Strapi.

# Alternatives

Issue [#3131](https://github.com/strapi/strapi/issues/3131) suggest having formula fields, which I imagine would have
a similar implementation (or likely more difficult), while being much less flexible.

# Unresolved questions

Unsure if the `virtualAttributes` functions should run before or after the lifehooks. Probably either way works,
however it should definitely be noted in the documentation.
