- Start Date: (fill in today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)

# Summary

Backend Implementation for SalesAssist Action Engine .

The focus of the action engine is to send a lead to an action page where they trigger one action and the result of this is sent back to the system. This is turn executes a set of workflows that send the result to any # of third parties, resulting in a historical feed of all the different actions the user has made over time.

The main db structure of the system is the following [Database Diagram](https://dbdiagram.io/d/60e7cf6b7e498c3bb3eecfee)

**Entities**
- **Action**: Holds all actions of the system.
- **Action Type**: Types of action system will support product, document, forms, commerce, etc , 
- **Action Workflow**: What are the default workflows system requires this action to execute once it ends.
- **Action Systems**: Each action type will be powered by its own system domain , thus allowing use to encapsulate system complexity . Example: Forms, Commerce and Content will each have its own set of features 
- **Action date type** : Specify the datatype this action will use. We define here the json structure the action expect the frontend to send , if the frontend structure doesn't match all the required fields in the data type, backend will throw a exception 
- **Action companies** : Reference what action is this company use / has created
- **Action companies receivers** : Each action has its own receiver uuid in order to know when the frontend send a message from each action taken on the action pages
- **Action companies workflow** : Besides the default action workflow , specify custom workflow created by the user withing this company
- **Action companies systems modules** : Specify the list of entities from a specific action system tied to this action . Example: we would hold here the list of products, content, form referenced use in this action for the specific company in order to generate the action page.
- **Engagement** : Result of the current lead interaction with a action page. Main thing to point out is the changes of the action stage and the reference to the message id which is the json object of the result interaction

# Structure

Since our action engine at the end of the day is based on 3 system, CRM, PIM and Actions but at this moment we dont want to have to manage 3 different API's lets follow the same DDD as we did in NZXT but update it based on a proper [DDD](https://stitcher.io/blog/laravel-beyond-crud-01-domain-oriented-laravel)

- API (aka apps)
  - crm
    - config
    - controller
    - public
    - routes
    - test
  - pim (Product information management)
    - config
    - controller
    - public
    - routes
    - test
  - action_engine?
    - config
    - controller
    - public
    - routes
    - test
- Domains
  - Leads
  - Products
  - ActionsEngine
- Libraries
- Cli
- Storage

# Detailed design

At a App level , we need to provide different APIs for the frontend to connect the necessary information to generate action pages and endpoints to send the end results in order to generate engagement messages.

- Public routes will be always guided by a UUID, please avoid any int 
- Internal CRUD calls can be guided by UUID or Int 

We will hae 3 different APi
- api.crm.domain.com
- api.products.domain.com
- api.action-engine.domain.com

For public API calls from receiver or action pages , it will be important to always send
- lead uuid
- visitor uuid
- action uuid

# Motivation

We need to make the necessary changes to provide a clean structure for our action engine , allowing use a good structure for future growth

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Big changes to our folder structure
- Will need to update our Kubernetes app structure for the APi
