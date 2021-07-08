OctoDB on Android Native
========================

OctoDB can be used on Android via the [SQLite Android Bindings](https://sqlite.org/android/doc/trunk/www/index.wiki)

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



### SAMPLE PROJECT

[OctoDB-AndroidStudio-SampleProject.tar.gz](http://octodb.io/download/OctoDB-AndroidStudio-SampleProject.tar.gz)


### INSTRUCTIONS

1.  Install OctoDB on a computer

2.  Start a SQLite shell as the primary node binding to a TCP address

3.  Modify the TCP address to connect to on the project code

4.  Build and run the project on the simulator or device


To open the database on the primary node we use a command like this:

    sqlite3 "file:test.db?node=primary&bind=tcp://0.0.0.0:1234"


### LIMITATIONS

This sample project includes the free version of OctoDB. It can be upgraded to the full version by replacing the `aar` file
