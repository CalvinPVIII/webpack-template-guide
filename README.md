# Simplified Webpack Setup

There are a lot of commands to run, and a lot of things to install and configure, here are the essentials of what you will need to do:

First we will begin by initializing `npm`, run this command in your terminal, in the directory of your template:

`npm init -y`

This will create the package.json and allow you to add npm packages

Next we will create our folders and files that will be needed:

`mkdir src src/assets src/assets/images css __tests__`

Creating files:

`touch src/index.html src/index.js css/styles.css .gitignore webpack.config.js`

Node modules and dist will be created when you run npm install and npm build, so we don't need to include them in our repo. That is what the `.gitignore` file is for. Any files you don't want on github should go here.
Add this to our .gitignore:

```
node_modules/
dist/
.DS_Store
```

Next we need to install webpack:

`npm install webpack@5.87.0 --save-dev --save-exact`

`npm install webpack-cli@5.1.4 --save-dev`

Create our `webpack.config.js` file. Inside that file, add this code:

```
const path = require('path');

module.exports = {
  entry: './src/index.js', // this indicates the main js file that everything will be bundled into
  output: {
    filename: 'bundle.js', // this is the file name for the bundled output
    path: path.resolve(__dirname, 'dist') // this is the directory for the bundled output, in this case it's 'dist'
  }
};
```

To have our css be bundled in, we will add `css-loader` and `style-loader`

`npm install style-loader@3.3.0 css-loader@6.8.1 --save-dev`

Configuring the loaders in webpack.config.js:

```
// this gets added right after the 'output' object, still within the `module.exports` object
module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      }
    ]
  }
```

In order for our css to actually get bundled and used, we need to import it into our JS. Add this line to the top `index.js`:

`import './css/styles.css';`

We'll let webpack handle generating html files, so we will install HTML webpack plugin:

`npm install html-webpack-plugin@5.5.3 --save-dev`

Add HTML webpack plugin to the webpack.config.js:

```
// below  const path = require('path');, add this line:
const HtmlWebpackPlugin = require('html-webpack-plugin');
```

In between `output` and `module` objects, add this new plugins array:

```
plugins: [
    new HtmlWebpackPlugin({
      title: 'Project', // this line will be the title that shows up in your browser tab
      template: './src/index.html', // everything will get injected into this file
      inject: 'body'
    })
  ],
```

Next we will install clean webpack plugin, which will clean up our environment whenever we make changes:

`npm install clean-webpack-plugin@4.0.0 --save-dev`

Add the plugin to our webpack config, right below HTML webpack plugin at the top of the file:

`const { CleanWebpackPlugin } = require('clean-webpack-plugin');`

Inside the plugins array, add this new line above `new Html WebpackPlugin()`:

`new CleanWebpackPlugin(),`

To prevent any issues with our bundled code and our browsers dev tools, we need to add this line, right above the `plugins` array:

`devtool: 'eval-source-map',`

To setup a dev live server, we need to install this:

`npm install webpack-dev-server@4.15.1 --save-dev --save-exact`

Add the devServer object to our webpack config, this goes right after the `output` object:

```
devServer: {
    static: {
      directory: path.join(__dirname, "dist"),
    },
  },

```

In order to actually run the server, we need to add in a script to our `package.json`. In the `scripts` object, add this line:

`"start": "webpack-dev-server --open --mode=development"`

Next we will install eslint, which will give us better errors and warnings:

`npm install eslint@8.42.0 --save-dev`

`npm install eslint-webpack-plugin@4.0.1 --save-dev`

We added a new plugin, so we need to update our webpack-config:

First we need to import eslint at the top of the file, underneath `CleanWebPackPlugin`

`const ESLintPlugin = require('eslint-webpack-plugin');`

Add it to our plugins array, above `new CleanWebpackPlugin()`:

`new ESLintPlugin(),`

To configure eslint, we need a `.eslintrc` file. Create the file and add this code:

