# NgUniversal

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 1.3.2.

## Development server

Run `npm start` for a dev server. Navigate to `http://localhost:8000/`.

## Steps

### for `package.json`

ADD this to `package.json`
```javascript
"main": "dist/server.js",
"engines": {
 "node": ">=6.0.0"
},
```

REPLACE the `scripts` section with:

```javascript
{
   "start": "npm run build && npm run server",
   "build": "webpack",
   "build:aot": "webpack --env.aot --env.client & webpack --env.aot --env.server",
   "build:prod": "webpack --env.aot --env.client -p & webpack --env.aot --env.server",
   "prebuild": "npm run clean",
   "prebuild:aot": "npm run clean",
   "prebuild:prod": "npm run clean",
   "clean": "rimraf dist",
   "server": "nodemon dist/server.js",
   "watch": "webpack --watch"
}
```

ADD this to `dependencies`
```javascript
  "@angular/platform-server": "^4.0.0",
  "@nguniversal/express-engine": "^1.0.0-beta.2",
  "express": "^4.15.2",
  "xhr2": "^0.1.4",
  "serialize-javascript": "^1.3.0",
```

ADD this to `devDependencies`
```javascript
  "@ngtools/webpack": "^1.3.1",
  "@types/express": "^4.0.35",
  "html-webpack-plugin": "^2.28.0",
  "nodemon": "^1.11.0",
  "rimraf": "^2.6.1",
  "raw-loader": "^0.5.1",
  "script-ext-html-webpack-plugin": "^1.7.1",
  "typescript": "~2.3.3",
  "webpack": "3.3.0",
  "webpack-merge": "^4.1.0",
  "webpack-node-externals": "^1.6.0",
  "less": "^2.5.3",
  "less-loader": "^2.2.1"
```

### ADD this to  `src/app/app.component.ts`
```javascript
 import { Component, OnInit } from '@angular/core';
 import { TransferState } from '../modules/transfer-state/transfer-state';
 import { REQUEST } from '@nguniversal/express-engine/tokens';
```

### ADD this to  `src/app/app.module.ts`
```javascript
 import { APP_BASE_HREF, CommonModule } from '@angular/common';
 import { HttpModule } from '@angular/http';
 import { RouterModule } from '@angular/router';
```
 in imports add:
```javascript
 CommonModule,
 HttpModule,
```

### CREATE `src/app/browser-app.module.ts`

```javascript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { AppModule } from './app.module';
import { BrowserTransferStateModule } from '../modules/transfer-state/browser-transfer-state.module';

@NgModule({
  bootstrap: [ AppComponent ],
  imports: [
    BrowserModule.withServerTransition({
      appId: 'my-app-id'
    }),
    BrowserTransferStateModule,
    AppModule
  ]
})
export class BrowserAppModule {}
```

### CREATE `src/app/server-app.module.ts`
```javascript
    import { NgModule, APP_BOOTSTRAP_LISTENER, ApplicationRef } from '@angular/core';
    import { ServerModule } from '@angular/platform-server';
    import { ServerTransferStateModule } from '../modules/transfer-state/server-transfer-state.module';
    import { AppComponent } from './app.component';
    import { AppModule } from './app.module';
    import { TransferState } from '../modules/transfer-state/transfer-state';
    import { BrowserModule } from '@angular/platform-browser';

    export function onBootstrap(appRef: ApplicationRef, transferState: TransferState) {
    return () => {
        appRef.isStable
        .filter(stable => stable)
        .first()
        .subscribe(() => {
            transferState.inject();
        });
    };
    }

    @NgModule({
    bootstrap: [AppComponent],
    imports: [
        BrowserModule.withServerTransition({
        appId: 'my-app-id'
        }),
        ServerModule,
        ServerTransferStateModule,
        AppModule
    ],
    providers: [
        {
        provide: APP_BOOTSTRAP_LISTENER,
        useFactory: onBootstrap,
        multi: true,
        deps: [
            ApplicationRef,
            TransferState
        ]
        }
    ]
    })
    export class ServerAppModule {

    }
```

### CREATE  `src/main.browser.ts`

