- Start Date: (fill in today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)

# Summary

Allow plugins to create `Custom Fields` by: 
- registering components in Backend API (to define the schema of our field)
- registering react components for edition in Admin Frontend

# Example

This is an example of implementing a Map view of GPS coordinates with Mapbox

## Registering custom fields

First we need to create a component in our plugin to describe which data it need

### Strapi Backend API

**file:** `./plugins/custom-field-mapbox/components/gps-point.json`
```javascript
{
  "collectionName": "custom_field_mapbox_components_gps_point",
  "info": {
    "name": "GPS point",
    "icon": "marker"
  },
  "options": {},
  "attributes": {
    "latitude": {
      "type": "float"
    },
    "longitude": {
      "type": "float"
    }
  }
}
```

> note: we have to determine a format for `collectionName`

<details>
  <summary>:rocket: Strapi v4 example: ./strapi-server.js</summary>
  
  **file:** `./strapi-server.js`
  ```javascript
  const components = require('./components');

  module.exports = (strapi) => {
    return {
      components,
    };
  };
  ```

  **file:** `./components/index.js`
  ```javascript
  const gpsPoint = require('./gps-point');

  module.exports = {
    gpsPoint,
  };
  ```

  **file:** `./components/gps-point.js`
  ```javascript

  module.exports = {
    collectionName: 'custom_field_mapbox_component_gps_point',
    info: {
      name: 'GPS point',
      icon: 'marker',
    },
    options: {},
    attributes: {
      latitude: {
        type: 'float',
      },
      longitude: {
        type: 'float',
      },
    },
  };
  ```
</details>

### Strapi Admin Frontend 

This optional step allow us to register a react component that would handle edition of our field
If no component is provided default component view is used

**file:** `./plugins/custom-field-mapbox/admin/src/index.js`

```javascript
import InputMapboxPoint from './components/InputMapboxPoint';

export default strapi => {
  const plugin = {
    // ...
  };

  strapi.registerField({ 
    pluginId: 'custom-field-mapbox',
    componentId: 'gps-point', 
    Component: InputMapboxPoint 
  });

  return strapi.registerPlugin(plugin);
};
```

<details>
  <summary>:rocket: Strapi v4 example: ./strapi-admin.js</summary>

  **file:** `./strapi-admin.js`

  ```javascript
  module.exports = () => {
    return {
      //...
      register(app) {
        app.fields.register({
          pluginId: 'custom-field-mapbox',
          componentId: 'gps-point', 
          Component: InputMapboxPoint 
        });
      },
    };
  };
  ```
</details>


## Using custom fields

We can use our field like any other component, notice the `plugin` entry

**file:** `./api/restaurant/models/restaurant.settings.json`
```javascript
// 
{
  "kind": "collectionType",
  "info": {
    "name": "restaurant",
    "description": "This represents the Restaurant Model"
  },
  "attributes": {
    "name": {
      "default": "",
      "type": "string"
    },
    "description": {
      "default": "",
      "type": "text"
    },
    "position": {
      "type": "component",
      "repeatable": false,
      "plugin": "custom-field-mapbox",
      "component": "gps-point"
    }
  }
}
```


# Motivation

The idea come from the [`Custom Fields`](https://portal.productboard.com/strapi/1-roadmap/c/10-custom-fields) on the roadmap  

It is actually not possible to cleanly extend the `content-manager` 
edition pages in the admin frontend _(unless overriding them in extentions)_.  

Using `components` to describe our fields reduce the complexity of this functionality  
We can see this as an evolution of `components`, instead of an evolutions of `fields`

Allowing plugins to handle logic about custom fields will provide Strapi community to create plugins such as:
- mapbox/google geo fields plugins
- seo fields plugins
- etc ...


# Detailed design

Describe the proposal in details:

- Explaining the design so that someone who knows Strapi can understand and someone who works on it can implement the proposal. 
- Think about edge-cases and include examples.

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Complexity
- Work load of implementation
- Can this be implemented outside of Strapi's core packages
- How does this proposal integrate with the current features being implemented
- Cost of migrating existing Strapi applications (is it a breaking change?)
- Does implementing this proposal mean reworking teaching resources (videos, tutorials, documentations)?

# Alternatives

What are the alternatives?

# Unresolved questions

Optional, but suggested for first draft proposals. What parts of the design are still TBD(To be defined)?
