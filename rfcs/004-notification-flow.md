- Start Date: 8/25/2021
- RFC PR: (leave this empty)

# Summary

At this moment any kanvas app can send notification can handle different type of notifications and send them via multiple channels, but , users cant configure what type of notification they want to receive , via what channels and with what frequency.

This proposal tries to address those 3 questions and its implementation in the Kanvas Core.

# Detailed design 

[Database Diagram](https://dbdiagram.io/embed/612720576dc2bb6073bbee53)

**Entities:**
- Users Notification Settings: Users settings per app of what notifications he want to receives and via what channels 
- Notification Types : List of the current app amiable notification by system module 
- Notification Frequency: Specify the frequency with what Users get their notification based on the weight specify by notifications types

Before the system can handle sending notification based on settings and frequency we have to provide Endpoints for the frontend to manage user settings

# API
## Notification Frequency Endpoints

- `GET - /v1/users/{id}/notifications_frequency` : list all notification frequency for the current user
    ```
    {
        entity_id : 100 {user_id},
        system_modules_id: 1 //based on the system module we will know the entity_namespace
        frequency_id : 1 //frequency reference for whats configure on this app
    }
    ```

- `GET - /v1/users/{id}/notifications_frequency/{entity_id}` : Get a specific notification frequency from one entity
    ```
    {
        entity_id : 100 {user_id},
        system_modules_id: 1 //based on the system module we will know the entity_namespace
        frequency_id : 1 //frequency reference for whats configure on this app
    }
    ```

- `POST - /v1/notifications_frequency` : Update notification frequency
    ```
    {
        entity_id : 100 {user_id},
        system_modules_id: 1 //based on the system module we will know the entity_namespace
        frequency_id : 1 //frequency reference for whats configure on this app
    }
    ```

## Notification Endpoints

- `GET - /v1/users/{id}/notifications` : List of the user notification settings (all notification available for the app)
    ```
    {
        name : 'Notification name',
        notifications_type_id : x
        is_enabled : 1,
        parent_id : 0, //list only the notifications that are main 
        channel : {
            'json',
            'email'
        }
    }
    ```

- `GET - /v1/users/{id}/notifications/{id}` : Get one notification and it children
    ```
    {
        notifications_type_id : x
        is_enabled : 1
        channel : {
            'json',
            'email'
        },
        related : [
            {
                name : 'Notification name',
                notifications_type_id : x
                notification_key : Canvas\Notifications\Signup
                is_enabled : 1
                channel : {
                    'json',
                    'email'
                }
            }
        ]
    }
    ```

- `PUT - /v1/users/{id}/notifications/{id}` : Get one notification
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

- `DELETE - /v1/users/{id}/notifications` : delete all notification settings for this user
<br />

## User Endpoints

- `GET - /v1/users/{id}` 
    ```
    {
        'new_notification': 1 , the user object will now return the total # of unread notification
    }
    ```

# Examples 

In order to use this update we will have to provide 2 new Controllers on Kanvas Core and a Trait to use NotificationFrequency on any entity within the developers App.

### UserNotificationSettingsController

```php
/**
 * Handle users notification Settings
 **/
class UsersNotificationSettingsController extends BaseController
{
    public function __construct()
    {
        $this->model = new NotificationsSettings();

        $this->model->users_id = $this->userData->getId();
        $this->additionalSearchFields = [
            ['apps_id', ':', $this->apps->getId()],
            ['users_id', ':', $this->userData->getId()],
            ['is_deleted', ':', 0],
        ];
    }

    public function index();
    public function getById();
}

```


To allow entity controllers to use Notification frequency , we have to use the NotificationFrequency Trait , by reading the `$this->model` instance type , it will know what system module to use and provide the rest of the necessary endpoints 
```
trait NotificationFrequency
{

}
```


# Motivation

Kanvas Core already provides with notification management but we have to expand it and allow users to controller what type of notification they receives, so we need to expand the system by adding this model

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Complexity
- Work load of implementation
- Can this be implemented outside of Kanvas's core packages
- How does this proposal integrate with the current features being implemented
- Cost of migrating existing Kanvas applications (is it a breaking change?)
- Does implementing this proposal mean reworking teaching resources (videos, tutorials, documentations)?

# Alternatives

None we will have this issue on all our future apps
# Unresolved questions

Should this be in the core or as a package?