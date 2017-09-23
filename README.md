# NgElectron

This project walks through how to create a basic Electron app using Angular and the [Angular CLI](https://github.com/angular/angular-cli)

# Setup
[how to setup node, angular, the cli correctly](https://gist.github.com/milesstanfield/17f980ad4ed6d038a255f8fc3b222add#file-angular-and-node-setup-md)
if you use electron-packager for production builds you will need to have [wine](https://www.winehq.org/) installed `brew install wine`

# Create an angular app with the cli
```
ng new my-electron-app && cd my-electron-app
```

# download the main electron.ts file
```
curl -o ./electron.ts https://raw.githubusercontent.com/milesstanfield/ng-electron/master/electron.ts
```

# use cli to generate a service then replace it with the default one from this repo
```
ng g s services/electron -m app.module
curl -o ./src/app/services/electron.service.ts https://raw.githubusercontent.com/milesstanfield/ng-electron/master/src/app/services/electron.service.ts
```

# add the following to the package.json file. Dont skip any of these!
```
...
"main": "electron.js",
"directories": {
  "buildResources": "dist",
  "app": "dist"
},
"scripts": {
  ...  
  "rimraf": "rimraf",
  "clean": "yarn run rimraf -- dist",
  "build:watch": "yarn run clean && yarn run build --watch true",
  "build:electron": "tsc electron.ts --outDir dist && copyfiles package.json dist && cd dist && yarn install --prod && cd ..",
  "start:electron": "electron ./dist --serve",
  "package": "electron-packager ./dist My Electron App --all"
},
...
```

#  Update the version key with electron version in package.json
you can find electron version by doing `yarn electron -v`
```
...
"version": "1.6.13",
...
```



# install electron and other dependencies
```
yarn add electron electron-reload electron-packager rimraf copyfiles --save-dev
```

there is a [security issue with electron v1.8.0](https://github.com/segmentio/nightmare/issues/1269) which has caused a revert to last stable version so make sure
you use `"electron": "1.7.6",` until that is resolved


# update bottom of typings.d.ts file with window logic
```
declare var window: Window;
interface Window {
  process: any;
  require: any;
}
```

# add the following to app.component.ts
```
...
import { ElectronService } from './services/electron.service';

export class AppComponent {
  ...

  constructor(public electronService: ElectronService) {
    if (electronService.isElectron()) {
      console.log('Mode electron');
      // Check if electron is correctly injected (see externals in webpack.config.js)
      console.log('c', electronService.ipcRenderer);
      // Check if nodeJs childProcess is correctly injected (see externals in webpack.config.js)
      console.log('c', electronService.childProcess);
    } else {
      console.log('Mode web');
    }
  }
}
```

# change index.html base tag to ./
```
<base href="./">
```

# do this in one tab to build the app and watch for changes
```
yarn run build:watch
```

# do this in another tab to build the electron portion and start it
```
yarn run build:electron
yarn run start:electron
```

# you can package the builds for app submission and/or download now with electron-packager
```
yarn run package
```
