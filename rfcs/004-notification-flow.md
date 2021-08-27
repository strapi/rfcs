- Start Date: 2021-08-21
- RFC PR: (leave this empty)

# Summary

Kanvas Apps can send notifications, can handle different types of notifications, and can send them via multiple channels. The downside is that users cannot configure which type of notification they want to receive, via which channels, and with what frequency.

This proposal tries to address these 3 concerns and their implementation into the Kanvas Core.

# Detailed design 

[Database Diagram](https://dbdiagram.io/embed/612720576dc2bb6073bbee53)

**Entities:**
- **Users Notifications Settings:** Users settings per app of which notifications they want to receive and via which channels.
- **Notifications Types:** List of the current app amiable notification by system module.
- **Users Notifications Entity Frequency:** Specify the frequency at which Users get their notifications based on the weight specified by notifications types.

Before the system can handle sending notifications based on settings and frequencies, we have to provide endpoints for allowing uses to manage their settings.

# API
## Notification Frequency Endpoints

- `GET - /v1/users/{id}/notifications_frequency` : list all notification frequency for the current user
    ```
    {
        id: 3,
        entity_id : 100 {user_id},
        system_modules_id: 1 //based on the system module we will know the entity_namespace
        frequency_id : 1 //frequency reference for whats configure on this app
    }
    ```

- `GET - /v1/users/{id}/notifications_frequency?q=(entity_id:{id},system_modules_id:{id})` : list the frequency for the curren entity id on the system module
- `GET - /v1/users/{id}/notifications_frequency/{entity_id}/system_modules_id/{id}` : list the frequency for the curren entity id on the system module
    ```
    {
        id: 3,
        entity_id : 100 {user_id},
        system_modules_id: 1 //based on the system module we will know the entity_namespace
        frequency_id : 1 //frequency reference for whats configure on this app
    }
    ```

- `POST - /v1/notifications_frequency` : Create notification frequency
    ```
    {
        entity_id : 100 {user_id},
        system_modules_id: 1 //based on the system module we will know the entity_namespace
        frequency_id : 1 //frequency reference for whats configure on this app
    }
    ```

- `PUT - /v1/notifications_frequency/{id}` : Update notification frequency
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
## User Endpoints

- `GET - /v1/users/{id}` 
    ```
    {
        new_notification: 1 , //the user object will now return the total # of unread notification
        system_modules_id: 2 , //add the system module reference to this object so it easy for the frontend to handle frequency
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
        $this->model->apps_id = $this->app->getId();
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

### UserNotificationFrequencyController

```php
/**
 * Handle users notification Settings
 **/
class UsersNotificationEntityFrequencyController extends BaseController
{
    public function __construct()
    {
        $this->model = new UserNotificationEntityFrequency();

        $this->model->users_id = $this->userData->getId();
        $this->model->apps_id = $this->app->getId();
        $this->additionalSearchFields = [
            ['apps_id', ':', $this->apps->getId()],
            ['users_id', ':', $this->userData->getId()],
            ['is_deleted', ':', 0],
        ];
    }

    public function index();

    /**
     * Verify that the user info belongs to the current logged in user
     */
    public function getById();
    public function getByEntity(string $entityId, int $systemModulesId);
}

```

# Motivation

Kanvas Core already provides with notification management but we have to expand it and allow users to controller what type of notification they receives, so we need to expand the system by adding this model

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Adding parent_id to notification will add a new layer of complexity
- Should it live on the core or as a package?
# Alternatives

None we will have this issue on all our future apps
# Unresolved questions

Should this be in the core or as a package?