# Simplified Webpack Setup

**This was written for Node version 16, if you have a different version of Node, this will not work. See the bottom of this file for more information**

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

`npm install webpack@4.46.0 --save-dev --save-exact`

`npm install webpack-cli@3.3.12 --save-dev`

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

`npm install style-loader@1.3.0 css-loader@3.6.0 --save-dev`

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

`npm install html-webpack-plugin@4.5.2 --save-dev`

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

`npm install clean-webpack-plugin@3.0.0 --save-dev`

Add the plugin to our webpack config, right below HTML webpack plugin at the top of the file:

`const { CleanWebpackPlugin } = require('clean-webpack-plugin');`

Inside the plugins array, add this new line above `new Html WebpackPlugin()`:

`new CleanWebpackPlugin(),`

To prevent any issues with our bundled code and our browsers dev tools, we need to add this line, right above the `plugins` array:

`devtool: 'eval-source-map',`

To setup a dev live server, we need to install this:

`npm install webpack-dev-server@3.11.3 --save-dev --save-exact`

Add the devServer object to our webpack config, this goes right after the `output` object:

```
devServer: {
    contentBase: './dist'
  },

```

In order to actually run the server, we need to add in a script to our `package.json`. In the `scripts` object, add this line:

`"start": "webpack-dev-server --open --mode=development"`

Next we will install eslint, which will give us better errors and warnings:

`npm install eslint@8.18.0 --save-dev`

`npm install eslint-webpack-plugin@2.7.0 --save-dev`

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

`npm install jest@24.9.0 --save-dev`

In order to actually run our tests, we need to add another script to the `scripts` object of `package.json`:

`"test": "jest --coverage",`

Next we will install Babel, which will translate any modern JS code into legacy code incase there are any compatibility errors:

`npm install @babel/core@7.18.6 --save-dev`

`npm install @babel/plugin-transform-modules-commonjs@7.18.6 --save-dev`

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

For images, we need file loader and html loader:

`npm install file-loader --save-dev`

`npm install html-loader@1.3.2 --save-dev`

Add this config to our webpack, this goes in the `rules` array, next to our previous `css-loader` configuration:

```
{
  test: /\.(gif|png|avif|jpe?g)$/,
  use: [
    {
      loader: 'file-loader',
      options: {
        name: '[name].[ext]',
        outputPath: 'assets/images/'
      }
    }
  ]
},
{
  test:/\.html$/,
  use: [
    'html-loader'
  ]
},
```

After all that, here is what your final `webpack.config.js` should look like:

```
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
    contentBase: "./dist",
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
        use: [
          {
            loader: "file-loader",
            options: {
              name: "[name].[ext]",
              outputPath: "assets/images/",
            },
          },
        ],
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

```
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
    "clean-webpack-plugin": "^3.0.0",
    "css-loader": "^3.6.0",
    "eslint": "^8.18.0",
    "eslint-webpack-plugin": "^2.7.0",
    "file-loader": "^6.2.0",
    "html-loader": "^1.3.2",
    "html-webpack-plugin": "^4.5.2",
    "jest": "^24.9.0",
    "style-loader": "^1.3.0",
    "webpack": "4.46.0",
    "webpack-cli": "^3.3.12",
    "webpack-dev-server": "3.11.3"
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

Or any errors that mention an `SSL`, make sure you are on Node version 16.

## If you are using the latest version of Node:

You can follow the guide as is, but there are a couple changes you will need to make.

- When installing packages, do not include any version numbers, and instead download the latest version of all packages.
  For example, instead of running this command:

  `npm install webpack-cli@3.3.12 --save-dev`

  You won't include the `@3.3.12`. The command will now look like this:

  `npm install webpack-cli --save-dev`

- You do not need to install or configure File Loader

- You will need to change the `devServer` config of `webpack.config.js`. Instead of this:

```
  devServer: {
    contentBase: "./dist",
  },
```

You will replace that with this:

```
devServer: {
  static: {
    directory: path.join(__dirname, "dist"),
  },
}
```
