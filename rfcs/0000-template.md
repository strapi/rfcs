- Start Date: 2019-06-21
- RFC PR: (leave this empty)

# Summary

- Introduce a global naming convention for controllers/services/models.
- Introduce getter APIs.

# Example

```js
'use strict';

module.exports = {
  find() {
    // access to a service
    strapi.service('admin::servicename');
    strapi.service('plugins::users-permissions.user');
    strapi.service('app::apiname.servicename');
    // or shortcut
    strapi.service('apiname.servicename');
  },
};
```

# Motivation

Currently, accessing a service, a controller or a model when customizing a Strapi application requires knowledge about Strapi's internal structure.

- The main goal of this proposal is to provide users with a clear naming convention on how to access some part of their business logic and plugins' business logic.

- This proposal will remove the naming collisions between apis that exist today.

- This proposal will also allow Strapi's core framework to evolve without breaking users' applications that rely on some internal apis.

# Detailed design

## Naming

Th first step is to define a naming convention so we can use it throughout Strapi's codebase.
The proposal is to create a naming convention to create a UID (unique identifier) for every service, controller and model in an application while avoiding collisions.

**Application**

```
app::api-name.thingy-name
```

**Plugins**

```
plugins::plugin-name.thingy-name
```

**Admin / Internals**

```
strapi::admin.thingy-name
```

## APIs

Now that every service, controller and model has a UID we need to make them accessible to users.

**Getters**

Getters are global function that will call a container to get a specific service, controller or model:

```js
strapi.service('app::api-name.service-name');
strapi.controller('plugins::plugin-name.controller-name');
strapi.model('app::api-name.model-name');
strapi.query('strapi::admin.model-name');
```

These getters can be implemented in a way that we could avoid using the `app::` prefix to access the application codebase.

```js
strapi.service('api-name.service-name'); // maps to strapi.service('app::api-name.service-name');
```

This syntax will only be available in the accessor functions.

For example: When refering to a model in the `Model.settings.json` file, the model name will need to have the `app::` prefix.

```json
{
  "attributes": {
    "key": {
      "collection": "app::api-name.model-name",
      "via": "refKey"
    }
  }
}
```

### Containers

Behind the **getters**, there are containers which contain the services, controllers and models.

The container APIs are not defined in this proposal because they will be part of a later proposal to build APIs for plugin developpers and improve end users programmatic customization possibilities.

# Tradeoffs

## Implications

Implementing this proposal means some refactoring in the core framework and the plugins will have to take place.

The main updates will be around models.

- Relations will change to point to a global model UID (no more plugin key there).
- Content-type and Content-manager apis will need to be refactored to use these UIDs
- The DB hooks (bookshelf and mongoose) will need to be refactored to account for these modifications too.

We will also need to refactor the current code to user the new **getters**.

## Migration

Now that the plugins and CRUD apis are in `node_modules`, upgrading is easy. The difficult part will be for users to migrate to those new APIs and mainly update there models manually.

## Documentation

A lot of work will have to be done to update and add the new documentation.

# Alternatives

Although creating this global naming convetion is mandatory to move Strapi forward, there are alternatives to the proposed naming convention with the same benefits:

**Path like**

```js
strapi.service('strapi/admin/servicename');
strapi.service('plugins/users-permissions/user');
strapi.service('app/apiname/servicename');
```

**Symbols**

```js
strapi.service('#admin.servicename'); // internals = #
strapi.service('@users-permissions.user'); // plugins = @
strapi.service('apiname.servicename');
```
