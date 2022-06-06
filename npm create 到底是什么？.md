# npm create 到底是什么？

在前端项目脚手架百花齐放的今天，王sad经常会使用脚手架搭建项目，使用脚手架的第一步往往都是看脚手架的使用文档，这时，往往 `npm create foo`  命令会映入眼帘。这行命令往往还伴随着以下命令出现。

```other
yarn create foo
pnpm create vite
```

文档告诉王sad，这些命令可以让他安装初始化脚手架，但他并不认识 `npm create` 这个命令。这时，王sad准备去 `npm CLI` 的文档中去寻找 `create` 命令的答案。

在文档的搜索框中搜索 npm-create，无果，他试着再搜索`cteate`，网页中出现了`npm-init` 的文档。

在文档的概述部分，王sad发现了 `init` **的别名为 `create` 和 `innit` ，原来脚手架文档普遍使用的 `npm create` 就是 `npm init`。**

[文档](https://docs.npmjs.com/cli/v8/commands/npm-init#description)是这样描述的：

```other
npm init [--force|-f|--yes|-y|--scope]
npm init <@scope> (same as `npx <@scope>/create`)
npm init [<@scope>/]<name> (same as `npx [<@scope>/]create-<name>`)
aliases: create, innit
```

> `npm init <initializer>` can be used to set up a new or existing npm package.

> `initializer` in this case is an npm package named `create-<initializer>`, which will be installed by [`npm-exec`](https://docs.npmjs.com/cli/v8/commands/npm-exec), and then have its main bin executed–presumably creating or updating `package.json` and running any other initialization-related operations.

> The init command is transformed to a corresponding `npm exec` operation as follows:

- > `npm init foo` -> `npm exec create-foo`
- > `npm init @usr/foo` -> `npm exec @usr/create-foo`
- > `npm init @usr` -> `npm exec @usr/create`

> If the initializer is omitted (by just calling `npm init`), init will fall back to legacy init behavior. It will ask you a bunch of questions, and then write a package.json for you. It will attempt to make reasonable guesses based on existing fields, dependencies, and options selected. It is strictly additive, so it will keep any fields and values that were already set. You can also use `-y`/`--yes` to skip the questionnaire altogether. If you pass `--scope`, it will create a scoped package.

> *Note:* if a user already has the `create-<initializer>` package globally installed, that will be what `npm init` uses. If you want npm to use the latest version, or another specific version you must specify it:

- > `npm init foo@latest` # fetches and runs the latest `create-foo` from the registry
- > `npm init foo@1.2.3` # runs `create-foo@1.2.3` specifically

> #### Forwarding additional options

> Any additional options will be passed directly to the command, so `npm init foo -- --hello` will map to `npm exec -- create-foo --hello`.

> To better illustrate how options are forwarded, here's a more evolved example showing options passed to both the **npm cli** and a create package, both following commands are equivalent:

- > `npm init foo -y --registry=<url> -- --hello -a`
- > `npm exec -y --registry=<url> -- create-foo --hello -a`

除此之外，文档还介绍了一些参数配置。

通过上述文档，王sad知道了这几件事：

1. `npm create` 不过是 `npm init` 的马甲。
2. `npm init foo`等价于`npm exec create-foo`。
3. `npm init foo`通过`npm-exec`执行本地（优先级更高）或远程的`create-foo`包去创建或更新 `package.json`文件。
4. 如果只执行`npm init`则会询问你一串问题，然后为你创建相应的`package.json`。
5. `npm init foo —- —-bar` 等价于`npm exec -- create-foo —bar`。

同时，在 Yarn 2  CLI 文档的中，王sad 也只发现了`yarn init`的[文档](https://yarnpkg.com/cli/init)。而`yarn create`只出现在 Yarn 1 CLI 的文档中，王sad想一定是在版本迁移的文档中有说明，果然，在[迁移的文档](https://yarnpkg.com/getting-started/migration#renamed)中找到了如下内容：

> Renamed

| Yarn Classic(1.x) | Yarn (2.x)               | Note                                                   |
| ----------------- | ------------------------ | ------------------------------------------------------ |
| `yarn create`     | `yarn dlx create-<name>` | `yarn create` still works, but prefer using `yarn dlx` |

所以，`npm` 和`yarn` 的最新文档中，并没有过多的介绍`npm create` 和 `yarn create` ，前者是`init` 的别名，后者是保留先前版本的命令名。

那为什么主流的脚手架文档还是会给出这两个命令呢？王sad思考如下：

1. `npm init`语义上为初始化包，而非创建包，使用`create` 别名更具有语义化；
2. `yarn dlx`命令需要保留包名中的 create- 部分，不如`yarn create`简洁；
3. 都使用`create`命令利于包管理器版本兼容，且方便开发者记忆。

至此，王sad对 `npm create`有了初步的了解和思考。

