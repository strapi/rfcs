- Start Date: 2020-04-09
- RFC PR: (leave this empty)

# Summary

Add new type of field (eg. **sorter**) in content-type-builder and content-manager to add ability to sort collection entries.

# Motivation

- Add ability to sort collection entries;
- Solves problem with some bussines scenarios when it's neccessary to be able to sort entries;

# Detailed design

- Add new field type based on integer as sorting type (plugin-content-type-builder);

![image](https://user-images.githubusercontent.com/2634448/111315390-00531680-866b-11eb-8ab7-136e957d3fed.png)

- Add support drag and drop in list view to be able to sort list (plugin-content-manager);

https://user-images.githubusercontent.com/2634448/111315496-1bbe2180-866b-11eb-92df-2b5eb136b146.mov

- Modify connector-bookshelf to integrate new type of field (same as email/password);

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Possibly wrong pattern, because all the fields should be seperated.
  Field creation, extending should be seperate logic with it's own env.

- It's part of the core untill strapi will support custom field creation.

# Alternatives

You can create seperate field for sorting, and manage it by incrementing numbers, but for dummy user it could cause more problems than solving it.

# Unresolved questions

- Name of field (personal suggestion - **sorter**)
- Buffet icon 🚀
- Maybe sorting should be core functionality without any sort field? (problem - that it will be possible to and it only on creation, like draft system)
- Pattern of creating field types, consider adding support to add custom fields, eg. google map, image map, etc.
