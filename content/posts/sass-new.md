---
title: 现代前端中的Sass
date: 2024-01-02T00:00:00.000+00:00
tags:
  - CSS
  - sass
  - scss
category: code
lang: zh
tocAlwaysOn: true
cover: /img/sass-new.jpg
images:
  - /img/sass-new.jpg
---

[原文](/en/posts/sass-new/)

最近，我发现了一个[项目](https://github.com/0xAliRaza/habitly)，实现了詹姆斯·克利尔的著名书籍《原子习惯》中提到的方法论；我成功地将所有软件包更新到最新版本以使其正常运行。

老实说，从vue-cli切换到vite非常顺利，因为在大版本中语法没有太大变化。然而，我遇到了一些与Bootstrap和Sass相关的错误。

Sass官方指出[`@import`规则将在不久的将来被弃用](https://github.com/sass/sass/blob/main/accepted/module-system.md#timeline)；这意味着我们需要在项目中使用@use和@forward。那么@use和@forward的规则是什么呢？

## `@use`

我们每天都在处理ES6标准的**模块**和导出。想象一下，有几个模块如“colors”、“fonts”和“mixins”。将它们全部以下划线开头定义，如`_colors.scss`。

```scss
// ./src/scss/colors.scss
$primary-color: #38f;
$secondary-color: #eee;
$flat-black: #333;
```

如何使用它？

在另一个文件中，通常是你想在`index.html`中链接的文件。`@use "colors"`将加载此样式表模块中定义的所有**成员**。但我们不能直接使用它；在访问它们时需要一个**命名空间**前缀。默认情况下是模块名；你可以使用“as”关键字重命名它。

> 成员在SASS语言中指“mixins”、“变量”和“函数”。

```scss
// ./src/scss/styles.scss
@use "./variables/colors" as c;
body{
 background-color: c.$secondary-color;
}
```

类似地，包含其他模块也非常容易：

```scss
// ./src/scss/styles.scss
@use "card";
@use "colors" as c;
@use "variables/fonts"; // 等于`@use "variables/fonts" as fonts`

$transition:all 0.5s ease-in-out;

body{
  font-size:fonts.$font-size;
  background-color: c.$secondary-color;
}
h1{
 color:c.$primary-color;
}
```

你可以通过将命名空间分配给`*`使此过程更简单。

```scss
@use "colors" as *;

h1{
 color:$primary-color;
}
```

注意：

* 样式表的`@use`规则必须在除`@forward`之外的任何规则之前，包括[样式规则](https://sass-lang.com/documentation/style-rules)。
* `@use`每次只加载每个文件一次。
* `@use`导入的变量可以被重写，并在所有模块中生效。你可以通过在任何地方添加`!default`标志来设置初始值。

## `@forward`

`@forward`规则将加载Sass样式表中的所有变量、mixins和函数，并在通过`@use`在另一个样式表中加载时使其可用。

典型的场景是管理一个文件夹内的不同模块，并在另一个样式表中一次性导入它们。

假设我们在以下路径中有两个模块`colors`和`fonts`。

```scss
// ./src/scss/variables/_fonts.scss
$font-size:1.5rem;
$font-weight:bold;
```

```scss
// ./src/scss/variables/_colors.scss
$primary-color: #38f;
$secondary-color: #eee;
$flat-black: #333;

```

然后我们在`variables`文件夹下创建一个名为`_index.scss`的新文件，使用`@forward`规则，前两个模块中的**成员**将在你使用`@use`时加载。

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

等于：

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
 color:$primary-color;
}
```

不过，这些成员在你的模块中不可用。如果你尝试使用`@forward`加载的变量，会发生错误。

```
@forward "colors";
@forward "fonts";

.box{
 color: colors.$color-primary;
 font-family: fonts.$font-primary;
}
// 错误：没有名为"colors"的模块
```

## 与Vue3+Vite配合使用

我们仍然可以在vite.config.js设置中添加预处理器选项，以在目标语言的每个样式内容中注入额外的代码。

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

`global`模块将在使用`@use`时加载其所有成员以及`variables`和`mixins`中的成员。

```scss
// /src/assets/scss/_global.scss
@forward "variables";
@forward "mixins";

$font-roboto: "Roboto", sans-serif;
```

由于我们最终移除了命名空间，我们可以在任何想要的组件中直接使用这些模块的所有成员：

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
