- Start Date: (fill in today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)

# Summary

When using `fields` in queries, users should also be able to use an `$omit` parameter so they don't have to explicitly list all of the parameters they want, but only those they do not want.

# Motivation

In situations like [this one](https://github.com/strapi/strapi/issues/13945), users might want to keep most of the information from a populate relation, but not all of it.

# Detailed design

- Allow the `fields` parameter to either be an array, like in the current implementation, or an object that accepts an `$omit` key and its relative array of fields.
- If `$omit` is specified, the Lodash omit function is run on the list of available attributes for the polled content type, so that each `$omit` query is actually transformed into a regular `fields` query and no implementations in the query builder are touched.

# Example

If we take [the context outlined in this issue](https://github.com/strapi/strapi/issues/13945) as an example, the user could achieve is goal by using this query:

```js
{
  populate: {
    image: {
      fields: {
        $omit: ["related"];
      }
    }
  }
}
```

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- The feature has to be documented

# Alternatives

Using long `fields` queries to exclude one or few fields.

# Unresolved questions

Would `$omit` be a good name for the new parameter?
I took inspiration from the filter operators for consistency, but since this isn't a filter, it might not make sense.