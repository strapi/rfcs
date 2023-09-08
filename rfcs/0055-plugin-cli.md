- Start Date: 2023-09-08
- RFC PR: https://github.com/strapi/rfcs/pull/55

# Summary

In v5, we want to stop merging plugins into the Strapi application before they are bundled. We instead want to bundle each plugin individually before it is imported, just like for any other dependency. This change will bring several benefits:

- **Avoiding breaking changes:** When managing internal Strapi dependencies, we won’t have to worry about the side effects it could have on plugins that rely on those same dependencies. We’ll therefore be able to move quicker and keep up with the ecosystem.
  - **Example: design system updates**
    On the design system, we often want to do breaking changes, in order to improve the APIs. Doing this would require shipping a new major version. We own the Strapi Admin codebase, so we would be able to change the version of its dependency on `@strapi/design-system`, and do the manual changes required by the new APIs.
    However, because many plugins rely on the design system version used in the admin, they would all be affected by this change. And since we don’t control their codebase, we’re not able to ensure that they will apply the updates required by the new APIs. Many plugins would therefore be broken.
    The workaround that we’ve been using is to never delete or change APIs from the design system, but to instead add new components that work differently, and mark the old ones as deprecated. This is why we have [a v2 folder](https://github.com/strapi/design-system/tree/main/packages/strapi-design-system/src/v2) with newer versions of several design system components. It does work, but it means that we’re maintaining a lot more code, and it can become tricky for users to understand which API they should use.
    Instead, if plugins were pre-bundled, each would specify its own design system version, so we would be free to release breaking changes on the DS without worrying about the admin dependencies breaking the plugin ecosystem.
- **Improved stability:** Bundling plugins individually ensures that errors caused by unlisted dependencies are caught early, preventing runtime issues. See the issues closed by [this PR](https://github.com/strapi/strapi/pull/15968) for examples of bugs caused by the lack of pre-bundling.
- **Simplified bundling process:** Bundling each dependency individually is how bundlers work by default, so relying on that behaviour will allow us to simplify our Vite config.
- **Clear dependency management:** Plugins explicitly declare their dependencies, reducing reliance on the admin's dependencies and enabling easier version control of shared packages.
- **Better DX:** Plugin developers would be able to rely on their IDE’s Intellisense more, as their tools will be able to know where the plugins they are requiring come from. They will also be able to enable the [no-extraneous-dependencies](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/no-extraneous-dependencies.md) ESLint rule which is commonly used.
- **Build performance:** Since plugins are already bundled, it’s less code that Vite will need to bundle, which should result in a faster build.

# Motivation

The motivation behind this proposal is to address the limitations of the current plugin bundling process in Strapi v4. The existing method of merging plugins into the admin before bundling was a workaround due to the lack of a plugin API in v3, because plugins were just files that got merged into the Strapi app’s directory, instead of being proper standalone packages. However, with the introduction of the Plugin API in v4, it is now possible to bundle plugins individually.

We also want to take this opportunity to improve the experience of developing plugins. It should be easier to create plugins that are located outside of a Strapi app directory.

# Detailed Proposal

## Generator

We would keep relying on the strapi generate plugin command, in order to ease the migration of users to v5, and to keep the more generic and interactive strapi generate command. Users will need to provide the path where they want to create the plugin, in order to make it easy to create a plugin outside of an application.

## Bundling & dependencies

When migrating to Vite for v5, we would simplify the bundling config to stop handling Strapi plugins separately, and instead rely on the default Vite behaviour which [pre-bundles all dependencies.](https://vitejs.dev/guide/dep-pre-bundling.html). We would also remove [the aliases system](https://github.com/strapi/strapi/blob/main/packages/core/admin/webpack.alias.js) we have set up on v4 with Webpack, so that each plugin can be entirely free to pick its dependencies, without Strapi overriding anything.

To ease developers into that new behaviour, we would update the plugin generator so that all dependencies are listed by default. This means editing [the `plugin-package-json.hbs` file](https://github.com/strapi/strapi/blob/main/packages/generators/generators/src/templates/js/plugin-package.json.hbs#L1) so that `react`, `react-dom`, `react-router-dom` and `styled-components` would be moved to the dependencies, as they will be required during the production build.

We could also go a step further and add an ESLint config to the plugin generators, so that we could enable the [no-extraneous-dependencies](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/no-extraneous-dependencies.md) rule by default, in order to make it obvious to developers that they need to reference them in the `package.json`.

## Hot reloading

We want to make sure that plugins can be hot-reloaded, even if they are located outside of a Strapi application. Similarly to [Sanity’s plugin kit](https://github.com/sanity-io/plugin-kit/tree/main), we will rely on two pieces to make it work:

- Use [yalc](https://github.com/wclr/yalc). Yalc resolves a lot of issues (like breaking the rules of hooks) caused by yarn link and npm link by treating local packages as if they were normal ones, instead of relying on symlinks.
- Provide a link-watch script. It will be available in the package.json scripts and exposed to the user in a root `link-watch.ts` file. It uses nodemon to push changes to yalc when the Vite output changes.

This would be the default functioning, but users are free to delete those two pieces, and to rely on another solution if they prefer.

## Commands

We'll be aiming to add the following commands:

- plugin:build – Build your plugin
- plugin:watch – Watch your plugin for changes and rebuild
- plugin:verify - Verify that your plugin is valid e.g. dependencies and package.json are correct
- plugin:link – Link your plugin to a Strapi app that is not local to the plugin

The above list is not exhaustive, but should indicidate the direction we are taking the CLI.

# Tradeoffs

Potential issues we need to consider:

- **Migration of existing plugins:** Not all of our changes can be done in v4 as it would break a lot of community plugins who rely on undeclared dependencies. So developers will need to make some changes to make their plugins compatible with v5. These changes could be facilitated by a codemod tool.
- **Bundle size:** Each Strapi app that has plugins may have many versions of the same package installed. This could make the bundle size bigger, though hopefully this won’t have too much of an impact with tree shaking and other Vite optimisations.

# Alternatives

The alternatives have been explored in v3 and v4 and have led to the current unsatisfying situation. We’re therefore quite confident about the need for pre-bundling plugins.
