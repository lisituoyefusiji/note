# eslint-plugin-vue

>https://eslint.vuejs.org/

### Introduction

Official ESLint plugin for Vue.js.

This plugin allows us to check the `<template>` and `<script>` of `.vue` files with ESLint, as well as Vue code in `.js` files.

- Finds syntax errors.
- Finds the wrong use of [Vue.js Directives](https://vuejs.org/api/built-in-directives.html).
- Finds the violation for [Vue.js Style Guide](https://vuejs.org/style-guide/).

ESLint editor integrations are useful to check your code in real-time.

> Status of Vue.js 3.x supports
>
> This plugin supports the basic syntax of Vue.js 3.2, `<script setup>`, and CSS variable injection, but the ref sugar, an experimental feature of Vue.js 3.2, is not yet supported.
> If you have issues with these, please also refer to the [FAQ](https://eslint.vuejs.org/user-guide/#does-not-work-well-with-script-setup). If you can't find a solution, search for the issue and if the issue doesn't exist, open a new issue.

### 💿 Installation

Via [npm](https://www.npmjs.com/):

bash

```
npm install --save-dev eslint eslint-plugin-vue
```

Via [yarn](https://yarnpkg.com/):

bash

```
yarn add -D eslint eslint-plugin-vue
```

> Requirements
>
> - ESLint v6.2.0 and above
> - Node.js v14.17.x, v16.x and above

### 📖 Usage

#### Configuration

Use `.eslintrc.*` file to configure rules. See also: https://eslint.org/docs/user-guide/configuring.

Example **.eslintrc.js**:

```javascript
module.exports = {
  extends: [
    // add more generic rulesets here, such as:
    // 'eslint:recommended',
    'plugin:vue/vue3-recommended',
    // 'plugin:vue/recommended' // Use this if you are using Vue.js 2.x.
  ],
  rules: {
    // override/add rules settings here, such as:
    // 'vue/no-unused-vars': 'error'
  }
}
```

See [the rule list](https://eslint.vuejs.org/rules/) to get the `extends` & `rules` that this plugin provides.

#### Bundle Configurations

This plugin provides some predefined configs. You can use the following configs by adding them to `extends`.

- `"plugin:vue/base"` ... Settings and rules to enable correct ESLint parsing.
- Configurations for using Vue.js 3.x.
  - `"plugin:vue/vue3-essential"` ... `base`, plus rules to prevent errors or unintended behavior.
  - `"plugin:vue/vue3-strongly-recommended"` ... Above, plus rules to considerably improve code readability and/or dev experience.
  - `"plugin:vue/vue3-recommended"` ... Above, plus rules to enforce subjective community defaults to ensure consistency.
- Configurations for using Vue.js 2.x.
  - `"plugin:vue/essential"` ... `base`, plus rules to prevent errors or unintended behavior.
  - `"plugin:vue/strongly-recommended"` ... Above, plus rules to considerably improve code readability and/or dev experience.
  - `"plugin:vue/recommended"` ... Above, plus rules to enforce subjective community defaults to ensure consistency

#### How to use a custom parser?

If you want to use custom parsers such as [@babel/eslint-parser](https://www.npmjs.com/package/@babel/eslint-parser) or [@typescript-eslint/parser](https://www.npmjs.com/package/@typescript-eslint/parser), you have to use the `parserOptions.parser` option instead of the `parser` option. Because this plugin requires [vue-eslint-parser](https://www.npmjs.com/package/vue-eslint-parser) to parse `.vue` files, this plugin doesn't work if you overwrite the `parser` option.

```diff
- "parser": "@typescript-eslint/parser",
+ "parser": "vue-eslint-parser",
  "parserOptions": {
+     "parser": "@typescript-eslint/parser",
      "sourceType": "module"
  }
```

The `parserOptions.parser` option can also specify an object to specify multiple parsers. See [vue-eslint-parser README](https://github.com/vuejs/vue-eslint-parser#readme) for more details.

#### How does ESLint detect components?

All component-related rules are applied to code that passes any of the following checks:

- `Vue.component()` expression
- `Vue.extend()` expression
- `Vue.mixin()` expression
- `app.component()` expression
- `app.mixin()` expression
- `createApp()` expression
- `defineComponent()` expression
- `export default {}` in `.vue` or `.jsx` file

However, if you want to take advantage of the rules in any of your custom objects that are Vue components, you might need to use the special comment `// @vue/component` that marks an object in the next line as a Vue component in any file, e.g.:

js

```
// @vue/component
const CustomComponent = {
  name: 'custom-component',
  template: '<div></div>'
}
```

js

```
Vue.component('AsyncComponent', (resolve, reject) => {
  setTimeout(() => {
    // @vue/component
    resolve({
      name: 'async-component',
      template: '<div></div>'
    })
  }, 500)
})
```

#### Disabling rules via `<!-- eslint-disable -->`

You can use `<!-- eslint-disable -->`-like HTML comments in the `<template>` and in the block level of `.vue` files to disable a certain rule temporarily.

For example:

vue

```
<template>
  <!-- eslint-disable-next-line vue/max-attributes-per-line -->
  <div a="1" b="2" c="3" d="4">
  </div>
</template>
```

If you want to disallow `eslint-disable` functionality in `<template>`, disable the [vue/comment-directive](https://eslint.vuejs.org/rules/comment-directive.html) rule.

#### Parser Options

This plugin uses [vue-eslint-parser](https://www.npmjs.com/package/vue-eslint-parser). For `parserOptions`, you can use the `vueFeatures` options of `vue-eslint-parser`.

json

```
{
  "parser": "vue-eslint-parser",
  "parserOptions": {
    "vueFeatures": {
      "filter": true,
      "interpolationAsNonHTML": false,
    }
  }
}
```

See the [`parserOptions.vueFeatures` documentation for `vue-eslint-parser`](https://github.com/vuejs/vue-eslint-parser#parseroptionsvuefeatures) for more details.

### 💻 Editor integrations

#### Visual Studio Code

Use the [dbaeumer.vscode-eslint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) extension that Microsoft provides officially.

You have to configure the `eslint.validate` option of the extension to check `.vue` files, because the extension targets only `*.js` or `*.jsx` files by default.

Example **.vscode/settings.json**:

```json
{
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "vue"
  ]
}
```

---

### ❓ FAQ

#### Conflict with Prettier

Use [eslint-config-prettier](https://github.com/prettier/eslint-config-prettier) for [Prettier](https://prettier.io/) not to conflict with the shareable config provided by this plugin.

Example **.eslintrc.js**:

```javascript
module.exports = {
  // ...
  extends: [
    // ...
    // 'eslint:recommended',
    // ...
    'plugin:vue/vue3-recommended',
    // ...
    "prettier"
    // Make sure "prettier" is the last element in this list.
  ],
  // ...
}
```

If Prettier conflicts with a rule you have set, [turn off that rule](https://prettier.io/docs/en/integrating-with-linters.html). For example, if you have `vue/html-indent` configured as `error` in `rules`, but it conflicts with Prettier, remove that line:

```diff
module.exports = {
  // ...
  rules: {
    // ...
-    "vue/html-indent": "error",
    // ...
  },
  // ...
}
```