- Start Date: 2022-05-11
- RFC PR: https://github.com/strapi/rfcs/pull/42

# Summary

Custom fields are a way to extend Strapi’s capabilities. If you want to know more about custom fields from a user’s point of view, you can read [our non-technical RFC](https://github.com/strapi/rfcs/pull/40).

The purpose of this RFC is to determine how custom fields should be implemented from a technical point of view. It also specifies how developers can create their own custom fields.

# Motivation

The goals of this RFC are to:

- Specify what new APIs we will introduce to allow custom fields creation
- Show a detailed example of how a custom field can use that API
- Get feedback from plugin developers to know if this new feature matches their expectations

It is **not** the goal of this RFC to:

- Explore custom field capabilities from an end-user perspective. This has been addressed in the [non-technical RFC](https://github.com/strapi/rfcs/pull/40).
- Explain how the new APIs will be implemented technically
- Add a way to register new database types in the Strapi backend

# Detailed design

A custom field needs to be registered in both the admin and server.

### Server

For the server, Strapi needs to be aware of all custom fields to ensure that an attribute using a custom field is valid. To do this, we will expose a new `customFields` object with a `register` method on the `Strapi` instance. The custom field can then be added to Strapi during the server [register lifecycle](https://docs.strapi.io/developer-docs/latest/developer-resources/plugin-api-reference/server.html#register). 

```ts
interface CustomFieldServerOptions {
  // The name of the custom field
  name: string;
  // The name of the plugin creating the custom field
  plugin?: string;
  // The existing Strapi data type the custom field will use
  type: string;
}

strapi.customFields.register(
  options: CustomFieldServerOptions | CustomFieldServerOptions[]
);
```

### Admin

For the admin, we will expose a new `customFields` object with a `register` method on the `StrapiApp` instance. The custom field can then be added to Strapi during the admin [bootstrap lifecycle](https://docs.strapi.io/developer-docs/latest/developer-resources/plugin-api-reference/admin-panel.html#bootstrap) by providing the following object to `customFields.register()`.

```tsx

// You can also pass an array of objects to register several custom fields at once
app.customFields.register(
  options: CustomFieldAdminOptions | CustomFieldAdminOptions[]
);

interface CustomFieldAdminOptions {
  // The name of the custom field
  name: string;
  // The name of the plugin creating the custom field
  pluginId?: string;
  // The existing Strapi data type the custom field will use
  type: string;
  // The translation for the name
  intlLabel: IntlObject;
  // The translation for the description
  intlDescription: IntlObject;
  // The icon for the custom field
  icon?: React.ComponentType;
  // The components needed to display the custom field in the Content Manager
  components: {
    // Input component for the Edit view
    Input: () => Promise<{ default: React.ReactComponent }>;
    // Read only component for the List view
    View: () => Promise<{ default: React.ReactComponent }>;
  };
  // The settings to extend in the Content-Type Builder
  options?: {
    base: CTBFormSection[],
    advanced: CTBFormSection[],
    validator: (args) => object
  }
}

interface IntlObject {
  id: string;
  defaultMessage: string;
}

interface CTBFormSection {
  sectionTitle: IntlObject;
  items: CTBFormInput[]
}

interface CTBFormInput {
  name: string;
  description: InltObject;
  type: string;
  intlLabel: IntlObject;
}
```

### Schema.json

An attribute using a custom field looks like this in a content type’s schema:

```json
{
  "attributes": {
    "attributeName": {
      "type": "customField",
      "customField": "plugin::plugin-name.fieldname"
    }
  }
}
```

We use a namespace system to ensure several plugins can’t reference the same `customField` value. If a custom field is directly in a Strapi application and not in a plugin, then the prefix should be `global::` instead of `plugin::`.

# Example

Let’s say we want to create a “color” type in Strapi. After creating a plugin called color-picker, we can register a new custom field on the server using `strapi-server.js`.

```js
module.exports = {
  register({ strapi }) {
    strapi.customFields.register({
      name: 'color',
      plugin: 'color-picker',
      type: 'text',
    });
  },
};
```

Then, we can register that custom field on the frontend. We provide the components, the name of the field and its underlying data type. For more control, we also provide an option to let the developer user select the color format in the base options.

```jsx
// strapi-admin.js
register(app) {
  app.customFields.register({
    name: "color",
    pluginId: "color-picker",
    type: "text", // store the color as a text in 
    intlLabel: {
      id: "color-picker.color.label",
      defaultMessage: "Color"
    },
    intlDescription: {
      id: "color-picker.color.description",
      defaultMessage: "Select any color",
    } 
    icon: ColorIcon,
    components: {
      Input: async () => import(/* webpackChunkName: "input-component" */ "./Input"),
      View: async () => import(/* webpackChunkName: "view-component" */ "./View"),
    } 
    options: {
      base: [
        {
          sectionTitle: {
            id: 'color-picker.color.section.format',
            defaultMessage: 'Format',
          },
          items: [
            {
              intlLabel: {
                id: 'color-picker.color.format.label'),
                defaultMessage: 'Color format',
              },
              name: 'pluginOptions.color-picker.format',
              type: 'select',
              value: 'hex',
              options: [
                {
                  key: 'hex',
                  value: 'hex',
                  metadatas: {
                    intlLabel: {
                      id: 'color-picker.color.format.hex',
                      defaultMessage: 'Hexadecimal',
                    },
                  },
                },
                {
                  key: 'rgba',
                  value: 'rgba',
                  metadatas: {
                    intlLabel: {
                      id: 'color-picker.color.format.rgba',
                      defaultMessage: 'RGBA',
                    },
                  },
                },
              ],
            },
          ],
        },
      ],
      advanced: [],
      validator: (args) => ({
        'color-picker': yup.object().shape({
          format: yup.string().oneOf(['hex', 'rgba']),
        }),
      }),
    },
  });
}
```

When someone adds an attribute to a content type using our “color” custom field, the `schema.json` looks like this:

```json
{
  "attributes": {
    "attributeName": {
      "type": "customField",
      "customField": "plugin::color-picker.color",
      "pluginOptions": {
        "color-picker": {
          "format": "hex"
        }
      }
    }
  }
}
```

# Tradeoffs

- We do not yet offer the ability to create a custom type in Strapi
- When extending a custom field’s base and advanced forms in the Content-type Builder, it is not yet possible to import custom input components.
- We do not allow custom fields to use the relation, component or dynamic zone types.
- Custom fields can only be shared using plugins. This may seem overkill for simple use cases. That’s why we are considering adding a generator to create a plugin with only a custom field inside.
