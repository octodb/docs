OctoDB on Xamarin
=================

There are 2 options to use OctoDB on Xamarin:

* SQLite.NET
* Microsoft.Data.SQLite


SQLite.NET
----------

The [SQLite.NET for OctoDB](https://github.com/octodb/sqlite-net) is a fork of [SQLite.NET](https://github.com/praeclarum/sqlite-net)
that uses the OctoDB library


### Instructions

First follow the instructions to compile OctoDB for mobile or use the pre-compiled native libraries

You can start downloading the free version:

* [Android](http://octodb.io/download/octodb-free-android-native-libs.tar.gz)
* [iOS](http://octodb.io/download/octodb-free-ios-native-libs.tar.gz)

Add the native libraries to the project according to the platform

#### Android

On Android you may have a folder structure like this:

    - lib
        - arm64-v8a
        - armeabi-v7a
        - x86
        - x86_64

On each sub-folder you may have 4 libraries:

    liboctodb.so
    libuv.so
    libbinn.so
    libsecp256k1-vrf.so

For each of the `.so` files, CONTROL + CLICK then set Build Action to AndroidNativeLibrary

#### iOS

On iOS we should have 4 fat libraries for static linking:

    liboctodb.a
    libuv.a
    libbinn.a
    libsecp256k1-vrf.a

CONTROL + CLICK on the Xamarin.iOS project then choose Add > Add Native Reference and select the 4 libraries from the folder

CONTROL + CLICK on each library, click Properties and enable these items:

    √ Force Load
    √ Smart Link


### Nuget Packages

Install with these commands:

    dotnet add package SQLitePCLRaw.core  --version 2.0.4
    dotnet add package SQLitePCLRawProvider.static  --version 2.0.4
    dotnet add package SQLitePCLRawProvider.OctoDB  --version 2.0.4


#### Finally

Add the [SQLite.cs](https://github.com/octodb/sqlite-net/blob/master/src/SQLite.cs) file to your project

And define the `USE_SQLITEPCL_RAW` conditional variable in the shared project


### Example code

```csharp
using SQLite;
using SQLitePCL;

if (Device.RuntimePlatform == Device.iOS)
{
    // use the OctoDB static native library
    SQLitePCL.raw.SetProvider(new SQLite3Provider_static());
}
else // if (Device.RuntimePlatform == Device.Android)
{
    // load the OctoDB dynamic native library
    SQLitePCL.raw.SetProvider(new SQLite3Provider_OctoDB());
}

// open the database
var uri = "file:test.db?node=primary&bind=tcp://0.0.0.0:1234";
var db = new SQLiteConnection(uri);

// check if the database is ready
if (db.IsReady()) {
    // the user is already logged in. show the main screen
    ...
} else {
    // the user is not logged in. show the login screen
    ...
    // wait until the login is successful
    db.OnReady(() => {
        // the log in was successful. show the main screen
        ...
    });
}

// subscribe to db sync notifications
db.OnSync(() => {
    // update the screen with new data
    ...
});
```



Microsoft.Data.SQLite
---------------------

The `Microsoft.Data.SQLite` uses the `SQLitePCLRaw` to interface with the native library


### Native Library

First follow the instructions to compile OctoDB for mobile or use the pre-compiled native libraries.
You can start downloading the free version:

* [Android](http://octodb.io/download/octodb-free-android-native-libs.tar.gz)
* [iOS](http://octodb.io/download/octodb-free-ios-native-libs.tar.gz)

Add the native libraries to the project according to the platform

### Android

On Android you may have a folder structure like this:

    - lib
        - arm64-v8a
        - armeabi-v7a
        - x86
        - x86_64

On each sub-folder you may have 4 libraries:

    liboctodb.so
    libuv.so
    libbinn.so
    libsecp256k1-vrf.so

For each of the `.so` files, CONTROL + CLICK then set Build Action to AndroidNativeLibrary

### iOS

On iOS we should have 4 fat libraries for static linking:

    liboctodb.a
    libuv.a
    libbinn.a
    libsecp256k1-vrf.a

CONTROL + CLICK on the Xamarin.iOS project then choose Add > Add Native Reference and select the 4 libraries from the folder

CONTROL + CLICK on each library, click Properties and enable these items:

    √ Force Load
    √ Smart Link


### Nuget Packages

Install with these commands:

    dotnet add package Microsoft.Data.Sqlite
    dotnet add package Microsoft.Data.Sqlite.Core
    dotnet add package SQLitePCLRaw.core  --version 2.0.4
    dotnet add package SQLitePCLRawProvider.static  --version 2.0.4
    dotnet add package SQLitePCLRawProvider.OctoDB  --version 2.0.4


### Example code

Make sure to load the native library before using `Microsoft.Data.Sqlite`

Also, avoid using a `SQLitePCLRaw` bundle package which might override the dynamic provider

```csharp
using Microsoft.Data.Sqlite;
using SQLitePCL;

namespace OctoDBExample
{
    class Program
    {
        static void Main()
        {

            if (Device.RuntimePlatform == Device.iOS)
            {
                // use the OctoDB static native library
                SQLitePCL.raw.SetProvider(new SQLite3Provider_static());
            }
            else // if (Device.RuntimePlatform == Device.Android)
            {
                // load the OctoDB dynamic native library
                SQLitePCL.raw.SetProvider(new SQLite3Provider_OctoDB());
            }

            // open the database
            var uri = "file:test.db?node=primary&bind=tcp://0.0.0.0:1234";
            var db = new SqliteConnection("Data Source=" + uri);
            db.Open();

            // check if the db is ready
            while (true) {
                SqliteCommand cmd = db.CreateCommand();
                cmd.CommandText = "pragma sync_status";
                string status = (string) cmd.ExecuteScalar();
                if (status == null) {
                    Console.WriteLine("the loaded library does not contain OctoDB");
                    return;
                }
                if (status.Contains("\"db_is_ready\": true")) break;
                System.Threading.Thread.Sleep(250);
            }

            // subscribe to db sync notifications
            db.CreateFunction("sync_notification", () => {
                // notification received on the worker thread - you can transfer it to the main thread here
                Console.WriteLine("db sync received");
            });

            // now we can use the database
            ...
        }
    }
}
```
