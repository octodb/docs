OctoDB on Qt
============

We can use OctoDB with the `QSQLITE` driver


### INSTALLATION

First follow the instructions to compile and install OctoDB from the source code or use the
pre-compiled binaries for your platform. You can start with the free version


### Linux

Modify the driver to use the OctoDB library:

    sudo apt install patchelf
    cd /usr/lib/x86_64-linux-gnu/qt5/plugins/sqldrivers
    sudo patchelf --replace-needed libsqlite3.so.0 liboctodb.so libqsqlite.so

Instead of `x86_64-linux-gnu` it can be `i386-linux-gnu` or `arm-linux-gnueabihf`, it depends on the architecture


To check if it was successful:

    ldd libqsqlite.so

It should appear `liboctodb.so` on the list


### Windows

Generally the `QSQLITE` driver on Windows already includes the SQLite engine embedded.
This means that we need to recompile it to use an external SQLite, on this case the one containing OctoDB.

Here are the steps:

1.  Install the Qt sources using the Qt Maintainance Tool

2.  Download the OctoDB binaries to a folder like C:\octodb

3.  Create a "include" folder and copy the 2 .h files there

4.  Create a "lib" folder and copy the LIB and DLL there

5.  Enter the lib folder and rename octodb.lib to sqlite3.lib (only this file)

6.  Click on the Windows Start button > Qt > Qt Terminal

7.  Run:

        cd %QTDIR%\qtbase\src\plugins\sqldrivers
        qmake -- -system-sqlite SQLITE_PREFIX=C:/OCTODB

When this step finishes, it will show 2 more commands to run to build and install the libraries

It is also necessary to copy the `octodb.dll` to the `Windows\System` folder OR to the same folder that the `qsqlite.dll` is installed



### EXAMPLE CODE

```cpp
QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
db.setDatabaseName("file:data.db?node=secondary&connect=tcp://server:port");
if (!db.open()) {
  qDebug() << "Can't Connect to DB!";
}

QSqlQuery query;
QString result;
while (true){
  query.exec("PRAGMA sync_status");
  query.first();
  result = query.value(0).toString();
  if (result.contains("\"db_is_ready\": true")) break;
  delay(250);
}
```

A complete working example is available [here](https://gitlab.com/litesync/QtExample)
