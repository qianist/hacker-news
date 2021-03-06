# Hacker News

Build a project from very scratch.

## Init

Create a directory for this project.

I'm going to use [React](https://reactjs.org/). Using `<script>` is an option, but [npm](https://www.npmjs.com/) seems to be a more popular choice. So ...

In order to use npm:

```shell
npm init -y
```

Install React:

```shell
npm install react react-dom
```

Create `src/index.html` and `src/index.js`.

Add `<script>` tag to load `index.js`, and a root element as container:

```html
<div id="root"></div>
<script src="index.js"></script>
```

To use React, import it in `index.js`

```js
import React from "react";
```

In the browser, an error occur:

```error
Uncaught SyntaxError: Unexpected identifier
```

Seems like the browser doesn't support import syntax yet.

Use [webpack](https://webpack.js.org/) to fix this. But before that, dealing with code format manually is a little annoying, and I don't have a strong bias of which format to use.

### Prettier

[Prettier](https://prettier.io/) is a nice solution. Since this is my own project, there is no need to force somebody to use it. So I'm going to use [Visual Studio Code](https://code.visualstudio.com/)'s [prettier plugin](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode), instead of installing prettier package locally.

In editor's setting panel, turn on `Prettier: Require Config` and `Editor: Format On Save`. This way, only project with `.prettierrc` or [other configuration file](https://prettier.io/docs/en/configuration.html) will "format on save".

Create `.prettierrc` file and add following:

```json
{}
```

Just an empty object. Default setting is fine.

At this point, it's probably a good idea to install [ESLint](https://eslint.org/) as well.

### ESLint

Install

```shell
npm install eslint --save-dev
```

Add configuration file:

```shell
npx eslint --init
```

Follow the instructions. Notice that webpack uses node, so under the question "Where does your code run?" Select both "Browser" and "Node".

Install VSCode [ESLint plugin](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint).

Restart to enable plugin (show the red underline).

I don't have a code style now. Maybe use a extension is a good choice, like [eslint-config-airbnb](https://www.npmjs.com/package/eslint-config-airbnb). Will see if needed along the way.

Now back to the syntax error.

### Webpack

Install

```shell
npm install webpack webpack-cli --save-dev
```

`webpack` is the core. `webpack-cli` is for running commands.

To run those commands, add scripts in `package.json`

```json
"scripts": {
  "build": "webpack"
}
```

Just run this script, see what happened.

```shell
npm run build
```

No Error! A filed is generated at `dist/main.js`. It's a complex minified js file.

Since webpack 4, no config is needed. The default behavior is to find file at `src/index.js`(entry), and bundle everything into `dist/main.js`(output).

But more complex config will be needed, so a config file is fine.

Create config file `webpack.config.js`. Add entry and output config

```js
module.exports = {
  entry: "./src/index.js",
  output: "./dist/main.js" // wrong, see below
};
```

Same as default, but explicit.

Webpack use the entry file as a starting point. Then find all the dependencies based on statements like `import`.

After `npm run build` and some modification, two errors were found:

```
1. configuration.output should be an object.
2. The output directory as **absolute path** (required)
```

Fix it.

```js
const path = require("path");

module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "main.js",
    path: path.resolve(__dirname, "dist")
  }
};
```

Use `path` package to generate absolute path.

No error now but a warning.

```
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
```

This is why the generated `main.js` file is minified.

Add mode config.

```js
{
  mode: "development";
}
```

Everything seems fine now. But there comes another problem. The `<script>` tag inside `index.html` point to `index.js`. But after adding webpack, the final output file is `dist/main.js`.

Two things about `dist` folder:

1. It's auto generated, and included in the `.gitignore` file
2. It should contain everything in order to be publish-ready.

Conclusion: `index.html` should be auto generated inside `dist` folder.

### [HtmlWebpackPlugin](https://github.com/jantimon/html-webpack-plugin)

```shell
npm install --save-dev html-webpack-plugin
```

`webpack.config.js`

```js
{
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, "src/index.html")
    })
  ];
}
```

Run `npm run build` and `index.html` file will be auto generated inside `dist` directory, using `stc/index.html` as template.

### [React.createElement](https://reactjs.org/docs/react-api.html#createelement)

`index.js`

```js
import React from "react";
import ReactDOM from "react-dom";

ReactDOM.render(
  React.createElement("h1", null, "Hacker News"),
  document.getElementById("root")
);
```

Run build command and open `dist/index.html`. Finally there is something on the page.

### [JSX](https://reactjs.org/docs/introducing-jsx.html)

Rewrite `index.js`

```js
import React from "react";
import ReactDOM from "react-dom";

function App() {
  return <h1>Hacker News</h1>;
}

ReactDOM.render(<App />, document.getElementById("root"));
```

As expect, there is a build error:

```
Module parse failed: Unexpected token (5:9)
You may need an appropriate loader to handle this file type,
```

A loader is needed to transform JSX into standard js, like the last part.

### [Babel](https://babeljs.io/)

Install packages

```shell
npm install --save-dev @babel/core @babel/preset-env @babel/preset-react babel-loader
```

The first three are used to compile new javascript syntax and jsx into browser-compatible javascript.

`babel-loader` is for webpack to let babel handle `.js` file bundle process.

Create `babel.config.js`

```js
const presets = ["@babel/preset-env", "@babel/preset-react"];

module.exports = {
  presets
};
```

Add config to `webpack.config.js`

```js
{
  module: {
    rules: [
      {
        test: /\.js$/,
        use: "babel-loader"
      }
    ];
  }
}
```

Now build command should work.

But for some reason, ESLint complains about jsx syntax.

```
Parsing error: Unexpected token /
```

After some research, I add `babel-eslint` parser option, and change `eslint-plugin-react` configuration

```
npm install babel-eslint --save-dev
```

```js
// .eslintrc.js
{
   parser: "babel-eslint",
   extends: ["eslint:recommended", "plugin:react/recommended"],
}
```

Code back to working again.

One last thing. Manually run `npm run build` and refresh browser is not very smooth.

### [webpack-dev-server](https://webpack.js.org/configuration/dev-server/)

```shell
npm install --save-dev webpack-dev-server
```

Add development script

```json
{
  "scripts": {
    "start": "webpack-dev-server --open"
  }
}
```

`--open` means to automatically open browser, sometimes it's useful, sometimes it's annoying.

---

That's a lot of work before diving into project implementation. Yet this is far from complete. There's still a lot of work to do.
