Synchronous Mode
================

On this case the authorizations are managed on the primary nodes

Your application registers a callback that is called when a request comes

> Important: this call is made using a **worker thread!**

The callback function must return the response (accept or reject) as a result of the function call

It can use the database connection to read or write to the database


### User Authorization

Here is how the callbacks for **user authorization** look like in Python: (you can use another language)

```python
def on_user_signup(signup_info, node_info):
  ...
  return result

def on_user_login(login_info, node_info):
  ...
  return result
```

The `signup_info` and `login_info` arguments contain the data that was sent by the secondary node
using the PRAGMA commands `user_signup` and `user_login`, respectively. See [here](auth-secondary-nodes.md) for more details

The `node_info` argument contains information about the device

Your application should verify these information and return a value according to this table:

 Return   | Meaning
--------- | --------------------------------------------------------------
<user_id> | Use this specific user id for this user (new or existing user)
0         | Let the system choose a new user id (new user)
-1        | Not authorized
-3        | General error

Example:

```python
  if error:
    return -3  # general error
  if not valid:
    return -1  # not authorized
  if existing_user:
    return user_id
  else:
    return 0   # choose any user id
```


### Node Authorization

The callback for **node authorization** has this format:

```python
def on_new_node(node_info):
  ...
  return "primary" | "secondary" | "async" | "unauthorized"
```

The `node_info` argument contains information about the device, in the same format as the functions above.

Your application should verify the node information and return a value according to this table:

 Return     | Meaning
----------- | ------------------------------------------------------------
"primary"   | Authorize it as a primary node
"secondary" | Authorize it as a secondary node
-1          | Not authorized
-3          | General error


### Example

To register the callback functions on the db connection:

```python
  db.register_function("user_signup", 2, on_user_signup)
  db.register_function("user_login", 2, on_user_login)
  db.register_function("new_node", 1, on_new_node)
```

You can check an example [here](...)


> Note: This method is relatively slow and it does not scale well, so if you plan to support
> many requests per second, please consider the asynchronous method or a dedicated service
