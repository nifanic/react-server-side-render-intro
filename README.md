# Intro to React Server Side Rendering (SSR)

This is a 2020-updated version of [An Introduction to React Server-Side Rendering](https://alligator.io/react/server-side-rendering/) by @alligatorio.

## Create New App with [`create-react-app`](https://github.com/facebook/create-react-app)

0. In Terminal, choose existing (or create new) folder, and make it current.

1. To create new React app named `react-ssr-intro`, run—

   ```
   $ npx create-react-app react-ssr-intro
   ```

1. Check if the app runs without errors—

   ```
   $ yarn start
   ```

## Create `Home` Component

```jsx harmony
// ./src/Home.js

import React from "react";

export default ({ name }) => <h1>Hello, {name}</h1>;
```

Use `Home` component in the `App`—

```jsx harmony
// ./src/App.js

import React from "react";
import Home from "./Home";

export default () => <Home name="Alligator" />;
```

## `hydrate` instead of `render`

Use ReactDOM's `hydrate` method to facilitate server side rendering—

```jsx harmony
// ./src/index.js

import React from "react";
import ReactDOM from "react-dom";
import App from "./App";

ReactDOM.hydrate(<App />, document.getElementById("root"));
``` 

## Set Up A Server

Express is a good candidate for a simple server that will render the app output and send it along.

0. Add the component—

   ```
   $ yarn add express
   ```

0. Create the `server` folder, on the `src` level, and the `index.js` file inside—

   ```
   $ mkdir server && touch server/index.js
   ```

0. Modify this `index.js`—

   ```jsx harmony
   // ./server/index.js
   
   import path from "path";
   import fs from "fs";
   
   import React from "react";
   import express from "express/lib/express";
   import ReactDOMServer from "react-dom/server";
   
   import App from "../src/App";
   
   const PORT = process.env.PORT || 3006;
   const app = express();
   
   const appDataFolder =
     process.env.APPDATA ||
     (process.platform === "darwin"
       ? process.env.HOME + "/Library/Preferences"
       : process.env.HOME + "/.local/share");
   console.log("AppData folder:", appDataFolder);
   
   // app.get("/", express.static("build"));
   
   app.get("/*", (req, res) => {
     const app = ReactDOMServer.renderToString(<App />);
     const indexFile = path.resolve("build/index.html");
     fs.readFile(indexFile, "utf8", (err, data) => {
       if (err) {
         console.error("Something went wrong:", err);
         return res.status(500).send("Oops, better luck next time!");
       }
   
       return res
         .status(200)
         .send(
           data.replace('<div id="root"></div>', `<div id="root">${app}</div>`)
         );
     });
   });
   
   app.listen(PORT, () => {
     console.log(`😎 Server is listening on port ${PORT}`);
   });
   
   ```

## Configure Webpack and Babel

0. Install components as `devDependencies`—

   ```
   $ yarn add --dev @babel/core @babel/preset-env @babel/preset-react babel-loader nodemon webpack-cli webpack-node-externals concurrently
   ```

0. In the project root, create Babel config file, `.babelrc`—

   ```
   {
     "presets": ["@babel/preset-env", "@babel/preset-react"]
   }
   ```

0. In the project root, create `webpack.server.js` config file—

   ```javascript
   const path = require("path");
   const nodeExternals = require("webpack-node-externals");
   
   module.exports = {
     entry: "./server/index.js",
     target: "node",
     externals: [nodeExternals()],
     output: {
       path: path.resolve("server-build"),
       filename: "index.js"
     },
     module: {
       rules: [
         {
           test: /\.js$/,
           exclude: /node_modules/,
           use: "babel-loader"
         }
       ]
     }
   };
   ```
   
   Webpack transpiles the server (written in ES6) to ES5 executable by NodeJS, by generating `server-build/index.js`.

## Add NPM scripts

Open `package.json`, and update its `scripts` node with all three `dev` entries—

```json
"scripts": {
  "start": "react-scripts start",
  "build": "react-scripts build",
  "test": "react-scripts test",
  "eject": "react-scripts eject",
  "dev:build-server": "NODE_ENV=development webpack --config webpack.server.js --mode=development -w",
  "dev:start": "nodemon ./server-build/index.js",
  "dev": "concurrently \"yarn build\" \"yarn dev:build-server\" \"yarn dev:start\""
}
```

## Run the App

```
$ yarn dev
```

Your app should be available on [`http://localhost:3006`](http://localhost:3006).
