Managing Authorizations on the Backend
======================================

When a secondary node connects to your backend (primary nodes) for the first time it will send an authorization request

Your backend is responsible for analyzing the request and either approve or reject it

It is also possible to save user or device information for further checking


There are many methods to manage the user or node authorization on your network:

- [On primary nodes - Synchronous mode](auth-sync.md)
- [On primary nodes - Asynchronous mode](auth-async.md)
- [Using a dedicated service](auth-service.md)
