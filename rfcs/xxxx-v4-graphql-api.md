- Start Date: 2021-05-24
- RFC PR: (leave this empty)

# Summary

Here is a first draft of what we would like the GraphQL API to look like in v4.

## Motivation

The idea behind those changes is to make the GraphQL API simpler, more powerful and customizable.
It'll also take advantage of the changes in the database layer (especially the new query engine).
Finally, it'll make it easier to add features over time.

## Overview

**url**: `/graphql`

## Retrieving Data

### Fetching Entities

#### Fetching a collection of document

**Request**

```graphql
{
  documents {
    data {
      id
      attributes {
        title
        locale
        categories {
          data {
            id
            attributes {
                name
            }
          }
        }
      }
    }
    meta {
      pagination {
        page
        pageSize
        total
        pageCount
      }
    }
  }
}
```

#### Fetching one entity

**Request**

```graphql
{
  document(id: 1, locale: "fr", publicationState: LIVE) {
    data {
      id
      attributes {
        title
        locale
        categories {
          data {
            id
            attributes {
              name
            }
          }
          meta {
            pagination {
              page
              pageSize
              total
              pageCount
            }
          }
        }
      }
    }
  }
}
```

### Sorting

**Parameter** `sort`

**Examples**

single value or array of value

```graphql
{
  documents(sort: "title") {
    data {
      id
    }
  }
}
```

```graphql
{
  documents(sort: "title:desc") {
    data {
      id
    }
  }
}
```

```graphql
{
  documents(sort: ["title:asc", "price:desc"]) {
    data {
      id
    }
  }
}
```

### Pagination

**Parameter** `pagination`

**Paginate by page**

- `pagination[page]`: page number (default: 0)
- `pagination[pageSize]`: page size (default: 100)

**Paginate by offset**

- `pagination[start]`: start value (default: 0)
- `pagination[limit]`: number of entities to return (limit: 100)

**Examples**

```graphql
{
  documents(pagination: { page: 0, pageSize: 10 }) {
    data {
      id
    }
    meta {
      pagination {
        page
        pageSize
        pageCount
        total
      }
    }
  }
}
```

```graphql
{
  documents(pagination: { start: 20, limit: 30 }) {
    data {
      id
    }
    meta {
      pagination {
        start
        limit
      }
    }
  }
}
```

### Filtering

**Context**

Two challenges were encountered when writing the RFC for the GraphQL filters syntax.

First, the symbols:
With the V4, we introduce a new query engine with its own syntax. In this syntax, filters starts with a `$` sign.
Unfortunately, GraphQL inputs don't allow keys to start with a symbol, thus preventing us to keep the `$` syntax here.

Then, the union types:
Since we want to allow custom scalar filters (`startsWith`, `contains`, `gt`, ...), we need to be able to pass objects with custom keys within. Ideally, we also wanted to be able to type `field: value` to create a shortcut to `field: { eq: value }`. Unfortunately (again), GraphQL don't allow input union type between scalars & object types. The decision was then to remove the shortcut, keeping only the filters object type for each scalar.

**Parameter** `filters`

**Examples**

```graphql
{
  documents(filters: { name: { eq: "test" }, or: [{ price: { gt: 10 }}, { title: { startsWith: "Book" }}] }) {
    data {
      id
    }
  }
}
```

## Entity

- `id`: entity id
- `attributes`: entity attributes

### Creating

```graphql
mutation createArticle {
  createArticle(data: { title: "Hello", relation: 1 }) {
    data {
      id
      attributes {
        title
      }
    }
  }
}
```

### Updating

**Request**

```graphql
mutation updateArticle {
  updateArticle(id: "1", data: { title: "Hello", relation: 1 }) {
    data {
      id
      attributes {
        title
      }
    }
  }
}
```

### Deleting

```graphql
mutation deleteArticle {
  deleteArticle(id: 1) {
    data {
      id
      attributes {
        title
      }
    }
  }
}
```

## Aggregation and Grouping (Connections)

> /!\ Warning: ETA: after v4.0.0
>
> The following specification is likely to change before its implementation after the release of Strapi V4.0.0

> Note: We're thinking about replacing the `values` attribute by a `nodes` attributes in the Connection queries,
> and we would like to have the opinion of our community members on this topic

### GroupBy

```graphql
query {
  restaurantsConnection {
    groupBy {
      open {
        key
      }
    }
  }
}
```

### Aggregate

```graphql
query {
  restaurantsConnection {
    aggregate {
      avg {
        nb_likes
      }
    }
  }
}
```

### Aggregate on relations

```graphql
query {
  restaurantsConnection {
    aggregate {
      avg {
        nb_likes
      }
    }

    values {
      categoriesConnection {
        aggregate {
          avg {
            price
          }
        }
      }
    }
  }
}
```

## Errors

To be defined in another RFC
