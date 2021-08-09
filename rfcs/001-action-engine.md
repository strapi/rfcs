- Start Date: (fill in today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)

# Summary

Backend Implementation for SalesAssist Action Engine .

The focus of the action engine is the send a lead to a action page where they can execute one action and the result of this action is sent back to the system in order to execute a set of workflow to send this result to any # of third party, resulting in a historical feed of all the diff action this users has taken over time.

The main db structure of the system is the following [Database Diagram](https://dbdiagram.io/d/60e7cf6b7e498c3bb3eecfee)

**Entities**
- **Action**: structure that holds all action from the system
- **Action Type**: what type of action is this? product, document, forms, commerce, etc , just a reference to the diff type of actions in the system
- **Action Workflow**: what are the default workflows this action will execute upon its end. Why? if we want to make sure some action always run a specific workflow the system creators define
- **Action Systems**: each action page is powered by a specific subsystem , we can expect product action to function in the same way as form actions. So here we specify what system powers this action . Some initial system will be (forms, commerce, content)
- **Action date type** :  specify the datatype this action will use. Example? product , content (as of now). cant this be used by the action system? to be discuss
- **Action Companies** : reference what action is this company use / has created
- **Action companies workflow** : Besides the default action workflow , specify custom workflow created by the user withing this company
- **Action companies systems modules** : Specify the list of entities from a specific action system tied to this action . Example: we would hold here the list of products, content, form referenced use in this action for the specific company in order to generate the action page.
- **Engagement** : Result of the current lead interaction with a action page. Main thing to point out is the changes of the action stage and the reference to the message id which is the json object of the result interaction

# Example

If the proposal requires changes to the current API or the creation of new ones, add a basic code example.

# Motivation

Please make sure to explain the motivation for this proposal. 
It means explaining the use case(s) and the functional feature(s) this proposal is trying to solve. 

Try to only talk about the intent not the proposed solution here.

# Detailed design

Describe the proposal in details:

- Explaining the design so that someone who knows Kanvas can understand and someone who works on it can implement the proposal. 
- Think about edge-cases and include examples.

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Complexity
- Work load of implementation
- Can this be implemented outside of Kanvas's core packages
- How does this proposal integrate with the current features being implemented
- Cost of migrating existing Kanvas applications (is it a breaking change?)
- Does implementing this proposal mean reworking teaching resources (videos, tutorials, documentations)?

# Alternatives

What are the alternatives?

# Unresolved questions

Optional, but suggested for first draft proposals. What parts of the design are still TBD(To be defined)?
