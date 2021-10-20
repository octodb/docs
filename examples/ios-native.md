OctoDB on iOS (Native)
======================

It is possible to use OctoDB natively on iOS, using Xcode and the Swift language

We do this by using the [SQLite.swift](https://github.com/stephencelis/SQLite.swift) wrapper


## Installation

... include ... in your project


## Native Libraries

...


## How to Use

Here is an example code:

```swift
// open the database
let fileURL = try! FileManager.default.url(for: .documentDirectory, in: .userDomainMask, appropriateFor: nil, create: false)
    .appendingPathComponent("app.db")
let uri = "file:" + fileURL.path + "?node=secondary&connect=tcp://server:port"
Connection db = try Connection(uri)

if db.isReady() {
    // the user is already logged in. show the main screen
    ...
} else {
    // the user is not logged in. show the login screen (and do not access the database)
    ...
    // wait until the login is done
    db.onReady {
        // the login was successful. show the main screen
        ...
    }
}

db.onSync {
    // the db received an update. update the screen with new data
    ...
}
```


## Opening a Database

```swift
let fileURL = try! FileManager.default.url(for: .documentDirectory, in: .userDomainMask, appropriateFor: nil, create: false)
    .appendingPathComponent("app.db")
let uri = "file:" + fileURL.path + "?node=secondary&connect=tcp://server:port"
Connection db = try Connection(uri)
```


## Database Status

:warning:  The application should **NOT** access the database before it is ready for read and write!

The app can check if the database is ready using the `isReady()` function.

The database is not ready when it is the first time the app is being open and/or the user has not signed up or logged in yet.

If the user has already signed up or logged in, then the database will be ready for access.

```swift
if db.isReady() {
  // the user is already logged in. show the main screen
  ...
} else {
  // the user is not logged in. show the login screen (and do not access the database)
  ...
}
```

When the login is successful, the database will fire the `onReady` event:

```swift
db.onReady {
  // the user is already logged in or the login was successful. show the main screen
  ...
}
```

It is also possible to subscribe to `sync` events, so the app can read fresh data from the
database and update the screen:

```swift
db.onSync {
  // the db received an update. update the screen with new data
  ...
}
```

To check the full status of the database we can use:

```swift
db.executeSql("pragma sync_status", [], (result) => {
  if (result.rows && result.rows.length > 0) {
    var status = result.rows.item(0);
    console.log('sync status:', status.sync_status);
  } else {
    console.log('OctoDB is not active')
  }
}, (msg) => {
  console.log('could not run "pragma sync_status":', msg)
});
```


## User Sign Up & User Login 

When the user is accessing your app for the first time (in any device), it will be required to sign up

The app can capture user data and send it to the backend user authorization service using this command:

```
pragma user_signup='<user_data>'
```

If the user is already signed up on your backend and is now using a new device, it should use the login option:

```
pragma user_login='<user_data>'
```

You can use any user data you want (phone number, username, OAuth...). The data can be encoded in a text (JSON, YAML...)
or a binary format. If using a text format, each single quote must be doubled (replace `'` with `''`).
If using a binary format then it must be encoded in base64 or hex because the command only accepts strings.

Here is an example code using JSON:

```swift
let info = "{\"email\": \"\(self.txtfName.text ?? "")\", \"passwd\": \"\(self.txtfPass.text ?? "")\"}"
try db.execute("pragma user_signup='" + info + "'")
```

Notice that you must implement the [backend service](https://github.com/octodb/docs/blob/master/auth-service.md)
that handles these authorization requests.


## Multi-User App

To support multiple users in a single app installation your app can have a database for each user.

Your app will need to keep track of which database is used for each user.
An easy way is to convert the username or e-mail into hex format and then use it as the database name:

```swift
let dbname = hex(email) + ".db"
let uri = "file:" + dbname + "?node=secondary&connect=tcp://111.222.33.44:1234"
```

### Sign Out

Just close the currently open database and display the signup & login screen

### Login

When the user enters its data (usually e-mail and password):

1. Get the database name based on the e-mail and open it
2. Wait for the db event. If not ready, then send signup/login info via the `pragma` command. If ready, then check if the password is correct


## Example Projects

Here are example Xcode projects using the [SQLite.swift](https://github.com/stephencelis/SQLite.swift) wrapper

They include the free version of OctoDB with some limitations (1 table per database + no encryption). These limitations are not present on the full libraries


### SQLite.swift - SQL

This project has a simple interface using SQL:

```swift
try db.run("INSERT INTO tasks (name,done) VALUES (?,?)", "Learn React Native", 1)
try db.run("INSERT INTO tasks (name,done) VALUES (?,?)", "Use SQLite", 1)
try db.run("INSERT INTO tasks (name,done) VALUES (?,?)", "Test OctoDB", 0)

for row in try db.prepare("SELECT * FROM tasks") {
    let id = row[0] as! Int64
    let name = row[1] as! String
    let done = Int(row[2] as! Int64)
    print("id: \(id) name: \(name) done: \(done)")
}
```

[Download the project](http://octodb.io/download/OctoDB-iOS-Xcode-SQLite.swift-SQL.tar.gz)


### SQLite.swift - Objects

This project has an interface using objects:

```swift
let tasks = Table("tasks")
let id = Expression("id")
let name = Expression("name")
let done = Expression("done")

try db.run(tasks.insert(name <- taskname, done <- isdone!))

for task in try db.prepare(tasks) {
    print("id: \(task[id]) name: \(task[name]) done: \(task[done])")
}
```

[Download the project](http://octodb.io/download/OctoDB-iOS-Xcode-SQLite.swift-Objects.tar.gz)



### Instructions

The above projects implement secondary nodes. To test them you need to run one or more primary nodes (backend)

1.  Install OctoDB on a computer
2.  Start a SQLite shell as the primary node binding to a TCP address
3.  Modify the TCP address to connect to on the project code
4.  Build and run the project on the simulator or device


To open the database on the primary node we use a command like this:

    sqlite3 "file:test.db?node=primary&bind=tcp://0.0.0.0:1234"
