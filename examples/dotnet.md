OctoDB on .NET
==============

There are many options to use OctoDB on .NET

On this page there are instructions for Desktop (Windows, Mac and Linux). For mobile please check [Xamarin](xamarin.md)


## SQLite.NET

The [SQLite.NET for OctoDB](https://github.com/octodb/sqlite-net) is a fork of [SQLite.NET](https://github.com/praeclarum/sqlite-net)
that uses the OctoDB library


### Instructions

First follow the instructions to compile and install OctoDB from the source code or use the
pre-compiled binaries for your platform. You can start with the free version

You can put the `.dll`, `.dylib` or `.so` file along side your application binaries or in a system-wide library path

Here is how to bundle the library on the project:

1. On the Solution Explorer click on "Add Existing" and then select the OctoDB library (octodb-0.1.dll or liboctodb.so/.dylib)

2. On properties of the library set "Build Action" to "Content" and set "Copy to Output Directory" to "Copy Always"

Add the [SQLite.cs](https://github.com/octodb/sqlite-net/blob/master/src/SQLite.cs) file to your project


### Tables

The library contains simple attributes that you can use to control the construction of tables

```csharp
public class TodoItem
{
    [PrimaryKey, AutoIncrement]
    public long ID { get; set; }
    public string Name { get; set; }
    public bool Done { get; set; }
}
```

On tables with integer primary keys the primary key column must be declared as `long` instead of `int` because OctoDB always use a 64-bit number for row ids


### Example Code

```csharp
using SQLite;

// open the database
var uri = "file:test.db?node=primary&bind=tcp://0.0.0.0:1234";
var db = new SQLiteConnection(uri);

// wait until the db is ready
while (!db.IsReady()) {
    System.Threading.Thread.Sleep(250);
}

// now we can use the database
db.CreateTable<TodoItem>();
...
```

Instead of waiting until the database is ready for access we can use an event notification:

```csharp
if (db.IsReady()) {
    // the database is ready to be accessed
    StartDatabaseAccess(db);
} else {
    // wait until the db is ready (and do not access the database in the meanwhile)
    db.OnReady(() => {
        // the database is ready to be accessed
        StartDatabaseAccess(db);
    });
}

void StartDatabaseAccess(SQLiteConnection db) {
    db.CreateTable<TodoItem>();
    ...
}
```

Your application can also receive a notification when the database receives a sync/update:

```csharp
// subscribe to db sync notifications
db.OnSync(() => {
    // update the screen with new data
    ...
});
```

To retrieve the synchronization status:

```csharp
var status = db.ExecuteScalar<string>("pragma sync_status");
```



## Microsoft.Data.SQLite

The `Microsoft.Data.SQLite` uses the `SQLitePCLRaw` to interface with the native library


### Native Library

First follow the instructions to compile and install OctoDB from the source code or use the
pre-compiled binaries for your platform. You can start with the free version.

You can put the `.dll`, `.dylib` or `.so` file along side your application binaries or in a system-wide library path

Here is how to bundle the library on the project:

1. On the Solution Explorer click on "Add Existing" and then select the OctoDB library (octodb.dll or liboctodb.so/.dylib)

2. On properties of the library set "Build Action" to "Content" and set "Copy to Output Directory" to "Copy Always"


### Nuget Packages

Install with these commands:

    dotnet add package Microsoft.Data.Sqlite
    dotnet add package Microsoft.Data.Sqlite.Core
    dotnet add package SQLitePCLRaw.core  --version 2.0.4
    dotnet add package SQLitePCLRawProvider.OctoDB  --version 2.0.4


### Example Code

Make sure to load the native library before using `Microsoft.Data.Sqlite`

```csharp
using Microsoft.Data.Sqlite;
using SQLitePCL;

namespace OctoDBExample
{
    class Program
    {
        static void Main()
        {
            // load the OctoDB native library
            SQLitePCL.raw.SetProvider(new SQLite3Provider_OctoDB());

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


## Mono SQLite

The Mono SQLite can also be used.

First follow the instructions to compile and install OctoDB from the source code or use the
pre-compiled binaries for your platform. You can start with the free version.

For the interface to work with the OctoDB library we must define `SQLITE_STANDARD` and modify
[this line](https://github.com/mono/mono/blob/master/mcs/class/Mono.Data.Sqlite/Mono.Data.Sqlite_2.0/UnsafeNativeMethods.cs#L57)
from "sqlite3" to "octodb"


## System.Data.SQLite

This wrapper is not supported. It is hard to maintain because it builds its own native library.
