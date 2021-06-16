Dedicated Authorization Service
===============================

Storing Authorizations on OctoDB
--------------------------------

In this case the service is used to verify the requests and generate the SQL commands that will be executed by the primary nodes


### User Signup

If the new user is approved, the service should send a message to the leader node containing the public key of the new device and an integer representing the user_id:

    {pubkey:"...",user_id:0}

The `pubkey` can be copied from the request message

The `user_id` should be `0` (zero), meaning that the system will choose a new user id

If you want to save user's data on OctoDB, then instead of the `user_id` field we should inform an SQL command to insert this data:

    {pubkey:"...","execute":["INSERT INTO xxx VALUES yyy"]}

In this case the SQL command is executed and the `user_id` is retrieved from the `last_insert_rowid()`


### User Login

If the existing user's new device is approved, the service should send an authorization message to the leader node containing an SQL command used for searching the registered user:

    {pubkey:"...","find":"SELECT id AS user_id FROM users_table WHERE ..."}

In this case the engine will use the `user_id` from the SQL query

If the user is not found then the device cannot be authorized


### New Node

If the new device is approved, the service should send a message to the leader node containing the public key of the new device and the `node_type`:

    {pubkey:"...",node_type:"..."}

The `pubkey` can be copied from the request message.

The `node_type` should be:

  Value   | Meaning
--------- | ---------------------------------------
primary   | Authorize it as a primary node
secondary | Authorize it as a secondary node


If you want to save additional data about the node:

    {pubkey:"...",node_type:"...","execute":["INSERT INTO xxx VALUES yyy"]}

In this case the SQL command it executed and the `node_id` is retrieved from the `last_insert_rowid()`
