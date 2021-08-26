- Start Date: 8/25/2021
- RFC PR: (leave this empty)

# Summary

At this moment any kanvas app can send notification can handle different type of notifications and send them via multiple channels, but , users cant configure what type of notification they want to receive , via what channels and with what frequency.

This proposal tries to address those 3 questions and its implementation in the Kanvas Core.

# Detailed design 

Description of the proposed feature or proposed changes.

If no notification settings configure -> send all
If one notification settings configured -> only set the ones enabled
They an enable or disable a notification type and send a json channels {'email', 'push'} , so we can confirm on the process() function what to send out

notification_frequency => 1, 2, 3 (all update | only relevant | never)

how will we handle frequency?
We will save the frequency on the user_config table and look it up by key

If the user doesn't have a frequency configure -> its 100%

how will the api handle updates to settings ?
PUT - /v1/users/{id}/notifications/{apps_id} ? sending a json
```
{
    notifications_type_id : x
    is_enabled : 1
    channel : {
        'json',
        'email'
    }
}
```

if the user doesn't have any notification settings configure , send all

how will the backend handle the notification_count per app?
on user config , set notification_count_{appId} and set the # of notifications unread

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
