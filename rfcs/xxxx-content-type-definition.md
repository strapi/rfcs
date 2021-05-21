- Start Date: 2021-05-21
- RFC PR: (leave this empty)

# Summary

## Summary

We will introduce an improved version of the content type definition files. In this RFC we are only describing the attributes types and not the options for now. We hope to make them more understandable and usable both manually and programmaticaly (CLI / parsing ...)

## Example

```js
const articleDef = {
  attributes: {
    title: {
      type: 'string',
    },
    seo: {
      type: 'component',
      component: 'seo',
    },
    body: {
      type: 'dynamiczone',
      components: ['txt.image-text', 'txt.rich'],
    },
    images: {
      type: 'media',
      multiple: true,
    }
    category: {
      type: 'relation',
      relation: 'manyToOne',
      target: 'category',
    },
  },
};
```

## Motivation

The main goal of this change is to simplify readability of the content-type files so developers understand better what they are working with. This will also help make the internal database layer more standard for future evolution.

We will also start reserving some property names for later features (e.g `parsing`, `formating`, `validation`)

## Detailed design

Every attribute **MUST** have a `type` property.

### Medias

To simplify the usage of media attributes we are also simplifying the corresponding content-type attribute.

```js
const articleDef = {
  attributes: {
    images: {
      type: "media",
      multiple: true | false,
    },
  },
};
```

### Relations

The major change is for relations that will be change to use a `type` property of relation and a `relation` property to describe which relation it is.

**Examples**

**Many `Articles` have One `Category`**

```js
const articleDef = {
  attributes: {
    category: {
      type: "relation",
      relation: "manyToOne",
      target: "category",
    },
  },
};
```

This relation is in one direction only (from Article to Category). It means you cannot fetch or edit the `articles` from a `category` in the Admin Panel.

To make the relation visible in both directions you can use the following setup

```js
const articleDef = {
  attributes: {
    category: {
      type: "relation",
      relation: "manyToOne",
      target: "category",
      inversedBy: "articles",
    },
  },
};

const categoryDef = {
  attributes: {
    category: {
      type: "relation",
      relation: "oneToMany",
      target: "article",
      mappedBy: "category",
    },
  },
};
```

When specifying `inversedBy` on one side, it defines this side as the owner of the relationship. This is used to generate join tables for example.
