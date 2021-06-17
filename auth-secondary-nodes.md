On Secondary Nodes (apps / devices)
===================================


Node Level Authorization
------------------------

Your app does not need to send data to the backend. The engine will collect device information and send them to
the authorization logic on the primary nodes

The app only needs to verify the authorization checking for the `db_is_ready` as shown in this example in Python:

```python
import octodb as sqlite3
import time
import json

# open the database
db = sqlite3.connect('file:app.db?node=secondary&connect=tcp://server1:port1,tcp://server2:port2,tcp://server3:port3')

# check if the db is ready
while True:
    result = db.cursor().execute("PRAGMA sync_status").fetchone()
    status = json.loads(result[0])
    if status["db_is_ready"]: break
    time.sleep(0.250)

# now the app can use the db connection
...
```


User Level Authorization
------------------------

If the user is not authorized on your network then the app should issue this command:

    PRAGMA user_signup="...signup data..."

If the user is already authorized but the device is not authorized, then the app should issue this command:

    PRAGMA user_login="...login data..."

Your application can send any data, but serialized in string format inside the quotes. If the string contains a quote, it must double it.

It can also use single quotes (better for JSON content)

    PRAGMA user_login='{key: "value", key2: "value2"}'

Your app can give both options (signup and login) to the user and let it choose.

You can also support authorization via third party OAuth providers like Google, Amazon, Alibaba, Wechat, Yandex...
In this case the app should send the signed token to the backend


#### Example

This is a partial example of a GUI application in Python:

```python
import octodb as sqlite3
import time
import json

# open the database
db = sqlite3.connect('file:app.db?node=secondary&connect=tcp://server1:port1,tcp://server2:port2,tcp://server3:port3')

# check if the user and device are already authorized
result = db.cursor().execute("PRAGMA sync_status").fetchone()
status = json.loads(result[0])
if status["node_id"] == 0:
    # display a screen for user signup or login
    show_user_auth_screen()
else:
    # wait until the db is ready for access
    start_db_ready_timer()

# called by the user auth screen
def do_user_auth(auth_type, user_data)
    if auth_type == USER_SIGNUP:
        db.execute("PRAGMA user_signup='" + user_data + "'")
    elif auth_type == USER_LOGIN:
        db.execute("PRAGMA user_login='" + user_data + "'")
    # wait until the db is ready for access
    start_db_ready_timer()

# called periodically by the timer
def on_check_db_ready():
    # check if the db is ready
    result = db.cursor().execute("PRAGMA sync_status").fetchone()
    status = json.loads(result[0])
    if status["db_is_ready"]:
        # destroy the timer
        ...
        # now the app can use the db connection
        on_db_is_ready()

# the database is ready for access
def on_db_is_ready():
    ...
```

Here is a simplified example without any GUI:


```python
import octodb as sqlite3
import time
import json

sent_user_registration = False
user_data = "..."

# open the database
db = sqlite3.connect('file:app.db?node=secondary&connect=tcp://server1:port1,tcp://server2:port2,tcp://server3:port3')

# check if the db is ready
while True:
    result = db.cursor().execute("PRAGMA sync_status").fetchone()
    status = json.loads(result[0])
    if status["db_is_ready"]: break

    # check if the user and device are already authorized
    if not sent_user_registration:
        db.execute("PRAGMA user_login='" + user_data + "'")
        sent_user_registration = True

    time.sleep(0.250)

# now the app can use the db connection
...
```

Checking the Result
-------------------

On all cases the app should wait for the authorization and it cannot access the database while it is not ready

This is done by running the `PRAGMA sync_status` command and checking if the value of `db_is_ready` is `true`

You can check examples on how to do this for each programming language in another section
