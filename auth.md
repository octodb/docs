Auth  (user and node authorization)
====

You can use OctoDB without node authorization for testing purposes. When using it on production it is recommended to enable some kind of user or node authorization, for security reasons.

There are 3 modes of operation:

- No authorization (for testing)
- Node level authorization
- User level authorization

Any of the above can also be used with encryption on communication.

The encryption is important for privacy reasons, to protect the users data from being read while they are transferred between their devices to the backend.


No Authorization
----------------

In this mode any external node can connect and be part of your network.

To use this mode just open the db connection without an `auth` parameter:

```
file:test.db?node=primary&bind=tcp://0.0.0.0:1234
```

It is possible to enable a simple protection with just encryption, where only nodes that know the network cipher and encryption key will be able to connect to your network. Example:

```
file:test.db?node=primary&bind=tcp://0.0.0.0:1234&cipher=xrc4&key=your_key_here
```


Node Level Authorization
------------------------

This mode is useful if you are working with machines or robots instead of humans (just one node per user)

In this mode each node connecting to your network will send a request to authorize itself. Your network is responsible to verify the request and accept or reject it.

Once a node is authorized on the network, there is no need to repeat the process. The nodes are automatically authenticated on re-connection.

To enable this authorization mode we must inform `auth=node` in the URI of all the primary nodes. Example:

```
file:test.db?node=primary&bind=tcp://0.0.0.0:1234&auth=node
```


User Level Authorization
------------------------

This mode is useful for humans, who can have many devices (nodes) connected to your network. OctoDB will synchronize the data on the user's devices.

In this mode each user connecting to your network will send a "sign up" request on the first time they connect. Your code is responsible to manage this requests and authorize or reject them.

Once a user is authorized, the next time it will access the network with another device it will send a "login" request.

The "sign up" authorization will record the new user and also its device. Each device has a private key that is used for authentication on further connections, so no new signup or login is required on this device.

The "login" authorization will record the new device from that user. Similar to the above, the "login" is done just once.

To enable this authorization mode we must inform `auth=user` in the URI of all the primary nodes. Example:

```
file:test.db?node=primary&bind=tcp://0.0.0.0:1234&auth=user
```

--------------

## Implementing the Authorization Logic

[On Secondary Nodes](auth-secondary-nodes.md) (apps / devices)

[On Primary Nodes](auth-backend.md) (backend)
