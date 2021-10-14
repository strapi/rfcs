- Start Date: 2021-10-14
- RFC PR: (leave this empty)

# Summary

Some times many users can interact with a single entity resulting in alot of notifications, we're attempting to solve this by creating a notification groping functionality that allows Kanvas to group notification if there are various notifications in a given period of time.

This solution will cotain:
- A flag to turn on and off this feature
- A soft cap and a hard cap to set the time of the agrupation
- A grouping mechanism based on the soft and hardcap

# Example

If the proposal requires changes to the current API or the creation of new ones, add a basic code example.

# Motivation

Enchance the way end users use notifications.

# Detailed design

Describe the proposal in details:

- Explaining the design so that someone who knows Kanvas can understand and someone who works on it can implement the proposal. 
- Think about edge-cases and include examples.

# Tradeoffs

# Alternatives

Moving this responsability to the client part

# Unresolved questions