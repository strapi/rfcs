- Start Date: 2020-05-20
- RFC PR: (leave this empty)

# Summary

When adding a relation from one content-type to another, currently, strapi only lets us select a pre-existing entry in the specified related content-type. This RFCS will try to describe the motivation and possible implementation to allow the creation of content-type entries via relational input fields.

# Example
The example below shows a basic working example - If no entries match the input field, the user is given the option to **create** the related content-type entry (via the `mainField` attribute).

![enter image description here](https://user-images.githubusercontent.com/11134131/81402093-0ade7e80-9131-11ea-8183-7bf5cc9e7fc8.gif)

It should not involve breaking or changing the current API.

# Motivation

The motivation for this feature arose when setting up a hashtag tagging mechanism on a specific content-type. I needed to add many-to-many relations between a Post type and a Tags type (in order to both retrieve tags per post and posts per tag). It quickly became tedious to first create the tag entry, then come back to the post type to add the relation. Even more when dealing with posts with multiple new tags.

This feature could potentially speed up a content editor's workflow when dealing with a similar situation (any type of content tagging mechanism, wether it be hashtags, categories, whatever..)

# Detailed design

This feature, if accepted, would have to be treated in the content-manager plugin.

- A new key would need to be added to the content-type settings' schema for relational fields - ie: `allowCreate`. (in the demo above, I've hard-coded it in the model's configuration for quick prototyping)
- The changes should be reflected in the **View Configuration** page when editing a relational field to give the user the option to activate the feature.
- This feature could then be quickly implemented in the `SelectWrapper`, `SelectMany` and `SelectOne` components by leveraging `react-select/creatable` :

```javascript
/* ie in SelectMany.js : */
import Select from 'react-select';
import CreatableSelect from 'react-select/creatable';

/* choose appropriate component based on new allowCreate key */
const Component = allowCreate ? CreatableSelect : Select;

/* handle relational entry creation */
const onCreateOption = async (inputValue) => {
  try {
    const formattedData = formatData([
      await request(requestUrl, {
        method: 'POST',
        body: {
          [mainField]: inputValue
        },
        signal,
      })
    ]);

    addRelation({ target: { name, value: formattedData } });
    strapi.notification.success('created.success');

  } catch(e) {
    strapi.notification.error('failed.to.create');
  }
};

/* Later in render : */
<Component /* ... existing props ... */ onCreateOption={onCreateOption} />

```

Edge-case :
This feature would come in handy for simple content-types (categories, tags..) - but would probably be useless for relations involving complex types where the creation of a new entry based on the type's `mainField` isn't enough.

# Tradeoffs

Potential tradeoffs involved with this proposal :
- Might be implemented outside of Strapi's core packages (it's basically what I did for my use-case), although it may be interesting to have this in the core.
- As this will most likely have no breaking-changes, the risk seems low for conflicts with current features being implemented, migrations of existing Strapi applications.. Implementing the proposal will mean updating the documentation.

# Alternatives
Can't think of any.

# Unresolved questions
The design of the content-type's settings' schema in strapi-plugin-content-manager (validations etc..)
