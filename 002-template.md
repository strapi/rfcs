- Start Date: 2021-07-23
- RFC PR: (leave this empty)

# Summary

The ADF have a huge growing, the ADF is a communication way between Dealer CRM, the ADF can has any information, using xml tags. Once Company can has a template for share the lead information, for example Demo Dealer 1 can share information of the lead, but Dealer Demo 2 can share information about lead, vehicle and vendor.

Both company must has a template for share the information. This template can be storage in the json params of ADF rules. We can set a default Rule with the default template , but if the company need a custom template must be duplicate the default rule and modify the new rule and set the custom template in the params.

For use ADF must use [Workflows](https://github.com/bakaphp/kanvas-packages/tree/0.3/src/WorkflowsRules)

# Example

## Add Rules to Entity

```
<?php 

declare(strict_types = 1);
use Kanvas\Packages\WorkflowsRules\Contracts\Traits\RulesTrait;

class Users extends BaseModel 
{
    use RulesTrait;
}

?>
```

## In Database
You must add new email templates called ADF, this template must use the volt syntax
<br>Example
```
<firstname>{{entity.firstname}}</firstname>
<lastname>{{entity.lastname}}</lastname>

```
## The entity ADF

This entity is a collection with the data of lead, and actual messages or feed. When this rule is fire, the Message entity sent the Lead as first argument and the actual feed as second arguments, after the workflow take the data and create a new array 
``` 
[
    'lead' => $arg[0]->toDocuments(),
    'messages; =>  $arg[1]->toDocuments(),
]
```

### Template Example
```
<?xml version="1.0" encoding="UTF-8"?>
<?adf version="1.0"?>
<adf>
    <prospect>
        <requestdate>{{$entity['messages'].created_at}}</requestdate>
        <vehicle>
            <year>{{$entity['messages'].messages()['year']}}</year>
            <make>{{$entity['messages'].messages()['make']}}</make>
            <model>{{$entity['messages'].messages()['year']}}</model>
        </vehicle>
        <customer>
            <contact>
                <name part="first">{{$entity['lead'].firstname}}</name>
                <name part="last">{{$entity['lead'].lastname}}</name>
                <phone>{{$entity['lead'].phone}}</phone>
                <email>{{$entity['lead'].email}}</email>
            </contact>
        </customer>
        <vendor>
            <contact>
                <name part="full">{{$entity['lead'].companies.name}}</name>
            </contact>
        </vendor>
    </prospect>
</adf>
```
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
