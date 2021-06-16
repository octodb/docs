Asynchronous Mode
=================

On this case the authorizations are managed on the primary nodes

Your application registers a callback that is called when a request comes

> Important: this call is made using a **worker thread!**

The callback function returns fast and the approval or rejection of the request is done later, in the main thread

The callback function can either:

- Transfer the request to the main thread and process it there

- Send a request to another service (external database) and receive the answer on the main thread

The callback function should return a value according to this table:

 Return   | Meaning
--------- | -------------------------------------------
-1        | Not authorized / Invalid request
-2        | Processing the request in asynchronous mode
-3        | General error


The main thread can authorize users and nodes using the commands below

For user signup:

    PRAGMA authorize_user="<pubkey>:<user_id>"

For user login (new device from existing user):

    PRAGMA authorize_user="<pubkey>:<user_id>"

For new node (when using node authorization) and for primary nodes:

    PRAGMA authorize_node="<pubkey>:primary"
    PRAGMA authorize_node="<pubkey>:secondary"


### Example

Here is an example in Python - the callbacks add the requests to a queue that is processed by the main thread:

```python
import octodb as sqlite3
import time
import json
from multiprocessing.queues import SimpleQueue

db = None
auth_queue = SimpleQueue()

# called by the worker thread:

def on_user_signup_async(signup_info, node_info):
    global auth_queue
    print "on_user_signup_async signup_info:", signup_info, "node_info:", node_info
    req = ("signup", signup_info, node_info,)
    auth_queue.put(req)
    return -2  # process in async mode

def on_user_login_async(login_info, node_info):
    global auth_queue
    print "on_user_login_async login_info:", login_info, "node_info:", node_info
    req = ("login", login_info, node_info,)
    auth_queue.put(req)
    return -2  # process in async mode

def on_new_node_async(node_info):
    global auth_queue, qcount
    print "on_new_node_async", node_info
    req = ("new_node", node_info, )
    auth_queue.put(req)
    return -2  # process in async mode

# called periodically by the main thread:

def process_auth_requests(db):
    global auth_queue
    while not auth_queue.empty():
        req = auth_queue.get()
        action = req[0]
        if action == "signup":
            signup_info = req[1]
            node = json.loads(req[2])
            user_id = check_user_signup(signup_info, node)
            if user_id >= 0:
                db.execute("pragma authorize_user='" + node["public_key"] + ":" + str(user_id) + "'")

        elif action == "login":
            login_info = req[1]
            node = json.loads(req[2])
            user_id = check_user_login(login_info, node)
            if user_id >= 0:
                db.execute("pragma authorize_user='" + node["public_key"] + ":" + str(user_id) + "'")

        elif action == "new_node":
            node = json.loads(req[1])
            node_type = check_new_node(node)
            if node_type:
                db.execute("pragma authorize_node='" + node["public_key"] + ":" + node_type + "'")


# open the database
db = sqlite3.connect('file:app.db?node=primary&bind=tcp://0.0.0.0:1234&auth=user')

# check if the db is ready
while True:
    result = db.cursor().execute("PRAGMA sync_status").fetchone()
    status = json.loads(result[0])
    if status["db_is_ready"]: break
    time.sleep(0.250)

# register the callback functions
db.register_function("user_signup", 2, on_user_signup_async)
db.register_function("user_login", 2, on_user_login_async)
db.register_function("new_node", 1, on_new_node_async)

# keep the app open
while True:
    process_auth_requests(db)
    time.sleep(0.5)
```
