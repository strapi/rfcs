- Start Date: 2023-09-05
- RFC PR: (leave this empty)

# Summary

When an option is not explicitly defined in a Strapi project configuration file, check for it in the environment variables with a predictable name.

**Note:** This RFC supports a WIP RFC for `Separate code and env from project configuration` and would also benefit from the config definition object mentioned in a WIP RFC for `Generate Complete/Commented Config Files` to ensure proper casting of data.

# Motivation

When new config properties are needed in a Strapi project, it currently requires users to update their configuration files and redeploy their Strapi app. If all configuration properties are available to be set from the environment variables, it only requires a server restart. This will give the ability to Cloud (and users in general) to add new or default values without modifying user code.

# Detailed design

## High level design

- When config is loaded during Strapi bootstrap, if an expected value from the `configuration definition object` (see `Separate code and env from project configuration` RFC if released) is not present in config files, we look for it in a specific location based on the property. For example, admin.js auditLogs enabled could be set from `STRAPI__ADMIN__AUDIT_LOGS__ENABLED`
  - **Alternative**: if we don't use the `configuration definition object`, we would  modify `config.get()` instead of the config loader to automatically look for any value requested in env if it does not already have a value. In this case we would lose data type casting and the safety of knowing if a config property exists in Strapi or not.
- The formula used will be:
  - Prefix: `STRAPI`
    - *Alternative*: Use `STRAPICONFIG` to be clear it is config files
    - All caps, with single underscores to map to camelCase properties
    - Double underscores between property names
    - By using double underscores, we don't break current variables like `STRAPI_LICENSE`
- ***ALTERNATIVE FORMULA***:
  - Use single underscores to separate property names
  - camelCase maps to UPPERCASE, so `admin.myProperty` would be found in `STRAPI_ADMIN_MYPROPERTY`
    - We do not have and will never allow two config properties that only differ in casing, so there will not be any naming conflicts due to casing
  - Non-config values such as `STRAPI_LICENSE` (see existing variables list below) will not be loaded into config because they are not present in the `configuration definition object`
- All possible configuration properties will be known (see `Generate Complete/Commented Config Files` RFC when released) and the `configuration definition object` can contain information to cast each property to the expected data type

## Existing `STRAPI_` prefixed environment variables

The following env variables are used within Strapi. In v5 we will take the opportunity to standardize as many as possible to config variables, this is not an exhaustive list but is a good starting point.

- `STRAPI_DISABLE_REMOTE_DATA_TRANSFER` → server.js as `STRAPI_SERVER_DATATRANSFER`
- `STRAPI_TELEMETRY_DISABLED` → server.js as `STRAPI_SERVER_TELEMETRY_DISABLED`
- `STRAPI_DISABLE_UPDATE_NOTIFICATIONS` → server.js as `STRAPI_SERVER_LOGGING_UPDATE_DISABLED`
- `STRAPI_HIDE_STARTUP_MESSAGE` → server.js as `STRAPI_SERVER_LOGGING_STARTUP_DISABLE`
- `STRAPI_UUID_PREFIX` → server.js as `STRAPI_SERVER_UUID_PREFIX`
- `STRAPI_PLUGIN_I18N_INIT_LOCAL_CODE` → plugins.js as `STRAPI_PLUGIN_I18N_INITLOCALCODE`

# Example

# Tradeoffs

- loading from `.env` can't (directly) be typed
  - However, we will be adding typings for config in v4 so typescript projects will be able to
  - Additionally, if we use a `config definition object` we can add data typings (and parsers such as for array types) to also cast each value, ensuring correct types even if the config is javascript. As long as we're aware of the config value (and we should be!) we will have a mapping for it.
  - The only cases we currently allow multiple data types are when we allow code, and since code can't be written in the environment variables we know to always accept those as (arrays of) strings from the environment (see `Separate code and env from project configuration` RFC if released)
- it may appear as “magic” to some users who are used to only looking in config files
  - It should be documented very clearly that Strapi attempts to load config first, and then falls back to `.env`

# Alternatives

## Config file for enabling/disabling features

Instead of allowing the DISABLE_ vars as special cases, create a features.js which would contain:

```js
// defaults will be on a per-feature basis
{
  // Currently available env variables
  // STRAPI_DISABLE_REMOTE_DATA_TRANSFER replacement
  dataTransfer: false, // disable feature that defaults to true

  // STRAPI_DISABLE_TELEMETRY replacement
  telemetry: false, // disable feature with false

  // probably not desired, but in theory
  // STRAPI_DISABLE_EE replacement
  ee: false, // to disable EE even when license is present
  // STRAPI_DISABLE_LICENSE_PING replacement
  licensePing: false // disable license ping

  // possible in the future, not currently implemented
  reviewWorkflows: true,
  auditLogs: true,
  someExperimentalFeature: true, // allow us to maintain feature flags in the future enable something that defaults to false until release

  // possible alternative to plugins.js pluginname.enabled to prevent strapi from loading plugins with this name, would need to explore how it would interact
  plugins: {
    somePlugin: false // explicitly block a plugin even if it's installed
  }
}
```

# Unresolved questions

- Will the following additional RFCs be approved and used as part of this one?
  - `Separate code and env from project configuration`
  - `Generate Complete/Commented Config Files`
- What format should we use to prevent conflicts? (Single/Double Underscores, etc)
