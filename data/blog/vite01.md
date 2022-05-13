---
title: 深入浅出 vite
date: '2022-04-13'
tags: ['vite', 'ESLint']
draft: false
summary: 利用 Lint 工具链来保证代码风格和质量
---

# 利用 Lint 工具链来保证代码风格和质量

> 提高代码质量、检查代码规范、自动修复、统一风格

通过`ESLint`、`Prettier`、`Commitlint`、`husky`、`lint-staged`、`VSCode插件`等生成完整的 Lint 工具链

## [ESLint](https://eslint.bootcss.com/)

安装

```shell
pnpm i eslint -D
```

初始化

```shell
echo {}> .eslintrc
```

配置如下 使用时去掉注释

```json
// .eslintrc
{
   "extends": [
     // 第1种情况
     "eslint: recommended",
     // 第2种情况，一般配置的时候可以省略 `eslint-config`
     "standard"
     // 第3种情况，可以省略包名中的 `eslint-plugin`
     // 格式一般为: `plugin:${pluginName}/${configName}`
     "plugin:@typescript-eslint/recommended",
   ]
}
```

> 详情查看文档

## [Prettier](https://www.prettier.cn/)

安装

```shell
pnpm i prettier -D
```

初始化

```shell
echo {}> .prettierrc
```

配置如下

```json
// .prettierrc
{
  "printWidth": 80, //一行的字符数，如果超过会进行换行，默认为80
  "tabWidth": 2, // 一个 tab 代表几个空格数，默认为 2 个
  "useTabs": false, //是否使用 tab 进行缩进，默认为false，表示用空格进行缩减
  "singleQuote": true, // 字符串是否使用单引号，默认为 false，使用双引号
  "semi": true, // 行尾是否使用分号，默认为true
  "trailingComma": "none", // 是否使用尾逗号
  "bracketSpacing": true // 对象大括号直接是否有空格，默认为 true，效果：{ a: 1 }
}
```

## 集成 ESLint + Prettier

eslint-config-prettier 用来覆盖 ESLint 本身的规则配置，而 eslint-plugin-prettier 则是用于让 Prettier 来接管 eslint --fix 即修复代码的能力。

```shell
pnpm i eslint-config-prettier eslint-plugin-prettier -D
```

配置如下 使用时去掉注释

```json
{
  "env": {
    "browser": true,
    "es2021": true
  },
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    // 1. 接入 prettier 的规则
    "prettier",
    "plugin:prettier/recommended"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaFeatures": {
      "jsx": true
    },
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  // 2. 加入 prettier 的 eslint 插件
  "plugins": ["@typescript-eslint", "prettier"],
  "rules": {
    // 3. 注意要加上这一句，开启 prettier 自动修复的功能
    "prettier/prettier": "error",
    "quotes": ["error", "single"],
    "semi": ["error", "always"],
    "react/react-in-jsx-scope": "off"
  }
}
```

添加 lint 检查、自动修复

```json
{
  "scripts": {
    // 省略已有 script
    "lint": "eslint --ext .js,.jsx,.ts,.tsx --fix --quiet ./"
  }
}
```

![format on save](/static/images/format.jpg)
在 vscode 设置中输入 formatonsave 点击 工作区 ☑️ 会在文件夹下生成`.vscode文件夹`
里面会生成共享配置文件

```json
// .vscode/settings.json
{
  "editor.formatOnSave": true
}
```

**之后每次保存时候都会自动格式化**

## ESLint in Vite

```shell
pnpm i vite-plugin-eslint -D
```

在`vite.config`中使用

```ts
// vite.config.ts
import viteEslint from 'vite-plugin-eslint'

{
  plugins: [viteEslint()] // 省略其它插件
}
```

重启项目 ESLint 的错误提示可以显示到命令行了

## husky + lint-staged 的 Git 提交工作流集成

### husky

```shell
  pnpm i husky -D
```

添加 husky 钩子

1. `npx husky install`
2. 在`package.json`中添加

```json
  "scripts": {
    // 会在安装 npm 依赖后自动执行
    "postinstall": "husky install"
  }
```

3. `npx husky add .husky/pre-commit "npm run lint"`
   会生成`.husky/pre-commit`文件包含了`git commit`前要执行的脚本
   现在每次`git commit` 的时候会执行 `npm run lint` 通过后才会提交

> 这样会有一定的性能问题 即使代码未变更 也会执行 lint 检查 有很大的性能消耗

### lint-staged

> 只会对缓存区的代码进行 lint 检查

```shell
pnpm i lint-staged -D
```

然后在 `package.json` 中添加如下的配置:

```json
{
  "lint-staged": {
    "**/*.{js,jsx,tsx,ts,json}": ["npm run lint", "git add --force"]
  }
}
```

在 husky 中使用 lint-stage，在.husky/pre-commit 脚本中，将 npm run lint 换成:

```shell
npx --no -- lint-staged
```

实现提交代码时的增量 Lint 检查

## 提交时的 commit 信息规范

> 规范 commit 信息

```shell
pnpm i commitlint @commitlint/cli @commitlint/config-conventional -D

echo {}> .commitlintrc.js
```

.commitlintrc.js 配置

```js
// .commitlintrc.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
}
```

常规的 commit 由两部分组成

```js
// type 指提交的类型
// subject 指提交的摘要信息
<type>: <subject>
```

常用的 type 值包括如下:

- `feat`: 添加新功能。
- `fix`: 修复 Bug。
- `chore`: 一些不影响功能的更改。
- `docs`: 专指文档的修改。
- `perf`: 性能方面的优化。
- `refactor`: 代码重构。
- `test`: 添加一些测试代码等等。

`commit` 功能集成到 `husky` 的钩子当中

```js
npx husky add .husky/commit-msg "npx --no-install commitlint -e $HUSKY_GIT_PARAMS"
```

生成`.husky/commit-msg` 脚本文件 `commitlint` 命令已经成功接入到 `husky` 的钩子当中, 错误的 `commit` 会直接报错， 规范的 `commit`
才会提交

## 总结

1. `JavaScript/TypeScript` 规范。主流的 Lint 工具包括 `Eslint`、`Prettier`

2. Git 提交规范。主流的 Lint 工具包括 `Commitlint`
