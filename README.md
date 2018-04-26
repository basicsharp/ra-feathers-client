# Feathers REST Client for React-Admin

The perfect match to build Backend and Frontend Admin, based on REST services.
For using [feathers](https://www.feathersjs.com) with [react-admin](https://github.com/marmelab/react-admin).

based on the code [aor-feathers-client](https://github.com/josx/aor-feathers-client)

## Features
* GET_MANY_REFERENCE
* GET_MANY
* GET_LIST
* GET_ONE
* CREATE
* UPDATE
* DELETE
* AUTH_LOGIN
* AUTH_LOGOUT
* AUTH_CHECK
* AUTH_ERROR
* AUTH_GET_PERMISSIONS
* Custom Id support

## Installation

In your react-admin app just add ra-feathers-client dependency:

```sh
npm install ra-feathers-client --save
```

or

```sh
yarn add ra-feathers-client
```

## Running tests

```sh
npm run test

```

## Example with Feathersjs stable (AUK)

Users service must have a roles field. We are using users.roles

```js
// in feathers-app/src/authentication.js

[...]

app.service('authentication').hooks({
    before: {
      create: [
        authentication.hooks.authenticate(config.strategies),
        // This hook adds the `roles` attribute to the JWT payload by
        // modifying params.payload.
        hook => {
          // make sure params.payload exists
          hook.params.payload = hook.params.payload || {};
          // merge in a `roles` property
          Object.assign(hook.params.payload, {roles: hook.params.user.roles});
        }
      ],
      remove: [
        authentication.hooks.authenticate('jwt')
      ]
    }
});

```

If your role field has another name than "roles" you must change hook.params.user.roles by hook.params.user.[yourRolesField] in Object.assign(hook.params.payload, {roles: hook.params.user.roles})

EACH feathers service MUST use a before hook as restrictToRoles. 

For example, 

```js
//in feathers-app/src/users.hooks.js

  before: {
    all: [],
    find: [ authenticate('jwt'), restrictToRoles({ roles: ['admin']}) ],
    get: [ ...restrict ],
    create: [ authenticate('jwt'), restrictToRoles({ roles: ['admin']}), hashPassword() ],
    update: [ ...restrict, hashPassword() ],
    patch: [ ...restrict, hashPassword() ],
    remove: [ authenticate('jwt'), restrictToRoles({ roles: ['admin']}) ]
  },
  
```


```js
// in src/feathersClient.js
import feathers from 'feathers-client';

const host = 'http://localhost:3030';

export default feathers()
    .configure(feathers.hooks())
    .configure(feathers.rest(host).fetch(window.fetch.bind(window)))
    .configure(feathers.authentication({ jwtStrategy: 'jwt', storage: window.localStorage }));
```

```js
// in src/App.js
import React from 'react';
import { Admin, Resource } from 'react-admin';
import { authClient, restClient } from 'ra-feathers-client';
import feathersClient from './feathersClient';
import { PostList } from './posts';

const authClientOptions = {
  storageKey: 'feathers-jwt',
  authenticate: { strategy: 'local' },
};

// to rename id field for *all* resources use this syntax:
const options = { id: '_id' };

// to rename id field(s) for specific resources use this syntax:
const options = {'my-resource': {id: '_id'}};

// Use HTTP PATCH method instead of PUT to implement UPDATE
const options = { usePatch: true };

const App = () => (
  <Admin
    authClient={authClient(feathersClient, authClientOptions)}
    restClient={restClient(feathersClient, options)}
  >
    {permissions => [
      permissions === 'admin' ? <Resource name="users" list={UsersList} /> : null,
      <Resource
        name="post"
        list={PostList}
        create={permissions === 'admin' ? PostCreate : null}
        edit={permissions === 'admin' ? PostEdit : null}
        remove={permissions === 'admin' ? Delete : null}
      />
    ]}
  </Admin>
);


export default App;
```

## License

This software is licensed under the [MIT Licence](LICENSE), and sponsored by [Cambá](https://www.camba.coop).
