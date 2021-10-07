- Start Date: (2021-10-07)
- RFC PR: (leave this empty)

# Summary

This RFC introduces a standard error format for Errors returned by Strapi's API.

# Example

```json
{
  "data": null,
  "error": {
    // corelation id
    "status": "", // http status
    "name": "", // Strapi error name (ValidationError)
    "message": "", // A human reable error message
    "details": {
      // error info specific to the error type
    }
  }
}
```

# Motivation

Our main goal with this standard is to improve user and developer experience.

We need to start by improving our API responses to be able to improve the Error displayed in the Admin Panel and in any SDK that consumes Strapi's API.

Here are some key points we want to improve:

- Usability of errors
- Errors more meaningful
- Error details to have more control
- Documenting Errors
- Standard Error format for integration builders and the Admin Panel
- Make Error responses as safe as possible by filtering out private details from the response (e.g: database error codes)

# Detailed design

## Content API Errors

In the Content API our goal is to expose to the end user a standard error format that can be handled consistently.

### REST

```json
{
  "data": null,
  "error": {
    "status": "", // http status
    "name": "", // Strapi error name (ValidationError)
    "message": "", // A human reable error message
    "details": // error info specific to the error type (can be anything depending on the error type)
  }
}
```

Here are a few other properties we are thinking about:

- `id`: corelation id. auto generated id that we can transmit along the stack and follow
- `timestamp`: a timestamps of when the error was thrown. can be usefull to trace error through multiple layers of services and third parties.
- `code` An error code we could add for practicality. We can have a variety of errors and finding good names will become harder and harder as we implement new error types. Using codes, we can separate error families from specific errors. (e.g we would have a `ValidationError` but multiple codes `VALIDATION_001`,`VALIDATION_002` ...)

### GraphQL

Following the recommendation on Apollo https://www.apollographql.com/docs/apollo-server/data/errors/#custom-errors.

We can create and wrap `ApplicationError` into `ApolloError` and pass the above properties into the error extensions.

## Admin Panel Errors

We can use the same format as in the REST API to be as consistent as possible.

## Programmatic Errors

We have multiple levels of errors

- Application
- Database
- Server
- HTTP

We need to make sure each level that is on top of another has access to the underlying errors to interpret and wrap them (e.g Application with Database Errors)

## Error list and meta

Here is a very short list of common errors we could have in different layers.

**Basic Errors**

- `Forbidden`
- `Unauthorized`
- `NotFound`
- `BadRequest`
- `InternalServerError`
- ....

**Application Errors**

- `ApplicationError`: generic application level error that can be transformed into `BadRequest` errors in http
- `InputError`: wrong input data (validation, type, format ....)
  - `QueryError`: wrong query parameters (sort / fields / populate / pagination / \_q / ...)
    - `FieldErrors`
    - `PopulateError`
    - `PaginationError`
    - `SortError`
    - `FiltersError`
  - `ValidationError`
- In case of db errors we can return a specific error type to wrap unique / not null constraints or a generic `ApplicationError`
- ....

**Database Errors**

All the database errors should be logged and replaced by `InternalServerError` to avoid leaking information

- `DatabaseError` (generic database errors wrapper)
- `Unique`
- `NotNullable`
- `InvalidType`
- ...

In our HTTP error middleware we will want to wrap all errors that are not `ApplicationError` or a subClass or an existing HTTP Error (forbidden, ...) into an `InternalServerError` to avoid leaking anything

### Errors with details

**ValidationError**

A common usecase is to provide detailed information about data validation to create a better user experience while inputing data into forms.

_Requirements:_

- Each property can have multiple errors.
- Each sub property can have multiple errors.
- We can have errors across multiple properties.
- We can have global errors.

To handle all those usecases a simple approach is to return an array of errors with a path, a message.

```json
{
  "error": {
    ...
    "name": "ValidationError",
    "meta": {
      "errors": [
        {
          "path": null,
          "message": "foo",
          "name": "SomeErrorName"
        },
        {
          "path": ["property", "subProperty"],
          "message": "property.subProperty is requried",
          "name": "RequiredError"
        },
        {
          "path": ["property", "subProperty"],
          "message": "property.subProperty min is 12",
          "name": "MinimumCharacterError"
        }
      ]
    }
  }
}
```

> We could also add a standard name or a code to each error for i18n and more context later

## Codes

We would like to introduce errors a bit later as an improvement to the new format.

Only errors returned by an API require a constant name or code that can be used as a reference for documentation or customization / localization etc. This would mainly apply to Errors returned in Strapi's REST or GraphQL APIs.

For Errors caught programmaticaly it could still make sense for traceability and flexibility to have codes but isn't a short term requirement. In javascript we can already work with Error types using `instanceof` to wrap/react to them.

### Code convention

**Ideas**

1. Codes can be two separate parts. One for the type of error and on for the error itself: (e.g: `01-001`)
2. Codes can be two separate parts to be more readable (e.g: `VALIDATION_001`)

# Alternatives

There are plenty alternatives for error types but we think this format is simple and clear enough to start with and gives enough room for future improvements.
