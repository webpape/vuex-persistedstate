# vuex-persistedstate

Persist and rehydrate your [Vuex](http://vuex.vuejs.org/) state between page reloads.

<hr />

[![NPM version](https://img.shields.io/npm/v/vuex-persistedstate.svg)](https://www.npmjs.com/package/vuex-persistedstate)
[![NPM downloads](https://img.shields.io/npm/dm/vuex-persistedstate.svg)](https://www.npmjs.com/package/vuex-persistedstate)
[![Prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg)](https://github.com/prettier/prettier)
[![MIT license](https://img.shields.io/github/license/robinvdvleuten/vuex-persistedstate.svg)](https://github.com/robinvdvleuten/vuex-persistedstate/blob/master/LICENSE)

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)
[![Code Of Conduct](https://img.shields.io/badge/code%20of-conduct-ff69b4.svg)](https://github.com/robinvdvleuten/vuex-persistedstate/blob/master/.github/CODE_OF_CONDUCT.md)

<a href="https://webstronauts.com/">
    <img src="https://webstronauts.com/badges/sponsored-by-webstronauts.svg" alt="Sponsored by The Webstronauts" width="200" height="65">
</a>

## Install

```bash
npm install --save vuex-persistedstate
```

The [UMD](https://github.com/umdjs/umd) build is also available on [unpkg](https://unpkg.com):

```html
<script src="https://unpkg.com/vuex-persistedstate/dist/vuex-persistedstate.umd.js"></script>
```

You can find the library on `window.createPersistedState`.

## Usage

```js
import { createStore } from "vuex";
import createPersistedState from "vuex-persistedstate";

const store = createStore({
  // ...
  plugins: [createPersistedState()],
});
```

For usage with for Vuex 3 and Vue 2, please see [3.x.x branch](https://github.com/robinvdvleuten/vuex-persistedstate/tree/3.x.x).

## Examples

Check out a basic example on [CodeSandbox](https://codesandbox.io).

[![Edit vuex-persistedstate](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/80k4m2598)

Or configured to use with [js-cookie](https://github.com/js-cookie/js-cookie).

[![Edit vuex-persistedstate with js-cookie](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/xl356qvvkz)

Or configured to use with [secure-ls](https://github.com/softvar/secure-ls)

[![Edit vuex-persistedstate with secure-ls (encrypted data)](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/vuex-persistedstate-with-secure-ls-encrypted-data-7l9wb?fontsize=14)

### Example with Vuex modules

New plugin instances can be created in separate files, but must be imported and added to plugins object in the main Vuex file.

```js
/* module.js */
export const dataStore = {
  state: {
    data: []
  }
}

/* store.js */
import { dataStore } from './module'

const dataState = createPersistedState({
  paths: ['data']
})

export new Vuex.Store({
  modules: {
    dataStore
  },
  plugins: [dataState]
})
```

### Example with Nuxt.js

It is possible to use vuex-persistedstate with Nuxt.js. It must be included as a NuxtJS plugin:

#### With local storage (client-side only)

```javascript
// nuxt.config.js

...
/*
 * Naming your plugin 'xxx.client.js' will make it execute only on the client-side.
 * https://nuxtjs.org/guide/plugins/#name-conventional-plugin
 */
plugins: [{ src: '~/plugins/persistedState.client.js' }]
...
```

```javascript
// ~/plugins/persistedState.client.js

import createPersistedState from 'vuex-persistedstate'

export default ({store}) => {
  createPersistedState({
    key: 'yourkey',
    paths: [...]
    ...
  })(store)
}
```

#### Using cookies (universal client + server-side)

Add `cookie` and `js-cookie`:

`npm install --save cookie js-cookie`
or `yarn add cookie js-cookie`

```javascript
// nuxt.config.js
...
plugins: [{ src: '~/plugins/persistedState.js'}]
...
```

```javascript
// ~/plugins/persistedState.js

import createPersistedState from 'vuex-persistedstate';
import * as Cookies from 'js-cookie';
import cookie from 'cookie';

export default ({ store, req }) => {
    createPersistedState({
        paths: [...],
        storage: {
            getItem: (key) => {
                // See https://nuxtjs.org/guide/plugins/#using-process-flags
                if (process.server) {
                    const parsedCookies = cookie.parse(req.headers.cookie);
                    return parsedCookies[key];
                } else {
                    return Cookies.get(key);
                }
            },
            // Please see https://github.com/js-cookie/js-cookie#json, on how to handle JSON.
            setItem: (key, value) =>
                Cookies.set(key, value, { expires: 365, secure: false }),
            removeItem: key => Cookies.remove(key)
        }
    })(store);
};
```

## API

### `createPersistedState([options])`

Creates a new instance of the plugin with the given options. The following options
can be provided to configure the plugin for your specific needs:

- `key <String>`: The key to store the persisted state under. Defaults to `vuex`.
- `paths <Array>`: An array of any paths to partially persist the state. If no paths are given, the complete state is persisted. If an empty array is given, no state is persisted. Paths must be specified using dot notation. If using modules, include the module name. eg: "auth.user" Defaults to `undefined`.
- `reducer <Function>`: A function that will be called to reduce the state to persist based on the given paths. Defaults to include the values.
- `subscriber <Function>`: A function called to setup mutation subscription. Defaults to `store => handler => store.subscribe(handler)`.

- `storage <Object>`: Instead of (or in combination with) `getState` and `setState`. Defaults to localStorage.
- `getState <Function>`: A function that will be called to rehydrate a previously persisted state. Defaults to using `storage`.
- `setState <Function>`: A function that will be called to persist the given state. Defaults to using `storage`.
- `filter <Function>`: A function that will be called to filter any mutations which will trigger `setState` on storage eventually. Defaults to `() => true`.
- `overwrite <Boolean>`: When rehydrating, whether to overwrite the existing state with the output from `getState` directly, instead of merging the two objects with `deepmerge`. Defaults to `false`.
- `arrayMerger <Function>`: A function for merging arrays when rehydrating state. Defaults to `function (store, saved) { return saved }` (saved state replaces supplied state).
- `rehydrated <Function>`: A function that will be called when the rehydration is finished. Useful when you are using Nuxt.js, which the rehydration of the persisted state happens asynchronously. Defaults to `store => {}`
- `fetchBeforeUse <Boolean>`: A boolean indicating if the state should be fetched from storage before the plugin is used. Defaults to `false`.
- `assertStorage <Function>`: An overridable function to ensure storage is available, fired on plugins's initialization. Default one is performing a Write-Delete operation on the given Storage instance. Note, default behaviour could throw an error (like `DOMException: QuotaExceededError`).

## Customize Storage

If it's not ideal to have the state of the Vuex store inside localstorage. One can easily implement the functionality to use [cookies](https://github.com/js-cookie/js-cookie) for that (or any other you can think of);

[![Edit vuex-persistedstate with js-cookie](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/xl356qvvkz?autoresize=1)

```js
import { Store } from "vuex";
import createPersistedState from "vuex-persistedstate";
import * as Cookies from "js-cookie";

const store = new Store({
  // ...
  plugins: [
    createPersistedState({
      storage: {
        getItem: (key) => Cookies.get(key),
        // Please see https://github.com/js-cookie/js-cookie#json, on how to handle JSON.
        setItem: (key, value) =>
          Cookies.set(key, value, { expires: 3, secure: true }),
        removeItem: (key) => Cookies.remove(key),
      },
    }),
  ],
});
```

In fact, any object following the Storage protocol (getItem, setItem, removeItem, etc) could be passed:

```js
createPersistedState({ storage: window.sessionStorage });
```

This is especially useful when you are using this plugin in combination with server-side rendering, where one could pass an instance of [dom-storage](https://www.npmjs.com/package/dom-storage).

### 🔐Obfuscate Local Storage

If you need to use **Local Storage** (or you want to) but want to prevent attackers from easily inspecting the stored data, you can [obfuscate it]('https://github.com/softvar/secure-ls').

**Important ⚠️** Obfuscating the Vuex store means to prevent attackers from easily gaining access to the data. This is not a secure way of storing sensitive data (like passwords, personal information, etc.), and always needs to be used in conjunction with some other authentication method of keeping the data (such as Firebase or your own server).

[![Edit vuex-persistedstate with secure-ls (obfuscated data)](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/vuex-persistedstate-with-secure-ls-encrypted-data-7l9wb?fontsize=14)

```js
import { Store } from "vuex";
import createPersistedState from "vuex-persistedstate";
import SecureLS from "secure-ls";
var ls = new SecureLS({ isCompression: false });

// https://github.com/softvar/secure-ls

const store = new Store({
  // ...
  plugins: [
    createPersistedState({
      storage: {
        getItem: (key) => ls.get(key),
        setItem: (key, value) => ls.set(key, value),
        removeItem: (key) => ls.remove(key),
      },
    }),
  ],
});
```

### ⚠️ LocalForage ⚠️

As it maybe seems at first sight, it's not possible to pass a [LocalForage](https://github.com/localForage/localForage) instance as `storage` property. This is due the fact that all getters and setters must be synchronous and [LocalForage's methods](https://github.com/localForage/localForage#callbacks-vs-promises) are asynchronous.

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.


## License

The MIT License (MIT). Please see [License File](LICENSE) for more information.
