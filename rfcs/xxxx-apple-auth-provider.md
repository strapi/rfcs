- Start Date: 14/09/2020
- RFC PR: _Work in progress_

# Summary

Implement new authentication provider : Apple SignIn

Apple mandatoring to add his own authentication system (Apple SignIn) if one other provider is use on an application (Facebook, Google, Twitter, Instagram...).
A lot of us use Strapi as an API with a mobile application (developed with Flutter, React, Expo, Swift, Kotlin, ...).

Today, Strapi is not provide a full documentation with configuration process for Apple SignIn.

The main idea of this RFC is to provide an authentication process with Apple SignIn in order to help all user to implement it quickly.

The second idea is to provide a simple process to configure Apple provider like others.

# Example

We need to change a couple of file, because Apple SignIn is a mixed with oAuth2 and OpenID authentication process.

### 1. Add Apple Grant main configuration

File `strapi-plugin-users-permissions/config/functions/bootstrap.js`
```javascript
...
apple: {
      enabled: false,
      icon: 'apple',
      key: '',
      secret: '', // First difficulty, the "secret" is a generated JWT token than need to be recreate usually
      callback: `${strapi.config.server.url}/auth/apple/callback`,
      scope: ['name', 'email'], // basic scope, but need to send a POST request
      nonce: true,
      state: true,
      custom_params: {
        response_type: 'code id_token',
        response_mode: 'form_post' // Force Grant to perform a POST response
      }
    },
...
```

### 2. Add a `POST` routes to received callback response from Apple (copy of GET)

File `strapi-plugin-users-permissions/config/routes.json`

```json
{
      "method": "POST",
      "path": "/connect/*",
      "handler": "Auth.connect",
      "config": {
        "policies": ["plugins::users-permissions.ratelimit"],
        "prefix": "",
        "description": "Connect a provider",
        "tag": {
          "plugin": "users-permissions",
          "name": "User"
        }
      }
    },
```

### Add Apple provider API call in `providers` services :


File `strapi-plugin-users-permissions/services/providers.js`
```javascript
...
const jwt = require('jsonwebtoken'); // require jwt decode
...
// /!\ DRAFT /!\ 
case 'apple': {
      const apple = purest({
        provider: 'apple',
        config: {
          apple: {
            'https://appleid.apple.com': {
              __domain: {
                auth: {
                  auth: { bearer: '[0]' },
                },
              },
              '{endpoint}': {
                __path: {
                  alias: '__default',
                },
              },
            },
          },
        }
      });
      apple
        .query()
        .post('auth/token') // Apple want receive only post request
        .auth(access_token)
        .request((err, res, body) => {
          if (err) {
            callback(err);
          } else {
            const idToken = jwt.decode(body.id_token);
            callback(null, {
              username: idToken.email.split('@')[0],
              email: idToken.email,
            });
          }
        });
      break;
    }
```

This part is a draft.

`Purest` has not been update since 4 years, many new providers are not available (like `discord` actually).
It's a sample config for Apple than want receive a Bearer access token.

Maybe it will be necessary to implement other library than `Purest`.

# Motivation

This feature is wanted by many users, and difficult to implement for now.
I've worked during a complete week in order to aggregate the most documentation as possible and help other to quickly implement this new provider on Strapi.

I wrote an issue to Simov/Grant  in order to have an example of implementation with the configuration above :
- [https://github.com/simov/grant/issues/193](https://github.com/simov/grant/issues/193)

I'm sure we can add it to Strapi, but with some modification on the plugin `strapi-plugin-users-permissions` :
- Find `Purest` configuration or used a new library (describe below) ?
- Perform GET/POST callback request with `access_token` readable
- Implement a 'generate token' system (except if Grant manage this part already ?)
- Manage private key `.p8` file to generate token for Apple authentication token ?

# Detailed design

None

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Complexity 
    * Medium

- Work load of implementation 
    * Development of this feature : 3 days with all information
    * Number of people need to test : min. 3

- Can this be implemented outside of Strapi's core packages
    - No, this need to be implemented into `strapi-plugin-users-permissions`, so people need to copy in `extensions` folder many files in order to have Apple SignIn.

- How does this proposal integrate with the current features being implemented
    - See example explained above

- Cost of migrating existing Strapi applications (is it a breaking change?)
    - No, we can add this feature and keep all other providers working well

- Does implementing this proposal mean reworking teaching resources (videos, tutorials, documentations)?
    - This provider require to write a complete documentation (but I've already done this part)

# Alternatives

Two main `node.js` package implement apple sign-in :
- [https://www.npmjs.com/package/apple-signin](https://www.npmjs.com/package/apple-signin)
- [https://www.npmjs.com/package/apple-auth](https://www.npmjs.com/package/apple-auth)

This is almost working, if we eject `Purest` and keep only `Grant`.
I've a working example available (private project, ask me to access on Slack).

# Unresolved questions
