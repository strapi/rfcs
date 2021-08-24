- Start Date: (fill in today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)

# Summary

Backend Implementation for SalesAssist Action Engine .

The purpose of the action engine is to generate an action page that collects information from leads and will trigger actions when this information is submitted back to our system, this will execute a set of workflows that send the result to any # of third parties, resulting in a historical feed of all the different actions the user has made over time.

The main db structure of the system is the following [Database Diagram](https://dbdiagram.io/d/60e7cf6b7e498c3bb3eecfee)

**Entities**
- **Action**: Holds all actions of the system.
- **Action Type**: Types of action system will support product, document, forms, commerce, etc , 
- **Action Workflow**: List of the default workflows system requires this action to execute once it ends.
- **Action Systems**: Each action type will be powered by its own system domain , in order to encapsulate system complexity . Example: Forms, Commerce and Content will each have its own set of features 
- **Action data type** : Specify the datatype this action will use. We define here the json structure the action expect the frontend to send , if the frontend structure doesn't match all the required fields in the data type, backend will throw a exception 
- **Action companies** : Reference what action is this company has
- **Action companies receivers** : UUID identifier to match external request from the frontend to a specific Action. 
- **Action companies workflow** : Besides the default action workflow , specify custom workflow created by the user withing this company
- **Action companies systems modules** : Specify the list of entities from a specific action system tied to this action . Example: we would hold here the list of products, content, form referenced use in this action for the specific company in order to generate the action page.
- **Engagement** : Result of the current lead interaction with a action page. Main thing to point out is the changes of the action stage and the reference to the message id which is the json object of the result interaction
- **Business Verticals** : List of industries the system will have default actions

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

# FAQ

- What is a lead receiver? <br />
  Automated API designed to receive and process the information that is used to create a lead in the system. 

  The following information is required:
  - Name
  - Email
  - Phone
  - UserCompany the lead belongs to

  Can receive the following information:
  - Any additional field that belongs to the UserCompany's lead settings fields (eg: Credit Score, Interests, Type of Business, etc.)

  Can receive leads from:
  - HTML forms
  - API's via Json
  - Webhooks
  
<br />

# DB Structure

business_verticals

| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| id      | integer       | 1   |
| name   | Text        | automobile      |

<br />
action_data_types

| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| id      | integer       | 1   |
| uuid   | varchar        | 714d7361-9c3c-4223-8ccc-dabb5da47a36|
| name   | Text        | dealerContent      |
| definition   | Text        | {json structure of reciever}      |

| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| id      | integer       | 2  |
| uuid   | varchar        | 714d7361-9c3c-4223-8ccc-dabb5da47a36|
| name   | Text        | Finance Product      |
| definition   | Text        | {json structure of reciever}      |

*json structure

```
{
  "leads_uuid": "a9f04a86-9aa5-44da-8c52-2289bd1c7dce",
  "message": {
    "lang": "EN",
    "headers": {"User-Agent":"Mozilla\/5.0 (Macintosh; Intel Mac OS X 11_2_3)"},
    "status": "submitted",
    "source": "web",
    "hashtagVisited": "#hello",
    "text": "hello",
		"visitor_id": "62f3e6ee-39cf-4f26-837d-3515edd96b22"
  },
  "verb": "trade-walk"
}
```

<br />
action_types

| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| id      | integer       | 1   |
| name   | Text        | document      |


<br />
actions

| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| id      | integer       | 1   |
| uuid   | Text        | 614d7361-9c3c-4223-8ccc-dabb5da47a36||
| apps_id   | integer        | 2||
| actions_systems_id   | integer        | 1||
| companies_id   | integer        | 0||
| users_id   | integer        | 1||
| pipelines_types_id   | integer        | 1||
| name   | varchar        | Dealer Content||
| parent_id   | integer        | 0||
| description   | text        | This is a action fro dealer content||
| business_Verticals_id   | text        | 1||


<br />
action_systems

| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| id      | integer       | 1   |
| actions_id   | integer        | 1      |
| sub_system_id   | integer        | 1      |
| actions_data_type_id   | integer        | 1      |
| actions_type_id   | integer        | 1      |

| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| id      | integer       | 2   |
| actions_id   | integer        | 1      |
| sub_system_id   | integer        | 1      |
| actions_data_type_id   | integer        | 2      |
| actions_type_id   | integer        | 1      |

<br />
actions_workflow

| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| id      | integer       | 1   |
| workflow_id   | integer        | 1  |

<br />
systems

| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| id      | integer       | 1   |
| uuid   | varchar        | 788d7361-9c3c-4223-8ccc-dabb5da47a36|
| name   | Text        | Landing Page      |
