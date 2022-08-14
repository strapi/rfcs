- Start Date: 2022-02-28
- RFC PR: 

# Summary

Allow repeatable components to use a relation as their title, screenshots of change in rejected PR here: https://github.com/strapi/strapi/pull/12650


# Example

https://github.com/strapi/strapi/compare/master...hamilton-b:bug/relations-not-showing-title?expand=1

Before - see blank title on repeatable:
<img width="839" alt="before" src="https://user-images.githubusercontent.com/31095524/155505579-6833d850-8192-49d8-a5c1-f61e225dbac3.png">

With change, it will now use the title of the relation
<img width="880" alt="after" src="https://user-images.githubusercontent.com/31095524/155505674-5463bc6a-a1b5-4012-83a6-067bba46ca15.png">

# Motivation

To allow the UI to be intuitive when creating repeatables with only relation properties (only way to create ordered array of relations)

# Detailed design

Describe the proposal in details:

screenshots of change in rejected PR here: https://github.com/strapi/strapi/pull/12650

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Complexity
- Work load of implementation
- Can this be implemented outside of Strapi's core packages
- How does this proposal integrate with the current features being implemented
- Cost of migrating existing Strapi applications (is it a breaking change?)
- Does implementing this proposal mean reworking teaching resources (videos, tutorials, documentations)?

# Alternatives

Create option of relation array

# Unresolved questions

Optional, but suggested for first draft proposals. What parts of the design are still TBD(To be defined)?
