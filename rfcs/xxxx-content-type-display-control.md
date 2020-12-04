- Start Date: 2020-12-04
- RFC PR: 

# Summary

Extend the Content Type format function and Model options definition by `isManaged` orio to allow hide / show specified Content Types on purpose.

# Example

Managed (typical) model
```
...
    attributes: {
        sample: {
            type: 'string'
        }
    },
...
```

Non-Manages (restricted) model
```
...
    options: {
        isManaged: false
    },
    attributes: {
        sample: {
            type: 'string'
        }
    },
...
```

# Motivation

All the motivation behind this proposal was to provide a possibity to the developers to exclude some of the Content Types from the RBAC and prevent listing them in the Left Menu. As a plugins dev team we've faced couple times the misunderstanding of our intention with plugin dedicated Models that should be handled by delivered plugin UI and logic instead of typical Content Manager flow for Collections.

Let's base on the real example for the [Comments plugin](https://github.com/VirtusLab/strapi-plugin-comments) and Models delivered by it.

1. Plugin provides `Comment` & `Report` models.
2. Plugin provides a custom moderation UI with number of additional features like tree view, block / unblock comment or thread, reviewing abuse reports. All the logic is highly customized and the flow, restrictions are not aligning to the Content Manager.
3. Delivered models are accessible via Collections section in the Left Menu what causes mistakes and gets rid of number of functionalities delivered with plugin, can affect the expected data consistency, relations etc. - [example issue raised recently](https://github.com/VirtusLab/strapi-plugin-comments/issues/14) 

As a plugin developer I would like to be able to restrict access (even for administrators) to some models that my plugin provides to be sure that end user will interact with it by expected way, without loosing the experience.

# Detailed design

- provide `isManaged` option in the model schemas to not affect built-in & hidden models. By default this prop is `true`, so nothing changes to the existing models
- extend `filterLinks` function to with both, built-in `isDisplayed` prop and custom `isManaged` prop
- display proper `(restricted)` indication for non-managed models in the RBAC plugin and disable the checkboxes

# Tradeoffs

- It have to be implemented as part of core implementation to avoid creating extensions in various areas (RBAC, admin)
- Update of training materials for RBAC might be required, but I think just the documentation update (also provided) is enough as by default

# Alternatives

The only alternative is to provide an extension repository from where developers might pull the changes and manually move to their existing projects. Anyway this solution is not a real option as will increase the complexity of migration to the new versions of Strapi.

# Unresolved questions

Optional, but suggested for first draft proposals. What parts of the design are still TBD(To be defined)?
