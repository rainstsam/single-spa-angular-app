<p float="left">
  <img src="https://single-spa.js.org/img/logo-white-bgblue.svg" width="50" height="50">
  <img src="https://angular.io/assets/images/logos/angular/angular.png" width="50" height="50">
</p>

[![npm version](https://img.shields.io/npm/v/single-spa-angular-app.svg?style=flat-square)](https://www.npmjs.org/package/single-spa-angular-app)
[![](https://data.jsdelivr.com/v1/package/npm/single-spa-angular-app/badge)](https://www.jsdelivr.com/package/npm/single-spa-angular-app)

# single-spa-angular-app

This is an Angular v8 application example used as NPM package in [single-spa-login-example-with-npm-packages](https://github.com/jualoppaz/single-spa-login-example-with-npm-packages) in order to register an application. See the installation instructions there.

## ✍🏻 Motivation

This is an Angular v8 application which contains two routed pages for embbed the app inside a root single-spa application.

## How it works ❓

There are several files for the right working of this application and they are:

- angular.json
- extra-webpack.config.ts
- src/main.single-spa.ts
- package.json

### angular.json
```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "single-spa-angular-app": {
      "projectType": "application",
      "schematics": {
        "@schematics/angular:component": {
          "style": "scss"
        }
      },
      "root": "",
      "sourceRoot": "src",
      "prefix": "app",
      "architect": {
        "build": {
          "builder": "@angular-builders/custom-webpack:browser",
          "options": {
            "outputPath": "dist",
            "index": "src/index.html",
            "main": "src/main.single-spa.ts",
            "polyfills": "src/polyfills.ts",
            "tsConfig": "tsconfig.app.json",
            "aot": false,
            "assets": [
              "src/favicon.ico",
              "src/assets"
            ],
            "styles": [
              "src/styles.scss"
            ],
            "scripts": [],
            "customWebpackConfig": {
              "path": "./extra-webpack.config.js"
            }
          },
          "configurations": {
            "production": {
              "fileReplacements": [
                {
                  "replace": "src/environments/environment.ts",
                  "with": "src/environments/environment.prod.ts"
                }
              ],
              "optimization": true,
              "outputHashing": "none",
              "sourceMap": false,
              "extractCss": true,
              "namedChunks": false,
              "aot": true,
              "extractLicenses": true,
              "vendorChunk": false,
              "buildOptimizer": true,
              "budgets": [
                {
                  "type": "initial",
                  "maximumWarning": "2mb",
                  "maximumError": "5mb"
                },
                {
                  "type": "anyComponentStyle",
                  "maximumWarning": "6kb",
                  "maximumError": "10kb"
                }
              ]
            }
          }
        },
        "serve": {
          "builder": "@angular-builders/custom-webpack:dev-server",
          "options": {
            "browserTarget": "single-spa-angular-app:build"
          },
          "configurations": {
            "production": {
              "browserTarget": "single-spa-angular-app:build:production"
            }
          }
        },
        "extract-i18n": {
          "builder": "@angular-devkit/build-angular:extract-i18n",
          "options": {
            "browserTarget": "single-spa-angular-app:build"
          }
        },
        "test": {
          "builder": "@angular-devkit/build-angular:karma",
          "options": {
            "main": "src/test.ts",
            "polyfills": "src/polyfills.ts",
            "tsConfig": "tsconfig.spec.json",
            "karmaConfig": "karma.conf.js",
            "assets": [
              "src/favicon.ico",
              "src/assets"
            ],
            "styles": [
              "src/styles.scss"
            ],
            "scripts": []
          }
        },
        "lint": {
          "builder": "@angular-devkit/build-angular:tslint",
          "options": {
            "tsConfig": [
              "tsconfig.app.json",
              "tsconfig.spec.json",
              "e2e/tsconfig.json"
            ],
            "exclude": [
              "**/node_modules/**"
            ]
          }
        },
        "e2e": {
          "builder": "@angular-devkit/build-angular:protractor",
          "options": {
            "protractorConfig": "e2e/protractor.conf.js",
            "devServerTarget": "single-spa-angular-app:serve"
          },
          "configurations": {
            "production": {
              "devServerTarget": "single-spa-angular-app:serve:production"
            }
          }
        }
      }
    }
  },
  "defaultProject": "single-spa-angular-app"
}
```

The essential config is in **@angular-builders/custom-webpack:browser** builder. Be care of this config is autogenerated when install **single-spa-angular**.

### extra-webpack.config.js

```javascript
const singleSpaAngularWebpack = require('single-spa-angular/lib/webpack').default

module.exports = (angularWebpackConfig, options) => {
  const singleSpaWebpackConfig = singleSpaAngularWebpack(angularWebpackConfig, options)

  // Feel free to modify this webpack config however you'd like to
  return singleSpaWebpackConfig
}
```

This file has no custom config. But we must set desired config here if needed.

### src/main.single-spa.ts

```javascript
import { enableProdMode, NgZone } from '@angular/core';

import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { Router } from '@angular/router';
import { AppModule } from './app/app.module';
import { environment } from './environments/environment';
import singleSpaAngular from 'single-spa-angular';
import { singleSpaPropsSubject } from './single-spa/single-spa-props';

if (environment.production) {
  enableProdMode();
}

const lifecycles = singleSpaAngular({
  bootstrapFunction: singleSpaProps => {
    singleSpaPropsSubject.next(singleSpaProps);
    return platformBrowserDynamic().bootstrapModule(AppModule);
  },
  template: '<app-root />',
  Router,
  NgZone,
  domElementGetter: () => document.getElementById('angular-app')
});

export const bootstrap = lifecycles.bootstrap;
export const mount = lifecycles.mount;
export const unmount = lifecycles.unmount;
```

The **lifecycles** object contains all **single-spa-angular** methods for the **single-spa** lifecycle of this app. All used config is default one but the custom config of the **domElementGetter** option. It's assumed that an element with **angular-app** id is defined in the **index.html** where this application will be mounted.

### package.json

```json
{
  "name": "single-spa-angular-app",
  "version": "0.1.3",
  "description": "Angular v8 application with two example pages for be included in a single-spa application as registered app.",
  "main": "dist/main-es2015.js",
  "scripts": {
    "ng": "ng",
    "build": "ng build",
    "test": "ng test",
    "lint": "ng lint",
    "build:single-spa": "ng build --prod"
  },
  "dependencies": {
    "@angular-builders/custom-webpack": "8.4.1",
    "@angular/common": "8.2.14",
    "@angular/compiler": "8.2.14",
    "@angular/core": "8.2.14",
    "@angular/forms": "8.2.14",
    "@angular/platform-browser": "8.2.14",
    "@angular/platform-browser-dynamic": "8.2.14",
    "@angular/router": "8.2.14",
    "rxjs": "6.4.0",
    "single-spa-angular": "3.1.0",
    "tslib": "1.10.0",
    "zone.js": "0.9.1"
  },
  "devDependencies": {
    "@angular-devkit/build-angular": "0.803.23",
    "@angular-devkit/build-ng-packagr": "0.803.23",
    "@angular/cli": "8.3.23",
    "@angular/compiler-cli": "8.2.14",
    "@angular/language-service": "8.2.14",
    "@types/node": "8.9.5",
    "@types/jasmine": "3.3.16",
    "@types/jasminewd2": "2.0.8",
    "codelyzer": "5.2.1",
    "jasmine-core": "3.4.0",
    "jasmine-spec-reporter": "4.2.1",
    "karma": "4.1.0",
    "karma-chrome-launcher": "2.2.0",
    "karma-coverage-istanbul-reporter": "2.0.6",
    "karma-jasmine": "2.0.1",
    "karma-jasmine-html-reporter": "1.5.1",
    "ng-packagr": "5.7.1",
    "protractor": "5.4.2",
    "ts-node": "7.0.1",
    "tsickle": "0.37.1",
    "tslint": "5.15.0",
    "typescript": "3.5.3"
  }
}
```

There are several scripts in this project:

- **ng**: for use global ng cli
- **test**: use for run unit tests
- **lint**: for run **eslint** in all project
- **build:single-spa**: for compile the application and build it as a **libray** in **umd** format