```
{
  "parserOptions": {
    "ecmaVersion": 2018,
    "sourceType": "module"
  },
  "extends": "eslint:recommended",
  "env": {
    "es6": true,
    "browser": true,
    "node": true
  },
  "rules": {
    "semi": 1,
    "indent": ["warn", 2],
    "no-console": "warn"
  }
}
```

Adding a lint script will tell us any errors we have when we run the command. Add this to the scripts in package.json:

`"lint": "eslint src --ext .js"`

Also I recommend installing the [eslint VS Code plugin](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint), which will show errors in our editor directly.

Next, for testing, we will need to install Jest:

`npm install jest@29.5.0 --save-dev`

In order to actually run our tests, we need to add another script to the `scripts` object of `package.json`:

`"test": "jest --coverage",`

Next we will install Babel, which will translate any modern JS code into legacy code incase there are any compatibility errors:

`npm install @babel/core@7.22.5 --save-dev`

`npm install @babel/plugin-transform-modules-commonjs@7.22.5 --save-dev`

To configure babel, we need a `.babelrc` file. Create that file in the root of your project and add the following code:

```
{
  "env": {
    "test": {
      "plugins": ["@babel/plugin-transform-modules-commonjs"]
    }
  }
}
```

Since we added the `--coverage` flag to our tests, a `coverage` folder will be generated. We don't really need that on our repo, so we can add it to the `.gitignore`:
`coverage/`

For images, we need html loader:

`npm install html-loader@4.2.0 --save-dev`

We will also need to configure the appropriate asset module

Add this config to our webpack, this goes in the `rules` array, next to our previous `css-loader` configuration:

```
{
  test: /\.(gif|png|avif|jpe?g)$/,
  type: "asset/resource",
  generator: {
    filename: "[name][ext]"
    publicPath: "assets/images/",
    outputPath: "assets/images/",
  },
},
{
  test:/\.html$/,
  use: [
    'html-loader'
  ]
},
```

After all that, here is what your final `webpack.config.js` should look like:

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const ESLintPlugin = require("eslint-webpack-plugin");
module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  devServer: {
    static: {
      directory: path.join(__dirname, "dist"),
    },
  },
  devtool: "eval-source-map",
  plugins: [
    new ESLintPlugin(),
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: "Project",
      template: "./src/index.html",
      inject: "body",
    }),
  ],
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.(gif|png|avif|jpe?g)$/,
        type: "asset/resource",
        generator: {
          filename: "[name][ext]",
          publicPath: "assets/images/",
          outputPath: "assets/images/",
        },
      },
      {
        test: /\.html$/,
        use: ["html-loader"],
      },
    ],
  },
};
```

And the final `package.json`:

```json
{
  "name": "project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "jest --coverage",
    "build": "webpack --mode=development",
    "start": "webpack-dev-server --open --mode=development"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "@babel/core": "^7.22.5",
    "@babel/plugin-transform-modules-commonjs": "^7.22.5",
    "clean-webpack-plugin": "^4.0.0",
    "css-loader": "^6.8.1",
    "eslint": "^8.42.0",
    "eslint-webpack-plugin": "^4.0.1",
    "file-loader": "^6.2.0",
    "html-loader": "^4.2.0",
    "html-webpack-plugin": "^5.5.3",
    "jest": "^29.5.0",
    "style-loader": "^3.3.3",
    "webpack": "^5.87.0",
    "webpack-cli": "^5.1.4",
    "webpack-dev-server": "^4.15.1"
  }
}
```

Some things to keep in mind:

All your source code, the code that you write, will primarily happen in the `src` folder. All your JS, HTML, CSS, and assets all belong in in the `src` folder.

For testing:

All files that contain our tests will go in the `__tests__` directory. The naming syntax is `{nameOfFileWeAreTesting}.test.js`. Naming is important here, as Jest will specifically look for tests in the `__tests__` directory, that have that naming syntax.

Once you have a solid template setup, you likely won't need to touch any of the files outside of the `src` or `__tests__` directories.

## Common Troubleshooting:

If you are getting an error that looks like this:

`error:0308010C:digital envelope routines::unsupported`

Or any errors that mention an `SSL`, make sure you are on Node version 16 or later.
