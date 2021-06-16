On Secondary Nodes (apps / devices)
===================================


Node Level Authorization
------------------------

Your app does not need to send data to the backend. The engine will collect device information and send them to
the authorization logic on the primary nodes

The app only needs to verify the authorization as described below


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


Checking the Result
-------------------

On both cases above the app should wait for the authorization and it cannot access the database while it is not ready

This is done by running the `PRAGMA sync_status` command and checking if the value of `db_is_ready` is `true`

You can check examples on how to do this for each programming language in another section


Example
-------

This example is not close to a real application with GUI and events, but it can be used to have an idea of
the sequence of operations or for testing purposes

```python
import octodb as sqlite3
import time
import json

# open the database
db = sqlite3.connect('file:app.db?node=secondary&connect=tcp://server1:port1,tcp://server2:port2,tcp://server3:port3')

# check if the user and device are already authorized
if user_not_registered:
    db.execute("PRAGMA user_signup='" + user_data + "'")
elif device_not_registered:
    db.execute("PRAGMA user_login='" + user_data + "'")

# check if the db is ready
while True:
    result = db.cursor().execute("PRAGMA sync_status").fetchone()
    status = json.loads(result[0])
    if status["db_is_ready"]: break
    time.sleep(0.250)

# now the app can use the db connection
...
```
