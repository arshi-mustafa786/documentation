---
title: Custom fields reference - Strapi Developer Docs
description: Learn how you can use custom fields to extend Strapi's content-types capabilities.
sidebarDepth: 3
canonicalUrl: https://docs.strapi.io/developer-docs/latest/development/custom-fields.html
---

# Custom fields

<!-- TODO: add links to user guide section(s) somewhere -->
Custom fields extend Strapi’s capabilities by adding new types of fields to content-types. Once created or installed, custom fields can be used in the Content-Types Builder and Content Manager just like built-in fields.

The present documentation is intended to custom fields creators: it describes which APIs and functions developers must use to create a new custom field. The [user guide](#) describes how to install and use custom fields from Strapi's admin panel.

<!-- TODO: uncomment and adjust content when blog post is ready -->
<!-- ::: strapi Prefer to learn by building?
If you'd rather directly jump to a concrete example, see our [Creating a color custom field guide](#) page for step-by-step instructions on how to build your first custom field from scratch.
::: -->

Custom fields are a specific type of Strapi [plugins](/developer-docs/latest/development/plugins-development.md) that include both a server part and an admin panel part. Both parts should be registered separately before a custom field is usable in Strapi's admin panel.

::: note NOTES
* Once registered, custom fields can be used in [models' `schema.json` files](/developer-docs/latest/development/backend-customization/models.md#model-schema).
<!-- * The section linked in the bullet point above will be part of another PR, but basically it's about mentioning that in schema files, custom fields are defined with `type: 'custom-field'` and a possible additional `customField` key for the uid (`global::…` or `plugin::…`), plus possibly additional keys (e.g. `options` for a `select` setting, etc.) -->
* Though the recommended way to add a custom field is through creating a plugin, app-specific custom fields can also be registered within the global `bootstrap` and `register` [functions](/developer-docs/latest/setup-deployment-guides/configurations/optional/functions.md) found in `./src/index.js`.

:::

<!-- TODO: make sure this # exists in the backend custom. > models docs — will come in another PR -->

<!-- ? should we document this? 👇 this was described in the [technical RFC](https://github.com/strapi/rfcs/blob/3eb034e169746558315d719ca5fb49cec854640a/rfcs/xxxx-custom-fields-api.md#motivation) but I'm unsure about what to do with it -->
<!-- ::: note
Custom fields can not be used to register new database types in the Strapi backend.
::: -->

## Registering a custom field on the server

::: prerequisites
!!!include(developer-docs/latest/development/snippets/custom-field-requires-plugin.md)!!!
:::

Strapi's server needs to be aware of all the custom fields to ensure that an attribute using a custom field is valid.

The `strapi.customFields` object exposes a `register()` method on the `Strapi` instance. This method is used to register custom fields on the server during the plugin's server [register lifecycle](/developer-docs/latest/developer-resources/plugin-api-reference/server.md#register).

`strapi.customFields.register()` registers one or several custom field(s) on the server by passing an object (or an array of objects) with the following parameters:

| Parameter                      | Description                                       | Type     |
| ------------------------------ | ------------------------------------------------- | -------- |
| `name`                         | The name of the custom field                      | `String` |
| `plugin`<br/><br/>(_optional_) | The name of the plugin creating the custom fields<br/><br/>If omitted, the custom field is registered within the `global` namespace.          | `String` |
| `type`                         | The data type the custom field will use           | `String` |

::: note
Currently custom fields can not add new data types to Strapi and should use existing, built-in Strapi data types (e.g. string, number, JSON — see [models documentation](/developer-docs/latest/development/backend-customization/models.md#model-attributes) for the full list).
:::

<!-- TODO: check this example with developers or create another one. The goal was to have an example different from the advanced color picker described in the guide. -->
::: details Example: Registering an example "color" custom field on the server:

```js
// path: ./src/plugins/my-custom-field-plugin/strapi-server.js

module.exports = {
  register({ strapi }) {
    strapi.customFields.register({
      name: 'color',
      plugin: 'color-picker',
      type: 'text',
    });
  },
};
```

:::

## Registering a custom field in the admin panel

::: prerequisites
!!!include(developer-docs/latest/development/snippets/custom-field-requires-plugin.md)!!!
:::

Custom fields must be registered in Strapi's admin panel to be available in the Content-type Builder and the Content Manager.

The `app.customFields` object exposes a `register()` method on the `StrapiApp` instance. This method is used to register custom fields in the admin panel during the plugin's admin [bootstrap lifecycle](/developer-docs/latest/developer-resources/plugin-api-reference/admin-panel.md#bootstrap).

`app.customFields.register()` registers one or several custom field(s) in the admin panel by passing an object (or an array of objects) with the following parameters:

| Parameter                        | Description                                                              | Type                  |
| -------------------------------- | ------------------------------------------------------------------------ | --------------------- |
| `name`                           | Name of the custom field                                             | `String`              |
| `pluginId`<br/><br/>(_optional_) | Name of the plugin creating the custom field<br/><br/>If omitted, the custom field is registered within the `global` namespace.                          | `String`              |
| `type`                           | Existing Strapi data type the custom field will use                  | `String`              |
| `icon`<br/><br/>(_optional_)     | Icon for the custom field                                            | `React.ComponentType` |
| `intlLabel`                      | Translation for the name                                             | [`IntlObject`](https://formatjs.io/docs/react-intl/) |
| `intlDescription`                | Translation for the description                                      | [`IntlObject`](https://formatjs.io/docs/react-intl/) |
| `components`                     | Components needed to display the custom field in the Content Manager (see [components](#components))            |
| `options`<br/><br/>(_optional_)  | Options to be used by the Content-type Builder (see [options](#options)) | `Object` |

<!-- TODO: check this example with developers or create another one. The goal was to have the most simple example, different from the advanced color picker described in the guide. -->
::: details Example: Registering an example "color" custom field in the admin panel:

```jsx
// path: ./src/plugins/color-picker/strapi-server.js

register(app) {
  app.customFields.register({
    name: "color",
    pluginId: "color-picker", // the custom field is created by a color-picker plugin
    type: "text", // the color will be stored as text
    intlLabel: {
      id: "color-picker.color.label",
      defaultMessage: "Color",
    },
    intlDescription: {
      id: "color-picker.color.description",
      defaultMessage: "Select any color",
    } 
    icon: ColorIcon,
    components: {
      Input: async () => import(/* webpackChunkName: "input-component" */ "./Input"),
      View: async () => import(/* webpackChunkName: "view-component" */ "./View"),
    } 
    options: {
      base: [
        /*
          Declare settings to be added to the "Base settings" section
          of the field in the Content-Type Builder
        */ 
        {
          sectionTitle: { // Add a "Format" settings section
            id: 'color-picker.color.section.format',
            defaultMessage: 'Format',
          },
          items: [ // Add settings items to the section
            {
              /*
                Add a "Color format" dropdown
                to choose between 2 different format options
                for the color value: hexadecimal or RGBA
              */
              intlLabel: {
                id: 'color-picker.color.format.label',
                defaultMessage: 'Color format',
              },
              name: 'options.format',
              type: 'select',
              value: 'hex', // option selected by default
              options: [ // List all available "Color format" options
                {
                  key: 'hex',
                  value: 'hex',
                  metadatas: {
                    intlLabel: {
                      id: 'color-picker.color.format.hex',
                      defaultMessage: 'Hexadecimal',
                    },
                  },
                },
                {
                  key: 'rgba',
                  value: 'rgba',
                  metadatas: {
                    intlLabel: {
                      id: 'color-picker.color.format.rgba',
                      defaultMessage: 'RGBA',
                    },
                  },
                },
              ],
            },
          ],
        },
      ],
      advanced: [
        /*
          Declare settings to be added to the "Advanced settings" section
          of the field in the Content-Type Builder
        */ 
      ],
      validator: (args) => ({
        'color-picker': yup.object().shape({
          format: yup.string().oneOf(['hex', 'rgba']),
        }),
      }),
    },
  });
}
```
:::

::: note
Relations, components or dynamic zones can't be used as a custom field's `type` parameter.
:::

### Components

`app.customFields.register()` must pass a `components` object with 1 or 2 of the following components:

| Component | Description                                                         |
| --------- | ------------------------------------------------------------------- |
| `Input`   | React component to use in the Content Manager's edit view           |
| `View`    | Read-only React component to use in the Content Manager's list view |

::: details Example: Registering Input and View components imported from dedicated files:

<!-- ? can we declare components inline as well? like Input: async () => <MyReactComponent>…<MyReactComponent/>? What would be the most simple React.component example we could describe for a custom field? -->
```js
// path: ./src/plugins/my-custom-field-plugin/strapi-server.js

register(app) {
  app.customFields.register({
    // …
    components: {
      Input: async () => import(/* webpackChunkName: "input-component" */ "./Input"),
      View: async () => import(/* webpackChunkName: "view-component" */ "./View"),
    } 
    // …
  });
}
```
:::

### Options

`app.customFields.register()` can pass an additional `options` object with the following parameters:

| Options parameter | Description                                                                     | Type                    |
| -------------- | ------------------------------------------------------------------------------- | ----------------------- |
| `base`         | Settings available in the _Base settings_ tab of the field in the Content-type Builder       | `Object` or  `Object[]` |
| `advanced`     | Settings available in the _Advanced settings_ tab of the field in the Content-type Builder   | `Object` or  `Object[]` |
| `validator`    | Validator function returning an object, used to sanitize input | `Function`              |

Both `base` and `advanced` settings accept an object or an array of objects, each object being a settings section. Each settings section must include:

- a `sectionTitle` to declare the title of the section as a `React IntlObject`
- and a list of `items` as an array of objects.

Each object in the `items` array can contain the following parameters:

<!-- TODO: fix the table content as the parameters given in the [example from the RFC](https://github.com/strapi/rfcs/blob/3eb034e169746558315d719ca5fb49cec854640a/rfcs/xxxx-custom-fields-api.md#example) and the [TS interface](https://github.com/strapi/rfcs/blob/3eb034e169746558315d719ca5fb49cec854640a/rfcs/xxxx-custom-fields-api.md#admin) described in the RFC are inconsistent 🤷  -->

| Items parameter | Description                                                                                  | Type     |
| --------------- | -------------------------------------------------------------------------------------------- | -------- |
| `intlLabel`     | Translation for the abel of the setting item                                                 | [`IntlObject`](https://formatjs.io/docs/react-intl/) |
| `name`          | Name of the setting item<br/>Must use the `options.setting-name` format.                     | `String` |
| `type`          | ?                                                                                            | `String` |
| `value`         | ?                                                                                            | `String` |
| `options`     | ?                                                                                              | `Array of Objects`  |



<!-- TODO: add a super simple example with one base setting, one advanced setting, and a simple validator -->
::: details Example: Declaring settings for an example "color" custom field:

```jsx
// path: ./src/plugins/my-custom-field-plugin/strapi-server.js

register(app) {
  app.customFields.register({
    // …
    options: {
      base: [
        {
          sectionTitle: {
            id: 'color-picker.color.section.format',
            defaultMessage: 'Format',
          },
          items: [
            {
              intlLabel: {
                id: 'color-picker.color.format.label',
                defaultMessage: 'Color format',
              },
              name: 'options.format',
              type: 'select',
              value: 'hex',
              options: [
                {
                  key: 'hex',
                  value: 'hex',
                  metadatas: {
                    intlLabel: {
                      id: 'color-picker.color.format.hex',
                      defaultMessage: 'Hexadecimal',
                    },
                  },
                },
                {
                  key: 'rgba',
                  value: 'rgba',
                  metadatas: {
                    intlLabel: {
                      id: 'color-picker.color.format.rgba',
                      defaultMessage: 'RGBA',
                    },
                  },
                },
              ],
            },
          ],
        },
      ],
      advanced: [],
      validator: (args) => ({
        'color-picker': yup.object().shape({
          format: yup.string().oneOf(['hex', 'rgba']),
        }),
      }),
    },
  });
}
```
:::

:::note
When extending a custom field’s base and advanced forms in the Content-type Builder, it is not currently possible to import custom input components.
:::
