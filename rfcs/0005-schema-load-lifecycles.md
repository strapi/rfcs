- Start Date: 2020-06-16
- RFC PR: [#15](https://github.com/strapi/rfcs/pull/15)

# Summary

It would be great if we could manually execute code against the schema object for either Mongoose or Bookshelf. That way, better customization would be possible.  
This would have the added benefit of being able to add compound indexes, for example, which is not possible at the moment while using Mongoose.

Related issue: [#5972](https://github.com/strapi/strapi/issues/5972)
Related PR: [#6515](https://github.com/strapi/strapi/pull/6515)

# Example

(not according to the implementation below, but just as an example of what could be done with this)

```js
module.exports = {
  schemaLifecycles: {
    beforeSchemaLoad(client, orm, model) { // or ({client, orm, model})

    },
    afterSchemaLoad(client, orm, schema) { // Mongoose Schema object
      console.log(schema);
      schema.index({
        content: "text",
        keywords: "text",
        about: "text",
      },
      {
        weights: {
          content: 10,
          keywords: 5
        },
        name: "TextIndex",
      });
    },
};
```

# Motivation

While designing a search box, I found myself needing to use weighted search indexes.
That's not possible without either modifying `strapi-connector-mongoose` or messing with the database manually.

Additionally, I feel like this could open the doors to further customization and optimization of models.

# Detailed design

## Idea 1

I'd say we could have another lifecycle in the `models.js` file, which would look like this:

```js
module.exports = {
  schemaLifecycles: {
    beforeSchemaLoad(client, orm, model) {
      // ...
    },
    afterSchemaLoad(client, orm, schema) {
      // ...
    },
};
```

Alternatively, the function parameters will be provided inside an object `{client, orm, model}` instead - to be further discussed.

With this setup, models could have a lifecycle like this, which would allow for changes based on the current database.

```js
module.exports = {
  schemaLifecycles: {
    afterSchemaLoad(client, orm, schema) {
      if (client.type === "mongoose") {
        // do something
      } else if (client.type === "sqlite") {
        // do something else
      }
    }
};
```

# Tradeoffs

This change would go against Strapi's goal of being database agnostic.
However, I think there is a need for further customizing the database integration
without having to overwrite the dependencies using something like `patch-package`.

# Alternatives

- Use `patch-package` to implement this.
- Add indexes directly on the database.

The `patch-package` approach is actually doable and stable, so that is the best alternative at the moment.

# Unresolved questions

1. What to do with Bookshelf? Since it supports various different databases (`SQLite`, `ProtgreSQL`, `MySQL`, etc),
   it might need a way to differentiate between which is currently in use.
2. Where to actually call the lifecycle on Bookshelf. I've looked at the code but couldn't figure out the best place yet.
3. ~~Debate idea 1 vs idea 2~~
