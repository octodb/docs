Dedicated Authorization Service
===============================

Storing Authorizations on an External Database
----------------------------------------------

### User Signup and User Login

If the new user or its new device is approved, the service should send a message to the leader
node containing the public key of the new device and an integer representing the user_id:

    {pubkey:"...",user_id:123}

The `pubkey` can be copied from the request message

The `user_id` can be:

  Value   | Meaning
--------- | --------------------------------------------------------------
 &gt; 0   | Use this specific user id for this user (new or existing user)
 0        | Let the system choose a new user id (new user)


### New Node

If the new device is approved, the service should send a message to the leader node containing the authorization command.

If you stored data about this node on an external database it is possible to use the
same `node_id` in OctoDB, by passing the value in the authorization message:

    {pubkey:"...",node_type:"...","node_id":...}

The `pubkey` can be copied from the request message

The `node_type` should be either:

  Value   | Meaning
--------- | ---------------------------------------
primary   | Authorize it as a primary node
secondary | Authorize it as a secondary node
