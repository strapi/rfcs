- Start Date: (fill in today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)

# Summary

Allow plugins to create `Custom Fields` by: 
- registering components in Backend API (to define the schema of our field)
- (optionally) registering React Components for edition in the Admin Frontend 

# Example

This is an example of implementing a field that represent a GPS point (latitude, longitude) and which will be editable with a marker on a map on the `content-manager` edition view

Let's say we use mapbox to render our map, and we have registered a new admin setting for the mapbox api key.

## Registering custom fields

First we need to create a component in our plugin to describe which data it will need.

Our component will be called `gps-point` and have two `float` fields (for `latitude` and `longitude`)  
_Those fields will be retreived in Rest API/GraphQL call such as any other components._

### Backend API

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

> :mailbox: **note:**  we have to determine a format for `collectionName` to avoid collisions


<details>
  <summary>
  :rocket:  Strapi v4 example
  </summary>

  > This is an example considering [Plugin API RFC](https://plugin-api-rfc.vercel.app/)
  
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

### Admin Frontend 

This **optional** step allow to register a React Component that would handle edition of our component in `content-manager`   

In this example, we want to display a map with a marker located at the coordinates of our `latitude` and `longitude` fields.  
We may also want the marker to be draggable to update models data.


**file:** `./plugins/custom-field-mapbox/admin/src/index.js`

```javascript
// InputMapboxPoint is the component that render the map
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

> :mailbox: **note:** we have to define propTypes for the inputs components  

<details>
  <summary>:rocket: Strapi v4 example</summary>

  > This is an example considering [Plugin API RFC](https://plugin-api-rfc.vercel.app/)

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

Due to the usage of `components` we will be able to associate our field to `models` like any other components, notice the `plugin` entry

Now when we will edit or create any `restaurant` we will have our mapbox displayed instead of the two native inputs for the floats values

**file:** `./api/restaurant/models/restaurant.settings.json`
```javascript
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

Allowing plugins to handle logic about custom fields will provide Strapi's community to create plugins such as:
- mapbox/google geo fields plugins
- seo fields plugins
- etc ...


# Detailed design

> :question: I want to discuss about the feature described in the example before going deeper in the implementation details

# Tradeoffs

As we are using `components` underneath, we are not able to add lifecycle hook.
We can not by this design use nested custom fields.

> :mailbox: **note:** About not creating real fields like `string`, `uid`, `time` is about separation of concerns: fields need to handle primitive values, but this rfc aim to  use structured data istead 
