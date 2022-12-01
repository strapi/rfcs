- Start Date: 2022-12-01
- RFC PR: (leave this empty)

# Summary

Make it easy to add new component anywhere in between dynamic zone

# Example

If the proposal requires changes to the current API or the creation of new ones, add a basic code example.

# Motivation

Making it easy for content manager to add new component in between of the dynamic zone. Currently it's very difficult to add a new component in the middle or start of the dynamic zone if the dynamic zone contains large list of components. For example if the dynamic zone has 200 components and you wanted to add new component at 10th index then you have to add component at 201th index and then move it up one by one to place it on the required location.

![Strapi New Component in Dynamic Zone](https://user-images.githubusercontent.com/21098295/205133227-e8fd95ed-736d-439c-81f1-086e6a389cf4.png)

# Detailed design

Adding `+` Icon after each component in the dynamic zone component which will open a Modal to add new component and will add component at the specific location from where the `+` button was click.
In the modal we can show same component `ComponentPicker` plugin as used for `Add new component` button at the bottom of the dynamic zone.

# Tradeoffs

I can't think of any tradeoff for this implementaion.
I have already implemented this on a patch package i can create a PR in strapi admin for this.

# Alternatives

Drag and Drop feature in dynamic zone component

# Unresolved questions

Optional, but suggested for first draft proposals. What parts of the design are still TBD(To be defined)?
