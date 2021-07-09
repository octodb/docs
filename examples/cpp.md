OctoDB on C++
=============

We can use most SQLite wrappers for C++ to interface with OctoDB

At the compilation time we must link to the `octodb` library instead of `sqlite3`

Example:

    cpp app.c -o app -loctodb

On the source code we need to make 2 changes:

1. Open the database using a URI with configuration parameters
2. Wait until the database is ready


## sqlite_modern_cpp

Here is an example for [sqlite_modern_cpp](https://github.com/SqliteModernCpp/sqlite_modern_cpp):

```cpp
#include <iostream>
#include <sqlite_modern_cpp.h>
#include <unistd.h>
using namespace sqlite;
using namespace std;

int main() {

   try {
      // open the database
      database db("file:app.db?node=primary&bind=tcp://0.0.0.0:1234");

      // wait until the database is ready
      while(1) {
        string status;
        db << "pragma sync_status" >> status;
        cout << "status : " << status << endl;
        if (status.find("\"db_is_ready\": true") != string::npos) break;
        sleep(1);
      }

      // now the application can access the database
      // check examples here:
      // https://github.com/SqliteModernCpp/sqlite_modern_cpp
      ...

   }
   catch (exception& e) {
      cout << e.what() << endl;
   }
}
```

Compile the code with:

    g++ test.cpp -std=c++14 -loctodb
