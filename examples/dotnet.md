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


### Example Code

```csharp
using SQLite;

// open the database
var uri = "file:test.db?node=primary&bind=tcp://0.0.0.0:1234";
var db = new SQLiteConnection(uri);

// check if the db is ready
while (true) {
    var status = db.ExecuteScalar<string>("pragma sync_status");
    if (status.Contains("\"db_is_ready\": true")) break;
    System.Threading.Thread.Sleep(250);
}

db.OnSync(() => {
    // notification received on the worker thread - you can transfer it to the main thread here
    Console.WriteLine("update received");
});

// now we can use the database
...
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
    dotnet add package SQLitePCLRaw.core
    dotnet add package SQLitePCLRaw.provider.dynamic


### Example Code

Make sure to load the native library before using `Microsoft.Data.Sqlite`

Also, avoid using a `SQLitePCLRaw` bundle package which might override the dynamic provider

```csharp
using Microsoft.Data.Sqlite;
using SQLitePCL;
using System;

namespace OctoDBExample
{
    // used to load the native library dynamically
    class NativeLibraryAdapter : IGetFunctionPointer
    {
        readonly IntPtr _library;
    
        public NativeLibraryAdapter(string name)
            => _library = NativeLibrary.Load(name);
    
        public IntPtr GetFunctionPointer(string name)
            => NativeLibrary.TryGetExport(_library, name, out var address)
                ? address
                : IntPtr.Zero;
    }

    class Program
    {
        static void Main()
        {
            // load the OctoDB native library
            SQLite3Provider_dynamic_cdecl.Setup("octodb", new NativeLibraryAdapter("octodb"));
            SQLitePCL.raw.SetProvider(new SQLite3Provider_dynamic_cdecl());

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

            db.CreateFunction("update_notification", () => {
                // notification received on the worker thread - you can transfer it to the main thread here
                Console.WriteLine("update received");
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
