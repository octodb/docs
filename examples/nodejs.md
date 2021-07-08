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

const timer = setInterval(function(){
  var res = db.prepare('PRAGMA sync_status').get();
  var status = JSON.parse(res.sync_status);
  if (status.db_is_ready) {
    clearInterval(timer);
    onDbReady();
  }
}, 1000);

function onDbReady() {

  const row = db.prepare('SELECT * FROM users WHERE id = ?').get(userId);
  console.log(row.firstName, row.lastName, row.email);

  setTimeout(function(){
    console.log('closing...');
    db.close();
  }, 2000);

}
```

-----

### TROUBLESHOOTING

The installation can fail if there is no pre-built wrapper binary for your platform.
In this case your computer needs to have the proper build tools.


### On Windows

Run this command:

    npm install --global --production windows-build-tools

Then try to install the wrapper again
