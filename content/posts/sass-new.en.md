---
title: Sass in Modern Frontend
date: 2024-01-02T00:00:00.000+00:00
tags:
  - CSS
  - sass
  - scss
category: code
lang: en
tocAlwaysOn: true
cover: /img/sass-new.jpg
images:
  - /img/sass-new.jpg
---


How to integrate the sass/scss features into your front-end frameworks? Watch this !

<!--more-->

Recently, I found a [project](https://github.com/0xAliRaza/habitly) that implemented the methodology mentioned in the famous book Atomic Habit by James Clear; I managed to update all the packages to the newest to get it up and running.

Honestly, switching from vue-cli to vite is pretty smooth since syntaxes didn't change much within a big version. However, I encountered several errors related to Bootstrap and Sass.

The Sass official indicated that [`@import`  rule will be deprecated in the near future](https://github.com/sass/sass/blob/main/accepted/module-system.md#timeline); that means we need to work with @use and @forward in our projects. So what are the at-rules of @use and @forward, then?

## `@use`

We all deal with ES6-standard **modules** and exports every day. Imagine there are several modules such as "colors", "fonts", and "mixins."  Define them all starting with an underscore like `_colors.scss`.

```scss
// ./src/scss/colors.scss
$primary-color: #38f;
$secondary-color: #eee;
$flat-black: #333;
```

How to use it?

In another file, typically the one you would love to linked in the `inedx.html`. A `@use "colors"` will load all the **members** defined in this stylesheet module. But we cannot use it directly; a **namespace** prefix is needed when you trying to access them. It's the module name by default; you can rename it using the "as" keyword.

> Members mean "mixins", "variables" and "functions" in SASS language.

```scss
// ./src/scss/styles.scss
@use "./variables/colors" as c;
body{
 background-color: c.$secondary-color;
}
```

Similarly, it's pretty easy to include other modules:

```scss
// ./src/scss/styles.scss
@use "card";
@use "colors" as c;
@use "variables/fonts"; // equals to `@use "variables/fonts" as fonts`

$transition:all 0.5s ease-in-out;

body{
  font-size:fonts.$font-size;
  background-color: c.$secondary-color;
}
h1{
 color:c.$primary-color;
}
```

You can make this process even simple by assigning namespace to `*` .

```scss
@use "colors" as *;

h1{
 color:$primary-color;
}
```

Notice:

* A stylesheet’s `@use` rules must come before any rules other than `@forward`, including [style rules](https://sass-lang.com/documentation/style-rules).
* `@use` only ever loads each file once.
* Variables imported by `@use` can be overridden and take effect in all modules. You can set the initial value by adding a `!default` flag anywhere.

## `@forward`

`@forward` rule will load all the variables, mixins and functions in a  Sass stylesheet available when you load it in another stylesheet via `@use` .

The typical scenario will be manage different modules inside a folder and import them once for all in another stylesheet.

Lets say we have two modules `colors` and `fonts` in the following path.

```scss
// ./src/scss/variables/_fonts.scss
$font-size:1.5rem;
$font-weight:bold;
```

```scss
// ./src/scss/variables/_fonts.scss
$primary-color: #38f;
$secondary-color: #eee;
$flat-black: #333;

```

Then we create a new file under `variables` folder names `_index.scss`, by using `@forward` rule, the **members** inside previous two modules will be loaded when you using `@use`.

```scss
// ./src/scss/variables/_index.scss
@forward "colors";
@forward "fonts";
```

```scss
@use "card";
@use "mixins" as *;
@use "./variables" as v;

$transition:all 0.5s ease-in-out;

body{
  font-size:v.$font-size;
  background-color: v.$secondary-color;
}
h1{
 color:v.$primary-color;
}

```

Equals to:

```scss
@use "card";
@use "mixins" as *;
@use "./variables" as *;

$transition:all 0.5s ease-in-out;

body{
  font-size:$font-size;
  background-color: $secondary-color;
}
h1{
 color:v.$primary-color;
}
```

Those members aren’t available in your module, though. If you try to use variables loaded by `@forward` , something bad will happen.

```
@forward "colors";
@forward "fonts";

.box{
 color: colors.$color-primary;
 font-family: fonts.$font-primary;
}
// ERROR: There is no module with the namespace "colors"
```

## Work with Vue3+Vite

We can still add preprocessor options in the vite.config.js setting to inject extra code in the target language for each style content.

> <https://github.com/vitejs/vite/issues/832>

```js
// vite.config.js
export default defineConfig({
    ...,
     css: {
  preprocessorOptions: {
    scss: {
      additionalData: `@use "@/assets/scss/global" as *;`,
    },
  },
 },
})
```

```scss
// /src/assets/scss/_variables.scss
$color-primary: #0074d9;
$color-secondary: #7fdbff;
$color-tertiary: #39cccc;
```

The `global` module will load all its members and memebers in `variables` and `mixins` when using `@use`.

```scss
// /src/assets/scss/_global.scss
@forward "variables";
@forward "mixins";

$font-roboto: "Roboto", sans-serif;
```

Since we removed the namespace eventually, we can use all the members from these modules directly in any component we want:

```vue
<script setup>
 ...
</script>

<template>
  ...
</template>

<style lang="scss" scoped >
h1{
 color:$color-primary;
    font-family: $font-roboto;
}
</style>

```
