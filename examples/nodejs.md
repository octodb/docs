OctoDB on Node.js
=================

The Node.js interface is based on [better-sqlite3](https://github.com/JoshuaWise/better-sqlite3)


### Installation

First follow the instructions to compile and install OctoDB from the source code or use the
pre-compiled binaries for your platform. You can start with the free version.

Then you can install the wrapper with this command:

    npm install better-sqlite3-octodb


### Example Code

```node.js
const options = { verbose: console.log };
const uri = 'file:test.db?node=secondary&connect=tcp://127.0.0.1:1234';
const db = require('better-sqlite3-octodb')(uri, options);

db.on('ready', function() {

  const row = db.prepare('SELECT * FROM users').all();
  console.log(row.name, row.email);

});

db.on('sync', function() {

  console.log('the database received updates');

});
```

Here is an example for an app with user login:

```js
// check if the db is ready
if (db.is_ready()) {
  // show the main screen
  ...
} else {
  // show the signup/login screen
  ...
  db.on('ready', () => {
    // login successful, show the main screen
    ...
  });
}
```

You can also check the status with:

```js
let res = db.prepare('PRAGMA sync_status').get();
let status = JSON.parse(res.sync_status);
```

-----

### TROUBLESHOOTING

The installation can fail if there is no pre-built wrapper binary for your platform.
In this case your computer needs to have the proper build tools.


### On Windows

Run this command:

    npm install --global --production windows-build-tools

Then try to install the wrapper again
