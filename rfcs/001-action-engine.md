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
- **Action companies** : reference what action is this company use / has created
- **Action companies recievers** :  each action from a company needs to generate its own receiver
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
  - pmi
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

# Detailed design

At a App level , we need to provide the different API's for the frontend to connect the necessary information to generate action pages and endpoints to send the end results in order to generate engagement messages.

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
