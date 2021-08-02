Managing Authorizations on the Backend
======================================

When a secondary node connects to your backend (primary nodes) for the first time it will send an authorization request

Your backend is responsible for analyzing the request and either approve or reject it

It is also possible to save user or device information for further checking


There are many methods to manage the user or node authorization on your network:

- [Using a backend service](auth-service.md) (preferred)
- [Directly on primary nodes - Synchronous mode](auth-sync.md)
- [Directly on primary nodes - Asynchronous mode](auth-async.md)


We recommend to use the backend service because if later you need to update the authorization logic
you can just restart the service, not affecting the primary nodes.
While if you implement the authorization logic directly on the primary nodes then they would need to be restarted
when updated, closing all the connections from the devices and between the primary nodes themselves.


## Listing Connected Nodes

To retrieve the connections from a specific primary node we can use this command:

    PRAGMA nodes


## Manual Authorizations

It is also possible to authorize nodes manually by using the commands below.

You need to know the node's public key to authorize a node. You can get it with the command above.


### User Signup

To authorize a device in which the user is new, we use:

    PRAGMA authorize_user="<pubkey>:0"

The `0` means that OctoDB will assign a new user id for this user.


### User Login

To authorize a new device from an existing user, we use:

    PRAGMA authorize_user="<pubkey>:<user_id>"

You must know the `user_id` of this existing user.


### New Node

To authorize primary nodes and also to authorize secondary nodes when using node level authorization (auth=node), we use:

    PRAGMA authorize_node="<pubkey>:<node_type>"

The `node_type` should be either `primary` or `secondary`
