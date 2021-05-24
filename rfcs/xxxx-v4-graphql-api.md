- Start Date: 2021-05-24
- RFC PR: (leave this empty)

# Summary

Here is a first draft of what we would like the REST API to look like in v4.

# Example

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
      }
      meta {
        availableLocales
      }
    }
  }
}
```

# Motivation

The idea behind those changes is to make the GraphQL API simpler, more powerful and customizable.
It'll also take advantage of the changes in the database layer (especially the new query engine).
Finally, it'll make it easier to add features over time.

# Detailed design

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
      }
      meta {
        availableLocales
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
      }
      meta {
        availableLocales
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
  documents(sort: "sort") {
    data {
      id
    }
  }
}
```

```graphql
{
  documents(sort: "-title") {
    data {
      id
    }
  }
}
```

```graphql
{
  documents(sort: ["title", "-price"]) {
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
  documents(pagination: { page: 0, pageSize: 10 }) {
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
  documents(pagination: { start: 20, limit: 30 }) {
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

**Parameter** `filters`

**Examples**

```graphql
{
  documents(filters: { name: "test", $or: [{ price: { $gt 10 }}, { title: { $startsWith: "Book" }}] }) {
    data {
      id
    }
  }
}
```

## Entity

- `id`: entity id
- `attributes`: entity attributes
- `meta`: entity metadata (available locales)

### Creating

```graphql
mutation createArticle {
  createArticle(data: { title: "Hello", relation: 1 }) {
    data {
      id
      attributes {
        title
      }
      meta {
        availableLocales
      }
    }
  }
}
```

**Response**

```json

```

### Updating

**Request**

```graphql
mutation updateArticle {
  updateArticle(params: { id: 1, locale: "fr" }, data: { title: "Hello", relation: 1 }) {
    data {
      id
      attributes {
        title
      }
      meta {
        availableLocales
      }
    }
  }
}
```

### Deleting

```graphql
mutation deleteArticle {
  deleteArticle(params: { id: 1, locale: "fr" }) {
    data {
      id
      attributes {
        title
      }
      meta {
        availableLocales
      }
    }
  }
}
```

## Aggregation and Grouping (Connections)

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
