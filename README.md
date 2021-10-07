# react-boilerplate-electron
How to setup react-boilerplate with electron

###### *This is the setup for `electron` on existing [react-boilerplate](https://github.com/react-boilerplate/react-boilerplate) MyWork_React.

---

**Getting started:**
- `$yarn dev:electron` - for local development that will run on `localhost:3000`
- `$yarn electron:pack-dev` - for compiling and to have the stand alone app dev mode environment. You can see the build window app at `/electron/dist`, there will be a `Setup` inside this directory. You can also check the cracked application at `/electron/dist/win-unpacked`.

**New installed libraries:**
```
- electron
- electron-builder
- electron-is-dev
- concurrently 
- wait-on
```

**Structure - changes for electron *(location of files that has changes or has been added.)*:**
```
├── app/
├── electron/
|  └── index.js
|  ├── build/ // this is the copy directory from the root 
|  └── dist/ // this is the compiled directory of electron
├── internals/
|  └── webpack/
|  |  └── webpack.base.babel.js
└── package.json
```

**Added and updated commits:**

*`electron/index.js`*
```js
const electron = require('electron');
const { app, BrowserWindow } = electron;
const path = require('path');
const isDev = require('electron-is-dev');

let mainWindow = null;
app.on('ready', createWindow);
app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') {
    app.quit()
  }
});
app.on('activate', () => {
  if (mainWindow === null || BrowserWindow.getAllWindows().length === 0) {
    createWindow();
  }
});

function createWindow() {
  mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true,
      webSecurity: false,
      contextIsolation: false
    },
    title: "React Electron"
  });
  mainWindow.loadURL(
    isDev ?
    'http://localhost:3000' :
    `file://${path.join(__dirname, "./build/index.html")}` // this will target the /electron/build/index.html
  );
  mainWindow.on('closed', function () {
    mainWindow = null
  })
  mainWindow.on('page-title-updated', function (e) {
    e.preventDefault()
  });

  if (isDev) {
    mainWindow.webContents.openDevTools({ mode: 'detach' });
  }
}
```

*`package.json`*
```js
{
  "main": "electron/index.js",
  "build": {
    "productName": "React Electron",
    "appId": "<com.your_app>",
    "directories": {
      "output": "electron/dist"
    },
    "win": {
      "target": [
        "nsis"
      ]
    }
  },
  "scripts": {
    "dev:electron": "concurrently -k \"npm run start\" \"npm:electron\"",
    "electron": "wait-on tcp:3000 && electron .",
    "electron:build-dev": "npm run build:dev && cp -r build/ electron/build",
    "electron:repack": "rm -rf electron/build electron/dist && electron-builder",
    "electron:pack-dev": "cross-env IS_ELECTRON=true env-cmd npm run electron:build-dev && electron-builder",
  }
}
```

*`\internals\webpack\webpack.base.babel.js`*
```
const isElectron = process.env.IS_ELECTRON
module.exports = options => ({
  output: Object.assign(
    {
      // Compile into js/build.js
      path: path.resolve(process.cwd(), isElectron ? 'build' : BUILD_FOLDER_PATH),
      publicPath: isElectron ? '' : PUBLIC_PATH
    },
    options.output
  ), // Merge with env dependent settings
  target: isElectron ? 'electron-renderer' : 'web', // Make variables accessible to webpack
})
```
