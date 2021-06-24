# DATABASE ENCRYPTION

OctoDB supports encryption of SQLite databases as well as encryption of the communication between the nodes

The available ciphers are ChaCha, which is faster than AES on small devices, and XRC4, an extended version of RC4 even faster than ChaCha.

To open an encrypted database we use the URI parameters cipher and key or hexkey. Example:

    "file:/path/to/file.db?cipher=...&key=..."

Or

    "file:/path/to/file.db?cipher=...&hexkey=..."


## THE CIPHER

The cipher argument can be set to one of these values:

* `xrc4` - fastest
* `chacha8` - fast
* `chacha12` - medium
* `chacha20` - strongest

## THE KEY

Using XRC4 the key can be up to 256 bytes. Example:

    "file:data.db?cipher=xrc4&key=testing"

Using ChaCha the key must be 32 bytes long. Example:

    "file:data.db?cipher=chacha20&key=ThisIsAReallyLongKeyWith32Bytess"

The key can also be in hex format, using the hexkey parameter:

    "file:data.db?cipher=...&hexkey=11223344556677889900AABBCCDDEEFF..."


## CREATING AN ENCRYPTED DATABASE

Open the database with an URI like the above ones and then create the tables and populate them.


## CONVERTING AN EXISTING DATABASE

Currently it is not possible to make a direct conversion of an existing database. We must create an encrypted database using the SQLite shell and then copy the data from the plain database to it.

This can be done in just 1 step:

    sqlite3 plain.db .dump | sqlite3 "file:encrypted.db?cipher=...&key=..."

Or in 2 steps if you want to inspect the generated SQL commands:

    sqlite3 plain.db .dump > out.sql
    sqlite3 "file:encrypted.db?cipher=...&key=..." < out.sql

If you want to use a page size different from the default open the generated out.sql file and add the pragma line at the top:

    PRAGMA page_size=...;

> **Attention:** This encrypted database file should be used only on one primary node. The other primary nodes will download this file from the first node when the connection is established.


## DATABASE HEADER

The first 100 bytes of an SQLite database form the header. Most of it is fixed data.

In encryption if we have a known plain data this helps an attacker to try to discover the encryption key.

For this reason the database header is not encrypted by default, but you have a total of 3 options:


### 1. Do not encrypt the db header

This is the default behavior


### 2. Do not encrypt the db header and use a custom header string

The first 16 bytes of the header form the header string. The default header string is "SQLite format 3" (with a null terminator).

If for some reason you don't want make clear that the file is a SQLite database you can change this string.

To enable the use of a custom header compile OctoDB with:

    CODEC_USE_CUSTOM_HEADER
    CODEC_CUSTOM_HEADER="ExampleNewHeader"

The custom header string must be 15 or 16 bytes long. If using 15 bytes, the last one will be a null terminator.


### 3. Encrypt the db header but do not encrypt the bytes 16-23 from it

These bytes are required by SQLite to be unencrypted.

To encrypt the database header compile OctoDB with:

    CODEC_ENCRYPT_DB_HEADER



## INTERNAL DESIGN

Each page is encrypted and decrypted on-demand, and each page has its own initialization vector (nonce). Each time a page is saved a new IV is used and then the entire encrypted page data will be different than before.

WAL pages are also encrypted.

The communication between the nodes use the same cipher and key selected for the database. Each message uses a different initialization vector so the same plain data is sent as a different crypto stream.
