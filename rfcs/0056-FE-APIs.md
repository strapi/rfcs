- Start Date: 2023-09-19
- RFC PR: https://github.com/strapi/rfcs/pull/56

# Summary

The following document proposes new public facing APIs aimed at plugin developers to be able to augment the `EditView` of the content-manager, the need came from the design proposals of Draft & Publish and would most likely be implemented only in V5 of the product under the Draft & Publish scope.

This document tries to perform the minimum required work to retain backwards compatible functionality yet deliver a solid developer experience and pave the way for more productive and intuitive public facing FE orientated APIs.

# Example

```jsx
// admin/src/index.ts
import type { DocumentAction } from "@strapi/admin/admin";
import { PublishMultipleLocales } from "./documentActions";

export default {
  bootstrap(app) {
    app.addDocumentActions([PublishMultipleLocales]);

    app.addEditViewSidePanels([
      {
        title: "Review Workflows",
        content: <ReviewWorkflowsSidePanel />,
      },
    ]);

    app.addDocumentHeaderAction([
      {
        label: "Select Locale",
        options: [{ label: "english", value: "en" }],
      },
    ]);
  },
};

// admin/src/documentActions.ts
export const PublishMultipleLocales = {
  label: "Publish multiple locales",
  onClick: () => window.alert("you're publishing!"),
};
```

# Motivation

Current injectionZones let you inject components into areas of the content-manager, while this is a good it's a somewhat limiting API, the `addDocumentHeaderAction` & `addEditViewSidePanels` APIs while the former may seem very constricted provides a balance between user's being able to add actions quicker but not affect the design of the product, the latter is incredibly flexible but a much clearer typed system compared to the current implementation of `InjectionZones`. The `addDocumentActions` API is a new API that allows you to add actions to be rendered in the DocumentActions menu (this is new to V5).

Overall, we want to move towards more developer friendly APIs that are quicker to use, more intuitive yet do not cause concerns for the usability of our product.

# Detailed proposal

> ⚠️ All types are rough and have not been validated, there may be reason to loosen types based on design specs.

We’ll add the new APIs to the `app` class to keep with the current imperative model of introducing things to the Strapi admin panel. These APIs will be overloaded so you can apply the addition so it’s always rendered, or you can conditionally apply it based on the context provided.

All the APIs dedicated to the EditView of the content manager will receive similar context payloads should user’s opt in to use the function overload, it can be illustrated like this:

```tsx
interface EditViewContext {
  activeTab: "draft" | "published"; // room for more.
  id: string;
  meta: DocumentMeta;
  document: Document;
}

// TBD – placeholder type
interface DocumentMeta {
  publishedAt: string;
  createdAt: string;
  createdBy: string;
  // other dimensions e.g. locale
  [key: string]: any;
}

// TBD – placeholder type
interface Document {
  // User attributes
  [key: string]: any;
}
```

We can then apply it to the APIs in a uniform fashion which creates consistency:

```tsx
interface ContentManagerEditViewAPIs {
  funcName<TConfig extends object>(config: TConfig[]): void;
  funcName<TConfig extends object>(configFn: ConfigFn<TConfig>): void;
}

type ConfigFn<TConfig extends object> = (
  prevConfigs: TConfig[],
  ctx: EditViewContext
) => TConfig[];
```

Because the function receives the previous list of configs, we expect the complete list of configs to be returned, this means if a plugin required it could remove a configuration, or mutate a current configuration building upon one another e.g. the publish action would be able to be mutated by the `i18n` plugin correctly.

## **Injecting into the Header**

A `HeaderAction` is a constrained API enabling users to inject a specific set of items into the header toolbar section e.g. a `Select` or `Button`. It’s specifically restricted and we can make it looser as requirements surface and are validated.

