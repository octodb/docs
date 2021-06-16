Dedicated Authorization Service
===============================

Storing Authorizations on External Database + OctoDB
----------------------------------------------------

### User Signup

If you want to save information about this user on the OctoDB database

    {pubkey:"...","user_id":123,"execute":["INSERT INTO xxx (user_id,...) VALUES yyy"]}

This can be used when the service gets the `user_id` from an external database and you need to have both databases using the same `user_id`

The engine will execute the SQL command(s) and will use the informed `user_id` for authorization


### User Login

On user login (existing user with new device) you can ..  the database.

When retrieving the `user_id` from an external database you can still update some tables on OctoDB:

    {pubkey:"...","user_id":123,"execute":["UPDATE xxx SET key=val WHERE zz=yy"]}


### New Node

If you stored data about this node on an external database it is possible to use the same `node_id` in OctoDB, by passing the value in the authorization message.

It is also possible to save data about this new node in OctoDB:

    {pubkey:"...",node_type:"...","node_id":...,"execute":["INSERT INTO xxx (node_id,...) VALUES yyy"]}

The command will fail if value returned from `last_insert_rowid()` differs from the supplied `node_id`
