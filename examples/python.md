OctoDB on Python
================

First follow the instructions to compile and install OctoDB from the source code or
use the pre-compiled binaries for your platform.
You can start with the free version.

Then install the `octodb` module according to the Python version:


### Python 3

    pip install octodb

### Python 2.7

    git clone --depth=1 https://gitlab.com/octodb/python2 octodb-python2
    cd octodb-python2
    python setup.py build
    sudo python setup.py install


### Example Code

The `octodb` module has the same API as the `sqlite3` module

```python
import octodb as sqlite3
import json
import time

conn = sqlite3.connect('file:app.db?node=secondary&connect=tcp://server:port')

# check if the db is ready
while True:
    result = conn.cursor().execute("PRAGMA sync_status").fetchone()
    status = json.loads(result[0])
    if status["db_is_ready"]: break
    time.sleep(0.250)

# now we can use the db connection
...
```

-----

### THROUBLESHOOTING

We can also use the Python's `sqlite3` module on Linux and Windows

On both cases the Python's `sqlite3` module interfaces with the system's SQLite library


### Linux

We have 2 options to make Python to use the modified SQLite library containing OctoDB:


A. Use the LD_LIBRARY_PATH when opening the application

    LD_LIBRARY_PATH=/usr/local/lib/octodb python app.py

B. Make the sqlite3 module use the LiteSync library

    sudo apt install patchelf
    patchelf --replace-needed libsqlite3.so.0 liboctodb.so /usr/lib/python2.7/lib-dynload/_sqlite3.so

To check if it was successful:

    ldd /usr/lib/python2.7/lib-dynload/_sqlite3.so


### Windows

Replace the sqlite3.dll file in the folder \Python\DLLs with the one containing OctoDB (rename octodb.dll to sqlite3.dll)


### Usage

In this case we use the `sqlite3` module instead of the OctoDB module

```python
import sqlite3
import json
import time

conn = sqlite3.connect('file:app.db?node=secondary&connect=tcp://server:port')

# check if the db is ready
while True:
    result = conn.cursor().execute("PRAGMA sync_status").fetchone()
    status = json.loads(result[0])
    if status["db_is_ready"]: break
    time.sleep(0.250)

# now we can use the db connection
...
```