```tsx
interface StrapiApp {
  addDocumentHeaderAction(action: HeaderAction[]): void;
  addDocumentHeaderAction(actionFn: HeaderActionFn): void;
}

type HeaderActionFn = (
  previousPanel: HeaderAction[],
  ctx: EditViewContext
) => HeaderAction[];

type HeaderAction = ButtonAction | SelectAction;

interface ButtonAction {
  label: string;
  icon?: React.ReactNode;
  type: "icon" | "default";
  onClick?: (
    event: React.MouseEvent<HTMLButtonElement>,
    ctx: EditViewContext
  ) => void;
}

interface SelectAction {
  label: string;
  options: Array<{ value: string; label: string; textValue?: string }>;
  onSelect?: (value: string, ctx: EditViewContext) => void;
}
```

## Injecting into the SideBar

A `PanelConfig` allows users to define a new configuration panel in the sidebar where they can put whatever they desire. e.g. Review Workflows could be it’s own panel.

```tsx
interface StrapiApp {
  addEditViewSidePanel(panel: PanelConfig[]): void;
  addEditViewSidePanel(panelFn: PanelConfigFn): void;
}

type PanelConfigFn = (
  previousPanel: PanelConfig[],
  ctx: EditViewContext
) => PanelConfig[];

interface PanelConfig {
  title: string;
  content: React.ReactNode;
}
```

For backwards compatibility we’ll render all components still using the injection-zone API in the default panel that holds the Document Actions.

## Extending Document Actions

A `DocumentAction` is a configuration object we pass and is rendered as part of the ActionButton menu in the EditView.

```tsx
// Partial type to illustrate an overload.
interface StrapiApp {
  addDocumentAction(action: DocumentAction[]): void;
  addDocumentAction(actionFunction: DocumentActionFunction): void;
}

type DocumentActionFunction = (
  previousActions: DocumentAction[],
  ctx: EditViewContext
) => DocumentAction[];

interface DocumentAction {
  label: string;
  onClick: (
    event: React.MouseEvent<HTMLButtonElement>,
    ctx: EditViewContext
  ) => void;
  icon?: React.ElementType;
  disabled?: boolean;
  dialog?: DialogOptions | NotifiationOptions | ModalOptions;
}

interface DialogOptions {
  type: "dialog";
  title: string;
  content?: React.ReactNode;
  onConfirm?: () => void;
  onCancel?: () => void;
}

interface NotifiationOptions {
  type: "notifcation";
  title: string;
  link?: {
    label: string;
    url: string;
    target?: string;
  };
  content?: string;
  onClose?: () => void;
  status?: "info" | "warning" | "softWarning" | "success";
  timeout?: number;
}

interface ModalOptions {
  type: "modal";
  title: string;
  content: React.ReactNode;
  footer: React.ReactNode;
  onClose?: () => void;
}
```

This gives users the ability to optionally render a `Notification`, a `ConfirmationDialog` or a `Modal` depending on the complexity of the action they’re performing. Consider the “Publish Multiple Locales” action, this would be very complex and most likely use the `Modal` option.

## Current InjectionZones

The current injection zones affected by this RFC are:

- `injectContentManagerComponent('editView', 'informations', conf)`

We would deprecate this one with a clear guide on moving towards adding your own panel and/or adding a document action instead. As we progress we could deprecate other injection zone APIs with much clear configuration objects.

# Tradeoffs

It also stands to reason that perhaps that something like the `SideBar` would be better off not having things injected, but rather you simply overwrite the entire component and `renderDefaults` around it, the issue with this could be that plugins would overwrite the component at X opportunities so perhaps this is better left for a user’s Strapi Application configuration file instead.

# Alternatives

The alternative would be to use the current InjectionZones pattern more freely, but that has very little type safety and the pattern is essentially an uncontrolled closed-box where you “just pass props” and assume it goes to the right component leading to hard to test circumstances and unpredictability which imo makes for bad software. Product has also had concerns over the viability of the current InjectionZones pattern...

# Unresolved questions

- Do we want to allow DocumentActions to be written as if they’re react components but return a data structure? This allows actions to be even more powerful, but their implementation in Strapi becomes potentially more complex.
- Should we live with injection zones in V4 and only deprecate them or completely remove them and do the additional work to define new APIs for the affected areas?
