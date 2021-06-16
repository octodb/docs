Dedicated Authorization Service
===============================

In this case we have a service that binds to a TCP port in which all the primary nodes would connect to

Your application running on all the primary nodes should register the service by executing this command:

```
PRAGMA authorizer_service="tcp://ip.address:port"
```

The authorization requests received on the primary nodes are redirected to the auth service

The supported protocol is TCP with length-prefix framing


Implementing the service
------------------------

All the primary nodes will connect to and send requests to the service, but the service MUST send the
responses to a single node, called the leader node

The leader node informs the service that it is the leader with a message exactly as this:

    {"leader":true}


The service will receive requests in this format:

    {action:"user_signup",data:"...",pubkey:"..."}

If the data sent by the secondary node is in JSON format, then the data is embedded in the parent JSON object, like this:

    {action:"user_signup",data:{your_json:"object"},pubkey:"..."}

The `action` can be:

* user_signup
* user_login
* new_node


As the service runs separated from the primary nodes it has 3 options for storing user authorization.
You can check the instructions according to the storage type:

[External database](auth-service-external-db.md)

[OctoDB](auth-service-octodb.md)

[Both external database and OctoDB](auth-service-external-octodb.md)
