## T.R.E.W for Dummies
Pragmatic starter guide for **Typescript**, **React**, **Electron** and **Webpack** application.

### Create project directory
```
mkdir project_name
cd project_name
```

### Initialize Node Project
```
npm init
```

### Install dependencies
#### Typescript
```
npm install --save-dev typescript tslint ts-node @types/node
```
* `ts-node` is needed to write **webpack config** file in **Typescript**.
* For **Vscode** users, make sure to use the local typescript and lint rules
  ```js
  // .vscode/settings.json
  {
    "typescript.tsdk": "./node_modules/typescript/lib",
    "tslint.configFile": "./tslint.json"
  }
  ```
#### React
```
npm install --save react react-dom
npm install --save-dev @types/react @types/react-dom
```
#### Electron
```
npm install --save-dev electron electron-builder
```
#### Webpack
```
npm install --save-dev webpack webpack-cli source-map-loader ts-loader html-webpack-plugin @types/webpack @types/html-webpack-plugin
```

### Create configuration files
#### tsconfig.json
```
npx typescript --init
```
Example
```ts
{
  "compilerOptions": {
    "target": "es6",
    "module": "commonjs",
    "jsx": "react",
    "sourceMap": true,
    "strict": true,
    "noImplicitAny": true,
    "esModuleInterop": true,
    /* OPTIONAL */
    "baseUrl": "./", // Required if "paths" option is used
    "paths": {
      "@components/*" : ["src/components/*"] // Enables cleaner import statements
    }
  }
}
```
#### tslint.json
```
npx tslint --init
```
Example
```ts
{
  "defaultSeverity": "error",
  "extends": ["tslint:recommended"],
  "jsRules": true,
  "rules": {
    "quotemark": [true, "single"],
    "trailing-comma": false,
    "only-arrow-functions": false,
    "object-literal-sort-keys": [true, "match-declaration-order-only"],
    "ordered-imports": false
  },
  "rulesDirectory": [],
  "linterOptions": {
    "exclude": ["node_modules/*", "dist/*"]
  }
}
```
#### webpack.config.ts
Example
```ts
import path from 'path';
import webpack from 'webpack';
import HtmlWebpackPlugin from 'html-webpack-plugin';

const base: webpack.Configuration = {
  mode: process.env.NODE_ENV === 'production' ? 'production' : 'development',
  devtool: 'source-map',
  module: {
    rules: [
      {
        exclude: [path.resolve('node_modules')],
        include: [path.resolve('src')],
        loader: 'ts-loader',
        test: /\.tsx?$/
      },
      {
        enforce: 'pre',
        loader: 'source-map-loader',
        test: /\.js$/
      }
    ]
  },
  resolve: {
    alias: {
      /* These need to match "paths" in tsconfig.json */
      '@components': path.resolve('src', 'components')
    },
    extensions: ['.json', '.ts', '.tsx', '.js']
  },
  node: {
    /*  
    Disables webpack's default "mock" behavior for below node globals
    Ref: https://github.com/electron/electron/issues/5107#issuecomment-299971806
    */
    __dirname: false,
    __filename: false
  }
};

const mainConfig: webpack.Configuration = {
  ...base,
  entry: path.resolve('src', 'index.ts'),
  output: {
    path: path.resolve('dist'),
    publicPath: '/dist/',
    filename: 'main.bundle.js'
  },
  target: 'electron-main' // Compiles for Electron main process, enabling Node.js APIs
};
const rendererConfig: webpack.Configuration = {
  ...base,
  entry: path.resolve('src', 'renderer.tsx'),
  output: {
    path: path.resolve('dist'),
    publicPath: '.',
    filename: 'renderer.bundle.js'
  },
  target: 'electron-renderer', // Compiles for Electron renderer process, enabling Node.js APIs
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve('src', 'index.html'),
      title: process.env.APP_NAME
    })
  ]
};

export default [mainConfig, rendererConfig];

```
* See [all webpack configuration options](https://webpack.js.org/configuration/).

### Sample application

#### src/index.html
**Webpack** will use this template to generate HTML file to be loaded by **Electron**.
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <!-- Webpack will define the title -->
    <title><%= htmlWebpackPlugin.options.title %></title>
</head>

<body>
    <!-- React will render the view -->
    <div id="app"></div>
</body>

</html>
```

#### src/index.ts
**Electron**'s main process.
```ts
// Modules to control application life and create native browser window
import { app, BrowserWindow } from 'electron';
import * as path from 'path';

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let mainWindow: Electron.BrowserWindow | null;

function createWindow() {
  // Create the browser window.
  mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      devTools: process.env.NODE_ENV === 'production' ? false : true,
      /*
      Enables Node.js APIs in renderer process.
      See related security risks to enabling this option.
      https://github.com/electron/electron/blob/master/docs/tutorial/security.md#isolation-for-untrusted-content
      */
      nodeIntegration: true
    }
  });

  // and load the index.html of the app.
  mainWindow.loadFile(path.join(__dirname, 'index.html'));

  // Open the DevTools.
  mainWindow.webContents.openDevTools();

  // Emitted when the window is closed.
  mainWindow.on('closed', function() {
    // Dereference the window object, usually you would store windows
    // in an array if your app supports multi windows, this is the time
    // when you should delete the corresponding element.
    mainWindow = null;
  });
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.on('ready', createWindow);