```javascript
import 'zone.js/dist/zone';
import 'reflect-metadata';

import 'rxjs/Observable';
import 'rxjs/add/operator/map';

import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { BrowserAppModule } from './app/browser-app.module';

export function main() {
  return platformBrowserDynamic().bootstrapModule(BrowserAppModule);
}

document.addEventListener('DOMContentLoaded', main, false);
```

### CREATE `src/main.server.ts`

```javascript
import 'zone.js/dist/zone-node';
import 'reflect-metadata';
import 'rxjs/Rx';
import * as express from 'express';
import { Request, Response } from 'express';
import { platformServer, renderModuleFactory } from '@angular/platform-server';
import { ServerAppModule } from './app/server-app.module';
import { ngExpressEngine } from '@nguniversal/express-engine';
import { ROUTES } from './routes';
import { enableProdMode } from '@angular/core';
enableProdMode();
const app = express();
const port = 8000;
const baseUrl = `http://localhost:${port}`;

app.engine('html', ngExpressEngine({
  bootstrap: ServerAppModule
}));

app.set('view engine', 'html');
app.set('views', 'src');

app.use('/', express.static('dist', {index: false}));

ROUTES.forEach((route: string) => {
  app.get(route, (req: Request, res: Response) => {
    console.time(`GET: ${req.originalUrl}`);
    res.render('../dist/index', {
      req: req,
      res: res
    });
    console.timeEnd(`GET: ${req.originalUrl}`);
  });
});

app.listen(port, () => {
	console.log(`Listening at ${baseUrl}`);
});
```

### COPY the MODULES folder to your repo from `src/modules/`

This folder will containe all the module configs needed. (will add more explanation later)

### CREATE `src/routes.ts` 'Someone explain this file in the future'
```javascript
/**
 * Routes which are handled by Angular in Express
 */
export const ROUTES: string[] = [
  '/',
];
```

### CREATE `src/tsconfig.browser.json`

```javascript
{
    "extends": "../tsconfig.json",
    "angularCompilerOptions": {
        "entryModule": "./app/browser-app.module#BrowserAppModule"
    },
    "exclude": [
        "./main.server.aot.ts"
    ]
}
```

### CREATE `src/tsconfig.server.json`

```javascript
{
    "extends": "../tsconfig.json",
    "angularCompilerOptions": {
        "entryModule": "./app/server-app.module#ServerAppModule"
    },
    "exclude": []
}
```

### MODIFY `tsconfig.json`

Replace `"sourceMap": true,` with `"sourceMap": false,`
ADD `"module": "es2015",`
ADD `"noImplicitAny": false,`

### CREATE `webpack.config.js`

```javascript
const ngtools = require('@ngtools/webpack');
const webpackMerge = require('webpack-merge');
const commonPartial = require('./webpack/webpack.common');
const clientPartial = require('./webpack/webpack.client');
const serverPartial = require('./webpack/webpack.server');
const prodPartial = require('./webpack/webpack.prod');
const { getAotPlugin } = require('./webpack/webpack.aot');

module.exports = function (options, webpackOptions) {
  options = options || {};

  if (options.aot) {
    console.log(`Running build for ${options.client ? 'client' : 'server'} with AoT Compilation`)
  }

  const serverConfig = webpackMerge({}, commonPartial, serverPartial, {
    plugins: [
      getAotPlugin('server', !!options.aot)
    ]
  });

  let clientConfig = webpackMerge({}, commonPartial, clientPartial, {
    plugins: [
      getAotPlugin('client', !!options.aot)
    ]
  });

  if (webpackOptions.p) {
    clientConfig = webpackMerge({}, clientConfig, prodPartial);
  }

  const configs = [];
  if (!options.aot) {
    configs.push(clientConfig, serverConfig);

  } else if (options.client) {
    configs.push(clientConfig);

  } else if (options.server) {
    configs.push(serverConfig);
  }

  return configs;
}
```

### COPY the `webpack` folder to your project

This contains all the configurations of webpack - will add more info later



## TODO

Figure out `ng generate component example` why it dose not work
Explain the need for another route file `src/routes.ts`.