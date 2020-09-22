- Start Date: 2020-09-14
- RFC PR: [https://github.com/strapi/rfcs/pull/19](https://github.com/strapi/rfcs/pull/19)

# Summary

Modify the component selector in the dynamic zone by adding the categories

# Motivation

We feel like the “category” on the component isn’t being used to it’s potential. When creating complex layout with lots of possible components, you end up with an horizontal scroll that make the experience quite uneasy. 

Below we seperated our blocks in two categories, one for the basic components, and the other for data associations. 

![categories](https://user-images.githubusercontent.com/505236/93095387-54247900-f6a3-11ea-96f5-d32429cc60a1.jpg)

By selecting them in a dynamic zone, it would be better to group them by category.

![liste](https://user-images.githubusercontent.com/505236/93095460-6acad000-f6a3-11ea-84c2-cda8c77adb17.jpg)

# Detailed design

The DynamicZone component of the content manager plugin should be modified.

`DynamicZone` should retrieve all the categories available and display for each category their components.

A collapsible menu would allow the user to access the components by category.

![collapse](https://user-images.githubusercontent.com/505236/93210607-310bcf00-f760-11ea-91df-ecfba777e99c.jpg)

NB: Both categories and components will be sorted by alphabetical order.

# Tradeoffs

What potential tradeoffs are involved with this proposal.

There should be no tradeoffs.

# Alternatives

No idea yet.

# Unresolved questions

- Add "pretty name" for categories
- UX for the selection of categories especially for projects with a large number of components