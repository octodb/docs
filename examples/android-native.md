OctoDB on Android (Native)
==========================

OctoDB can be used natively on Android via the [SQLite Android Bindings](https://sqlite.org/android/doc/trunk/www/index.wiki)
(for Java and Kotlin)

Is has the same interface as the `android.database.sqlite` namespace with some minor changes:

`1`  We must load the native SQLite library before using it using this code:

    System.loadLibrary("octodb");

`2`  Replace all occurrences of `android.database.sqlite` with `org.sqlite.database.sqlite`. Example:

    import org.sqlite.database.sqlite.SQLiteDatabase;

`3`  Replace these two references:

    android.database.SQLException
    android.database.DatabaseErrorHandler

By these:

    org.sqlite.database.SQLException
    org.sqlite.database.DatabaseErrorHandler


## How to Use

Here is an example code:

```java
// open the database
File dbfile = new File(getFilesDir(), "app.db");
String uriStr = "file:" + dbfile.getAbsolutePath() + "?node=secondary&connect=tcp://server:port";
SQLiteDatabase db = SQLiteDatabase.openOrCreateDatabase(uriStr, null);

if (db.isReady()) {
    // the user is already logged in. show the main screen
    ...
} else {
    // the user is not logged in. show the login screen (and do not access the database)
    ...
    // wait until the login is done
    db.onReady(() -> {
        // the login was successful. show the main screen
        ...
    });
}

db.onSync(() -> {
    // the db received an update. update the screen with new data
    ...
});
```


## Opening a Database

```java
File dbfile = new File(getFilesDir(), "app.db");
String uriStr = "file:" + dbfile.getAbsolutePath() + "?node=secondary&connect=tcp://server:port";
SQLiteDatabase db = SQLiteDatabase.openOrCreateDatabase(uriStr, null);
```


## Database Status

:warning:  The application should **NOT** access the database before it is ready for read and write!

The app can check if the database is ready using the `isReady()` function.

The database is not ready when it is the first time the app is being open and/or the user has not
signed up or logged in yet.

If the user has already signed up or logged in, then the database will be ready for access.

```java
if (db.isReady()) {
    // the user is already logged in. show the main screen
    ...
} else {
    // the user is not logged in. show the login screen (and do not access the database)
    ...
}
```

When the login is successful, the database will fire the `onReady` event:

```java
db.onReady(() -> {
  // the user is already logged in or the login was successful. show the main screen
  ...
});
```

It is also possible to subscribe to `sync` events, so the app can read fresh data from the
database and update the screen:

```java
db.onSync(() -> {
  // the db received an update. update the screen with new data
  ...
});
```

To check the full status of the database we can use:

```java
SQLiteStatement statement = db.compileStatement("PRAGMA sync_status");
String result = statement.simpleQueryForString();
statement.close();
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

```java
String email = edtEmail.getText().toString().trim();
String password = edtPassword.getText().toString().trim();
JSONObject json = new JSONObject();
json.put("email", email);
json.put("passwd", password);
String user_info = json.toString();
db.execSQL("pragma user_signup='" + user_info + "'");
```

Notice that you must implement the [backend service](https://github.com/octodb/docs/blob/master/auth-service.md)
that handles these authorization requests.


## Multi-User App

To support multiple users in a single app installation your app can have a database for each user.

Your app will need to keep track of which database is used for each user.
An easy way is to convert the username or e-mail into hex format and then use it as the database name:

```java
String dbname = hex(email) + ".db";
String uri = "file:" + dbname + "?node=secondary&connect=tcp://111.222.33.44:1234";
```

### Sign Out

Just close the currently open database and display the signup & login screen

### Login

When the user enters its data (usually e-mail and password):

1. Get the database name based on the e-mail and open it
2. Wait for the db event. If not ready, then send signup/login info via the `pragma` command. If ready, then check if the password is correct


## Sample Project

[OctoDB-AndroidStudio-SampleProject.tar.gz](http://octodb.io/download/OctoDB-AndroidStudio-SampleProject.tar.gz)


### Instructions

1.  Install OctoDB on a computer
2.  Start a SQLite shell as the primary node binding to a TCP address
3.  Modify the TCP address to connect to on the project code
4.  Build and run the project on the simulator or device

To open the database on the primary node we use a command like this:

    sqlite3 "file:test.db?node=primary&bind=tcp://0.0.0.0:1234"


### Limitations

This sample project includes the free version of OctoDB. It can be upgraded to the full version by replacing the `octodb.aar` file
