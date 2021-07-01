- Start Date: 2021-07-01
- RFC PR: (leave this empty)

# Summary

In v4 we are build a new database layer to support the future evolution of Strapi and we are introducing a new query engine along with it.

The query engine should mostly be used by plugin developers, developers adding custom business logic to their applications and the Strapi core team.

# Example

```js
db.query("article").findMany({
  where: {
    title: {
      $startWith: "2021",
      $endsWith: "v4",
    },
  },
  populate: {
    category: true,
  },
});
```

# Motivation

We want to make sure we have a more flexible and powerful database layer to build future features of Strapi onto.

Moreover we want to provide a better experience and more possibilities for plugin developers and application developers to build advanced business logic into their plugins or applications.

This layer is really made for database queries and is at more low-level than our previous version. This layer isn't abstracting a lot of things so developers can have full control.

# Detailed design

## Query API

To work with entities in the database you can user the Query api:

**Example**

```js
const query = db.query("{modelUID}");

const entry = await query.findOne({
  where: { id: 1 },
  populate: { someRelation: true },
});
```

### `findOne(params)` ⇒ `Entry`

Finds the first entry matching the params

**Params**

| Param    | Type                           | Description                                |
| -------- | ------------------------------ | ------------------------------------------ |
| select   | `string[]`                     | Select some attributes to return           |
| where    | [`WhereParam`](#filtering)     | Filter                                     |
| limit    | `number`                       | Always `1` when used with `findOne`        |
| offset   | `number`                       | How many entries to skip                   |
| orderBy  | [`OrderByParam`](#ordering)    | How to order the entries                   |
| populate | [`PopulateParam`](#populating) | Select which relations should be populated |

**Example**

```js
const entry = await db.query("article").findOne({
  select: ["title", "description"],
  where: { title: "Hello World" },
  orderBy: { title: "DESC" },
  populate: { category: true },
});
```

### `findMany(params)` ⇒ `Entry[]`

Finds entries matching the params

**Params**

| Param    | Type                           | Description                                |
| -------- | ------------------------------ | ------------------------------------------ |
| select   | `string[]`                     | Select some attributes to return           |
| where    | [`WhereParam`](#filtering)     | Filter                                     |
| limit    | `number`                       | How many entries to return                 |
| offset   | `number`                       | How many entries to skip                   |
| orderBy  | [`OrderByParam`](#ordering)    | How to order the entries                   |
| populate | [`PopulateParam`](#populating) | Select which relations should be populated |

**Example**

```js
const entries = await db.query("article").findMany({
  select: ["title", "description"],
  where: { title: "Hello World" },
  orderBy: { title: "DESC" },
  populate: { category: true },
});
```

### `findWithCount(params)` => `[Entry[], number]`

Finds and counts entries matching the params

**Params**

| Param    | Type                           | Description                                |
| -------- | ------------------------------ | ------------------------------------------ |
| select   | `string[]`                     | Select some attributes to return           |
| where    | [`WhereParam`](#filtering)     | Filter                                     |
| limit    | `number`                       | How many entries to return                 |
| offset   | `number`                       | How many entries to skip                   |
| orderBy  | [`OrderByParam`](#ordering)    | How to order the entries                   |
| populate | [`PopulateParam`](#populating) | Select which relations should be populated |

**Example**

```js
const [entries, count] = await db.query("article").findWithCount({
  select: ["title", "description"],
  where: { title: "Hello World" },
  orderBy: { title: "DESC" },
  populate: { category: true },
});
```

### `create(params)` => `Entry`

Creates one entry and returns it

**Params**

| Param    | Type                           | Description                                |
| -------- | ------------------------------ | ------------------------------------------ |
| select   | `string[]`                     | Select some attributes to return           |
| populate | [`PopulateParam`](#populating) | Select which relations should be populated |
| data     | `object`                       | Input data                                 |

**Example**

```js
const entry = await db.query("article").create({
  data: {
    title: "My Article",
  },
});
```

### `update(params)` => `Entry`

Updates one entry and returns it

**Params**

| Param    | Type                           | Description                                |
| -------- | ------------------------------ | ------------------------------------------ |
| select   | `string[]`                     | Select some attributes to return           |
| populate | [`PopulateParam`](#populating) | Select which relations should be populated |
| where    | [`WhereParam`](#filtering)     | Filter                                     |
| data     | `object`                       | Input data                                 |

**Example**

```js
const entry = await db.query("article").update({
  where: { id: 1 },
  data: {
    title: "xxx",
  },
});
```

### `delete(params)` => `Entry`

Deletes one entry and returns it

**Params**

| Param    | Type                           | Description                                |
| -------- | ------------------------------ | ------------------------------------------ |
| select   | `string[]`                     | Select some attributes to return           |
| populate | [`PopulateParam`](#populating) | Select which relations should be populated |
| where    | [`WhereParam`](#filtering)     | Filter                                     |

**Example**

```js
const entry = await db.query("article").delete({
  where: { id: 1 },
});
```

## Bulk operation

> ⚠️ To avoid performance pitfalls, bulk operations do not work with relations (attach/detach...).
> We need to define how to handle this in a more suitable way.

### `createMany(params)` => `{ count: number }`

Creates multiple entries

**Params**

| Param | Type       | Description         |
| ----- | ---------- | ------------------- |
| data  | `object[]` | Array of input data |

**Example**

```js
await db.query("article").createMany({
  data: [
    {
      title: "ABCD",
    },
    {
      title: "EFGH",
    },
  ],
});

// { count: 2 }
```

### `updateMany(params)` => `{ count: number }`

Updates multiple entries matching the params

**Params**

| Param | Type                       | Description         |
| ----- | -------------------------- | ------------------- |
| where | [`WhereParam`](#filtering) | Filter              |
| data  | `object`                   | Array of input data |

**Example**

```js
await db.query("article").updateMany({
  where: {
    price: 20,
  },
  data: {
    price: 18,
  },
});

// { count: 42 }
```

### `deleteMany(params)` => `{ count: number }`

Deletes multiple entries matching the params

**Params**

| Param | Type                       | Description |
| ----- | -------------------------- | ----------- |
| where | [`WhereParam`](#filtering) | Filter      |

**Example**

```js
await db.query("article").deleteMany({
  where: {
    title: {
      $startsWith: "v3",
    },
  },
});

// { count: 42 }
```

## Aggregations

### `count(params)` => `number`

Counts entries matching the params

**Params**

| Param | Type                       | Description |
| ----- | -------------------------- | ----------- |
| where | [`WhereParam`](#filtering) | Filter      |

```js
const count = await db.query("article").count({
  where: {
    title: {
      $startsWith: "v3",
    },
  },
});

// 12
```

### Avg,Min,Max,Sum

> The rest of this API (avg,min,max,sum,groupby) will specified at a later time.

## Query Builder

When in need fore more control over the queries than with the Query API you can use the underlying query builder.

> The API will specified at a later time

## Filtering

Every operator is prefixed with a `$` to make their name explicit.

### Logical operators

#### `$and`

Every nested conditions must be `true`

**Example**

```js
const entries = await db.query("article").findMany({
  where: {
    $and: [
      {
        title: "Hello World",
      },
      {
        title: {
          $contains: "Hello",
        },
      },
    ],
  },
});
```

`$and` will be used implicitly when passing an object with nested conditions.

```js
const entries = await db.query("article").findMany({
  where: {
    title: "Hello World",
    rating: 12,
  },
});
```

#### `$or`

One or many nested conditions must be `true`

**Example**

```js
const entries = await db.query("article").findMany({
  where: {
    $or: [
      {
        title: "Hello World",
      },
      {
        title: {
          $contains: "Hello",
        },
      },
    ],
  },
});
```

#### `$not`

Negates the nested conditions.

**Example**

```js
const entries = await db.query("article").findMany({
  where: {
    $not: {
      title: "Hello World",
    },
  },
});
```

### Attribute Operators

#### `$not`

Negates nested condition. The `not` operator can be use in an attribute condition too.

**Example**

```js
const entries = await db.query("article").findMany({
  where: {
    title: {
      $not: {
        $contains: "Hello World",
      },
    },
  },
});
```

#### `$eq`

Attribute equals input value.

**Example**

```js
const entries = await db.query("article").findMany({
  where: {
    title: {
      $eq: "Hello World",
    },
  },
});
```

`$eq` can be omitted

```js
const entries = await db.query("article").findMany({
  where: {
    title: "Hello World",
  },
});
```

#### `$ne`

Attribute does not equal input value.

**Example**

```js
const entries = db.query("article").findMany({
  where: {
    title: {
      $ne: "ABCD",
    },
  },
});
```

#### `$in`

Attribute is contained in the input list.

**Example**

```js
const entries = db.query("article").findMany({
  where: {
    title: {
      $in: ["Hello", "Hola", "Bonjour"],
    },
  },
});
```

`$in` can be ommited when passing an array of values

```js
const entries = db.query("article").findMany({
  where: {
    title: ["Hello", "Hola", "Bonjour"],
  },
});
```

#### `$nin`

Attribute is not contained in the input list.

**Example**

```js
const entries = db.query("article").findMany({
  where: {
    title: {
      $nin: ["Hello", "Hola", "Bonjour"],
    },
  },
});
```

#### `$lt`

Attribute is less than the input value.

**Example**

```js
const entries = db.query("article").findMany({
  where: {
    rating: {
      $lt: 10,
    },
  },
});
```

#### `$lte`

Attribute is less than or equal to the input value.

**Example**

```js
const entries = db.query("article").findMany({
  where: {
    rating: {
      $lte: 10,
    },
  },
});
```

#### `$gt`

Attribute is greater than the input value.

**Example**

```js
const entries = db.query("article").findMany({
  where: {
    rating: {
      $gt: 5,
    },
  },
});
```

#### `$gte`

Attribute is greater than or equal to the input value.

**Example**

```js
const entries = db.query("article").findMany({
  where: {
    rating: {
      $gte: 5,
    },
  },
});
```

#### `$between`

Attribute is between the two input values.

**Example**

```js
const entries = db.query("article").findMany({
  where: {
    rating: {
      $between: [1, 20],
    },
  },
});
```

#### `$contains`

Attribute contains the input value.

**Example**

```js
const entries = db.query("article").findMany({
  where: {
    title: {
      $contains: "ABCD",
    },
  },
});
```

#### `$startsWith`

Attribute starts with input value.

**Example**

```js
const entries = db.query("article").findMany({
  where: {
    title: {
      $startsWith: "ABCD",
    },
  },
});
```

#### `$endsWith`

Attribute ends with input value.

**Example**

```js
const entries = db.query("article").findMany({
  where: {
    title: {
      $endsWith: "ABCD",
    },
  },
});
```

#### `$null`

Attribute is null`.

**Example**

```js
const entries = db.query("article").findMany({
  where: {
    title: {
      $null: true,
    },
  },
});
```

#### `$notNull`

Attribute is not null.

**Example**

```js
const entries = db.query("article").findMany({
  where: {
    title: {
      $notNull: true,
    },
  },
});
```

## Populating

With this new database layer, relations and components have a unified API for populating them.

You can populate by passing an array of attribute names

```js
db.query("article").findMany({
  populate: ["componentA", "relationA"],
});
```

For more advanced usage you can pass an object

```js
db.query("article").findMany({
  populate: {
    componentB: true,
    dynamiczoneA: true,
    relation: someLogic || true,
  },
});
```

You can also apply where filters and select or populate nested relations

```js
db.query("article").findMany({
  populate: {
    relationA: {
      where: {
        name: {
          $contains: "Strapi",
        },
      },
    },

    repeatableComponent: {
      select: ["someAttributeName"],
      orderBy: ["someAttributeName"],
      populate: {
        componentRelationA: true,
      },
    },

    // NOTE: We can't do the same on dynamic zones as their polymorphic nature prevents it for now.
    // We will explore some concepts like graphQL fragments to allow this later on
    dynamiczoneA: true,
  },
});
```

## Ordering

**Single**

```js
db.query("article").findMany({
  orderBy: "id",
});

// single with direction
db.query("article").findMany({
  orderBy: { id: "asc" },
});
```

**Multiple**

```js
db.query("article").findMany({
  orderBy: ["id", "name"],
});

// multiple with direction
db.query("article").findMany({
  orderBy: [{ id: "asc" }, { name: "desc" }],
});
```

**Relatonal ordering**

```js
db.query("article").findMany({
  orderBy: {
    author: {
      name: "asc",
    },
  },
});
```
