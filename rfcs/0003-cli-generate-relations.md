- Start Date: 2019-12-18
- RFC PR: (leave this empty)

# Summary

* Support relation type in cli generators

# Motivation

Currently, it is necessary to use the frontend or have knowledge about the internal structure of strapi build up an api from scratch. The CLI isn't feature complete, so you can only create basic attributes. I would like to be able to bootstrap strapi apis for automated scenarios without manual interaction:
* Using strapi as a microservice
* Using strapi in a ci/cd pipeline
* Automated testing

# Detailed design

From a user perspective, the generate:api command should be extended by every possible data type existing in the frontend:

https://strapi.io/documentation/3.0.0-beta.x/cli/CLI.html#strapi-generate-api

 For optimal automation, it would be great to have an option to parse a json file containing the api definition. Due to the complexity of relations, for normal users an interactive cli would be great.

To add the complexe relation type, there're many different natures. The file containing the api definition should have the following structure:

```json
{
  "apis": {
    "<api_name>": {
      "attributes": {
        "<attribute_name>": {
          "type": "relation",
          "nature": "oneWay|manyWay|oneToOne|oneToMany|manyToOne|manyToMany",
          "relatedAttribute": "<related_api>"
        },
        "<attribute_name>": {
          "type": "media",
          "multiple": "true|false",
          "required": "true|false",
          "unique": "true|false"
        },
        "<attribute_name>": {
          "type": "string|text",
          "default": "<attribute_default>",
          "required": "true|false",
          "unique": "true|false",
          "minLength": "<min_length>",
          "maxLength": "<max_length>",
        },
        "<attribute_name>": {
          "type": "richtext",
          "default": "<attribute_default>",
          "required": "true|false",
          "minLength": "<min_length>",
          "maxLength": "<max_length>",
        },
        "<attribute_name>": {
          "type": "json",
          "required": "true|false",
          "unique": "true|false",
        },
        "<attribute_name>": {
          "type": "enumeration",
          "enum": ["<enum_1>", "<enum_2>", "..."],
          "default": "<enum>",
          "enumName": "<enum_name>",
          "required": "true|false",
          "unique": "true|false",
        },
        "<attribute_name>": {
          "type": "password",
          "required": "true|false",
          "minLength": "<min_length>",
          "maxLength": "<max_length>",
        },
        "<attribute_name>": {
          "type": "integer|biginteger|float|decimal",
          "default": "<attribute_default>",
          "required": "true|false",
          "unique": "true|false",
          "min": "<min>",
          "max": "<max>",
        },
        "<attribute_name>": {
          "type": "date",
          "default": "<attribute_default>",
          "required": "true|false",
          "unique": "true|false",
        },
        "<attribute_name>": {
          "type": "boolean",
          "required": "true|false",
          "unique": "true|false",
        }
      }
    }
  }

}
```



From a developers perspective, I think the following code in [before.js|https://github.com/strapi/strapi/blob/master/packages/strapi-generate-api/lib/before.js] on line 83 returns the attribute which is added into the models settings file:

```javascript
scope.attributes = scope.args.attributes.map(attribute => {
 if (_.isString(attribute)) {
   const parts = attribute.split(':');

   parts[1] = parts[1] || 'string';

   // Handle invalid attributes.
   if (!parts[1] || !parts[0]) {
     invalidAttributes.push(
       'Error: Invalid attribute notation `' + attribute + '`.'
     );
     return;
   }

   return {
     name: _.trim(_.deburr(parts[0].toLowerCase())),
     params: {
       type: _.trim(_.deburr(parts[1].toLowerCase())),
     },
   };
 } else {
   return _.has(attribute, 'params.type') ? attribute : undefined;
 }
});
```

To think about edge-cases, I include the following examples:

Relation from Source to Destination

## many to many

models/Source.settings.json

```json
"destionations": {
  "collection": "destionation",
  "via": "sources",
  "dominant": true
}
```

models/Destionation.settings.js

```json
"sources": {
  "collection": "source",
  "via": "destionations"
}
```

## hasone

models/Source.settings.json

```json
"destionation": {
  "model": "destionation"
}
```

## onetoone

models/Source.settings.json

```json
"destionation": {
  "model": "destionation",
  "via": "source"
}
```

models/Destionation.settings.js

```json
"source": {
  "model": "source",
  "via": "destionation"
}
```

## onetomany

models/Source.settings.json

```json
"destionations": {
  "collection": "destionation",
  "via": "source"
}
```

models/Destionation.settings.js

```json
"source": {
  "model": "source",
  "via": "destionations"
}
```

## manytoone

models/Source.settings.json

```json
"destionation": {
  "model": "destionation",
  "via": "sources"
}
```

models/Destionation.settings.js

```json
"sources": {
  "collection": "source",
  "via": "destionation"
}
```

## hasmany

models/Source.settings.json

```json
"destionations": {
  "collection": "destionation"
}
```

# Tradeoffs

* The solution demand some extension in the complexity of the cli:
  * In basic attribute types, the type is determined by the string given by the user. Relation doesn't use the json attribute "type" but "model" or "collection", which is determined by the form of relation
  * the current cli only edits the api file currently in creation. This is in hasone and hasmany cases no restriction. In every other form, the destination model setting has also to be edited. Therefore, the existing destination model has to be opened, parsed, modified and saved
* The work load could be high due to some major additions to cli
* A solution could be outside strapis core package, although currently cli is part of strapi
* I don't think this enhancment would break any other feature of strapi
* No migration cost
* A change would demand change on documentation how to use relations on cli

# Alternatives

Write a cli outside of strapi to offer a feature complete generation of an api

# Unresolved questions

* are there any other changes in the app when using relations (config, controllers, routes)?
* Is there any other way to bootstrap a strapi api on first start without manual interaction?