// Quit when all windows are closed.
app.on('window-all-closed', function() {
  // On macOS it is common for applications and their menu bar
  // to stay active until the user quits explicitly with Cmd + Q
  if (process.platform !== 'darwin') {
    app.quit();
  }
});

app.on('activate', function() {
  // On macOS it's common to re-create a window in the app when the
  // dock icon is clicked and there are no other windows open.
  if (mainWindow === null) {
    createWindow();
  }
});

// In this file you can include the rest of your app's specific main process
// code. You can also put them in separate files and require them here.
```

#### src/renderer.tsx
**Electron**'s renderer process using **React**.
```tsx
import * as React from 'react';
import * as ReactDOM from 'react-dom';

import App from '@components/App';

ReactDOM.render(<App />, document.getElementById('app'));
```

#### src/components/App.tsx
Top-level **React** component.
```tsx
import * as React from 'react';

export default class App extends React.Component {
  public render() {
    return (
      <div>
        {/* All of the Node.js APIs are available in this renderer process. */}
        <h1>{process.env.APP_NAME}</h1>
        <div>Node.js: {process.versions.node}</div>
        <div>Chromium: {process.versions.chrome},</div>
        <div>Electron: {process.versions.electron}</div>
      </div>
    );
  }
}
```

### Build & Run 
These steps assume above configurations and files.

###### Build the modules
```
APP_NAME='your_app_name' npx webpack
```
* For a production build, set environment variable `NODE_ENV=production`.
###### Run Electron app
```
APP_NAME='your_app_name' npx electron ./dist/main.bundle.js
```

### Create installer
Update `package.json`
```ts
{
  /* Other properties */

  // Entry point of the app
  "main": "./dist/main.bundle.js",

  // Electron-builder
  "build": {
    "appId": "your.id",
    "directories": {
      "app": "./", // Relative path to package.json's directory
      "output": "./installer/"
    },
    "mac": {
      /* MacOS specific options */
    },
    "win": {
      /* Windows specific options */
    }
  }
}
```
* See [all available builder options](https://www.electron.build/configuration/configuration)

Create a production build
```
NODE_ENV=production APP_NAME='your_app_name' npx webpack
```
Create app installer
```
npx electron-builder build --{win | mac} --x64
```

### Reference
###### Documentation
* [Typescript Official Documentation](https://www.typescriptlang.org/docs/home.html)
* [React Official Documentation](https://reactjs.org/docs/getting-started.html)
* [Electron Official Documentation](https://electronjs.org/docs)
* [Webpack Official Documentation](https://webpack.js.org/concepts/)
###### GitHub Repository
* [electron / electron-quick-start-typescript](https://github.com/electron/electron-quick-start-typescript)
* [electron-userland / electron-builder
](https://github.com/electron-userland/electron-builder)
* [Devtography / electron-react-typescript-webpack-boilerplate
](https://github.com/Devtography/electron-react-typescript-webpack-boilerplate)
###### Medium
* [How to store user data in Electron](https://medium.com/cameron-nokes/how-to-store-user-data-in-electron-3ba6bf66bc1e)
###### Music I listened to while writing this
* [Best of Shingo Nakamura 01 (2-Hour Melodic Progressive House Mix)
](https://www.youtube.com/watch?v=WYp9Eo9T3BA&list=RDsnX0aKTCw-0&index=4)