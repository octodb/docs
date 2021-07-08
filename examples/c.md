OctoDB on C
===========

We can use OctoDB on C using the same SQLite API. There are just 2 differences:

1. Open the database using a URI with parameters
2. Wait until the database is ready


### Example Code

```c
#include <sqlite3.h>

char *uri = "file:app.db?node=secondary&connect=tcp://server:port";

int main() {
  sqlite3 *db;

  /* open the database */
  sqlite3_open(uri, &db);

  /* check if the db is ready */
  while(1){
    char *json_str = sqlite3_query_value_str(db, "PRAGMA sync_status");
    BOOL db_is_ready = strstr(json_str, "\"db_is_ready\": true") > 0;
    sqlite3_free(json_str);
    if (db_is_ready) break;
    sleep_ms(250);
  }

  /* now the application can access the database using the SQLite API */

}
```

### Compiling

Compile the application linking it to the OctoDB library:

#### Linux and Mac

    gcc app.c -o app -loctodb

#### Windows

    gcc app.c -o app -L/path/to/octodb -loctodb
